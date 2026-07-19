# As telas — panorama geral

Inventário completo da plataforma: **51 telas** em 8 seções, mais 4 rotas de
detalhe. Cada linha traz a rota, a pergunta que a tela responde e o arquivo de
dados que a alimenta no modo demonstração (`demo/<endpoint com _>.json`).

> Como ler a coluna "dados": no modo demo o front converte cada endpoint da API
> em um arquivo estático — `/api/fornecedores/setores` vira
> `demo/fornecedores_setores.json`. Ver [arquitetura.md](arquitetura.md#o-modo-demonstração).

---

## Panorama

| Rota | Tela | O que responde | Dados |
|---|---|---|---|
| `/` | Visão geral | o país num painel só | vários |
| `/proposta` | A proposta: rastreabilidade | o que a plataforma cobre, o que falta e por quê | `proposta_cobertura`, `proposta_fios`, `proposta_sancionadas`, `proposta_inidoneas` |

A tela `/proposta` é a mais autocrítica do conjunto: `proposta_cobertura` lista 5
mapas do poder (poder, influência, dinheiro eleitoral, contratações, …) e marca
cada um como `coberto`, `parcial` ou `lacuna`, com o campo `lacuna` dizendo o que
falta — por exemplo, que não existe dado aberto padronizado de **nomeações** para
cargos comissionados e dirigentes de estatais.

---

## Violência · o dano

| Rota | Tela | O que responde | Dados |
|---|---|---|---|
| `/violencia` | O mapa da violência | onde se morre no Brasil | `violencia_*` |
| `/violencia/diagnostico` | Diagnóstico de segurança | o quadro consolidado | `diagnostico_*` |
| `/pensamento` | Abordagens e pensamento | o manifesto, as 12 teses e o canon | `diagnostico_pensamento` |
| `/violencia/leituras` | Leituras dos dados | as interpretações possíveis do mesmo número | — |
| `/violencia/regioes` | O Brasil em 5 regiões | como a violência muda de região | `regioes_*` |
| `/violencia/quem-morre` | Quem morre | perfil das vítimas | `violencia_risco-grupos` |
| `/violencia/justica` | O funil da impunidade | por que a lei não chega ao fim | `justica_panorama` |
| `/violencia/armas` | Com o que se mata | o instrumento da morte | — |
| `/violencia/juventude` | Juventude interrompida | a idade que mais morre | — |
| `/violencia/tempo` | A economia do tempo | o tempo de vida que se perde | — |
| `/violencia/conjugal` | Violência conjugal | quem agride dentro de casa | `conjugal_*`, `conjugal_lgbtqia` |
| `/violencia/genero` | Gênero e LGBTQIA+ | o recorte de gênero da violência | `conjugal_lgbtqia` |
| `/indigenas` | Povos e terras indígenas | terra, língua, garimpo e morte | `indigena_panorama`, `indigena_dinheiro` |
| `/quilombolas` | Povos e territórios quilombolas | o Censo, a titulação que não vem e o dinheiro | `quilombola_panorama` |
| `/violencia/suicidio` | O mapa do suicídio | a morte que não vira manchete | — |
| `/violencia/alcool` | A droga da esquina | o álcool como variável estrutural | `alcool_*` |
| `/violencia/relacao-toxica` | A relação tóxica | os ciclos que se retroalimentam | — |
| `/violencia/mundo` | O Brasil no mundo | como o país se compara | `mundo_*` |
| `/violencia/bares` | Bares e bebidas | a geografia do álcool | — |
| `/violencia/saude-mental` | Para onde levar quem sofre | a rede CAPS existe onde precisa? | — |

**Novo neste ciclo:** `/violencia/justica`, `/violencia/leituras`, `/indigenas`,
`/quilombolas`, e o recorte LGBTQIA+ da violência conjugal.

O `conjugal_lgbtqia` trouxe também uma correção de nomenclatura nas telas antigas:
"Mulheres agredidas" virou "Mulheres **cis** agredidas", com as categorias
"Vítima trans/travesti", "Vítima LGB" e "Parceiro do mesmo sexo" passando a existir
separadamente. O arquivo carrega um glossário (`nomenclatura[10]`) e uma
`nota_subcontagem` — a notificação do SINAN subconta essa população, e o painel diz
isso em vez de apresentar o número como completo.

`/violencia/mulheres` redireciona para `/violencia/conjugal` (rota antiga preservada).

---

## Violência · o dinheiro

| Rota | Tela | O que responde |
|---|---|---|
| `/violencia/pressao-social` | A carência e a morte | o que falta onde se morre |
| `/violencia/presidio-escola` | Presídio ou escola | onde o Estado escolhe investir |
| `/violencia/seguranca` | O mercado da segurança | quem lucra com o medo |
| `/violencia/bancada` | Quem representa quem | a bancada e a pauta da segurança |
| `/violencia/dinheiro` | O dinheiro e a violência | o cruzamento central das duas camadas |

---

## Economia *(seção nova)*

| Rota | Tela | O que responde | Dados |
|---|---|---|---|
| `/economia` | Painel macroeconômico | o quadro macro oficial | `economia_panorama` |
| `/economia-brasil` | O Brasil dos negócios | o tecido empresarial do país | `economia-brasil_panorama`, `economia-brasil_cruzamento`, `economia-brasil_cruzamento-atividades` |

`economia_panorama` traz 5 grupos de séries do **Banco Central (SGS)** — Trabalho,
Crédito e endividamento, Dívida pública, Juros e inflação, Câmbio — cada série com
`definicao`, `variacao_12m`, a série temporal em `pontos[]` e o campo **`melhor`**,
que diz qual direção é a boa para aquela métrica (nem toda alta é boa notícia).

`economia-brasil` é a novidade estrutural: o **cadastro completo de CNPJ** —
27,8 milhões de empresas ativas de um total de 72,3 milhões já registradas. Isso
permite o cruzamento que faltava: **penetração** — quantas empresas de cada setor
existem no Brasil × quantas efetivamente recebem dinheiro público, e quanto.
É a diferença entre "o setor X recebeu muito" e "o setor X tem poucas empresas
capturando muito".

---

## Poder público

| Rota | Tela | O que responde | Dados |
|---|---|---|---|
| `/deputados` · `/deputados/:id` | Deputados | quem gasta a cota, e em quê | `deputados_*` |
| `/senadores` | Senadores | idem, no Senado | `senadores` |
| `/custo-politica` | O custo da política | quanto custa o Legislativo | `custo-politica_panorama` |
| `/pensoes` | Inativos e supersalários | aposentadorias e pensões do Executivo federal | `pensoes_panorama` |
| `/municipal` | Vereadores e ciclo local | o dinheiro da eleição municipal | `municipal_panorama` |
| `/municipios` · `/municipios/:codigo` | Municípios | o retrato consolidado de uma cidade | `municipios_*` |
| `/templos` | Templos religiosos | a geografia das organizações religiosas | `templos_*` |

**`/custo-politica`** é um bom exemplo do padrão epistêmico do projeto: soma o que
é **medido** (cota CEAP + CEAPS, R$ 251 mi em 2025) com o que é **constante
conhecida** (subsídio de 594 parlamentares, R$ 367 mi) — cada componente marcado
com seu `estatuto` — e declara em `nao_incluido[5]` os itens que ficaram de fora e
por quê. O total, R$ 618,3 mi, é apresentado como **parcial e por baixo**, não como
o custo real da política.

**`/pensoes`** abre o SIAPE: 652.647 inativos, R$ 87,3 bi/ano, 1.161 supersalários
acima do teto, e séries que revelam a estrutura do sistema — pensões por década de
início, pensões herdadas por filho, a mais antiga ainda ativa iniciada em **1921**.

---

## Dinheiro público

| Rota | Tela | O que responde | Dados |
|---|---|---|---|
| `/licitacoes` | Licitações | o que o Estado comprou | `licitacoes` |
| `/fornecedores` · `/fornecedores/:cnpj` | Fornecedores | quem recebeu | `fornecedores_*` |
| `/setores` | Setores da economia | para que ramo vai o dinheiro | `fornecedores_setores` + 28 drill-downs |
| `/atividades` | Atividades (agro, alimentos…) | o mesmo, em recorte fino de CNAE | `fornecedores_atividades` |
| `/habitacao` | Habitação e construção | o dinheiro da moradia | `habitacao_panorama` |
| `/doacoes` | Doações e 2026 | quem financia a política | `doacoes_resumo`, `doacoes_ligacoes`, `doacoes_projecao-2026` |
| `/diarios` | Diários oficiais | o que os municípios publicam | Querido Diário (ao vivo) |

**`/setores`** classifica **97,6%** de R$ 1,60 trilhão contratado em 28 setores
CNAE, cada um com ticket médio e concentração (top-1 e top-5) — e cada setor abre
num arquivo próprio (`fornecedores_setores_<slug>.json`) com as 100 maiores
empresas e sua fatia.

**`/doacoes`** carrega o número mais político do conjunto: **77,69%** do
financiamento de campanha (R$ 10,64 bi) já entra como **fundo público via partido**,
não pelo eleitor. E `doacoes_ligacoes` mostra as **98 empresas** que doaram e também
receberam dinheiro público — com `proveniencia` embutida e a ressalva de que
alavancagem não prova causalidade.

---

## Investigação

| Rota | Tela | O que responde | Dados |
|---|---|---|---|
| `/apanhado` | Apanhado geral | a síntese do sistema + a auditoria dos próprios números | `apanhado_geral.json` (raiz) |
| `/rastro` · `/rastro/:cnpj` | Rastro do dinheiro | o dossiê 360º de uma empresa | 84 × `rastro_<cnpj>.json` |
| `/grafo` | Grafo de relações | como as entidades se ligam | `grafo_fraude` |
| `/fraude` | Radar de alertas | os 15 gatilhos de indício | `fraude_alertas` |
| `/rombos` | Rombos e vazamentos 2026/28 | a exposição a risco, hoje e projetada | `fraude_projecao` |
| `/analises` | Análises por personas | a leitura interpretativa, com contra-leitura | `analises_personas.json`, `analises_violencia.json` (raiz) |
| `/modelo` | Modelo de risco | o score de ML por dentro | `fraude_scores` |

**`/rastro/:cnpj`** é a tela que amarra tudo num CNPJ só: o que a empresa recebeu
(contratos + cota), o que doou, a **alavancagem** entre as duas coisas, os órgãos e
deputados envolvidos, o quadro societário com as **candidaturas de cada sócio**, a
flag `tem_socio_politico`, as sanções e os contratos detalhados. É o ciclo
*cota → empresa → sócio → doação → parlamentar* materializado numa página.

**`/rombos`** projeta a exposição a risco: R$ 64,18 bi hoje sobre uma base de
R$ 1,60 tri, em 14 tipos de sinal, com cenários `conservador`/`central`/`agravado`
para 2026 e 2028 — e `ressalvas[4]` afirmando explicitamente que são **sinais de
exposição, não fraude confirmada**.

---

## Sistema

| Rota | Tela | O que responde | Dados |
|---|---|---|---|
| `/dados` | Central de dados | de onde vem cada número, com que lastro | `fontes`, `cruzamentos`, `lastro*`, `stats_inventario`, `sync_status` |

A porta de entrada da camada documentada em
[lastro-e-proveniencia.md](lastro-e-proveniencia.md).

---

## O que mudou neste ciclo

19 rotas novas, nenhuma removida:

- **Seção nova:** Economia (`/economia`, `/economia-brasil`)
- **Território e povos:** `/indigenas`, `/quilombolas`
- **Justiça:** `/violencia/justica`, `/violencia/leituras`
- **Custo do Estado:** `/custo-politica`, `/pensoes`, `/municipal`
- **Dinheiro público em recortes:** `/setores`, `/atividades`, `/habitacao`, `/doacoes`
- **Investigação:** `/apanhado`, `/rastro`, `/rombos`, `/analises`
- **Meta:** `/proposta`
