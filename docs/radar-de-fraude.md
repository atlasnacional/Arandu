# Radar de fraude — os 15 gatilhos em detalhe

Código: `backend/app/services/fraude.py`. Cada gatilho é uma função `g_*` que
devolve objetos `Alerta`; `recalcular()` apaga a tabela `alertas` e roda todos
(um gatilho quebrado não derruba os demais). Rode com
`python -m app.cli calcular-fraude` ou pelo botão da página.

**Todos os alertas são indícios.** A coluna "como ler" abaixo diz o que verificar
antes de tirar qualquer conclusão.

## Gatilhos de cruzamento de bases (mais graves)

| Gatilho | Critério exato | Como ler |
|---|---|---|
| `ciclo_retroalimentacao` | deputado/senador pagou empresa pela cota E um sócio (nome normalizado) dessa empresa doou como pessoa física para a campanha do MESMO parlamentar | o dinheiro público volta ao circuito de quem o gastou; confirmar CPF sócio×doador (a Receita mascara) |
| `fornecedor_sancionado` | CNPJ com sanção **vigente** (sem data final ou data final ≥ hoje) no CEIS/CNEP recebendo da cota ou com contrato | ver categoria da sanção e órgão sancionador na descrição; sanção estadual não impede contrato federal automaticamente |
| `doador_fornecedor` | doador (CPF/CNPJ) da campanha do eleito que depois aparece como fornecedor da cota do mesmo eleito | legal doar; o problema é a via de mão dupla — checar datas e valores |
| `doador_contratado` | doador PJ de campanhas federais com qualquer contrato no PNCP | fraco isolado (contrato pode ser em outro estado); ganha força combinado |
| `doador_contratado_municipal` | doador (CNPJ) da campanha de prefeito/vereador com contrato no PNCP com órgão do MESMO município; severidade alta se o candidato foi eleito | o gatilho municipal mais direto: financiou quem governa e vende para a prefeitura |
| `familiar_fornecedor` | sócio de empresa paga pela cota compartilha **sobrenome raro** com o nome CIVIL do parlamentar (Silva/Santos/Oliveira etc. são ignorados; partículas e sufixos idem) | contratar parentes com a cota é vedado; sobrenome compartilhado NÃO prova parentesco — investigar |
| `socio_homonimo` | sócio com nome **idêntico** ao do parlamentar pagador (ou contido nele) | homônimos existem; conferir CPF |

## Gatilhos de padrão estatístico

| Gatilho | Critério exato | Como ler |
|---|---|---|
| `concentracao_acumulada` | fornecedor nº1 ≥ 35% de TUDO que o parlamentar gastou em todos os anos, com ≥ R$ 150 mil (alta: ≥ 50% e ≥ R$ 300 mil) | relação exclusiva e duradoura; pode ter explicação (ex.: aluguel de escritório) |
| `concentracao_fornecedor` | ≥ 40% da cota de UM ano num único fornecedor (≥ R$ 50 mil no ano) | idem, janela anual |
| `notas_repetidas` | mesmo deputado + fornecedor + ano com 5+ notas de valor idêntico ≥ R$ 1.000 (alta: 10+) | pode ser mensalidade legítima (aluguel); checar os PDFs das notas (`url_documento`) |
| `empresa_recente` | primeiro pagamento público ≤ 365 dias após a abertura do CNPJ, total ≥ R$ 10 mil (alta: ≤ 120 dias) | empresa criada "sob medida"? ver capital social e CNAE |
| `situacao_irregular` | CNPJ BAIXADO/INAPTO/SUSPENSO/NULO na Receita recebendo | pagamento pode ser anterior à baixa — ver datas |
| `fracionamento` | mesmo órgão + fornecedor: 3+ contratos em ≤ 120 dias, todos abaixo do limite de dispensa (R$ 62.725,72) somando acima dele | padrão clássico de fatiamento para evitar licitação |
| `dispensa_alto_valor` | contratação por dispensa/inexigibilidade ≥ R$ 1 mi (alta: ≥ R$ 10 mi) | dispensas legítimas existem (emergência, exclusividade) — ler o objeto |
| `fornecedor_hub` | fornecedor atendendo 15+ deputados no ano com ≥ R$ 200 mil, excluindo setores de rede (aéreo, telefonia, combustível, correios) | hubs regionais (gráficas, locadoras) podem ser legítimos |

## Ajustando limiares

Os números estão no código, no topo de cada função (`services/fraude.py`):
`LIMIAR_DISPENSA`, os `0.4`/`0.35` de concentração, os `5`/`10` de notas
repetidas, os `365`/`120` dias, a lista `_SOBRENOMES_COMUNS` etc. Depois de
mudar, rode `calcular-fraude` de novo.

## Severidade e índice por parlamentar

- `alta` = cruzamento forte entre bases independentes; `media` = padrão atípico;
  `baixa` = contexto.
- O ranking da página soma alertas ponderados: alta ×5, média ×2, baixa ×1
  (`GET /api/fraude/ranking-parlamentares`).

## Saída

Tabela `alertas`: `tipo`, `severidade`, `titulo`, `descricao` (o critério com os
números), `cnpj`, `parlamentar_*`, `orgao_nome`, `ano`, `valor`, `quantidade`,
`detalhes` (JSON com os dados brutos do cruzamento — é o que os grafos usam para
desenhar os ciclos).
