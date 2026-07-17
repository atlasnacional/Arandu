# Instalação passo a passo (para iniciantes)

Este guia assume que você **nunca rodou um projeto Python + Node**. Cada comando
vem com a explicação do que ele faz.

## 1. Instale as ferramentas

1. **Python 3.11+** — <https://www.python.org/downloads/>
   - No instalador do Windows, marque **"Add Python to PATH"**.
2. **Node.js 18+** — <https://nodejs.org/> (versão LTS).

Feche e reabra o terminal, depois confirme:

```powershell
python --version    # Python 3.11.x ou maior
node --version      # v18.x ou maior
npm --version       # vem junto com o Node
```

## 2. Baixe o projeto

Com git: `git clone <url-do-repo>` e `cd trans`.
Sem git: baixe o ZIP no GitHub (botão **Code → Download ZIP**) e extraia.

> O nome da pasta criada ao clonar depende do repositório (pode ser `Arandu`).
> Os comandos abaixo usam `cd trans` — ajuste para o nome da sua pasta. O produto
> é o **Arandu**; `trans` é apenas o nome herdado do diretório/banco.

## 3. Backend (o servidor de dados)

```powershell
cd trans\backend

# Cria um "ambiente virtual": uma pasta .venv com um Python isolado,
# para as dependências do projeto não misturarem com o resto do sistema.
python -m venv .venv

# Instala as dependências (FastAPI, SQLAlchemy, scikit-learn…) DENTRO da .venv.
.\.venv\Scripts\pip install -r requirements.txt
```

> **Linux/Mac:** os executáveis ficam em `.venv/bin/` em vez de `.venv\Scripts\`.
> Ex.: `./.venv/bin/pip install -r requirements.txt`.

Importe uma amostra de dados (leva 5–10 min, precisa de internet):

```powershell
.\.venv\Scripts\python -m app.cli sync-rapido
```

Suba o servidor e **deixe este terminal aberto**:

```powershell
.\.venv\Scripts\python -m uvicorn app.main:app --port 8000
```

Se aparecer `Uvicorn running on http://0.0.0.0:8000`, funcionou.
Teste: abra <http://localhost:8000/docs> no navegador.

## 4. Frontend (a interface)

Abra **outro terminal**:

```powershell
cd trans\frontend
npm install     # baixa as dependências do site (só na primeira vez)
npm run dev     # sobe o site em modo desenvolvimento
```

Abra <http://localhost:5173>. Pronto.

## 5. Carregando mais dados

Use a página **Central de dados** (menu Sistema) — cada card tem um botão
"Importar" — ou os comandos do [README](../README.md#4-carregando-a-base-completa-na-ordem-certa).
Ordem recomendada e tempos estimados:

| Passo | Comando | Tamanho/tempo |
|---|---|---|
| Cota Câmara (por ano) | `sync-camara-despesas-bulk --ano 2024` | ~5 MB, segundos |
| CEAPS (por ano) | `sync-ceaps --ano 2024` | ~10 MB, segundos |
| PNCP | `sync-pncp --dias 90` | API paginada, ~10–30 min |
| Sanções | `sync-sancoes` | ~10 MB, ~30 s |
| Candidaturas TSE | `sync-tse --ano 2022 --uf SP` | 1º download ~400 MB, depois segundos por UF |
| Doações TSE | `sync-doacoes --ano 2022` | 0,4–1,2 GB, 10–40 min |
| Receita Federal | `enriquecer-empresas --limite 500` | ~1 req/s, ~10 min |
| Radar de fraude | `calcular-fraude` | ~1–3 min |
| Modelo ML | `treinar-modelo` | ~1–2 min |

## 6. Erros comuns

| Erro | O que fazer |
|---|---|
| `python não é reconhecido` | reinstale o Python marcando "Add to PATH", reabra o terminal |
| `execution of scripts is disabled` (PowerShell) | rode `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` |
| `port 8000 already in use` | já existe um backend rodando — feche-o ou troque `--port` |
| Site abre mas tudo vazio | o backend não está rodando, ou o banco está vazio (`sync-rapido`) |
| `database is locked` | duas importações pesadas simultâneas; espere uma terminar |
| Download lento/caiu | rode o comando de novo — os ZIPs ficam em cache e retomam de onde parou |
