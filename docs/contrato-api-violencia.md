# Contrato — `/api/violencia/*`

Camada de violência: liga o **dano** (homicídios, por município e ano) ao **dinheiro**
(verba de segurança, contratos do PNCP, cota parlamentar) e a **quem decide**
(deputados eleitos naquele território).

Base: a tabela `violencia` já declarada em `backend/app/models.py`:

```
codigo_ibge (7) · municipio · uf · ano
homicidios · taxa_homicidios (por 100 mil) · homicidios_jovens (15–29)
homicidios_mulheres · populacao
```

O frontend consome o contrato abaixo. **Enquanto um endpoint não existir, ele deve
responder 404** — a interface trata isso e mostra "camada ainda não importada",
em vez de quebrar.

---

## 1. `GET /api/violencia/resumo?ano=`

```jsonc
{
  "ano": 2023,
  "anos_disponiveis": [2010, 2011, "…"],
  "brasil": {
    "homicidios": 46409,
    "taxa": 22.8,                  // por 100 mil habitantes
    "homicidios_jovens": 21800,    // 15–29 anos
    "homicidios_mulheres": 3900,
    "populacao": 203000000
  },
  "variacao": { "homicidios": -0.032, "taxa": -0.041 },  // fração vs. ano anterior
  "por_uf": [
    { "uf": "BA", "homicidios": 5100, "taxa": 34.6, "homicidios_jovens": 2600,
      "homicidios_mulheres": 380, "populacao": 14100000 }
  ]
}
```

## 2. `GET /api/violencia/serie?uf=&codigo_ibge=`

Série temporal do Brasil (sem filtro), de uma UF, ou de um município.

```jsonc
[{ "ano": 2023, "homicidios": 46409, "taxa": 22.8, "homicidios_jovens": 21800, "homicidios_mulheres": 3900 }]
```

## 3. `GET /api/violencia/municipios?uf=&ano=&ordem=taxa|homicidios&limite=50`

```jsonc
[{ "codigo_ibge": "2927408", "municipio": "Salvador", "uf": "BA", "ano": 2023,
   "homicidios": 1420, "taxa": 48.9, "homicidios_jovens": 780,
   "homicidios_mulheres": 96, "populacao": 2900000 }]
```

> Ordenar por `taxa` exige piso de população (sugestão: ≥ 30 mil hab.), senão o
> topo do ranking vira ruído estatístico de cidade pequena — um município de 5 mil
> habitantes com 3 homicídios "lidera" o país. O backend aplica o piso; o frontend
> informa qual foi.

## 4. `GET /api/violencia/juventude?ano=`

O eixo central: **juventude interrompida**.

```jsonc
{
  "ano": 2024,
  "total": 19801,
  "pct_do_total": 0.465,         // fração dos homicídios que são de jovens 15–29
  "taxa": 41.11,                 // por 100 mil JOVENS de 15 a 29 (denominador do IBGE)
  "taxa_por_habitante": 9.31,    // a métrica antiga: por 100 mil habitantes de qualquer idade
  "jovens": 48161680,            // o denominador, explícito
  "taxa_base": "por 100 mil jovens de 15 a 29 anos (IBGE: projeção por idade)",
  "anos_de_vida_perdidos": 1089055,
  "denominador": {
    "fonte": "IBGE — SIDRA 7358 (projeção por sexo e idade); SIDRA 9514 (Censo 2022) nos municípios",
    "disponivel": true,
    "primeiro_ano": 2000
  },
  "serie": [{ "ano": 2010, "total": 28562, "taxa": 54.9, "taxa_por_habitante": 14.6, "pct_do_total": 0.53 }],
  "por_uf": [{ "uf": "BA", "total": 3440, "taxa": 98.69, "taxa_por_habitante": 23.2, "jovens": 3486000, "pct_do_total": 0.51 }],
  "piores_municipios": [{ "codigo_ibge": "…", "municipio": "…", "uf": "…", "total": 0, "taxa": 0.0, "jovens": 0 }]
}
```

> **O denominador foi consertado.** Até julho/2026 esta taxa dividia as mortes de
> jovens pela população **inteira** e saía ~4× menor que a real (9,3 em vez de 41,1
> em 2024) — a nota do endpoint declarava a falha, mas a tela mostrava o número
> errado, e a página de mulheres chegava a rotular "por 100 mil mulheres" um valor
> que era por 100 mil habitantes. O Ipeadata não publica série etária; o IBGE
> publica (SIDRA 7358/9514, sem token), e é de lá que vem `populacao_grupo`
> (ver `backend/app/etl/ibge_populacao.py`). Com o denominador certo, a taxa passa
> a bater com a que o próprio Ipea divulga (juvenil ~60 em 2018).
>
> **`taxa` é `null` antes de 2000** — a projeção etária do IBGE começa ali. Nesses
> anos só existe `taxa_por_habitante`. Devolver o número errado para preencher o
> buraco seria trocar a régua no meio da série.

## 5. `GET /api/violencia/presidio-escola?ano=`

A comparação que o produto existe para fazer: **quanto cada estado empenha em
prender vs. em educar** — e quantos morrem lá.

Despesa por função (Ipea, séries `DFDEFSE`/`DFDEFSM` para segurança e as
equivalentes de educação), em R$ correntes.

```jsonc
{
  "ano": 2023,
  "brasil": { "seguranca": 120e9, "educacao": 350e9, "razao": 0.34, "homicidios": 46409 },
  "por_uf": [
    { "uf": "BA", "seguranca": 8.1e9, "educacao": 14.2e9,
      "razao": 0.57,                  // seguranca / educacao
      "por_habitante_seguranca": 574.0,
      "por_habitante_educacao": 1007.0,
      "homicidios": 5100, "taxa": 34.6, "populacao": 14100000 }
  ]
}
```

> **Uma advertência que a interface exibe junto**: gastar mais em segurança não
> "causa" mais homicídio, nem o contrário — estados violentos gastam mais
> justamente porque são violentos (causalidade reversa). O gráfico mostra
> correlação e a página diz isso com todas as letras.

## 6. `GET /api/violencia/mulheres?ano=`

```jsonc
{
  "ano": 2024,
  "total": 3642,
  "taxa": 3.27,                  // por 100 mil MULHERES (denominador do IBGE)
  "taxa_por_habitante": 1.71,    // a métrica antiga: por 100 mil habitantes dos dois sexos
  "mulheres": 111339419,
  "taxa_base": "por 100 mil mulheres (IBGE: população feminina)",
  "denominador": { "fonte": "IBGE — SIDRA 7358 / 9514", "disponivel": true, "primeiro_ano": 2000 },
  "serie": [{ "ano": 2010, "total": 4477, "taxa": 4.5, "taxa_por_habitante": 2.3 }],
  "por_uf": [{ "uf": "RR", "total": 40, "taxa": 11.9, "taxa_por_habitante": 5.6, "mulheres": 336000 }],
  "piores_municipios": [{ "codigo_ibge": "…", "municipio": "…", "uf": "…", "total": 0, "taxa": 0.0, "mulheres": 0 }]
}
```

> Mesmo conserto de denominador da seção 4: a taxa era por 100 mil **habitantes**
> (metade da real) enquanto a interface a rotulava "por 100 mil mulheres". Agora o
> rótulo e o número dizem a mesma coisa. `taxa` é `null` antes de 2000.

## 7. `GET /api/violencia/seguranca/contratos?uf=&pagina=&tamanho=`

Contratos e licitações do PNCP cujo objeto é do **mercado da segurança**
(armamento, munição, viatura, videomonitoramento, presídio, algema, colete…).
Classificação por palavra-chave no campo `objeto` — é um filtro, não uma verdade:
a página avisa.

```jsonc
{
  "total_valor": 4.2e9,
  "por_categoria": [{ "categoria": "viaturas", "contratos": 320, "valor": 890e6 }],
  "por_uf": [{ "uf": "SP", "contratos": 210, "valor": 700e6, "taxa_homicidios": 8.1 }],
  "itens": { "total": 0, "pagina": 1, "tamanho": 20, "paginas": 0, "itens": ["…Contrato"] },
  "alertas_cruzados": 12   // quantos desses fornecedores têm alerta no radar de fraude
}
```

## 8. `GET /api/violencia/bancada?uf=&limite=50`

Quem representa os territórios mais violentos — e o que faz com a verba.

```jsonc
[{
  "deputado_id": 204554, "nome": "…", "partido": "…", "uf": "BA",
  "taxa_uf": 34.6, "homicidios_uf": 5100,
  "total_cota": 1200000.0,        // CEAP acumulada
  "alertas": 3                    // alertas do radar de fraude ligados a ele
}]
```

---

## Adendo — campos que faltaram na primeira versão

Descobertos ao escrever o frontend contra o contrato. **Todos são obrigatórios.**

1. **`anos_disponiveis: number[]`** em *todas* as respostas que aceitam `?ano=`
   (`/juventude`, `/mulheres`, `/presidio-escola`), não só em `/resumo`. Sem isso
   a página não consegue povoar o seletor de ano — `/presidio-escola` ficou sem
   seletor nenhum, porque não tem nem `serie` de onde deduzir.

2. **`piso_populacao: number`** em toda resposta que traz ranking de município
   (`/municipios`, `piores_municipios` de `/juventude` e `/mulheres`). O contrato
   manda a interface dizer qual piso foi aplicado; sem o campo ela só consegue
   descrever a regra por escrito, o que não é a mesma coisa.

3. **`tem_alerta: boolean`** (ou `alertas: number`) **por contrato** em
   `/seguranca/contratos`. Hoje só existe o agregado `alertas_cruzados`, então a
   tabela não consegue destacar QUAIS linhas têm fornecedor no radar — que é
   justamente o cruzamento que dá sentido à tela.

4. **`categoria: string`** por contrato em `/seguranca/contratos` (a mesma
   classificação usada em `por_categoria`). Sem ela o gráfico agrega por categoria
   mas a tabela não mostra a de cada linha.

---

## 9. `GET /api/violencia/dinheiro-politica?ano=`

O paralelo central: **o dinheiro da política × a violência do território**. Junta,
por UF, tudo o que a plataforma já sabe sobre dinheiro e põe ao lado do número de
mortos.

```jsonc
{
  "ano": 2024,
  "por_uf": [{
    "uf": "BA",
    "homicidios": 5100, "taxa": 40.8, "populacao": 14100000,
    "doacoes_campanha": 210e6,      // TSE: doações a candidatos daquela UF
    "doacoes_por_habitante": 14.9,
    "gasto_seguranca": 8.1e9,       // despesa_funcao
    "gasto_educacao": 14.2e9,
    "cota_bancada": 42e6,           // CEAP somada dos deputados eleitos pela UF
    "contratos_seguranca": 320e6,   // PNCP, objeto do mercado da segurança
    "alertas": 47                   // radar de fraude ligado àquela bancada
  }],
  "brasil": { "…mesmos campos agregados…" },
  "correlacoes": {                  // Pearson entre taxa de homicídios e cada eixo
    "taxa_x_doacoes_por_habitante": 0.1188,
    "taxa_x_gasto_seguranca_por_hab": 0.1718,
    "taxa_x_razao_seguranca_educacao": 0.0084
  },
  "origem_do_dinheiro": [           // de onde vem a receita declarada de campanha
    { "origem": "Recursos de partido político", "valor": 5112800000.0, "doacoes": 378190 }
  ],
  "financiadores_da_seguranca": [   // empresa da segurança cujo SÓCIO PF doou — ver abaixo
    { "cnpj": "…", "nome": "…", "doado": 0.0, "contratado": 0.0, "candidatos": 3,
      "ufs": ["BA"], "alertas": 1, "socios": ["…"], "via": "socio" }
  ],
  "cobertura_cruzamento": { "fornecedores_seguranca": 14, "com_qsa_em_cache": 14 },
  "ano_contratos": [2026],
  "anos_disponiveis": [2022, 2024]  // só anos de eleição: precisa de violência E doação
}
```

> **Advertência que a interface exibe:** correlação não é causa, e aqui a
> causalidade reversa é quase certa — um estado violento gasta mais em segurança
> **porque** é violento. O coeficiente é apresentado como uma pergunta, não como
> uma resposta.

### Três limites do dado, medidos no banco — e o que mudou por causa deles

**1. "Empresa que doou E vendeu" não existe — o cruzamento ingênuo acha lixo.**
A primeira versão desta seção pedia CNPJs que doaram a campanhas *e* vendem
segurança ao Estado. Mas doação de PJ a candidato é **proibida desde 2015** (ADI
4650/STF). Medido no banco: dos R$ 3,2 bi declarados com CNPJ em 2022, **R$ 3,15 bi
(98%) são "Recursos de partido político"** — o fundo eleitoral repassado pela
legenda (União Brasil R$ 417 mi, Progressistas R$ 288 mi, PT R$ 217 mi…). O
cruzamento CNPJ-doador × CNPJ-fornecedor devolve **2 CNPJs**: uma gráfica que doou
R$ 400 e outra empresa que doou R$ 200. Publicar isso sob o título "Quem financia a
segurança" seria ruído vendido como investigação.

→ `financiadores_da_seguranca` passou a ser: empresa do mercado da segurança cujo
**sócio pessoa física** aparece como doador — o mesmo caminho que o gatilho de ciclo
fechado do radar de fraude já usa. O casamento é por **nome normalizado** (o CPF do
sócio vem mascarado pela Receita), então homônimo é possível: é indício para apurar,
nunca prova. `origem_do_dinheiro` expõe a estrutura que explica tudo isso, e a
página a exibe como achado.

**2. Zero pode ser ausência de dado.** `cobertura_cruzamento` diz de quantos
fornecedores da segurança temos o quadro societário. Sem esse campo, uma lista vazia
leria como "ninguém faz isso" quando pode ser "não sabemos quem são os sócios".

**3. Os contratos do PNCP não são uma série anual.** `contratos_seguranca` **não
varia com `?ano=`** — é a janela já importada (`ano_contratos`). Filtrar por ano
daria zero, que leria como "não houve contrato". `cota_bancada` vem `null` (nunca 0)
nos anos em que a CEAP não foi importada.

---

## 10. `GET /api/violencia/suicidio?ano=`

O mapa do suicídio — que é quase o **negativo** do mapa do homicídio.

```jsonc
{
  "ano": 2024,
  "total": 16751,
  "taxa": 7.88,                    // por 100 mil habitantes
  "variacao": { "total": -0.0148, "taxa": -0.015 },
  "homicidios": 42590,             // o contraponto, no mesmo objeto
  "taxa_homicidios": 20.04,
  "correlacao_com_homicidio": -0.4177,   // Pearson entre as duas taxas, nas 27 UFs
  "serie": [{ "ano": 2010, "suicidios": 9448, "taxa": 4.83, "homicidios": 53016, "taxa_homicidios": 27.12 }],
  "por_uf": [{ "uf": "RS", "suicidios": 0, "taxa": 13.78, "homicidios": 0, "taxa_homicidios": 15.15, "populacao": 0 }],
  "piores_municipios": [{ "codigo_ibge": "…", "municipio": "…", "uf": "…", "total": 0, "taxa": 0.0, "taxa_homicidios": 0.0 }]
}
```

> **Suicídio nunca é somado a homicídio, e o dado explica por quê.** Entre as 27 UFs
> as duas taxas correlacionam **negativamente** (r ≈ −0,42): RS e SC lideram em
> suicídio e estão entre os menores em homicídio; PE e MA fazem o inverso. São
> fenômenos opostos — violência sofrida versus autoinfligida — com políticas públicas
> que não têm nada em comum. Agregá-los em "mortes violentas" apagaria a informação.
> Por isso toda resposta carrega o homicídio ao lado: o número de suicídios sozinho
> convida à leitura errada.
>
> Fonte: `AVIOL12_SUICID` (Atlas/Ipeadata, nível municipal, 1980–2024). Suicídio =
> óbito por lesão autoprovocada (CID-10 X60–X84).
>
> **A interface exibe o CVV (188) nesta tela.** Uma página sobre suicídio que não
> oferece ajuda é uma página irresponsável.

## 11. `GET /api/violencia/pressao-social?ano=`

O que anda junto com o homicídio: a carência ou a polícia?

```jsonc
{
  "ano": 2024,
  "anos_disponiveis": [2012, "…", 2024],
  "indicadores": [                 // ordenado pela FORÇA da relação (módulo de r)
    { "chave": "pobreza", "rotulo": "Pobreza", "unidade": "% da população",
      "social": true, "correlacao": 0.669, "ufs_na_conta": 27,
      "por_uf": [{ "uf": "AC", "valor": 41.1 }] },
    { "chave": "gasto_seguranca_por_habitante", "rotulo": "Gasto em segurança por habitante",
      "unidade": "R$ por habitante/ano", "social": false, "correlacao": 0.172, "ufs_na_conta": 27,
      "por_uf": ["…"] }
  ],
  "por_uf": [{ "uf": "BA", "homicidios": 0, "taxa_homicidios": 0.0, "populacao": 0,
               "desemprego": 0.0, "gini": 0.0, "pobreza": 0.0, "analfabetismo": 0.0,
               "gasto_seguranca_por_habitante": 0.0, "razao_seguranca_educacao": 0.0 }],
  "fonte_social": "Ipeadata, base Social (PNAD Contínua/IBGE) — por UF e ano"
}
```

Medido entre as 27 UFs em 2024 — os quatro eixos de **carência** ficam todos acima
dos dois de **dinheiro público**:

| | r com a taxa de homicídios |
|---|---|
| Pobreza | **+0,67** |
| Analfabetismo | +0,52 |
| Desemprego | +0,45 |
| Desigualdade (Gini) | +0,22 |
| Gasto em segurança por habitante | +0,17 |
| Prioriza segurança sobre educação | +0,01 |

> O gasto em segurança entra na **mesma régua** de propósito: sem o contraponto, o
> ranking não responderia à pergunta da tela. E a advertência corta para os dois
> lados — isto **não** prova que a pobreza causa homicídio (pode haver uma terceira
> causa comum), nem que gastar em segurança é inútil (naquele eixo a causalidade
> reversa *rebaixa* o coeficiente, não o infla). O que o ranking impede é a leitura
> oposta e silenciosa: a de que a morte é um problema exclusivamente policial.
>
> Fonte: `PNADCT_TXDSCUPUF` (desemprego, trimestral → média anual), `PNADCA_GINIUF`,
> `PNADCA_TXPNUF` (pobreza), `PNADCA_TXA15MUF` (analfabetismo) — base Social do
> Ipeadata, nível "Estados".
>
> **Arma de fogo e recorte racial NÃO estão aqui**, e não por escolha: o Ipeadata não
> publica nenhuma série de homicídio por arma de fogo nem por cor/raça (verificado nas
> 3.595 séries). Esses cortes só existem no **microdado do SIM/DataSUS** (campos
> `CAUSABAS` = X93–X95 e `RACACOR`), que é um arquivo DBC por UF/ano — outra extração,
> ainda não feita.

---

## Notas de método (a interface exibe cada uma delas)

- **Fonte**: Atlas da Violência (Ipea/FBSP), a partir do SIM/DataSUS. Homicídio =
  óbito por agressão (CID-10 X85–Y09) + intervenção legal (Y35–Y36).
- **Ano de referência** costuma ficar 2 anos atrás do corrente: o SIM consolida tarde.
- **Taxa** = óbitos / população × 100.000. Comparar municípios por *número absoluto*
  favorece cidade grande; por *taxa*, cidade pequena. As duas visões coexistem.
- **A taxa de um grupo usa o denominador daquele grupo.** Mortes de jovens ÷ jovens
  (não ÷ população inteira); mortes de mulheres ÷ mulheres (não ÷ os dois sexos).
  O denominador vem do IBGE — SIDRA **7358** (projeção por sexo e idade, Brasil e UF,
  2000–2060) e SIDRA **9514** (Censo 2022, municípios). Sem token; ver
  `backend/app/etl/ibge_populacao.py`. Antes de 2000 não há projeção etária: a taxa
  do grupo vem `null` e só `taxa_por_habitante` existe.
- **Anos de vida perdidos**: Σ (expectativa de vida na idade − idade na morte), o
  jeito honesto de dizer que um homicídio aos 19 anos custa mais tempo de vida
  que um aos 70. Se o dado só vier agregado em faixa 15–29, usar o ponto médio (22)
  e declarar a aproximação.
- **O "mercado da segurança" é um filtro por palavra-chave no objeto do contrato, e
  ele erra nos dois sentidos.** Erra tanto que precisou de uma lista de **exclusão**:
  `CARTUCHO` casava com *"CARTUCHO DE TONER XEROX"* (toner virava munição) e
  `UNIDADE PRISIONAL` casava com *"gêneros alimentícios destinado aos sentenciados"*
  (laticínio e hortifruti viravam indústria prisional — eram **metade** dos contratos
  que a tela chamava de "segurança"). Comprar comida para um presídio é despesa do
  sistema prisional, mas quem entrega leite não é do mercado da segurança. Depois da
  exclusão: 43 → 24 contratos. Ver `EXCLUSOES_SEGURANCA` em `api/violencia.py`.
- **Correlação não é causa** — vale para verba × violência e para bancada × violência.
