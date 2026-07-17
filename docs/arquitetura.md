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
                    │                       (ML → scores_fraude)  │
                    └─────────────────────────────────────────────┘
```

O padrão é sempre o mesmo: **ETL baixa → normaliza → grava no SQLite; os
serviços cruzam as tabelas; a API expõe; o React desenha.**

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
