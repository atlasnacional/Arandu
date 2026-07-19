# Lastro, proveniência e auditoria

> A camada que responde à pergunta mais importante que se pode fazer a um número:
> **"como você sabe disso?"**

Todo dado desta plataforma é um cruzamento de bases públicas — e todo cruzamento
erra de algum jeito. O Arandu trata isso como parte do produto, não como rodapé:
cada indicador carrega, **junto com o valor**, a ficha do que ele mede, do que ele
**não** mede, de onde veio e com quanta confiança.

São quatro peças, do geral ao específico:

| Peça | Arquivo | O que responde |
|---|---|---|
| **Fontes** | `demo/fontes.json` | de quais sistemas públicos o dado vem, sob que licença e cobertura |
| **Cruzamentos** | `demo/cruzamentos.json` | por qual chave duas bases foram unidas, e com que filtro |
| **Lastro** | `demo/lastro.json` + `demo/lastro_<slug>.json` | a ficha metodológica completa de um indicador, com índice de confiança |
| **Apanhado / auditoria** | `apanhado_geral.json` | recomputa as métricas publicadas e compara com o que está no ar |

---

## 1. Fontes — o catálogo

`demo/fontes.json` é o registro formal das fontes. Uma entrada por sistema
público, com o que um jornalista ou auditor precisa para refazer o caminho:

```jsonc
{
  "slug": "pncp",
  "orgao": "Governo Federal",
  "sistema": "Portal Nacional de Contratações Públicas (PNCP)",
  "url": "https://www.gov.br/pncp/",
  "licenca": "Dados abertos, sem cadastro para consultas disponibilizadas",
  "cobertura": "Editais, atas e contratos dos entes aderentes; adesão subnacional crescente (2021→), obrigatória desde 2023",
  "frequencia": "Contínua",
  "exige_chave": false,
  "observacao": "Campo valorGlobal aceita valores corrompidos; ver contrato_plausivel() em services/qualidade.py"
}
```

Dois campos merecem destaque:

- **`cobertura`** é a honestidade sobre o universo. O PNCP não cobre todo o gasto
  público brasileiro — cobre os entes que aderiram, com adesão crescente desde
  2021. Quem lê "R$ 1,6 tri contratados" precisa saber que o denominador não é o
  Brasil inteiro.
- **`observacao`** guarda a pegadinha conhecida da fonte. O `valorGlobal` do PNCP
  aceita lixo (contratos de trilhões digitados errado) — daí o filtro de
  plausibilidade descrito em [fontes-de-dados.md](fontes-de-dados.md).

`exige_chave: false` em todas as entradas é uma **invariante do projeto**: se uma
fonte exigir token ou cadastro, ela não entra.

---

## 2. Cruzamentos — a chave do join, explícita

Quase toda afirmação forte do Arandu nasce de unir duas bases que o Estado mantém
separadas. `demo/cruzamentos.json` documenta cada união com a chave **literal**:

| Slug | O que une | Chave | Filtro |
|---|---|---|---|
| `doador_fornecedor` | quem doou a campanha (TSE) × quem recebeu contrato (PNCP) ou cota (CEAP) | CNPJ 14 díg. (`doador_cnpj_cpf = fornecedor_ni`/`fornecedor_cnpj`) | só pessoa jurídica; contratos plausíveis (≤ R$ 1 bi) |
| `fornecedor_sancionado` | fornecedor (PNCP) × cadastro de sanção (CGU) | CNPJ (`fornecedor_ni = sancoes.cnpj_cpf`) | sinal de exposição; sanção pode ter escopo/prazo |
| `fornecedor_setor` | fornecedor × ramo econômico (CNAE da Receita) | CNPJ | classificação por CNAE principal |

Escrever a chave por extenso não é preciosismo: é o que permite a outra pessoa
**contestar** o cruzamento. Um join por CNPJ é forte (documento único); um join
por nome normalizado é fraco (homônimos). O leitor tem direito de saber qual dos
dois sustenta o número que está vendo.

O filtro `≤ R$ 1 bi` em `doador_fornecedor` é o filtro de plausibilidade: sem ele,
um punhado de contratos com valor corrompido domina qualquer ranking.

---

## 3. Lastro — a ficha metodológica do indicador

**Lastro** é a ficha completa de um indicador: o que ele mede, o que ele não mede,
como foi extraído e transformado, e quanto se pode confiar nele.

`demo/lastro.json` é o índice — os indicadores que já têm ficha, com sua definição
curta e o índice de confiança:

| Indicador | Confiança | O que mede — e o que **não** mede |
|---|---|---|
| `fornecedores_setores` | **91** | para que setor econômico vai o dinheiro dos contratos. Não mede lucro nem se o gasto foi bem-empregado |
| `economia_panorama` | **91** | séries macroeconômicas oficiais do Banco Central |
| `doacoes_ligacoes` | **82** | empresas que doaram e também receberam dinheiro público. **Não prova que a doação comprou o contrato** |
| `fraude_projecao` | **81** | a fatia do gasto que carrega sinal de risco, hoje e projetada. **Não afirma fraude nem prevê resultado** |
| `habitacao_panorama` | **78** | para onde vai o dinheiro público de moradia |

Cada `demo/lastro_<slug>.json` abre a ficha inteira:

| Campo | O que traz |
|---|---|
| `definicao` | o que mede — e a negativa explícita do que não mede |
| `fonte_primaria`, `fontes[]` | a origem, com as entradas de `fontes.json` embutidas |
| `referencia_original` | o endpoint e a tabela de onde saiu |
| `extracao`, `transformacao`, `formula` | o caminho do dado bruto ao número publicado |
| `dimensao`, `cobertura` | a unidade e o universo coberto |
| `qualidade`, `incerteza` | os defeitos conhecidos e a margem |
| `licenca`, `reproducao` | como refazer o cálculo por conta própria |
| `indice_confianca` | 0–100 |

### Como ler o índice de confiança

O índice **não** é probabilidade de estar certo. É uma nota de sustentação que
combina qualidade da fonte, força da chave de cruzamento e quantidade de suposições
no caminho. A leitura prática:

- **90+** — medição quase direta. O número é o que a fonte publica, reagrupado.
- **80–89** — cruzamento sólido por chave forte (CNPJ), mas a *interpretação* exige
  cuidado. É onde mora `doacoes_ligacoes`: a coincidência entre doador e fornecedor
  é fato; a leitura de que uma coisa causou a outra **não é**.
- **70–79** — há proxy ou recorte assumido no meio do caminho. `habitacao_panorama`
  depende de classificar contratos por objeto, e objeto é texto livre.

Abaixo de 70, o indicador não deveria ir ao ar sem ressalva na própria tela.

### A regra da negativa

Todo `definicao` de lastro tem duas metades: o que mede **e o que não mede**. Não é
estilo — é a defesa central do projeto contra o próprio poder de sugestão de um
gráfico. Um gráfico de "empresas que doaram e receberam" insinua corrupção mesmo
quando o texto diz que não prova nada; a negativa explícita, colada ao número,
é o que segura essa insinuação.

O mesmo padrão aparece espalhado por quase todos os JSONs de painel, no campo
`nota` (e em `nota_especulacao`, `nota_ligacao`, `ressalvas`). Ao adicionar um
painel novo, **o campo `nota` não é opcional**.

---

## 4. Apanhado geral — a auditoria que recomputa

`apanhado_geral.json` (carregado na rota `/apanhado`) faz o que quase nenhum painel
público faz: **recalcula do banco as métricas que estão publicadas e compara com o
que está no ar.**

```jsonc
"auditoria": {
  "metricas": [
    { "nome": "Base contratada (bi)", "recomputado": "1601.9", "publicado": "1601.9", "status": "OK" },
    { "nome": "Doações — fatia pública", "recomputado": "0,7769", "publicado": "0,7769", "status": "OK" },
    { "nome": "Empresas que doaram e receberam", "recomputado": "98", "publicado": "98", "status": "OK" }
  ],
  "divergencias": [],
  "tabelas": { "contratos": 4468769, "doacoes": 2150088, "alertas": 110542, "sancoes": 24869 }
}
```

Se um ETL rodar pela metade ou um filtro mudar sem que o texto acompanhe, a
divergência aparece aqui em vez de virar um número errado circulando como fato.
`divergencias: []` é o estado saudável.

O arquivo também carrega a leitura sistêmica: `cadeia[6]` (as etapas do dinheiro
público), `ligacoes[14]` (como uma etapa alimenta a outra), `sintese`, e
**`maior_lacuna`** — o que a plataforma reconhece que ainda não consegue ver.

---

## 5. Análises por personas — leitura com contra-leitura

`analises_personas.json` e `analises_violencia.json` (rota `/analises`) são a camada
interpretativa: insights gerados por "personas" analíticas distintas, cada um
amarrado aos números que o sustentam e aos pensadores do [canon](canon.md).

O campo que importa aqui é **`contra_leitura`**: o contra-argumento explícito de
cada insight, publicado junto com ele. Um insight sobre concentração de fornecedores
vem acompanhado da leitura alternativa — que concentração pode ser efeito de
mercado técnico, não de captura.

Cada insight traz ainda `forca` (grau de sustentação) e `referencias`, e o bloco
`contagem` mostra quantos insights foram gerados, quantos sobreviveram à checagem
(`sustentados`) e quantos foram publicados (`finais`). O funil fica visível: o que
não se sustentou não vai ao ar, mas o fato de ter sido descartado é registrado.

---

## Ao adicionar um indicador novo

1. A fonte já está em `fontes.json`? Se não, adicione — com `cobertura` e
   `observacao` preenchidas de verdade.
2. Houve cruzamento? Documente a chave literal em `cruzamentos.json`.
3. Escreva o `lastro_<slug>.json` **antes** de publicar a tela. Se você não
   consegue preencher `incerteza` e `qualidade`, você ainda não entendeu o dado.
4. O `definicao` precisa da negativa: *"mede X; NÃO prova Y"*.
5. O painel precisa do campo `nota`.
6. Se a métrica é destaque, acrescente-a à `auditoria.metricas` do apanhado.

> ⚖️ Nada disto transforma indício em prova. O objetivo do lastro é o oposto:
> deixar claro, em cada número, exatamente onde termina o que os dados sustentam.
