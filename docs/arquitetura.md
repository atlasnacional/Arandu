# Arquitetura e fluxo de dados

## Visão geral

```
                    ┌─────────────────────────────────────────────┐
 FONTES PÚBLICAS    │                 BACKEND (FastAPI)           │      FRONTEND
                    │                                             │
 Câmara (dump CSV) ─┤► etl/camara_bulk ─┐                         │
 Senado (CSV/API) ──┤► etl/senado ──────┤                         │
 PNCP (API) ────────┤► etl/pncp ────────┤   SQLite (data/trans.db)│
 TSE (ZIPs) ────────┤► etl/tse ─────────┼──► models.py (tabelas) ─┼──► api/* (REST)
 TSE receitas ──────┤► etl/tse_receitas ┤          │              │        │
 CGU CEIS/CNEP ─────┤► etl/sancoes ─────┤          ▼              │        ▼
 Receita (API) ─────┤► services/receita ┘   services/fraude.py    │   React + ECharts
                    │                       (15 gatilhos→alertas) │   páginas em src/pages
 IBGE / Q.Diário ───┤► api/live (proxy ao vivo, sem armazenar)    │
                    │                       services/modelo.py    │
 ─── CAMADA DE VIOLÊNCIA ────────────────── (ML → scores_fraude)  │
 IBGE/SIDRA (pop) ──┤► etl/ibge_populacao ─┐                       │
 Ipea/Atlas Viol. ──┤► etl/ipea ───────────┤                       │
 SIM/DataSUS (FTP) ─┤► etl/sim ────────────┤   domains/*/router.py │  ~20 telas em
 CNES (FTP) ────────┤► etl/cnes ───────────┼──►(alcool, conjugal,  ├─►  src/pages/violencia
 OMS/GHO + OWID ────┤► domains/mundo/etl ───┤   mundo, diagnostico, │
 Receita (dump CNPJ)┤► etl/bares, templos ──┘   regioes)           │
                    └─────────────────────────────────────────────┘
```

O padrão é sempre o mesmo: **ETL baixa → normaliza → grava no SQLite; os
serviços cruzam as tabelas; a API expõe; o React desenha.**

O app tem hoje **três camadas**:

1. **Transparência/fraude** — parlamentares, licitações, fornecedores, setores da
   economia, doações, rastro por CNPJ, radar de fraude.
2. **Violência** — o maior bloco, com ~25 telas em `frontend/src/pages/violencia/`
   (Panorama, Regiões, Juventude, Quem morre, Armas, Suicídio, Álcool, Bares,
   Mundo, Saúde mental, Conjugal, Gênero, Justiça, etc.), mais os painéis de
   territórios indígenas e quilombolas. É servida por **domínios modulares** em
   `backend/app/domains/` (`alcool`, `conjugal`, `mundo`, `diagnostico`,
   `regioes`), cada um com seu `router.py` e `etl.py`, registrados em `main.py`.
3. **Lastro e proveniência** — a camada que descreve as outras duas: catálogo de
   fontes, chaves de cruzamento, ficha metodológica por indicador e a auditoria que
   recomputa as métricas publicadas. Ver
   [lastro-e-proveniencia.md](lastro-e-proveniencia.md).

O inventário completo das telas está em [telas.md](telas.md).

## O modo demonstração

Este repositório publica a **build estática** — não há backend rodando. O front
resolve isso convertendo cada endpoint da API num arquivo estático, trocando `/` por
`_`:

```js
// frontend: resolvedor do modo demo
`./demo/${endpoint.replace(/^\/api\//, "").replace(/[/]/g, "_")}.json`
```

Ou seja:

| Endpoint | Arquivo no repositório |
|---|---|
| `/api/fornecedores/setores` | `demo/fornecedores_setores.json` |
| `/api/fornecedores/setores/saude-e-farmacia` | `demo/fornecedores_setores_saude-e-farmacia.json` |
| `/api/rastro/00000000000191` | `demo/rastro_00000000000191.json` |
| `/api/lastro/fraude_projecao` | `demo/lastro_fraude_projecao.json` |

Consequências práticas:

- **Rotas paramétricas viram um arquivo por valor.** Hoje são 28 setores, 84 CNPJs
  de rastro e 5 lastros congelados. Filtro ou busca que caia fora desse conjunto
  não retorna nada no modo demo — é limitação esperada, não bug.
- **Três arquivos ficam na raiz, não em `demo/`**: `apanhado_geral.json`,
  `analises_personas.json` e `analises_violencia.json`. São carregados direto por
  caminho (`./apanhado_geral.json`), fora do resolvedor acima.
- Ao publicar uma tela nova, o snapshot precisa ser gerado junto — senão a rota
  existe e a tela abre vazia.

## Tabelas principais (`backend/app/models.py`)

| Tabela | Conteúdo | Chave de cruzamento |
|---|---|---|
| `deputados`, `senadores` | quem está em exercício | nome parlamentar (≈ nome de urna do TSE) |
| `despesas` | cota CEAP da Câmara | `fornecedor_cnpj` (14 díg.) ou CPF |
| `despesas_senado` | CEAPS | `fornecedor_cnpj`, `senador_nome` |
| `contratacoes`, `contratos` | PNCP | `fornecedor_ni`, `orgao_cnpj`, `municipio` |
| `candidatos` | candidaturas TSE (todos os cargos) | `nome` (civil) ↔ `nome_urna` |
| `doacoes` | receitas de campanha | `doador_cnpj_cpf`, `candidato_nome`, `municipio` |
| `sancoes` | CEIS/CNEP | `cnpj_cpf` |
| `empresas` | cadastro Receita + `qsa` (sócios em JSON) | `cnpj` |
| `alertas` | saída do radar de fraude | `cnpj`, `parlamentar_id`, `tipo` |
| `scores_fraude` | saída do modelo ML | `cnpj` |
| `sync_log` | histórico de importações | — |

`models.py` tem **21 tabelas** no total (não 11) — as demais são da camada de
violência:

| Tabela | Conteúdo | Fonte |
|---|---|---|
| `violencia` | séries de homicídio/violência por território | Ipea/Atlas (`etl/ipea.py`) |
| `populacao_grupo` | população por recorte demográfico (denominador de taxas) | IBGE/SIDRA (`etl/ibge_populacao.py`) |
| `mortes_violentas` | microdado de óbito por causa externa | SIM/DataSUS (`etl/sim.py`) |
| `saude_mental` | rede CAPS/leitos de saúde mental por município | CNES (`etl/cnes.py`) |
| `indicadores_sociais` | indicadores sociais de contexto | Ipeadata (`etl/ipea.py`) |
| `despesa_funcao` | despesa pública por função | Ipeadata (`etl/ipea.py`) |
| `bares_bebidas` | bares/comércio de bebidas por município | dump CNPJ Receita (`etl/bares.py`) |
| `templos_religiosos` | organizações religiosas por município | dump CNPJ Receita (`etl/templos.py`) |

### Como as entidades se ligam (o "grafo")

```
 deputado ──cota (despesas)──► FORNECEDOR (cnpj) ◄──contratos── órgão público
    ▲                             │        ▲
    │ doação (doacoes)            │ qsa    │ sanção (sancoes)
    └────── doador ◄──────────── sócio     CEIS/CNEP
             (o ciclo fecha quando doador ≈ sócio do fornecedor)
```

A ligação **candidato ↔ parlamentar** é por *nome de urna normalizado* (sem
acentos, maiúsculas). O CSV de receitas do TSE não traz nome de urna, então
`etl/tse_receitas.atualizar_nomes_urna()` preenche cruzando com `candidatos`
(por isso candidaturas devem ser importadas ANTES das doações).

A ligação **sócio ↔ pessoa** é por *nome completo normalizado* (a Receita mascara
o CPF do sócio: `***123456**`) — por isso todo cruzamento societário é rotulado
como indício, não prova.

## Convenções do código

- **Idempotência**: todo ETL apaga-e-regrava seu escopo (`delete WHERE ano=X` antes
  de inserir) — rodar duas vezes não duplica.
- **Cache de downloads**: ZIPs ficam em `backend/data/downloads/`; só baixa de novo
  com `--forcar-download` ou apagando o arquivo.
- **Documentos**: CNPJ/CPF sempre como dígitos puros (`re.sub(r"\D", "", x)`).
- **Nomes**: comparação sempre via `_norm()` (remove acentos, uppercase, colapsa
  espaços) — em `services/fraude.py`.
- **SQLite**: WAL + `busy_timeout=30s` (ETLs concorrentes esperam o lock em vez de
  falhar). O banco inteiro é um arquivo: `backend/data/trans.db`.
- **Grafos**: `services/graph.py` devolve `{nodes: [{id, label, tipo, valor}],
  edges: [{source, target, valor, rotulo?, severidade?}]}` — consumido direto pelo
  ECharts em `GrafoChart.tsx`. Tipos de nó novos só precisam de cor em
  `theme.ts:NODE_COLORS` e rótulo em `GrafoChart.tsx:TIPO_LABEL`.

## Onde adicionar coisas

| Quero… | Mexa em |
|---|---|
| nova fonte de dados | `etl/nova_fonte.py` + tabela em `models.py` + comando em `cli.py` + entrada em `api/sync.py:_FONTES` |
| novo gatilho de fraude | função `g_*` em `services/fraude.py` + registrar em `GATILHOS` e `ROTULOS` |
| nova feature do ML | `services/modelo.py`: adicionar em `FEATURES` + calcular em `montar_features()` |
| nova página | `frontend/src/pages/X.tsx` + rota e link em `App.tsx` |
| novo endpoint | router em `backend/app/api/` + registrar em `main.py` |
| novo domínio de violência | pasta `app/domains/<nome>/` com `router.py` + `etl.py` (padrão modular) + registrar o router em `main.py` |
