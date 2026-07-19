# Fontes de dados e pegadinhas

Todas as fontes são oficiais e **sem token**. Esta página documenta URLs, formatos
e as armadilhas que já custaram debugging real neste projeto.

## Câmara dos Deputados — cota parlamentar (CEAP)

- **Dump anual (use este)**: `https://www.camara.leg.br/cotas/Ano-{ano}.csv.zip`
  — o ano inteiro em um CSV de poucos MB. ETL: `etl/camara_bulk.py`.
- API paginada (`dadosabertos.camara.leg.br/api/v2`) existe como alternativa
  (`etl/camara.py`) mas é MUITO mais lenta (513 deputados × N páginas).
- ⚠️ `txtCNPJCPF` vem sujo (espaços, pontuação inconsistente) — limpe com
  `re.sub(r"\D", "", x)`, senão nenhum cruzamento por documento bate.
- ⚠️ Linhas sem `ideCadastro` são despesas de lideranças/órgãos — são puladas.
- Valor correto da despesa é `vlrLiquido` (não `vlrDocumento`).

## Senado — senadores e CEAPS

- Senadores: `https://legis.senado.leg.br/dadosabertos/senador/lista/atual` (JSON).
- CEAPS: `https://www.senado.gov.br/transparencia/LAI/verba/despesa_ceaps_{ano}.csv`.
- ⚠️ O servidor de arquivos responde **406** se o request tiver
  `Accept: application/json` — envie `Accept: */*`.
- ⚠️ A primeira linha do CSV é um aviso de atualização; o cabeçalho real começa
  na linha que abre com `"ANO"`. Encoding latin-1, separador `;`, valores `1.234,56`.
- O CSV identifica o senador **apenas pelo nome** (sem código).

## PNCP — licitações e contratos (Lei 14.133)

- Base: `https://pncp.gov.br/api/consulta` — janelas por data de publicação.
- ⚠️ `codigoModalidadeContratacao` é **obrigatório** em `/contratacoes/publicacao`
  (o ETL itera pelas modalidades).
- ⚠️ Rate-limit agressivo (429) e 500 intermitentes — o cliente (`etl/http.py`)
  respeita `Retry-After` e faz backoff exponencial; janelas grandes demoram.
- Valor `null` em licitação = sigiloso (mostrado como "sigiloso/—" na UI).

## TSE — candidaturas

- `https://cdn.tse.jus.br/estatistica/sead/odsele/consulta_cand/consulta_cand_{ano}.zip`
  — um CSV por UF dentro do ZIP (latin-1, `;`). ETL: `etl/tse.py`.
- Colunas-chave: `NM_CANDIDATO` (civil), `NM_URNA_CANDIDATO`, `DS_CARGO`,
  `DS_SIT_TOT_TURNO` (resultado), `NM_UE` (município em eleições municipais).
- ⚠️ `DS_SIT_TOT_TURNO` tem valores como "ELEITO POR QP", "ELEITO POR MÉDIA",
  "NÃO ELEITO", "SUPLENTE" — cuidado: `LIKE '%ELEITO%'` também pega "NÃO ELEITO".

## TSE — doações de campanha (prestação de contas)

- A pasta certa do CDN é `odsele/prestacao_contas/`:
  `https://cdn.tse.jus.br/estatistica/sead/odsele/prestacao_contas/prestacao_de_contas_eleitorais_candidatos_{ano}.zip`
- ⚠️ A pasta `odsele/prestacao_de_contas_eleitorais_candidatos/...` dá **404**.
  Se o TSE mudar de novo, descubra a URL pela API CKAN:
  `https://dadosabertos.tse.jus.br/api/3/action/package_show?id=<dataset>`
  (o dataset de 2022 chama `dadosabertos-tse-jus-br-dataset-prestacao-de-contas-eleitorais-2022`).
- ZIPs grandes: 2022 ≈ 435 MB, 2024 ≈ 1,17 GB. Ficam em cache.
- ⚠️ O CSV `receitas_candidatos_*` **não tem `NM_URNA_CANDIDATO`** — só o nome
  civil (`NM_CANDIDATO`) e `NR_CPF_CANDIDATO`. O nome de urna (que liga ao nome
  parlamentar da Câmara) é preenchido depois cruzando com `consulta_cand`
  (`etl/tse_receitas.atualizar_nomes_urna`). **Importe candidaturas antes.**
- Filtro por eleição: anos múltiplos de 4 = municipais (prefeito/vereador);
  demais = gerais (deputado federal/senador). Doador: `NR_CPF_CNPJ_DOADOR`
  (11 díg. = pessoa física, 14 = empresa), `NM_DOADOR_RFB` é o nome na Receita.
- 📌 Doação de PJ direta a candidato é proibida desde 2015 — CNPJs de 14 dígitos
  em doações são quase sempre órgãos partidários; os ciclos reais aparecem via
  **pessoa física sócia** de fornecedor.

## CGU — CEIS e CNEP (sanções)

- `https://portaldatransparencia.gov.br/download-de-dados/{ceis|cnep}/{AAAAMMDD}`
  — ZIP diário com um CSV (latin-1, `;`).
- ⚠️ Nem todo dia tem arquivo: o ETL tenta hoje e retrocede até 15 dias, validando
  que o conteúdo começa com `PK` (assinatura de ZIP).
- Colunas variam entre CEIS e CNEP — o parser acha colunas por fragmento do nome
  ("CPF OU CNPJ", "DATA INÍCIO", "VALOR DA MULTA"…).

## Receita Federal — cadastro de CNPJ

- Primária: `https://minhareceita.org/{cnpj}` · fallback:
  `https://brasilapi.com.br/api/cnpj/v1/{cnpj}` (mesmos nomes de campos do dump
  oficial). Cache local de 30 dias na tabela `empresas`.
- ⚠️ O CPF dos sócios vem **mascarado** (`***123456**`) — cruzamentos societários
  são por NOME normalizado ⇒ sempre indício, não prova.
- Seja educado: `enriquecer-empresas` faz ~1 req/s. Não paralelize.

## PNCP — o filtro de plausibilidade

⚠️ Pegadinha que merece seção própria: o campo `valorGlobal` do PNCP **aceita
valores corrompidos**. Existem contratos registrados com valores na casa dos
trilhões — erro de digitação ou de unidade na origem. Um único desses domina
qualquer ranking e envenena qualquer soma.

Por isso todo cruzamento que envolve valor passa por `contrato_plausivel()`
(`services/qualidade.py`), que hoje corta em **R$ 1 bi** por contrato. O corte está
documentado no campo `filtro` de `cruzamentos.json` — quem contesta o número
precisa saber que ele existe.

## Banco Central — séries macroeconômicas (SGS)

- Sistema Gerenciador de Séries Temporais do BCB. Alimenta o painel `/economia`
  com 5 grupos: Trabalho, Crédito e endividamento, Dívida pública, Juros e
  inflação, Câmbio.
- Cada série carrega `unidade`, `origem`, `definicao` e o campo **`melhor`**, que
  registra qual direção é a desejável — sem isso, uma alta de inadimplência e uma
  alta de emprego seriam pintadas do mesmo jeito no gráfico.

## SIAPE — inativos e pensões do Executivo federal

- Alimenta `/pensoes`: 652.647 inativos, aposentadorias e pensões com valor,
  tipo, órgão e data de início.
- ⚠️ Cobre **apenas o Executivo federal** — não inclui Legislativo, Judiciário,
  militares nem entes subnacionais. Qualquer leitura de "supersalários no Brasil"
  a partir daqui é subcontagem.
- O campo `acima_teto` compara com o teto constitucional vigente; benefícios
  antigos podem ter amparo jurídico próprio (decisão judicial, direito adquirido)
  — por isso é sinalizado, não acusado.

## Censo 2022, FUNAI, INCRA, Fundação Palmares e CIMI

- Alimentam `/indigenas` e `/quilombolas`.
- O **Censo 2022** contou os quilombolas pela primeira vez (1,32 milhão) e
  atualizou a demografia indígena por UF e etnia — a comparação com 2010 existe,
  mas ⚠️ a **metodologia mudou** entre os dois censos; crescimento aqui mistura
  aumento real com melhora de captação e reafirmação identitária.
- Demarcação (FUNAI) e titulação quilombola (INCRA/Palmares) vêm por fase do
  processo. O campo `anos_no_ritmo_atual` é uma projeção aritmética do ritmo
  histórico, não uma previsão institucional.
- Violência contra indígenas combina SIM/DataSUS com o relatório do **CIMI** —
  fontes com metodologias diferentes, apresentadas lado a lado e não somadas.

## SINAN — violência interpessoal e o recorte LGBTQIA+

- Notificação compulsória de violência interpessoal/autoprovocada. Alimenta as
  telas conjugais, incluindo o recorte por identidade de gênero e orientação
  sexual.
- ⚠️ **Subcontagem estrutural**: o SINAN registra o que chegou ao serviço de saúde
  *e* foi notificado com o campo preenchido. Para população trans e LGB a
  subnotificação é sabidamente maior. O painel publica isso em `nota_subcontagem`
  em vez de apresentar o número como total.
- O arquivo traz um glossário (`nomenclatura[]`) porque as categorias do formulário
  não coincidem com os termos usados politicamente pelas próprias populações.

## Receita Federal — dump completo de CNPJ (o Brasil dos negócios)

- Além dos recortes de bares e templos (abaixo), o dump completo alimenta
  `/economia-brasil`: **27,8 mi de empresas ativas** de 72,3 mi já registradas,
  agrupadas em 28 setores e 41 atividades CNAE.
- É o que torna possível a métrica de **penetração** — empresas existentes por
  setor × empresas que efetivamente recebem dinheiro público.
- ⚠️ A classificação usa o **CNAE principal**; empresa com atuação diversificada é
  contada num setor só.

## IBGE e Querido Diário (ao vivo, sem armazenar)

- Malha das UFs: `https://servicodados.ibge.gov.br/api/v3/malhas/...` (proxy em
  `api/live.py` para evitar CORS).
- Querido Diário: a API fica em `https://api.queridodiario.ok.org.br`
  (⚠️ o path `/api` do site principal serve HTML, não JSON).

---

# Camada de violência

As fontes abaixo alimentam a camada de violência (tabelas `violencia`,
`mortes_violentas`, `saude_mental`, `populacao_grupo`, `bares_bebidas`,
`templos_religiosos`, etc.). Todas sem token.

## Ipea / Atlas da Violência (via Ipeadata)

- ETL: `etl/ipea.py`. Fonte programática: a API OData4 do **Ipeadata**:
  `http://www.ipeadata.gov.br/api/odata4/Metadados` e
  `http://www.ipeadata.gov.br/api/odata4/ValoresSerie(SERCODIGO='AVIOL12_HOMIC')`.
- ⚠️ O site novo do Atlas (`ipea.gov.br/atlasviolencia`) foi reescrito em Next.js e
  o endpoint antigo `/atlasviolencia/api/v1/...` responde **404** — por isso usa-se
  o Ipeadata, que é a mesma base (SIM/DataSUS consolidado pelo Ipea) e é
  programática. Alimenta também `indicadores_sociais` e `despesa_funcao`.

## SIM/DataSUS — microdado de óbito

- ETL: `etl/sim.py`. Arquivos:
  `ftp.datasus.gov.br` → `/dissemin/publicos/SIM/CID10/DORES/DO<UF><ANO>.dbc`.
- ⚠️ O `ftp.datasus.gov.br` responde **só na porta 21 (FTP)** — 80/443 dão timeout,
  então httpx não serve; usa-se `ftplib`.
- ⚠️ O `.dbc` é um **DBF comprimido com Blast (PKWare)** — não é gzip nem zip.
  Descomprime com `datasus_dbc.decompress()` e lê o DBF com `dbfread`.

## CNES — rede de saúde mental

- ETL: `etl/cnes.py`. Mesmo FTP do SIM (também **só na porta 21**):
  `/dissemin/publicos/CNES/200508_/Dados/ST/ST<UF><AAMM>.dbc` (estabelecimentos) e
  `.../LT/LT<UF><AAMM>.dbc` (leitos).
- ⚠️ O DBF do CNES **não escreve o byte terminador (0x0D)** da lista de campos —
  pegadinha na leitura do cabeçalho.

## OMS/GHO e Our World in Data (domínio mundo)

- ETL: `backend/app/domains/mundo/etl.py`.
- WHO GHO OData: `https://ghoapi.azureedge.net/api/{CODE}` — indicadores de saúde
  por país.
- Our World in Data: `https://ourworldindata.org/grapher/{slug}.csv` — usado para o
  consumo de álcool (o GHO não tem o agregado; o OWID cobre 185 países).

## Receita Federal — dump de CNPJ (bares e templos)

- ETLs: `etl/bares.py` (comércio de bebidas) e `etl/templos.py` (organizações
  religiosas, CNAE 9491-8/00). É o **dump de CNPJ (Estabelecimentos)** da Receita,
  não uma API de consulta.
- Acesso (verificado jul/2026) via WebDAV:
  `https://arquivos.receitafederal.gov.br/public.php/webdav/{AAAA-MM}/{arquivo}`,
  autenticando com usuário = TOKEN e senha vazia (`curl -u "TOKEN:"`).
- ⚠️ O diretório HTTP antigo (`.../dados_abertos_cnpj/AAAA-MM/`) foi desativado em
  jan/2026 e dá **404**. O token pode mudar; quando muda, reaparece na URL do share
  em dados.gov.br. O token corrente fica em `settings`.
- ⚠️ O campo de município no CSV é o **código TOM da Receita (4 díg.)**, não o IBGE.
