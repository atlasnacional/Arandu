# Documentação do Arandu

Atlas Nacional da Transparência. A documentação vai da **ideia** (por que o projeto
existe e como ele lê o Brasil) à **prática** (como rodar, importar dados e publicar).

## Conceito — princípios e teorias

Antes do dado vem a lente. Esta é a base que sustenta a leitura do país.

| Guia | O que é |
|---|---|
| [O manifesto e as teses](manifesto.md) | o "grito": o Brasil como sistema vivo, e as 12 teses originais da plataforma |
| [O Framework Integrado da prevenção](principios.md) | os 5 princípios que costuram todos os pensadores num modelo único |
| [O canon](canon.md) | a base teórica: **197 pensadores em 28 eixos**, cada um ligado ao que o dado mede |
| [Abordagens e métodos](abordagens-e-metodos.md) | o que o mundo já testou na prática, e as formas de medir a metrópole |

## Método — como o número se sustenta

| Guia | O que é |
|---|---|
| [Lastro, proveniência e auditoria](lastro-e-proveniencia.md) | como cada indicador declara o que mede, **o que não mede** e com que confiança — e como a plataforma audita os próprios números |

## Técnico — como funciona

| Guia | Para quem |
|---|---|
| [As telas — panorama geral](telas.md) | quem quer o mapa das 51 telas e o dado que alimenta cada uma |
| [Arquitetura e fluxo de dados](arquitetura.md) | quem quer entender ou modificar o código |
| [Fontes de dados e pegadinhas](fontes-de-dados.md) | quem vai importar/atualizar dados |
| [Radar de fraude — os 15 gatilhos](radar-de-fraude.md) | quem vai investigar ou ajustar limiares |
| [Modelo de risco (ML)](modelo-de-risco.md) | quem quer entender/estender o score |
| [Contrato da API de violência](contrato-api-violencia.md) | quem consome os endpoints de violência |

## Operacional — rodar e publicar

| Guia | Para quem |
|---|---|
| [Instalação passo a passo](instalacao.md) | quem nunca rodou um projeto Python+Node |
| [Carregar e subir os dados](carregar-e-subir.md) | quem vai popular o banco (ordem dos importadores) |
| [Publicando a demo no GitHub Pages](github-pages.md) | quem quer compartilhar o projeto |

## Planejamento — para onde vai

| Guia | O que é |
|---|---|
| [Roadmap 2026](ROADMAP_2026.md) | o que já existe e as próximas fases |
| [Expansão de dados](EXPANSAO_DE_DADOS.md) | fontes já integradas e as candidatas |
| [Melhorias do frontend](MELHORIAS_FRONTEND.md) | wishlist de interface |
| [Aprofundamento (round 2)](aprofundamento-round2.md) | backlog de dado subaproveitado |

---

A referência viva de TODAS as rotas da API fica em <http://localhost:8000/docs>
(Swagger gerado pelo FastAPI) com o backend rodando.
