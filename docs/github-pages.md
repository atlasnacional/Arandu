# Publicando a demo no GitHub Pages

O GitHub Pages só serve **arquivos estáticos** — não roda Python/FastAPI. Por
isso o projeto tem um **modo demo**: o frontend é buildado lendo *snapshots JSON*
da API (congelados no momento da exportação) em vez do backend ao vivo.

## Como funciona

```
 (local)  python -m app.cli exportar-demo
            └► gera frontend/public/demo/*.json  (snapshots das rotas GET)

 (GitHub)  push na main
            └► .github/workflows/pages.yml
                 └► npm run build com VITE_DEMO=1
                      • base relativa (./) para funcionar em /seu-repo/
                      • HashRouter (rotas #/fraude funcionam sem servidor)
                      • src/lib/api.ts lê demo/*.json em vez de /api/*
                 └► publica frontend/dist no GitHub Pages
```

No modo demo um banner avisa o visitante; filtros/paginação/importações ficam
congelados no snapshot (POSTs não existem).

## Passo a passo

1. **Gere os snapshots** com o banco populado do jeito que você quer exibir:

   ```powershell
   cd backend
   .\.venv\Scripts\python -m app.cli exportar-demo
   ```

   Confira a pasta `frontend/public/demo/` (≈ 30 arquivos .json).

2. **Crie o repositório no GitHub** e envie o projeto:

   ```powershell
   cd ..            # raiz do projeto
   git init
   git add .
   git commit -m "Trans — transparência pública BR"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/SEU-REPO.git
   git push -u origin main
   ```

   (O `.gitignore` já exclui banco, downloads e node_modules — os snapshots da
   demo SÃO versionados de propósito.)

3. **Ative o Pages**: no GitHub, `Settings → Pages → Source: GitHub Actions`.

4. Pronto. Cada push na `main` republica. A URL fica
   `https://SEU-USUARIO.github.io/SEU-REPO/`.

## Atualizando a demo

Importou dados novos ou recalculou a fraude? Rode `exportar-demo` de novo,
commit e push:

```powershell
cd backend && .\.venv\Scripts\python -m app.cli exportar-demo
cd .. && git add frontend/public/demo && git commit -m "atualiza snapshots da demo" && git push
```

## Testando a demo localmente antes de publicar

```powershell
cd frontend
$env:VITE_DEMO = "1"; npm run build
npx vite preview     # abre o build estático (sem backend!) — confira as páginas
Remove-Item Env:VITE_DEMO
```

## Limitações do modo demo (por design)

- Busca, filtros, paginação e ordenação mostram sempre o snapshot padrão.
- Botões de importar/recalcular/treinar não funcionam (não há backend).
- Páginas de detalhe (deputado individual, município individual) só funcionam
  para o que estiver nos snapshots exportados.
- Diários oficiais (proxy ao vivo) não funciona.

Para a experiência completa, o visitante clona o repo e segue o
[guia de instalação](instalacao.md).
