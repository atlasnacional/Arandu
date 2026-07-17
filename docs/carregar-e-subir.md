# Carregar os dados e subir a plataforma

Guia operacional: do banco vazio ao app no ar. Três partes — **preparar**, **carregar
os dados** e **levantar** (rodar) — mais uma de **deploy**.

O banco é um único arquivo SQLite (`backend/data/trans.db`). Não há servidor de banco
para subir: carregar os dados é rodar os ETLs, que escrevem nesse arquivo.

---

## 1. Preparar o ambiente (uma vez)

```bash
# backend: venv + dependências
cd backend
python -m venv .venv
.venv/Scripts/activate        # Windows;  no Linux/Mac: source .venv/bin/activate
pip install -r requirements.txt

# frontend: dependências
cd ../frontend
npm install
```

Ou, na raiz: `make setup` (cria venv, instala tudo) — ou `./dev.ps1` / `./dev.sh`, que
preparam e já sobem.

---

## 2. Carregar os dados

### O jeito rápido de ver o app funcionando

```bash
cd backend
python -m app.cli sync-rapido
```

Carrega 4 fontes leves (deputados, senadores, despesas da Câmara, amostra do PNCP). Bom
para abrir o app em minutos — mas **não** enche a camada de violência, SIM, álcool, bares
nem violência conjugal.

### A carga COMPLETA (recomendada)

```bash
# tudo o que é rápido/médio, na ordem de dependência:
python -m app.cli sync-tudo

# incluindo os dois PESADOS (SIM ~90 min por FTP; bares ~5,3 GB de dump de CNPJ):
python -m app.cli sync-tudo --pesados
```

`sync-tudo` é **idempotente** (repetir não duplica — cada ETL substitui a sua fatia) e
**tolerante a falha** (uma fonte que cai não derruba a carga; o erro vai para o log e a
carga segue). No fim imprime um resumo `ok / erro / pulado`.

**Retomar uma carga interrompida** (as etapas anteriores não re-rodam):

```bash
python -m app.cli sync-tudo --pesados --continuar-de conjugal
```

**A ordem de dependência** que o `sync-tudo` respeita (é o que importa se você rodar à
mão):

| # | Etapa | Depende de | Peso |
|---|-------|-----------|------|
| 1 | `populacao` | — | leve — é o **denominador** de toda taxa |
| 2 | `violencia` (+ despesa, social) | população | ~min (260 MB de JSON do Ipeadata) |
| 3 | `sim` | — | **PESADO** — alimenta álcool/conjugal/armas/quem-morre |
| 4 | `saude-mental` (CNES) | — | ~min |
| 5 | `consumo-alcool` | violencia | leve (PNS via SIDRA) |
| 6 | `conjugal` | sim + violencia | leve (OMS/GHO) |
| 6b | `sinan` | violencia | **PESADO** — microdado SINAN Violência (~40 MB/ano) |
| 7 | `bares` | violencia + sim | **PESADO** — 5,3 GB |
| 8 | câmara / senado / ceaps / sanções | — | leve |
| 9 | `tse-candidatos` / `tse-doacoes` | — | médio (ZIP grande, fica em cache) |
| 10 | `pncp-licitacoes` / `pncp-contratos` | — | médio (janela de `--pncp-dias`) |
| 11 | `classificar-seguranca` | PNCP | leve (roda DEPOIS do PNCP) |
| 12 | `enriquecer-empresas` | contratos | médio (consulta a Receita) |
| 13 | `fraude` | as fontes acima | ~min |
| 14 | `modelo` | fraude | ~min (treina sklearn) |

**Fora do `sync-tudo`** (pedem parâmetro ou são muito pesados — rode à parte):

```bash
python -m app.cli sync-pncp-historico ...   # varre anos do PNCP (horas)
python -m app.cli sync-tse --ano 2024 --uf MG   # cada UF é um ZIP; sync-tudo só faz --tse-uf
python -m app.cli sync-sim --de 2015 --ate 2023 --ufs PE,SP   # SIM de UFs específicas
```

### Só uma camada

Cada etapa também é um comando solto — útil para atualizar uma fonte sem recarregar tudo:

```bash
python -m app.cli sync-consumo-alcool   # atualiza a PNS
python -m app.cli sync-conjugal         # atualiza as estimativas da OMS/UNODC
python -m app.cli sync-bares            # rebaixa o dump de CNPJ
```

Veja todos: `python -m app.cli --help`.

---

## 3. Levantar (rodar)

### Desenvolvimento

```bash
# dois terminais:
cd backend  && .venv/Scripts/activate && python -m uvicorn app.main:app --reload --port 8000
cd frontend && npm run dev
```

Ou, na raiz, tudo de uma vez: `./dev.ps1` (Windows) / `./dev.sh` (Linux/Mac), ou
`make dev-backend` + `make dev-frontend`.

- Frontend de dev: **http://localhost:5173** (o Vite faz proxy de `/api` para o backend).
- Backend/API: **http://localhost:8000** (docs interativas em `/docs`).

### Produção (backend serve o frontend já buildado)

```bash
cd frontend && npm run build          # gera frontend/dist
cd ../backend && python -m uvicorn app.main:app --port 8000
```

Com o `dist` presente, o backend serve a SPA e a API na **mesma porta 8000** — um processo
só. As rotas do React Router caem no `index.html`; qualquer `/api/*` inexistente devolve
404 de verdade (o contrato depende disso).

### Docker

```bash
docker compose up --build
```

Sobe backend (8000) e frontend (5173). O volume `./backend/data` persiste o banco — a
carga de dados (passo 2) roda dentro do container do backend ou fora, contra o mesmo
arquivo.

---

## 4. Deploy sem backend (GitHub Pages)

Para uma versão pública estática (sem servidor), o modo **demo** serve snapshots JSON:

```bash
cd backend && python -m app.cli exportar-demo    # gera frontend/public/demo/*.json
cd ../frontend && VITE_DEMO=1 npm run build
```

Filtros e paginação ficam congelados no snapshot. Detalhes em `docs/github-pages.md`.

---

## Onde ver o estado da carga

- O endpoint `GET /api/sync/status` e a tabela `sync_log` guardam cada execução (fonte,
  status, contagem, erro). O `sync-tudo` grava um registro por etapa (`sync-tudo:<etapa>`).
- A **Central de dados** (`/dados` no app) mostra o inventário de tabelas e a última
  sincronização de cada fonte.
