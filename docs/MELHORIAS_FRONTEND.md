# Melhorias do Frontend — Trans

Guia de aprimoramentos visuais e de funcionalidade para acompanhar a expansão de dados.

---

## 🎨 Design System Refinado

### Paleta Semântica

```css
/* Cores neutras — com intenção */
--cor-fundo: #fafaf9;      /* Warm stone (não branco puro) */
--cor-fundo-alt: #f5f5f4;
--cor-texto: #1c1c1c;       /* Charcoal (não puro preto) */
--cor-texto-secundario: #6b7280;
--cor-divisor: #e5e7eb;

/* Semântica — não decorativa */
--cor-bom: #059669;         /* Verde saúde */
--cor-aviso: #d97706;       /* Âmbar cuidado */
--cor-critico: #dc2626;     /* Vermelho urgência */
--cor-info: #0284c7;        /* Azul informação */

/* Accent — propósito específico */
--cor-accent: #7c3aed;      /* Violeta (investigação/fraude) */
--cor-accent-alt: #ea580c;  /* Laranja (dados públicos) */

/* Tema escuro */
@media (prefers-color-scheme: dark) {
  --cor-fundo: #1a1a1a;
  --cor-fundo-alt: #262626;
  --cor-texto: #f5f5f5;
  --cor-texto-secundario: #9ca3af;
  --cor-divisor: #404040;
  /* Semântica se adapta mas preserva contraste */
}
```

### Tipografia

- **Display** (Títulos): `Georgia` ou `Crimson Text` (serif, autoridade)
  - H1: 32px / 1.2 / 600
  - H2: 24px / 1.3 / 600
  - H3: 20px / 1.4 / 500

- **Body** (Texto corrido): `Inter` (sans-serif, legibilidade)
  - Parágrafo: 16px / 1.6 / 400
  - Compacto: 14px / 1.5 / 400
  - Label: 12px / 1.5 / 500 + `letter-spacing: 0.05em`

- **Data** (Números/tabelas): `Menlo` ou `IBM Plex Mono` (monospace)
  - 14px / 1.4 / 400 + `font-variant-numeric: tabular-nums`

### Componentes Base Melhorados

#### 1. **Card Variant System**

```tsx
// src/components/Card.tsx
interface CardProps {
  variante?: 'padrao' | 'destaque' | 'alerta' | 'sucesso' | 'dados';
  titulo?: string;
  acoes?: ReactNode;
}

// Uso:
<Card variante="alerta" titulo="Ciclo Fechado Detectado">
  <CicloFechadoViz />
</Card>

<Card variante="dados" titulo="Despesa Federal">
  <TabelaDespesa />
</Card>
```

Variantes:
- `padrao`: Border sutil, fundo neutro
- `destaque`: Border + accent color, sombra maior
- `alerta`: Border esquerda em `--cor-critico`
- `sucesso`: Border esquerda em `--cor-bom`
- `dados`: Tipografia monospace, grid visual

#### 2. **Badge Sistema**

```tsx
<Badge severidade="alta">Ciclo Fechado</Badge>
<Badge tipo="compliance">Conformidade</Badge>
<Badge tipo="hotspot">🔥 Hotspot</Badge>
<Badge tipo="audit">✓ Auditado</Badge>
```

Cores por tipo (não por nome):
- `alta` → `--cor-critico`
- `media` → `--cor-aviso`
- `baixa` → `--cor-info`
- `compliance` → `--cor-bom`
- `hotspot` → Laranja animado (pulse leve)

#### 3. **Timeline Componente**

```tsx
<Timeline>
  <TimelineItem
    ano={2022}
    titulo="Auditoria TCU"
    severidade="media"
    valor={150000}
  />
  <TimelineItem
    ano={2023}
    titulo="Contrato recebido"
    severidade="critica"
    valor={500000}
  />
</Timeline>
```

#### 4. **Mapa Coroplético (Hotspots)**

```tsx
<MapaHotspots
  municipios={data}
  escala="taxa_homicidios"  // ou "alertas", "despesa"
  overlay="fornecedores"
/>
```

Cores gradiente: azul (baixo risco) → vermelho (alto risco)

#### 5. **Scorecard & Gauge**

```tsx
<Scorecard
  titulo="Score de Risco"
  valor={0.72}
  faixa={[0, 1]}
  cores={['--cor-bom', '--cor-aviso', '--cor-critico']}
  detalhes={['CEIS', 'Ciclo fechado', '3 alertas']}
/>
```

---

## 📄 Novas Páginas

### 1. **`/municipios/:ibge` — Dashboard Municipal**

Consolidar tudo que sabemos sobre um município:
- **Header**: Nome, UF, população (IBGE), taxa de homicídios (IPEA)
- **4 seções**:
  1. **Segurança** (violência + crimes por tipo)
  2. **Orçamento** (despesa federal + estadual + municipal)
  3. **Contratos** (fornecedores, alertas, hotspot score)
  4. **Política** (deputados/senadores daquela UF, votações relevantes)

**Componentes**:
- Breadcrumb: Brasil → UF → Município
- Timeline de eventos (contratações, auditorias)
- Mapa interativo (município destacado)
- Cards comparativos (vs. média estadual, vs. Brasil)

### 2. **`/fornecedores/:cnpj/risco` — Scorecard de Risco**

Expandir página de fornecedor com:
- **Top** (hero): Razão social, CNAE, data abertura, situação (Receita)
- **Score**: Gauge de risco (0–1)
  - Components: `score_fraude`, ciclos fechados, sanções, auditoria
- **Timeline**: Contratos + alertas + auditorias (linha do tempo)
- **Análise**: Sócios (QSA), histórico, comparativas
- **Recomendação**: Botão "Investigar Ciclo" (link para Grafo)

### 3. **`/investigacao/hotspots` — Mapa de Risco Municipal**

Novo sub-aba dentro de "Investigação":
- Mapa Brasil com municípios coloridos (risco)
- Filtros: por UF, por tipo de risco (fraude | violência | despesa)
- Clique no município → `/municipios/:ibge`
- Tabela com top 20 hotspots (ranking)

### 4. **`/investigacao/conformidade` — Auditoria & Feedback**

Novo sub-aba:
- **Filtros**: Tribunal (TCU | TCE), órgão, fornecedor, ano
- **Tabela**: Achados + valor envolvido + status
- **Timeline**: Auditado em ano N → contratado em N+1? (compliance check)
- **Badges**: ✓ Conformidade (corrigiu) | ⚠️ Reincidente

### 5. **`/dados/fluxo` — Pipeline de Dados**

Página de transparência sobre dados:
- **Origem**: Quais fontes, quando atualizadas
- **Processamento**: ETL pipeline (diagrama)
- **Qualidade**: % de cobertura por fonte, missing values
- **Histórico**: Log de sincronizações (SyncLog)

Botões:
- "Download dataset" (CSV de cada tabela)
- "Documentação técnica" (link para `EXPANSAO_DE_DADOS.md`)

---

## 🎯 Melhorias em Páginas Existentes

### Dashboard (Home)

**Adicionar seção**: "Hotspots Emergentes"
```
┌─────────────────────────────────┐
│ 🔥 Municípios em Risco          │
├─────────────────────────────────┤
│ 1. Manaus, AM    Score: 0.89    │
│ 2. Belém, PA     Score: 0.87    │
│ 3. Fortaleza, CE Score: 0.85    │
└─────────────────────────────────┘
[Ver Mapa] [Investigar]
```

**Atualizar gráficos**:
- Série temporal: Despesa Federal vs. Homicídios (correlação)
- Mapa: Sobreposição fraude + violência

### Fraude (Radar)

**Novo filtro**: "Por Contexto Municipal"
- Mostrar alertas agrupados por hotspot
- Severity aumenta se município tem risco = não priorizado

**Timeline**: Adicionar eventos de auditoria
- Se fornecedor foi auditado, mostrar na timeline

### Deputados / Senadores

**Novo card**: "Conflitos de Interesse"
```
Fornecedores frequentes: 15
Ciclos fechados: 3
Doações recebidas: R$ 450k
```

**Aba**: "Votações Relevantes" (link para API Câmara)
- Votou a favor de projeto que beneficiou fornecedor X?

### Fornecedores

**Novo card**: "Histórico de Auditoria"
- Se em `achados_auditoria`, mostrar achados + multas

**Status Visual**:
- 🟢 Conforme (auditado, sem irregularidades)
- 🟡 Atenção (auditorias pendentes, achados menores)
- 🔴 Crítico (em CEIS, ou achados graves)

---

## 🚀 Features Funcionais

### 1. **Comparador (Multi-Select)**

```tsx
// Permitir seleção de múltiplos itens e comparação
<TabelaDados
  rows={fornecedores}
  selecionavel={true}
  onCompare={(selecionados) => navigate('/compare', {state: selecionados})}
/>
```

Página de comparação:
- Tabela side-by-side
- Gráficos justapostos (despesa, alertas, score)
- "Baixar comparativa em PDF"

### 2. **Exportação Melhorada**

```tsx
<ExportarDados
  formatos={['csv', 'json', 'pdf']}
  incluir={['estrutura', 'análises', 'contexto']}
/>
```

- CSV: Estruturado, UTF-8, com guia de colunas
- JSON: Completo, tipado
- PDF: Relatório visual (com gráficos, colorido, capa)

### 3. **Alertas em Real-time (Polling)**

Se backend atualizar `alertas`:
```tsx
useQuery({
  queryKey: ['alertas'],
  queryFn: () => api('/api/fraude/alertas/novos', {desde: lastCheck}),
  refetchInterval: 5 * 60 * 1000,  // 5 min
  staleTime: 2 * 60 * 1000,
})
```

Notificação visual:
```
🆕 3 novos alertas de ciclo fechado
[Ver] [Ignorar]
```

### 4. **Busca Avançada com Facetas**

```tsx
<BuscaAvancada
  campos={[
    {nome: 'fornecedor', tipo: 'texto'},
    {nome: 'municipio', tipo: 'geo', valores: municipios},
    {nome: 'severidade', tipo: 'checkbox', valores: ['alta', 'media', 'baixa']},
    {nome: 'data', tipo: 'intervalo'},
  ]}
  onBusca={(filtros) => ...}
/>
```

Salvar buscas:
- "Todos os ciclos fechados em SP este ano"
- "Fornecedores sancionados que recebem contratos"

### 5. **Breadcrumb Dinâmico + Contexto**

```tsx
<Breadcrumb>
  <BreadcrumbItem to="/"> Brasil </BreadcrumbItem>
  <BreadcrumbItem to="/municipios?uf=SP"> SP </BreadcrumbItem>
  <BreadcrumbItem to="/municipios/3550308"> São Paulo </BreadcrumbItem>
  <BreadcrumbItem to="/fornecedores?municipio=3550308"> Fornecedores </BreadcrumbItem>
  <BreadcrumbItem current> Acme Corp </BreadcrumbItem>
</Breadcrumb>
```

---

## 🎬 Animações & Micro-interações

### 1. **Transições Suaves**

```css
/* Ao navegar entre rotas */
@keyframes slide-in {
  from {
    opacity: 0;
    transform: translateX(10px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

main {
  animation: slide-in 0.3s ease-out;
}
```

### 2. **Skeleton Loaders Refinados**

```tsx
<SkeletonCard /> — shimmer animation
<SkeletonGrafico /> — pulso suave
<SkeletonTabela rows={10} /> — linhas como padrão
```

Não usar "Loading..." — visual é melhor.

### 3. **Hover & Focus Inteligentes**

```css
/* Fileira da tabela */
tr:hover {
  background: var(--cor-fundo-alt);
  box-shadow: inset 0 0 0 1px var(--cor-divisor);
  cursor: pointer;
}

/* Badge interativa */
button:has(Badge):hover {
  transform: scale(1.05);
}

/* Card destaque */
.card:has(:focus-within) {
  border-color: var(--cor-accent);
}
```

### 4. **Notificações Toast Contextuais**

```tsx
// Sucesso
toast.success('Dados exportados', { icon: '✓' })

// Aviso
toast.warning('3 registros faltantes', { icon: '⚠️' })

// Crítico
toast.error('Ciclo fechado detectado', { icon: '🚨', duration: 0 })
```

---

## 📊 Visualizações Novas

### 1. **Heatmap Municipal** (Violência vs. Fraude)

```tsx
<Heatmap
  x="municipios"
  y="anos"
  valores="taxa_homicidios"
  overlay="alertas"  // cores sobrepostas
  escala="divergente"  // azul-branco-vermelho
/>
```

### 2. **Sankey: Dinheiro no Ciclo**

Fluxo visual:
```
Parlamentar
  ↓ Usa Cota R$100k
Fornecedor
  ↓ Sócio
Pessoa Física
  ↓ Doa
Candidato
  ↓ Eleito
Parlamentar (ciclo fecha)
```

### 3. **Scatter: Risco vs. Investimento**

```
X = Valor de contratos
Y = Score de fraude
Tamanho = Número de alertas
Cor = Severidade

Padrão: fornecedor de alto valor com alto risco = destaque
```

### 4. **Gauges Progressivos**

Para scorecard (risco, conformidade, etc):
```
┌─────────────────────┐
│  Conformidade       │
│                     │
│  ▅▅▅▅▅▅▅░░░░  72%  │
│                     │
│ ✓ Auditado         │
│ ⚠️  1 Achado         │
└─────────────────────┘
```

---

## 🔍 Busca & Navegação Melhorada

### Search Semântica

```tsx
<BuscaGlobal
  placeholder="Municipio, fornecedor, parlamentar, lei..."
  modo="rapido"  // vs "avancado"
/>
```

Resultados agrupados:
- 👥 Parlamentares (3)
- 🏭 Fornecedores (12)
- 🏘️ Municípios (2)
- 📋 Documentos (5)

### Quick Links por Contexto

Se em `/fornecedores/12345`, sidebar mostra:
- 📍 Localização: [Ver no Mapa]
- 🔗 Sócios: [Listar]
- 📊 Contratos: [Ver]
- ⚠️ Alertas: [3 críticos]
- 🏛️ Auditoria: [TCU 2023]

---

## 🎭 Temas & Personalizações

### Temas Nomeados

```tsx
// Além de claro/escuro
<TemaSelector
  temas={[
    'claro',
    'escuro',
    'impressao' (minimalista para PDF),
    'alto-contraste' (acessibilidade),
  ]}
/>
```

### Preferências de Usuário

```
// localStorage
{
  tema: 'escuro',
  densidade: 'confortavel' | 'compacto',
  moedaLocal: 'BRL',
  notificacoes: true,
  buscas_salvas: [],
}
```

---

## ♿ Acessibilidade

- [ ] WCAG 2.1 AA em todas as cores
- [ ] Focus visible em tudo interativo
- [ ] Labels com `htmlFor`
- [ ] ARIA: `role`, `aria-label`, `aria-expanded`
- [ ] Keyboard nav: Tab, Enter, Escape
- [ ] Screen reader: estrutura semântica (headings, landmarks)

---

## 📱 Responsividade

| Breakpoint | Uso |
|-----------|-----|
| < 480px | Mobile (hamburger menu, stack vertical) |
| 480–768px | Tablet (2 colunas, sidebar colapsável) |
| 768–1200px | Desktop pequeno (3 colunas) |
| > 1200px | Desktop grande (4+ colunas, sidebar fixo) |

---

## 🚀 Implementação (Prioridade)

### Alto (Semana 1–2)
- [ ] Card variants + Badges
- [ ] Scorecard de fornecedor
- [ ] Hotspots mapa básico

### Médio (Semana 3–4)
- [ ] Busca avançada
- [ ] Comparador
- [ ] Dashboard municipal

### Baixo (Semana 5+)
- [ ] Sankey, Scatter
- [ ] Conformidade audit
- [ ] Personalização avançada

---

## 📌 Referências de Design

- Material Design 3: https://m3.material.io/
- Radix UI: https://www.radix-ui.com/
- Visx (gráficos): https://visx-demo.vercel.app/
- ECharts (referência): https://echarts.apache.org/

**Objetivo**: Design que fala (dados auto-explicativos), não grita (sem cores aleatórias).
