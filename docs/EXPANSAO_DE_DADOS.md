# Expansão de Dados — Arandu

Plano de enriquecimento com novas fontes públicas brasileiras e análises cruzadas com fundamento e referência de base.

> Nota: a pasta do código e o banco (`trans/`, `data/trans.db`) mantêm o nome herdado; o produto é o **Arandu — Atlas Nacional da Transparência**.

---

## 📊 Fontes Atuais (já implementadas)

| Fonte | Cobertura | API/Dump | Tabela | Status |
|-------|-----------|---------|--------|--------|
| **Câmara dos Deputados** | Deputados + CEAP (cota) | API + dump CSV | `deputados`, `despesas` | ✅ Ativo |
| **Senado Federal** | Senadores + CEAPS (cota) | API + dump CSV | `senadores`, `despesas_senado` | ✅ Ativo |
| **PNCP** | Licitações + contratos públicos | API (com backoff) | `contratacoes`, `contratos` | ✅ Ativo |
| **TSE** | Eleições + doações campanha | Dump ZIP | `candidatos`, `doacoes` | ✅ Ativo |
| **Portal Transparência** | CEIS/CNEP (sanções) | Dump diário (sem token) | `sancoes` | ✅ Ativo |
| **IPEA / Atlas da Violência** | Homicídios, indicadores sociais, despesa por função | Ipeadata (`etl/ipea.py`) | `violencia`, `despesa_funcao`, `indicadores_sociais` | ✅ Ativo |
| **Receita Federal** | CNPJ cache (QSA, situação) | minhareceita.org + BrasilAPI | `empresas` | ✅ Ativo |
| **Querido Diário** | Diários oficiais municipais | API pública (proxy ao vivo em `api/live.py`) | — (página `Diarios.tsx`) | ✅ Ativo |
| **IBGE — população** | População por recorte demográfico (denominador de taxas) | SIDRA (`etl/ibge_populacao.py`) | `populacao_grupo` | ✅ Ativo |
| **SIM/DataSUS** | Microdado de óbito por causa externa | FTP porta 21 (`etl/sim.py`) | `mortes_violentas` | ✅ Ativo |
| **CNES** | Rede de saúde mental (CAPS/leitos) por município | FTP porta 21 (`etl/cnes.py`) | `saude_mental` | ✅ Ativo |
| **OMS/GHO + OWID** | Referência global (álcool, saúde) por país | GHO OData + OWID (`domains/mundo/etl.py`) | (domínio mundo) | ✅ Ativo |
| **Receita — dump CNPJ** | Bares/comércio de bebidas e organizações religiosas | dump CNPJ Receita (`etl/bares.py`, `etl/templos.py`) | `bares_bebidas`, `templos_religiosos` | ✅ Ativo |
| **Receita — dump CNPJ (completo)** | Tecido empresarial do país: 27,8 mi de empresas ativas, 28 setores e 41 atividades CNAE | dump CNPJ Receita | (painel `/economia-brasil`) | ✅ Ativo |
| **Banco Central — SGS** | Séries macro: trabalho, crédito, dívida, juros/inflação, câmbio | API SGS | (painel `/economia`) | ✅ Ativo |
| **SIAPE** | Inativos e pensões do Executivo federal, supersalários | Dump | (painel `/pensoes`) | ✅ Ativo |
| **IBGE — Censo 2022** | Demografia indígena e quilombola (1ª contagem quilombola) | SIDRA/Censo | (painéis `/indigenas`, `/quilombolas`) | ✅ Ativo |
| **FUNAI / INCRA / Fundação Palmares** | Demarcação de terras indígenas e titulação quilombola | Dados abertos | (painéis de território) | ✅ Ativo |
| **CIMI** | Violência contra povos indígenas (relatório anual) | Relatório | (painel `/indigenas`) | ✅ Ativo |
| **SINAN/DataSUS** | Violência interpessoal notificada, com recorte LGBTQIA+ | Dump | (telas conjugais) | ✅ Ativo |

> Todas mantêm a invariante do projeto: **sem token, sem cadastro**.

---

## 🎯 Novas Fontes Recomendadas (Sem Token)

### Grupo 1: Poder Público & Gestão

#### 1. **Portal da Transparência — Execução Orçamentária**
- **Endereço**: `https://portaldatransparencia.gov.br/api`
- **O que traz**: Gastos federais por órgão, função, subfunção; empenho, liquidação, pagamento
- **Cobertura**: União (2008–hoje)
- **Integração**: Enriquecer `despesa_funcao` e criar `despesa_federal`
- **Por que**: Paralelo à violência / segurança pública / educação em nível federal
- **Referência**: https://portaldatransparencia.gov.br/api/docs
- **Tabelas propostas**:
  ```sql
  CREATE TABLE despesa_federal (
    id INTEGER PRIMARY KEY,
    ano INTEGER,
    mes INTEGER,
    orgao_codigo STRING,
    orgao_nome STRING,
    funcao STRING,
    subfuncao STRING,
    uf STRING,
    valor FLOAT,
    tipo STRING  -- empenho|liquidacao|pagamento
  );
  ```

#### 2. **IBGE — Contagem de População Municipal**
- **Status**: ✅ Já implementado (`etl/ibge_populacao.py` → tabela `populacao_grupo`).
- **Endereço**: `https://servicodados.ibge.gov.br/api/v1`
- **O que traz**: População por município, UF, recenseamento
- **Cobertura**: 2010, 2020 (e projeções)
- **Integração**: Usar em taxas (crimes/habitante, despesa/habitante)
- **Por que**: Atualmente usamos dados antigos; necessário para normalizar indicadores
- **Referência**: https://servicodados.ibge.gov.br/api/docs/localidades
- **Tabelas propostas**:
  ```sql
  CREATE TABLE populacao (
    codigo_ibge STRING PRIMARY KEY,
    municipio STRING,
    uf STRING,
    ano INTEGER,
    populacao INTEGER UNIQUE(codigo_ibge, ano)
  );
  ```

#### 3. **CGU — Convênios e Transferências**
- **Endereço**: `https://www.convenios.gov.br/api`
- **O que traz**: Transferências federal → municípios (educação, saúde, infraestrutura)
- **Cobertura**: 2013–hoje
- **Integração**: Cruzar com contratos locais (PNCP) + violência
- **Por que**: Mostra alocação real de recursos (não é planejado, é executado)
- **Referência**: https://www.convenios.gov.br/
- **Tabelas propostas**:
  ```sql
  CREATE TABLE convenios (
    id STRING PRIMARY KEY,
    ano INTEGER,
    concedente STRING,
    uf STRING,
    municipio STRING,
    valor_repasse FLOAT,
    valor_contrapartida FLOAT,
    status STRING,
    data_assinatura DATE
  );
  ```

---

### Grupo 2: Propriedade & Patrimônio

#### 4. **SPU — Terras da União**
- **Endereço**: `https://gestaodelicitacoes.spu.planejamento.gov.br/acesso-publico/mapa-web`
- **O que traz**: Imóveis federais (localização, uso, concessões)
- **Cobertura**: Imóveis de domínio da União
- **Integração**: Cruzar com fornecedores que são proprietários (SP/ML)
- **Por que**: Identifica imóveis em concessão → possível conflito de interesse
- **Referência**: https://www.gov.br/spu/pt-br
- **Tabelas propostas**:
  ```sql
  CREATE TABLE imoveis_uniao (
    id STRING PRIMARY KEY,
    municipio STRING,
    uf STRING,
    area_hectares FLOAT,
    tipo STRING,  -- concessao|ocupacao|dominial
    valor_tributario FLOAT,
    data_atualizacao DATE
  );
  ```

#### 5. **Receita Federal — Restituição & Compensação de IR**
- **Endereço**: `https://www8.receita.fazenda.gov.br/simplesnacional/`
- **O que traz**: Restituições / compensações de IR por CNPJ
- **Cobertura**: Simples Nacional + regime geral
- **Integração**: Enriquecer `empresas` com score de conformidade
- **Por que**: Evita / identifica esquemas de compensação fraudulenta
- **Referência**: https://www8.receita.fazenda.gov.br/SimplesNacional/

---

### Grupo 3: Crime & Segurança (Complementar ao IPEA)

#### 6. **SIM-PC — Sistema de Informação de Mortalidade (DATASUS)**
- **Status**: ✅ Já implementado (`etl/sim.py` → tabela `mortes_violentas`, via FTP porta 21).
- **Endereço**: `https://datasus.saude.gov.br/transferencia-de-arquivos/`
- **O que traz**: Óbitos por causa (inclui homicídios, mas também suicídios, etc)
- **Cobertura**: 1979–hoje, por município
- **Integração**: Completar `violencia` com dados do SIM (vs. Atlas da Violência)
- **Por que**: Atlas da Violência é agregado; SIM tem granularidade municipal
- **Referência**: https://datasus.saude.gov.br/
- **Adendo**: `violencia` ganha coluna `fonte` (atlas|sim|isp)

#### 7. **ISP-SP, ISP-RJ e outros — Estatísticas de Segurança Pública**
- **Endereço**: Varies by state (e.g., `https://www.ispdados.rs.gov.br/`)
- **O que traz**: Crimes por tipo, por delegacia/região (mais granular que IPEA)
- **Cobertura**: Cada estado (não é nacional)
- **Integração**: Criar `crimes_estadual` com tipologia
- **Por que**: Permite análise por tipo (furto, roubo, tráfico, etc) → correção > investimento
- **Referência**: Variar por UF; exemplo SP: https://www.ispdados.rs.gov.br/
- **Tabelas propostas**:
  ```sql
  CREATE TABLE crimes_estadual (
    id INTEGER PRIMARY KEY,
    uf STRING,
    ano INTEGER,
    mes INTEGER,
    delegacia STRING,
    tipo_crime STRING,
    quantidade INTEGER UNIQUE(uf, ano, mes, delegacia, tipo_crime)
  );
  ```

---

### Grupo 4: Reputação Corporativa

#### 8. **Proteste/SPC (público)**
- **Endereço**: `https://www.scpc.org.br/`
- **O que traz**: Restritivos (negativações) por CNPJ
- **Cobertura**: Pessoas jurídicas com restrição de crédito
- **Integração**: Enriquecer `empresas` com flag de "mau pagador"
- **Por que**: Fornecedor inadimplente é risco político-fiscal
- **Referência**: Sem API pública; requer web scraping ou contrato
- **Status**: ⏳ Requer negociação

#### 9. **Relatórios de Auditoria (TCU, TCE)**
- **Endereço**: Variam; exemplo TCU: `https://portal.tcu.gov.br/`
- **O que traz**: Achados de auditoria (irregularidades identificadas)
- **Cobertura**: União (TCU) + Estados (TCE)
- **Integração**: Criar `auditoria` com achados por órgão/fornecedor
- **Por que**: Controles ex-post (após o gasto) podem validar/desmentir alertas de fraude
- **Referência**: Variar por órgão
- **Status**: ⏳ Requer normalização (formatos variam)
- **Tabelas propostas**:
  ```sql
  CREATE TABLE achados_auditoria (
    id STRING PRIMARY KEY,
    ano INTEGER,
    tribunal STRING,  -- tcu | tce_uf
    orgao_auditado STRING,
    fornecedor_cnpj STRING,  -- se aplicável
    tipo_achado STRING,  -- irregularidade | falha | fraqueza
    valor_envolvido FLOAT,
    descricao TEXT
  );
  ```

---

### Grupo 5: Dados de Texto & Inteligência

#### 10. **Querido Diário — Ampliação**
- **Endereço**: `https://api.queridodiario.ok.org.br`
- **O que traz**: Extratos de diários oficiais municipais (texto livre)
- **Cobertura**: 3000+ municípios brasileiros
- **Integração**: Parsing de menções a CPF/CNPJ → link com atores
- **Por que**: Diários publicam contratações emergenciais, nomeações, licitações canceladas
- **Referência**: https://queridodiario.ok.org.br
- **Status**: ✅ Já integrado (mas não explorado em parsing)
- **Ação**: Ampliar com NER (Named Entity Recognition) para extrair menções

#### 11. **LexML — Legislação & Regulamentações**
- **Endereço**: `http://www.lexml.gov.br/`
- **O que traz**: Leis, decretos, portarias federais e estaduais
- **Cobertura**: Desde 1946 (Federal); estadual varia
- **Integração**: Contexto de sanções (qual lei foi violada)
- **Por que**: Comprovar que sanção tem base legal
- **Referência**: http://www.lexml.gov.br/
- **Ação**: Indexar fundamentações de sanções contra textos de lei

---

## 🔗 Análises Cruzadas Recomendadas

### Análise 1: **Ciclo Fechado Expandido**
- **Base**: Fornecedor recebe contrato (PNCP) → seus sócios doam para parlamentar → parlamentar usa cota com fornecedor
- **Extensão**: Adicionar
  - Concessões em SPU (imóvel pago pela União, controlado por sócio)
  - Convênios (transferência → fornecedor → sócio → doação)
  - Restituições (fornecedor devolve recurso de forma irregular)
- **Score**: Aumentar severidade do alerta conforme número de elos confirmados

### Análise 2: **Sanção Contínua**
- **Base**: CEIS/CNEP (empresa punida)
- **Extensão**:
  - Está em CEIS mas continua recebendo contratos (PNCP posterior a data_inicio)?
  - Qual órgão violou a proibição?
  - Quanto custou à administração pública?
- **Tabela**: `violacoes_sancao` (contrato_id, sancao_id, dias_decorridos)

### Análise 3: **Despesa vs. Resultado**
- **Base**: Parlamentar gasta em cota parlamentar (CEAP) por tipo
- **Extensão**:
  - Correlacionar com votações (API Câmara: `votacoes/deputados`)
  - Ex: Deputado A vota favor de projeto que beneficia fornecedor X, depois usa cota com X?
  - Criar `conflito_voto` (deputado_id, votacao_id, fornecedor_id, score_conflito)
- **Ref**: https://dadosabertos.camara.leg.br/api/v2/votacoes

### Análise 4: **Aglomeração Geográfica**
- **Base**: Contratos PNCP + fornecedores em mesmo município
- **Extensão**:
  - Mapa de calor de "hot spots" (municípios com muitos alertas + aglomeração de fornecedores)
  - Correlacionar com violência (município com risco alto de segurança = menos fiscalização = mais fraude?)
  - Tabela: `hotspots` (municipio, uf, ano, alertas, fornecedores, taxa_homicidios, score_risco)

### Análise 5: **Conformidade & Auditoria**
- **Base**: Fornecedor auditado + achados negativos
- **Extensão**:
  - Se auditado no ano N, não contrata em N+1?
  - Se auditado e encontrou irregularidade, qual a multa? Pagou?
  - Criar `feedback_auditoria` (fornecedor_id, auditoria_id, tempo_para_contratacao, multa_paga)

---

## 📈 Tabelas Propostas (Novo Schema)

```sql
-- Execução orçamentária federal (Portal Transparência)
CREATE TABLE despesa_federal (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  ano INTEGER NOT NULL,
  mes INTEGER NOT NULL,
  orgao_codigo STRING,
  orgao_nome STRING,
  uf STRING,
  funcao STRING,
  subfuncao STRING,
  valor FLOAT,
  tipo STRING,  -- empenho|liquidacao|pagamento
  data_documento DATE,
  UNIQUE(ano, mes, orgao_codigo, funcao, subfuncao, uf, tipo)
);
CREATE INDEX ix_despesa_federal_ano_uf ON despesa_federal(ano, uf);

-- Transferências (convênios)
CREATE TABLE convenios (
  id STRING PRIMARY KEY,
  ano INTEGER NOT NULL,
  concedente STRING,
  uf STRING NOT NULL,
  municipio STRING NOT NULL,
  valor_repasse FLOAT,
  valor_contrapartida FLOAT,
  status STRING,
  data_assinatura DATE,
  CREATE INDEX ix_convenios_ano_uf ON convenios(ano, uf);

-- População por município (IBGE)
CREATE TABLE populacao (
  codigo_ibge STRING PRIMARY KEY,
  municipio STRING NOT NULL,
  uf STRING NOT NULL,
  ano INTEGER NOT NULL,
  populacao INTEGER NOT NULL,
  UNIQUE(codigo_ibge, ano)
);
CREATE INDEX ix_populacao_ano_uf ON populacao(ano, uf);

-- Imóveis da União (SPU)
CREATE TABLE imoveis_uniao (
  id STRING PRIMARY KEY,
  municipio STRING,
  uf STRING,
  area_hectares FLOAT,
  tipo STRING,  -- concessao|ocupacao|dominial
  valor_tributario FLOAT,
  data_atualizacao DATE
);

-- Crimes por tipo (ISP estadual)
CREATE TABLE crimes_estadual (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  uf STRING NOT NULL,
  ano INTEGER NOT NULL,
  mes INTEGER NOT NULL,
  delegacia STRING,
  tipo_crime STRING NOT NULL,
  quantidade INTEGER NOT NULL,
  UNIQUE(uf, ano, mes, delegacia, tipo_crime)
);
CREATE INDEX ix_crimes_uf_ano ON crimes_estadual(uf, ano);

-- Achados de auditoria (TCU/TCE)
CREATE TABLE achados_auditoria (
  id STRING PRIMARY KEY,
  ano INTEGER NOT NULL,
  tribunal STRING NOT NULL,  -- tcu | tce_uf
  orgao_auditado STRING NOT NULL,
  fornecedor_cnpj STRING,
  tipo_achado STRING NOT NULL,
  valor_envolvido FLOAT,
  descricao TEXT,
  data_auditoria DATE
);
CREATE INDEX ix_achados_tribunal_ano ON achados_auditoria(tribunal, ano);

-- Violações de sanção (ex-post)
CREATE TABLE violacoes_sancao (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  sancao_id INTEGER NOT NULL REFERENCES sancoes(id),
  contrato_id INTEGER NOT NULL REFERENCES contratos(id),
  dias_decorridos INTEGER,
  orgao_violador STRING,
  valor_contratado FLOAT,
  UNIQUE(sancao_id, contrato_id)
);

-- Aglomeração (hot spots) de risco
CREATE TABLE hotspots (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  municipio STRING NOT NULL,
  uf STRING NOT NULL,
  ano INTEGER NOT NULL,
  alertas INTEGER,
  fornecedores_unicos INTEGER,
  taxa_homicidios FLOAT,
  despesa_seguranca FLOAT,
  despesa_educacao FLOAT,
  score_risco FLOAT,  -- 0-1, blend de indicadores
  UNIQUE(municipio, uf, ano)
);
```

---

## 🔧 Plano de Implementação (Fases)

### Fase 1: Dados de Base (Semana 1–2)
- [ ] IBGE — população
- [ ] Portal Transparência — despesa federal
- [ ] Integrar com violência existente

**Por que começar**: Dados de contexto (população, despesa) alimentam normalização de todas as outras análises

### Fase 2: Enriquecimento de Atores (Semana 3–4)
- [ ] Convênios (CGU)
- [ ] Imóveis (SPU)
- [ ] Ampliar `empresas` com restituições/conformidade

**Por que**: Expande perfil de fornecedores → melhora ciclo fechado

### Fase 3: Validação & Controle (Semana 5–6)
- [ ] Achados de auditoria (TCU/TCE)
- [ ] Violações de sanção (pós-análise)
- [ ] Conformidade & feedback

**Por que**: Dados ex-post validam alertas de fraude

### Fase 4: Análises Cruzadas (Semana 7–8)
- [ ] Hotspots (municípios de risco)
- [ ] Ciclo expandido
- [ ] Dashboard de contexto por município

**Por que**: Junta tudo em narrativa coerente

### Fase 5: Inteligência Textual (Semana 9–10, opcional)
- [ ] NER em Querido Diário
- [ ] Indexação de leis (LexML)
- [ ] Busca semântica

**Por que**: Qualitativo complementa quantitativo

---

## 📚 Referências de Fundamento

| Análise | Fundamento | Referência |
|---------|-----------|-----------|
| Ciclo fechado | Lei Nº 12.846/2013 (Lei Anticorrupção) — conflito de interesse | Art. 5º, Lei 12.846 |
| Sanção contínua | Lei Nº 14.133/2021 (Licitações) — efeito ex tunc | Art. 173 |
| Correlação despesa–violência | Debate acadêmico: gasto público vs. criminalidade | Atlas da Violência (IPEA) 2023 |
| Hotspot municipal | Análise SIG (Sistemas de Informação Geográfica) + estatística espacial | IBGE / Ipea |
| Conformidade auditoria | Ciclo de accountability pública | TCU — Referencial de Controles Internos |

---

## 🎯 Impacto Esperado

- ✅ **Cobertura de dados**: ~13 fontes já ativas (as futuras abaixo somam sobre elas)
- ✅ **Análises cruzadas**: De ciclo simples → 5+ análises multidimensionais
- ✅ **Fundamento legal**: Cada alerta terá referência a norma ou estudo
- ✅ **Contexto municipal**: Dashboard de "por que este município, por que este fornecedor"
- ✅ **Rastreabilidade**: Audit trail completo: dado → transformação → alerta

---

## 🚀 Como Começar

1. **Escolha Fase 1** (IBGE + Portal Transparência)
2. **Crie ETLs** em `backend/app/etl/`:
   - `ibge.py` — população
   - `transparencia.py` — execução orçamentária
3. **Atualize `models.py`** com novas tabelas
4. **Rode CLI**:
   ```bash
   python -m app.cli etl-ibge
   python -m app.cli etl-despesa-federal --ano 2023
   ```
5. **Crie routers** em `backend/app/api/` para expor dados
6. **Frontend**: Visu­alize em novas páginas

---

**Próxima reunião**: Definir prioridade entre Fases 1–2
