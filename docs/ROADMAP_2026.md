# Roadmap Arandu 2026

Plano estratégico de expansão de dados e melhoria do frontend com fundamento em APIs públicas brasileiras.

> Nota: a pasta do código e o arquivo de banco (`trans/`, `data/trans.db`) ainda carregam o nome herdado do projeto; o produto é o **Arandu — Atlas Nacional da Transparência**.

---

## 📋 Resumo Executivo

### O que você tem agora
- ~20 fontes de dados públicas, todas sem token ✅
- **51 telas** navegáveis em 8 seções ([inventário](telas.md)) ✅
- Radar de fraude com 15 gatilhos ✅
- Camada de lastro e auditoria: proveniência por indicador, índice de confiança e recomputação das métricas publicadas ✅
- **Já concluído** (era proposto aqui e hoje existe no código):
  - Importador IBGE — população (`backend/app/etl/ibge_populacao.py` + tabela `populacao_grupo`) ✅
  - Router de municípios (`backend/app/api/municipios.py`) ✅
  - Dashboard municipal consolidado (`frontend/src/pages/MunicipioDetalhe.tsx` na rota `/municipios/:codigo`) ✅
  - Classificação de fornecedores por setor/atividade CNAE (`/setores`, `/atividades`) ✅
  - Dossiê 360º por CNPJ — o "rastro do dinheiro" (`/rastro/:cnpj`) ✅
  - Painel macroeconômico (BCB/SGS) e tecido empresarial do país (`/economia`, `/economia-brasil`) ✅
  - Custo da política, inativos e supersalários, ciclo eleitoral municipal ✅
  - Painéis de povos indígenas e quilombolas, e o funil da impunidade ✅
  - Projeção de exposição a risco 2026/2028 (`/rombos`) ✅

### Lacunas reconhecidas
A tela `/proposta` publica o que a plataforma **não** cobre. A maior:
**não existe dado aberto padronizado de nomeações** — cargos comissionados,
dirigentes de estatais e agências, e quem os indicou. Sem isso, o mapa do poder
fica parcial por limitação da fonte, não do projeto.

### O que pode adicionar
- **Novas fontes de dados** ainda futuras (Portal Transparência execução, CGU convênios, SPU, TCU/TCE, etc)
- **5+ análises cruzadas** (ciclo expandido, hotspots municipais, conformidade, etc)
- **Novas páginas** (scorecard de risco, mapa de hotspots, etc — o dashboard municipal já existe)
- **Design system refinado** (paleta, tipografia, componentes)
- **20+ features funcionais** (comparador, busca avançada, exportação, etc)

### Diferencial competitivo
Nenhuma plataforma brasileira integra:
- Fraude + violência + orçamento público + auditoria **em um lugar**
- Análise municipal consolidada (município = contexto único)
- Referência legal para cada alerta (não é achismo, é base)

---

## 🎯 Fases de Implementação

### **Fase 1: Alicerce (Semanas 1–2)**
**Foco**: Dados de contexto municipal

Adicionar:
- [x] IBGE — população (atualiza normalização de indicadores)
- [x] Portal Transparência — execução orçamentária federal
- [x] Novo componente: Card variants + Badges

**Impacto**:
- Qualquer número agora tem contexto (taxa por habitante, % do orçamento, etc)
- Preparação para análises multidimensionais

**Esforço**: ⭐⭐ (10–20h)

---

### **Fase 2: Atores (Semanas 3–4)**
**Foco**: Enriquecer perfil de fornecedores e parlamentares

Adicionar:
- [x] CGU — convênios (transferência federal)
- [x] SPU — imóveis da União (concessões)
- [x] Auditoria (TCU/TCE) — achados ex-post
- [x] Scorecard de risco refinado (frontend)

**Impacto**:
- Ciclo fechado agora incluir concessões + transferências
- Validação de alertas via auditoria (confiabilidade)

**Esforço**: ⭐⭐⭐ (30–40h)

---

### **Fase 3: Análises (Semanas 5–6)**
**Foco**: Cruzamentos inteligentes

Adicionar:
- [x] Hotspots municipais (fraude + violência + despesa)
- [x] Violações de sanção (pós-análise)
- [x] Conformidade & feedback (auditado → recontratou?)
- [x] Dashboard municipal consolidado (frontend)

**Impacto**:
- "Por que este município?" — resposta completa
- "Por que este fornecedor?" — histórico visível

**Esforço**: ⭐⭐⭐⭐ (50–60h)

---

### **Fase 4: Interface (Semanas 7–8)**
**Foco**: Design system + funcionalidades

Adicionar:
- [x] Paleta + tipografia refinada
- [x] Mapa de hotspots interativo
- [x] Comparador (múltiplos fornecedores)
- [x] Busca avançada com facetas
- [x] Exportação (CSV, JSON, PDF)

**Impacto**:
- Profissionalismo visual
- Descoberta fácil (busca, filtros, comparação)

**Esforço**: ⭐⭐⭐⭐ (40–50h)

---

### **Fase 5: Inteligência (Semanas 9–10, OPCIONAL)**
**Foco**: Qualitativo + semântica

Adicionar:
- [ ] NER em Querido Diário (extrair menções de CPF/CNPJ)
- [ ] Indexação de leis (LexML — contexto legal)
- [ ] Alertas em tempo real (polling)
- [ ] Análise de violação por lei específica

**Impacto**:
- "Não achei em tabelas? Procure no texto da lei."
- Notificações de novo ciclo detectado

**Esforço**: ⭐⭐⭐⭐⭐ (60–80h, com ML)

---

## 💰 Custo-Benefício

| Fase | Dados | Análises | Frontend | Impacto | Horas | ROI |
|------|-------|----------|----------|---------|-------|-----|
| 1 | ⭐⭐ | ⭐ | ⭐ | 🔧 Fundação | 15h | Alto (prepara 2–4) |
| 2 | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ | 🎯 Atores | 35h | Alto (ciclo completo) |
| 3 | — | ⭐⭐⭐⭐ | ⭐⭐⭐ | 🌍 Contexto | 55h | Altíssimo (narrativa) |
| 4 | — | — | ⭐⭐⭐⭐ | 👁️ UX/Descoberta | 45h | Alto (uso efetivo) |
| 5 | ⭐⭐ | ⭐⭐ | ⭐ | 🚀 Automação | 70h | Médio (nice-to-have) |

**Recomendação**: Fazer Fases 1–4 sequencialmente (6–8 semanas). Fase 5 fica para depois.

---

## 📁 Arquivos Criados

### Documentação

1. **`EXPANSAO_DE_DADOS.md`** (Este arquivo)
   - 11 novas fontes descritas com URL + tabelas SQL
   - 5 análises cruzadas com fundamento legal
   - Plano de ETL por fase

2. **`MELHORIAS_FRONTEND.md`**
   - Design system (paleta, tipografia)
   - 5 novas páginas (municipal, scorecard, hotspots, conformidade, fluxo)
   - 20+ features e componentes
   - Priorização

3. **`ROADMAP_2026.md`** (Este arquivo)
   - Phases timeline
   - Estimativas de esforço
   - Próximos passos

### Backend (Proposto)

```
backend/app/etl/
├── ibge.py                (população)
├── transparencia.py       (orçamento federal)
├── convenios.py          (CGU)
├── spu.py                (imóveis)
├── auditoria.py          (TCU/TCE)
└── crimes_estadual.py    (ISP estadual)

backend/app/services/
├── analises_cruzadas.py  (hotspots, ciclos, violações)
└── conformidade.py       (feedback auditoria)

backend/app/api/
├── municipios.py         (JÁ IMPLEMENTADO)
├── scorecard.py          (novo router)
└── hotspots.py           (novo router)
```

### Frontend (Proposto)

```
frontend/src/pages/
├── MunicipioDetalhe.tsx       (JÁ IMPLEMENTADO — rota /municipios/:codigo)
├── FornecedorRisco/:cnpj.tsx  (ampliado)
└── Investigacao/
    ├── Hotspots.tsx           (novo)
    └── Conformidade.tsx       (novo)

frontend/src/components/
├── CardVariant.tsx            (novo)
├── Badge.tsx                  (novo)
├── Timeline.tsx               (novo)
├── Scorecard.tsx              (novo)
├── Comparador.tsx             (novo)
├── BuscaAvancada.tsx          (novo)
└── MapaHotspots.tsx           (novo)

frontend/src/lib/
└── analises.ts                (helpers para cálculos)
```

---

## 🚀 Como Começar Hoje

### Opção A: Fazer de verdade (Recomendado)
```bash
# 1. Implementar Fase 1 (IBGE + Portal Transparência)
cd backend

# Criar novo ETL
touch app/etl/ibge.py
touch app/etl/transparencia.py

# Atualizar modelos
# → adicionar tabelas: populacao, despesa_federal

# Rodar CLI
python -m app.cli sync-ibge --ano 2023
python -m app.cli sync-despesa-federal --anos 2020-2023

# 2. Criar routers
touch app/api/municipios.py

# 3. Frontend — novo page
touch frontend/src/pages/Municipio.tsx
```

### Opção B: Preparação (Próximas reuniões)
```bash
# Revisar esta documentação com a equipe
# Decidir prioridade entre fases
# Estimar recursos (pessoas, tempo, infra)
# Acordar sobre referências de dados (qual IPEA? qual TCU?)
```

---

## 🎓 Referências & Fundamento

### Fontes públicas confiáveis
- Portal Transparência: `https://portaldatransparencia.gov.br` ✅
- IBGE Localidades: `https://servicodados.ibge.gov.br` ✅
- CGU Convênios: `https://www.convenios.gov.br` ✅
- SPU Imóveis: `https://www.gov.br/spu` ✅
- Ipeadata: `https://ipeadata.gov.br` ✅
- Atlas da Violência: `https://www.ipea.gov.br/atlasviolencia/` ✅

### Legislação
- Lei 12.846/2013 (Anticorrupção) — ciclos fechados
- Lei 14.133/2021 (Licitações) — sanções
- Lei Complementar 131/2009 (Transparência Pública)
- Lei 12.527/2011 (LGPD — privacidade)

### Benchmarks
- **Portal Transparência (CGU)**: Gastos federais (modelo de referência)
- **Serenata de Amor**: Análise de CEAP, open-source
- **CNJ — Cadastro Unificado de Demandas**: Processos judiciais
- **Poder360**: Análise política (filiação, votações)

---

## 📊 Métricas de Sucesso

### Depois de Fase 1
- ✅ Taxa de normalização de indicadores sobe (n/pop)
- ✅ Dashboard carrega sem erro
- ✅ +2 novas rotas na API

### Depois de Fase 2
- ✅ Score de risco por fornecedor calculado
- ✅ +30% mais alertas (ciclos expandidos)
- ✅ Auditoria data vira campo em alertas

### Depois de Fase 3
- ✅ Hotspots municipais mapeados (20–50 no BR)
- ✅ Dashboard municipal carrega completo
- ✅ "Por que este município?" tem resposta clara

### Depois de Fase 4
- ✅ Busca avançada com 5+ facetas
- ✅ Exportação PDF com 10+ páginas
- ✅ Comparador ativo (2–5 fornecedores)

---

## 🤝 Próximos Passos

1. **Semana 1**: Review desta documentação com stakeholders
2. **Semana 2**: Decidir Fase 1 vs. Fase 2 vs. "tudo"
3. **Semana 3**: Kick-off desenvolvimento
4. **Semana 4+**: Sprints conforme plano

---

## 📞 Dúvidas Frequentes

**P: Por que IBGE antes de PNCP expandido?**
R: IBGE (população) alimenta normalização de TUDO. PNCP já está bem coberto.

**P: E se o Portal Transparência rate-limit?**
R: Usar dump CSV mensal (mais rápido, mais confiável).

**P: Precisa de machine learning para hotspots?**
R: Não. É soma ponderada de 3 indicadores (fraude, violência, despesa). ML é Fase 5 (opcional).

**P: E se a lei mudar?**
R: Cada alerta referencia lei/portaria. Se lei muda, atualiza tabela `referencias_legais`.

**P: Quanto custaria rodar tudo on-cloud?**
R: SQLite + Python rodam em qualquer VPS (R$ 50/mês). Escalável até 100M linhas.

---

## 🎯 Visão Final

Transformar o Arandu de **"dashboard de dados"** em **"narrativa de accountability"**:

```
Dado bruto (CEAP, PNCP, IPEA, TSE)
    ↓
Enriquecimento (IBGE, auditoria, sanção)
    ↓
Análise cruzada (ciclo, hotspot, conformidade)
    ↓
Contexto municipal ("aqui, neste município, estes atores")
    ↓
Interface que conta a história (design + busca + comparação)
    ↓
Ação pública (denúncia, investigação, ajuste fiscal)
```

---

**Pronto para começar? Vamos ao Roadmap detalhado em `EXPANSAO_DE_DADOS.md` e `MELHORIAS_FRONTEND.md`.**
