# Modelo de risco (ML) — como funciona por dentro

Código: `backend/app/services/modelo.py`. Treina com
`python -m app.cli treinar-modelo` ou pelo botão da página **Modelo de risco**.
A metodologia espelha os desafios de ML do autor (adia/obesity — CrunchDAO):
boosting da sklearn como workhorse, ensemble por blend escalar, validação
out-of-fold rigorosa e rejeição de componentes que não validam.

## O problema

Não existe rótulo "fraude confirmada" em dado público. Saídas honestas:

1. **Rótulo fraco** — usar os alertas graves do radar de regras como proxy de
   "suspeito" e treinar um classificador que generalize esses padrões.
2. **Anomalia não supervisionada** — apontar fornecedores estatisticamente
   atípicos, sem depender de regra nenhuma.

O score final combina os dois. Ele **rankeia para investigação**; não é veredito.

## As 17 features (uma linha por CNPJ fornecedor)

| Grupo | Features |
|---|---|
| Volume | `log_total_recebido`, `n_notas`, `valor_medio_nota`, `valor_max_nota`, `meses_ativos`, `fontes` (CEAP/CEAPS/PNCP) |
| Padrão de notas | `pct_notas_redondas` (múltiplos de R$ 100), `pct_valor_repetido` (frequência do valor mais comum), `hhi_pagadores` (concentração de quem paga) |
| Cadastro (Receita) | `idade_empresa_dias` (no 1º pagamento público), `log_capital_social`, `razao_recebido_capital`, `situacao_irregular`, `n_socios` |
| Cruzamentos | `tem_sancao` (CEIS/CNEP), `doou_campanha` |
| Rede | `n_pagadores` (quantos parlamentares) |

Fornecedores com menos de 3 notas ficam de fora (features instáveis).
Campos ausentes usam sentinela `-1` (o HGB da sklearn lida bem com isso).

## Rótulo fraco (sem circularidade)

`y=1` se o CNPJ tem alerta de severidade **alta** em gatilho de **cruzamento
externo**: sanção, ciclo fechado, doador→fornecedor, parentesco, homônimo,
cadastro irregular, empresa recém-aberta.

Os gatilhos **estatísticos** (notas repetidas, concentração…) ficam FORA do
rótulo de propósito: eles são derivados das mesmas features numéricas — usá-los
faria o modelo "reaprender as regras" e o score não acrescentaria nada.

## A lição de vazamento (importante!)

O primeiro treino deu **AUC 0.32 — pior que o acaso**. Causa: os gatilhos-rótulo
dependem do cadastro da Receita, e só ~2 mil dos 17 mil fornecedores estavam
enriquecidos ⇒ positivos só existiam no subconjunto enriquecido ⇒ o modelo
aprendia "essa empresa foi consultada na Receita" (via `capital_social`,
`n_socios=-1`…) em vez de padrão de risco.

**Correção:** o supervisionado treina e pontua **só o subconjunto enriquecido**;
fornecedores sem cadastro recebem apenas o score de anomalia. AUC honesto
depois da correção: **0.93**.

## Validação e ensemble

- `HistGradientBoostingClassifier(max_iter=300, lr=0.08, max_depth=6,
  class_weight="balanced")`.
- Probabilidades **out-of-fold** com `StratifiedGroupKFold(5)` agrupado por UF —
  o modelo nunca pontua um fornecedor que viu no treino, e estados inteiros
  ficam de fora por fold (evita "decorar" padrões regionais). A estratificação
  evita folds sem positivos (que colapsariam o AUC).
- `IsolationForest(n_estimators=300)` nas mesmas features; score normalizado 0–1.
- Blend por linha: `score = 0.55·prob + 0.45·anomalia` para enriquecidos,
  `score = anomalia` para o resto (`PESO_MODELO` em `modelo.py`).
- **Guarda-corpo:** se o AUC out-of-fold < 0.55, o componente supervisionado é
  rejeitado do blend automaticamente (peso 0) — reportado no meta.json.
- Explicabilidade: `permutation_importance` (scoring=roc_auc) — exibida na página.

## Artefatos e API

| Artefato | Conteúdo |
|---|---|
| `backend/data/modelo_fraude.joblib` | modelo final + lista de features |
| `backend/data/modelo_fraude.meta.json` | AUC, nº de positivos, importâncias, data |
| tabela `scores_fraude` | score, score_modelo, score_anomalia, percentil, features (JSON) por CNPJ |

Rotas: `GET /api/fraude/scores` (ranking paginado), `GET /api/fraude/modelo`
(histograma + meta), `POST /api/fraude/modelo/treinar`.

## Como melhorar (próximos passos naturais)

- Mais matéria-prima: `enriquecer-empresas --limite 2000` (mais cadastros ⇒ mais
  rótulos e features reais) e mais anos de doações.
- Features novas: crescimento ano-a-ano por fornecedor, distância entre a sede da
  empresa e a UF do parlamentar, share de notas no fim do mês.
- Calibração: os scores são ranking, não probabilidade calibrada — se precisar de
  probabilidade real, aplicar `CalibratedClassifierCV`.
