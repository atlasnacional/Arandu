# Plano de aprofundamento — round 2 (49 propostas confirmadas contra o banco)

Auditoria multi-agente 2026-07-16. Implementados: Conjugal (reframe espectro completo), Genero (psicológica).


## Alcool
- **[alta/dado_subaproveitado]** O endpoint já entrega a composição das causas (do que o álcool mata) e a tela não desenha nada
  - dado: mortes_violentas.alcool_cat, ano=2023, piso (alcool_cat IS NOT NULL): figado=11246, transtorno_mental=7685, pancreatite=524, cardiomiopatia=372, neuropatia=210, intoxicacao=189, outro=2. O service já monta isso: AlcoolService.panorama() -> 
  - mudanca: Adicionar em frontend/src/pages/violencia/Alcool.tsx um card com barra horizontal usando d.por_categoria (categoria/rotulo/total/pct) — os rótulos já vêm prontos (ROTULO_ALCOOL em backend/app/domains/alcool/service.py:19). Zero mudança de b
- **[alta/dado_subaproveitado]** quem_morre traz 4 cortes (sexo, faixa, raça, local) e a tela só desenha faixa etária
  - dado: Piso 2023 por sexo: M=18081, F=2147 (89,4% homens). Por local: saude=12591, domicilio=6643, outro=538, via_publica=442 (62% morrem em unidade de saúde). Por raça: parda=10143, branca=7116, preta=2476. O service preenche schemas.QuemMorre co
  - mudanca: Em Alcool.tsx, usar quem_morre.sexo (donut ou par de barras: 9 em cada 10 mortes são de homens) e quem_morre.local (a maioria morre no hospital, não na rua — o oposto do homicídio). Puramente frontend. Para raça, cuidado: a contagem bruta e
- **[media/dado_subaproveitado]** O ranking dos 60 municípios que mais matam por álcool é calculado e nunca renderizado
  - dado: AlcoolService._ranking_municipios() já roda (service.py:186,202), aplica o piso de 30 mil hab (PISO_POPULACAO) e retorna municipios[:60] com municipio/uf/total/populacao/taxa — o tipo Alcool.municipios[] existe em api.ts:574. Query bruta 20
  - mudanca: Adicionar em Alcool.tsx uma tabela/ranking municipal lendo d.municipios (já ordenado por taxa), igual ao que a tela irmã Bares faz com seu municipios[:60] (Bares.tsx:321). Zero backend. O CSV também poderia incluir os municípios.

## Armas
- **[media/indicador_raso]** A tela de armas mostra o % de arma de fogo só no total e por UF, ignorando faixa etária/sexo — perde que a arma de fogo se concentra brutalmente nos jovens (82%) e some nos idosos (40%)
  - dado: mortes_violentas tem faixa e sexo, mas /armas (violencia.py L1579) agrupa apenas por meio×uf×ano. Prova (homicídio 2022, % arma de fogo por faixa): 0-14=53%, 15-29=82%, 30-44=69%, 45-59=54%, 60+=40%. Ou seja, o meio muda drasticamente com a
  - mudanca: Backend (violencia.py endpoint armas): adicionar bloco por_faixa (e opcional por_sexo) com pct_arma_fogo por faixa etária no ano. Frontend (frontend/src/pages/violencia/Armas.tsx): gráfico de % arma de fogo por faixa etária, deixando visíve

## Bancada
- **[media/dado_subaproveitado]** A tela é uma tabela plana de até 200 deputados; a coluna 'partido' nunca é agregada, escondendo quais legendas concentram os territórios mais violentos
  - dado: deputados.partido está preenchido. Agregando a taxa de homicídios média do estado que elegeu cada deputado por partido (n≥10): PDT 30,0 | PSB 28,9 | PCdoB 28,8 | UNIÃO 24,3 ... PL 19,9 | PT 19,7 | PODE 17,4 | PSOL 12,6. Há um gradiente de ~
  - mudanca: Adicionar um bloco `por_partido` no handler bancada (backend/app/api/violencia.py ~L850-911) agregando taxa_uf média (e cota/alertas somados) por partido, ou computar no frontend a partir da lista já retornada (frontend/src/pages/violencia/

## Bares
- **[media/dado_subaproveitado]** A tabela bares_bebidas tem a coluna 'total' (CNPJs já registrados) e o endpoint só lê 'ativos' — dá para expor a rotatividade do setor, como a tela de Templos já faz
  - dado: PRAGMA bares_bebidas mostra colunas ativos E total. Ex.: bar em DF (5300108) = 3727 ativos de 13365 já registrados (72% baixados/inativos); nacional por categoria ativos: varejo=1.026.859, lanchonete=848.124, bar=289.137. A query do endpoin
  - mudanca: No endpoint /api/violencia/bares (violencia.py:2189) somar também BaresBebidas.total e devolver brasil.taxa_rotatividade = (total-ativos)/total; em Bares.tsx trocar/adicionar um stat com essa taxa. Reaproveita a lógica de taxa_fechamento de

## Conjugal
- **[alta/dado_subaproveitado]** O ranking de violência de parceiro NOTIFICADA por estado (taxa por 100 mil mulheres) é calculado no backend e nunca renderizado — a tela mostra só o mapa da MORTE
  - dado: schemas.Notificada.por_uf (List[NotificadaUF] com uf/total/parceiro/parceiro_repeticao/parceiro_alcool/taxa_parceiro) é montado em service.py:_notificada (linhas 218-231, ordenado por taxa_parceiro desc) e chega ao frontend em api.ts:810 (C
  - mudanca: Adicionar um card em Conjugal.tsx que renderiza `nf.por_uf` (já no payload) como ranking de estados pela taxa_parceiro (violência de parceiro NOTIFICADA por 100 mil mulheres) — complemento ao card 'Onde se mata mais mulheres', que é a ponta

## Dashboard
- **[alta/indicador_raso]** O gráfico 'Licitações por modalidade' ordena as barras por QUANTIDADE, dando a leitura de que Dispensa é a maior — quando por VALOR ela é a menor das grandes; o total_estimado já é calculado e fica só no tooltip
  - dado: /api/stats/licitacoes-por-modalidade já devolve quantidade E total_estimado (stats.py:207-222), mas Dashboard.tsx (série linhas 240-248) plota m.quantidade. Query mostra a inversão: Dispensa 29.200 licitações / R$1,96 bi; Pregão-Eletrônico 
  - mudanca: Frontend: em Dashboard.tsx oferecer alternância (ou barra dupla) quantidade × valor no opcaoModalidade, plotando m.total_estimado — o dado já chega. Alternativamente ranquear por valor, que é o que importa para 'onde vai o dinheiro'.

## Deputados
- **[alta/dado_subaproveitado]** A tela lista quem gasta e por partido, mas nunca mostra EM QUÊ a Câmara gasta a cota — Divulgação (R$251M) é a maior categoria e fica invisível
  - dado: despesas (Câmara) por tipo, nacional: SELECT tipo,sum(valor) FROM despesas GROUP BY tipo → DIVULGAÇÃO DA ATIVIDADE PARLAMENTAR R$251,3M (disparado 1º) · LOCAÇÃO DE VEÍCULOS R$104,3M · MANUTENÇÃO DE ESCRITÓRIO R$81,4M · COMBUSTÍVEIS R$56,7M.
  - mudanca: Adicionar card 'Onde vai a cota da Câmara' com barras por tipo — um novo endpoint stats (SELECT tipo,sum(valor) FROM despesas GROUP BY tipo) ou incluir no /api/stats. Arquivos: backend/app/api/stats.py (endpoint), frontend/src/pages/Deputad

## Diagnostico
- **[alta/indicador_raso]** O achado do gasto em segurança mostra 1 número (R$140bi/2025) e afirma que 'gastar mais não baixou a morte' — mas o banco tem a série 1985-2025 do gasto E a série da taxa de homicídio, e ela mostra a morte CAINDO 33% enquanto o gasto dobrava
  - dado: despesa_funcao (funcao='seguranca'): 2014=R$59,3bi → 2025=R$140,1bi, 27 UFs todos os anos. violencia UF somada: taxa homicídio 2014=29,82 → 2024=20,04 (/100 mil). Query: SELECT ano,sum(valor) FROM despesa_funcao WHERE funcao='seguranca' GRO
  - mudanca: Backend: em backend/app/domains/diagnostico/service.py._gasto_seguranca, além do total do último ano, montar duas séries (gasto seguranca por ano; taxa de homicídio nacional por ano) e adicioná-las ao schema Achado (novo campo opcional 'ser
- **[media/dado_subaproveitado]** A tabela despesa_funcao carrega 'educacao' (1965-2025, 27 UFs) inteiramente ignorada — a comparação segurança×educação, que sustenta a tese preventiva da própria tela, está no banco e nunca é usada
  - dado: SELECT funcao,count(*),min(ano),max(ano) FROM despesa_funcao GROUP BY funcao → ('educacao',1556,1965,2025) e ('seguranca',1107,1985,2025). Em 2025: educação R$250,6bi vs segurança R$140,1bi (razão 1,79×). Ambos por UF e por ano.
  - mudanca: Backend: novo achado em diagnostico/service.py (ou estender _gasto_seguranca) comparando gasto estadual em educação vs segurança ao longo do tempo, usando a série já presente. Frontend: novo CartaoAchado. Nenhum ETL — a coluna já está popul
- **[media/dado_subaproveitado]** O achado pobreza×homicídio usa só 'pobreza'; a tabela indicadores_sociais tem desemprego, analfabetismo e gini (2012-2025, 27 UFs) — e a Ficha Técnica JÁ afirma usar 'pobreza, desemprego, Gini', mas o código não os toca
  - dado: Correlação com taxa_homicídio UF 2024 (Pearson, n=27, dado já no banco): pobreza r=0,673; analfabetismo r=0,511; desemprego r=0,447; gini r=0,057. Query indicadores_sociais GROUP BY indicador confirma os 4 indicadores, todos 27 UFs até 2025
  - mudanca: Backend diagnostico/service.py._pobreza: calcular a correlação para os 4 indicadores sociais e devolver um pequeno ranking (qual fator social acompanha o homicídio). Frontend: exibir os 4 r's. Sem ETL.

## DinheiroPolitica
- **[alta/bug_residual]** No ano padrão (2024) a tela soma doações 100% MUNICIPAIS (Vereador+Prefeito, R$7 bi) como 'doações de campanha' por UF e as correlaciona/exibe ao lado da cota da bancada FEDERAL — conflando três níveis de governo sem expor o corte 'cargo' que existe no banco
  - dado: doacoes.cargo existe e separa: 2022 = Deputado Federal (R$3,26 bi) + Senador (R$341 mi); 2024 = Prefeito (R$3,86 bi) + Vereador (R$3,18 bi), ZERO federal. Confirmado: em 2024 SELECT por cargo dá 0 doações a Deputado Federal/Senador. A corre
  - mudanca: No handler dinheiro_politica (backend/app/api/violencia.py ~L956-1263) a agregação de doacoes (L1002-1008) ignora `cargo`. Adicionar GROUP BY cargo e devolver `doacoes_por_cargo` e/ou rotular o ano com o nível eleitoral (2022=federal/estadu

## Fornecedores
- **[alta/indicador_raso]** O endpoint top-contratos já devolve 'orgaos' (nº de órgãos distintos que pagaram o fornecedor), mas o frontend descarta o campo — some o sinal de concentração
  - dado: Retornado por /api/fornecedores/top-contratos (fornecedores.py:60, func.count(distinct orgao_nome)) e declarado em TopContrato (Fornecedores.tsx:38), porém o tipo Linha (linhas 44-49, 80-85) só mantém 'contratos' e joga 'orgaos' fora. Query
  - mudanca: Frontend puro: em frontend/src/pages/Fornecedores.tsx incluir 'orgaos' no tipo Linha e no map da aba pncp (linha 80), e adicionar na tabela (linhas 251-289) uma coluna 'Órgãos' — ou a razão contratos/órgãos como medida de concentração. Só a
- **[media/dado_subaproveitado]** O ranking de fornecedores da cota agrega os 3 anos disponíveis num só total, e o parâmetro 'ano' do endpoint (que existe) não é exposto — não dá para ver quem dominou cada ano
  - dado: /api/fornecedores/top-despesas já aceita 'ano' (fornecedores.py:15,20-21) mas o frontend chama sem ele (Fornecedores.tsx:63). Query despesas por ano: 2024=173.796, 2025=170.141, 2026=79.483 registros — três anos misturados no ranking único.
  - mudanca: Frontend: em Fornecedores.tsx adicionar um seletor de ano (aba cota) e de UF (aba pncp) que injetam os params já suportados na chamada api(); incluir o ano/uf na queryKey. Zero mudança de backend.

## Fraude
- **[alta/indicador_raso]** Um único gatilho (fornecedor sancionado) representa 96% de tudo e engole o gráfico 'O que cada regra encontrou' e a stat 'Gravidade alta'
  - dado: alertas.tipo/severidade. SELECT severidade,count(*) FROM alertas GROUP BY severidade => alta 106.396, media 4.111, baixa 35. Desses 106.396 'alta', 98.874 são tipo='fornecedor_sancionado' (SELECT count(*) FROM alertas WHERE tipo='fornecedor
  - mudanca: backend app/api/fraude.py::resumo já devolve 'gatilhos' com alta/media/baixa por tipo; frontend/src/pages/Fraude.tsx opcaoGatilhos deveria (a) usar eixo logarítmico OU quebrar a barra 'fornecedor_sancionado' à parte, e (b) a Stat 'Gravidade
- **[media/dado_subaproveitado]** detalhes.fonte e detalhes.cadastro dos alertas de sanção existem mas nunca aparecem — 98.776 dos 98.874 são contratos PNCP (R$52,5 bi), não cota parlamentar
  - dado: alertas.detalhes (JSON). Agregando fornecedor_sancionado: fonte => {pncp: 98.776, ceap: 78, ceaps: 20}; valor por fonte => {pncp: R$52,5 bi, ceap: R$1,4 mi, ceaps: R$127 mil}; cadastro => {CEIS: 97.959, CNEP: 915}. Ou seja: praticamente tod
  - mudanca: backend app/api/fraude.py: (1) no resumo, incluir contagem/valor por detalhes.fonte e detalhes.cadastro (json_extract já dá para agrupar); (2) endpoint /alertas ganhar filtro 'fonte' (pncp|ceap|ceaps) e 'cadastro' (CEIS|CNEP) via json_extra

## Genero
- **[alta/dado_subaproveitado]** O homicídio é quebrado só por sexo; a RAÇA da vítima (78,7% dos homens mortos são negros) existe no banco 2015-2023 e a tela afirma 'negro' no texto sem mostrar o dado
  - dado: mortes_violentas tem coluna `raca` (preta/parda/branca/amarela/indigena/ignorada) para tipo='homicidio', anos 2015-2023. Query 2023: entre homens assassinados com raça informada, negros (preta+parda)=30.381 de 38.621 = 78,7%; entre mulheres
  - mudanca: Backend: nova função repository.homicidios_por_raca_sexo(ano) (GROUP BY raca,sexo em mortes_violentas WHERE tipo='homicidio'); expor em GeneroPanorama (schemas.py) e service.genero. Frontend: um Stat '% de negros entre os assassinados' e/ou
- **[alta/bug_residual]** O gráfico 'A violência tem sexo' plota Total/Por parceiro/Física/Sexual e OMITE Psicológica — que é maior que Sexual e já vem no DTO
  - dado: SexoResumo.psicologica é preenchido em service.py:289 (psicologica=d['psicologica']) e tipado em api.ts. Mas opcaoSexo (Genero.tsx:49-67) usa cats=['Total','Por parceiro','Física','Sexual'] e data=[x.total,x.parceiro,x.fisica,x.sexual] — `x
  - mudanca: Frontend-only: em opcaoSexo incluir 'Psicológica' em cats e x.psicologica em data (o campo já existe no objeto SexoResumo). Arquivo: frontend/src/pages/violencia/Genero.tsx (linhas 51 e 62).
- **[media/indicador_raso]** Homicídio por sexo é mostrado só no ano selecionado; a série 2015-2023 (queda de 57,9 mil homens em 2017 para 39,0 mil em 2023) existe e não vira tendência
  - dado: mortes_violentas tem série completa por ano×sexo. Query: M 2015=52.730, 2017=57.916 (pico), 2019=39.750, 2023=39.028; F 2015=4.573, 2017=4.877, 2023=3.867. O endpoint só devolve homicidios_por_sexo de UM ano (homic_ano). Não há nenhuma séri
  - mudanca: Backend: repository.homicidios_serie_por_sexo() (GROUP BY ano,sexo); expor em GeneroPanorama. Frontend: gráfico de linhas M vs F ao longo do tempo no card 'Duas violências'. Arquivos: backend/app/domains/conjugal/repository.py, service.py, 
- **[media/dado_subaproveitado]** Faixa etária e meio (arma de fogo) do homicídio existem por sexo e sustentariam o 'jovem, na rua' que a tela só afirma
  - dado: mortes_violentas tem `faixa` (0-14/15-29/30-44/45-59/60+) e `meio` (arma_fogo/arma_branca/outro) por sexo, tipo='homicidio'. Query 2023: homens 15-29=18.735 de ~38k (49% dos mortos com idade informada são jovens); homens por arma de fogo=28
  - mudanca: Backend: agregações homicidios_por_faixa_sexo e/ou por_meio_sexo; expor em GeneroPanorama. Frontend: Stats '% jovens (15-29) entre os assassinados' e '% por arma de fogo' no card 'Duas violências'. Arquivos: backend/app/domains/conjugal/rep
- **[media/dado_subaproveitado]** Repetição e agressor alcoolizado do TOTAL notificado (por sexo) vêm no DTO e nunca são exibidos — 40% das notificações são de violência de repetição
  - dado: SexoResumo carrega `repeticao` e `autor_alcool` (service.py:290-291), tipados em api.ts, mas Genero.tsx nunca os lê (só total/parceiro/fisica/sexual/pct_*). Query SINAN 2024 geral: repeticao=248.328 de 616.386 = 40,3%; autor_alcool=139.788 
  - mudanca: Frontend-only: dois Stats no bloco de gênero — '% de violência de repetição' e '% com agressor alcoolizado' — usando campos já presentes em `por_sexo`. A aba Conjugal só mostra isso DENTRO de parceiro; aqui seria o total, por sexo. Arquivo:

## Grafo
- **[media/dado_subaproveitado]** Os endpoints do grafo aceitam ano e partido, mas o frontend nunca envia — o grafo da Câmara mistura silenciosamente 3 exercícios e não há corte por partido
  - dado: app/api/grafo.py: despesas-camara aceita ano E partido; despesas-senado aceita ano. Dados existem: SELECT ano,count(*) FROM despesas => 2024, 2025, 2026; FROM despesas_senado => 2022..2025; SELECT count(distinct partido) FROM deputados => 2
  - mudanca: frontend/src/pages/Grafo.tsx: adicionar select de ano (para camara/senado) e de partido (para camara), passando-os em queryFn — o backend já trata. Nenhuma mudança de backend necessária.
- **[media/bug_residual]** A aba 'Rede de relações' sem filtro ordena por valor e enche o grafo só com os contratos PNCP de empresa sancionada (R$52,5 bi), enterrando as relações de cota/sócio/doação que a legenda promete
  - dado: graph.py::grafo_fraude faz order_by(Alerta.valor.desc()).limit(250). Como fornecedor_sancionado/PNCP concentra R$52,5 bi (visto acima) e valores unitários bilionários, os top-250 por valor são quase todos esses. Além disso grafo_fraude só e
  - mudanca: backend app/services/graph.py::grafo_fraude: quando não há filtro de tipo, ou balancear a amostra por gatilho (não puxar tudo por valor), ou permitir excluir fornecedor_sancionado do default; e alinhar o texto/legenda da aba em frontend/src

## Juventude
- **[alta/dado_subaproveitado]** A juventude interrompida é tratada como um número único, mas o SIM permite quebrar por raça e sexo — e revela que o morto típico é o jovem pardo/preto homem, não 'o jovem'
  - dado: mortes_violentas (tipo='homicidio', faixa='15-29') tem raca e sexo. O endpoint /juventude (violencia.py L282) usa só violencia.homicidios_jovens (contagem agregada do Atlas) — nunca abre por raça/sexo. Prova (2022, homicídios jovens 15-29 p
  - mudanca: Backend (violencia.py, endpoint juventude): adicionar bloco 'composicao' com GROUP BY raca, sexo sobre mortes_violentas tipo='homicidio' AND faixa='15-29' no ano; opcionalmente taxa de jovens negros vs brancos usando populacao_grupo (jovens

## Licitacoes
- **[alta/dado_subaproveitado]** A aba Licitações mostra só o valor ESTIMADO e ignora o valor_homologado — o resultado real do certame — que já vem no payload e no tipo TS
  - dado: contratacoes.valor_homologado. Query: SELECT count(*) FROM contratacoes WHERE valor_homologado IS NOT NULL => 51.233 de 72.862 linhas (70%). Agregado nas que têm os dois: estimado R$47.712.756.893 vs homologado R$42.425.254.659 em 49.202 li
  - mudanca: Frontend puro (nenhuma mudança de backend): em frontend/src/pages/Licitacoes.tsx, na tabela de licitações (bloco thead/tbody 'aba === licitacoes', linhas ~248-283) acrescentar coluna 'Valor homologado' usando l.valor_homologado, e idealment
- **[alta/dado_subaproveitado]** A aba Contratos não expõe a coluna materializada categoria_seguranca, que já classifica 30 mil contratos em armamento, munição, viaturas, presídios e videomonitoramento
  - dado: contratos.categoria_seguranca (materializada, indexada). Query: SELECT categoria_seguranca, count(*) FROM contratos GROUP BY 1 => videomonitoramento 13.535, presidios 8.450, viaturas 5.628, armamento 1.897, municao 905, protecao 250, conten
  - mudanca: Backend: em backend/app/api/licitacoes.py, no listar_contratos, adicionar param 'categoria: str | None = None' e um where Contrato.categoria_seguranca == categoria (é indexado, filtro barato mesmo em 4,4 M linhas). Frontend: em Licitacoes.t
- **[media/dado_subaproveitado]** A aba Licitações não expõe o filtro de situação (o backend aceita), nem os cortes esfera e poder — licitações Revogadas/Anuladas/Suspensas ficam impossíveis de isolar
  - dado: listar_contratacoes já aceita 'situacao' (licitacoes.py:18,32-33), mas o formulário (Licitacoes.tsx:155-193) só tem busca, UF e modalidade. Query situacao: 'Divulgada no PNCP' 71.856, 'Revogada' 579, 'Anulada' 308, 'Suspensa' 119 — 1.006 ce
  - mudanca: Frontend: adicionar <select> de situação (e opcionalmente esfera: Municipal/Federal/Estadual) em Licitacoes.tsx, passando o param já suportado; para esfera/poder incluir os params novos no backend (where simples e indexado). O de situação é

## ModeloRisco
- **[alta/indicador_raso]** As 2 features mais 'importantes' do modelo (idade da empresa e situação irregular) só existem para 8,8% dos fornecedores — o gráfico de importância engana sobre o que decide a nota da maioria
  - dado: data/modelo_fraude.meta.json importancias => idade_empresa_dias 0.195, situacao_irregular 0.190, e TODAS as demais 0.0. Mas scores_fraude.features.enriquecida (tem cadastro da Receita) => {False: 15.548, True: 1.498} de 17.046 (só 8,8%). Pa
  - mudanca: backend app/api/fraude.py::modelo_info devolver o total 'enriquecidos' (contar features.enriquecida). frontend/src/pages/ModeloRisco.tsx: no card 'O que mais pesa na decisão' e/ou na Stat 'Fornecedores analisados', dizer que só N (8,8%) têm
- **[media/dado_subaproveitado]** A decomposição score_modelo × score_anomalia prova o que o texto técnico afirma, mas nunca é quantificada: dos 'fora do padrão', 205 são achados SÓ pela anomalia e 0 pelo classificador
  - dado: scores_fraude.score_modelo / score_anomalia. SELECT count(*) FROM scores_fraude WHERE score_anomalia>=0.7 AND score_modelo<0.3 => 205 (anomalia pura); WHERE score_modelo>=0.7 AND score_anomalia<0.3 => 0 (modelo puro). SELECT count(*) WHERE 
  - mudanca: backend app/api/fraude.py::modelo_info: agregar contagens por origem (anomalia-only / modelo-only / ambos). frontend/src/pages/ModeloRisco.tsx: uma Stat ou pequena barra 'X casos vêm do reconhecimento de padrão já denunciado, Y são anomalia

## Mundo
- **[media/dado_subaproveitado]** O service já computa extremos_baixos e global_oms, mas a tela só mostra 'Mais alto' e nunca a linha do agregado mundial da OMS no strip-plot
  - dado: referencia_global tem pais='GLOBAL' para homicidio_taxa (6,098), suicidio_taxa e alcool_per_capita; e países no extremo baixo (homicídio: Singapore/Japan 0,181, Bahrain 0,272, Switzerland 0,443). Campos extremos_baixos e global_oms já em fr
  - mudanca: Frontend frontend/src/pages/violencia/Mundo.tsx: no IndicadorCard, adicionar a marca 'mundo (OMS)' com ind.global_oms no markLine (hoje só há mediana e Américas) e uma linha 'Mais baixo:' com ind.extremos_baixos ao lado de 'Mais alto:'. Sem

## Municipios
- **[alta/indicador_raso]** "Quem financiou a eleição" é enganoso: ~90% é dinheiro de PARTIDO (fundo partidário), e os maiores 'doadores' são os próprios partidos — a coluna `origem` que separa isso é ignorada
  - dado: doacoes tem coluna `origem`. Rio de Janeiro: SELECT origem,sum(valor) ... WHERE municipio='RIO DE JANEIRO' → Recursos de partido político R$154,7M vs Pessoas físicas R$11,7M vs Próprios R$3,1M (partido = 90% do total). Top 'doadores' que a 
  - mudanca: Quebrar as doações por `origem` (partido / pessoas físicas / próprios / outros candidatos) no /detalhe e /dossie, e separar 'financiamento externo (PF)' do 'repasse partidário'. Arquivos: backend/app/api/municipios.py (detalhe e dossie), fr
- **[media/dado_subaproveitado]** As doações do município misturam campanha de prefeito e de vereador; a coluna `cargo` separa os dois mas a tela não quebra
  - dado: doacoes tem `cargo`. Rio: SELECT cargo,sum(valor) ... WHERE municipio='RIO DE JANEIRO' → Prefeito R$59,3M · Vereador R$112,5M. O /detalhe (backend/app/api/municipios.py) soma tudo em `doado_2024` e lista doadores sem distinguir a qual dispu
  - mudanca: Adicionar corte por cargo no total e nos doadores (quanto foi para a corrida de prefeito vs para as 50+ campanhas de vereador). Arquivos: backend/app/api/municipios.py (detalhe), frontend/src/pages/Municipios.tsx.

## Panorama
- **[media/dado_subaproveitado]** O panorama expõe homicídios de jovens e de mulheres, mas oculta a coluna homicidios_homens — apagando que ~91% das vítimas são homens
  - dado: A tabela violencia tem a coluna homicidios_homens preenchida em série completa 1980-2024, mas resumo/serie (violencia.py L160/L209) só devolvem homicidios_jovens e homicidios_mulheres. Prova (Brasil): 2023 homicidios=45747, homens=41744, mu
  - mudanca: Backend (violencia.py resumo+serie e por_uf): incluir homicidios_homens (já na model) e, opcionalmente, taxa_homens/taxa_mulheres via populacao_grupo. Frontend (frontend/src/pages/violencia/Panorama.tsx): mostrar a divisão por sexo (barra h
- **[media/dado_subaproveitado]** acidentes_transito e acidentes_transito_jovens existem na tabela violencia em série completa e não são expostos por endpoint nenhum — uma causa de morte violenta maior que o suicídio fica invisível
  - dado: Colunas acidentes_transito e acidentes_transito_jovens da tabela violencia estão preenchidas 1980-2024 e nenhum dos endpoints as lê. Prova (Brasil): 2024 acidentes_transito=38253 (jovens=10388), 2023=35938, 2015=39543. No microdado SIM (mor
  - mudanca: Backend (violencia.py resumo/serie): devolver acidentes_transito e acidentes_transito_jovens no bloco Brasil, por_uf e na série. Frontend (frontend/src/pages/violencia/Panorama.tsx): segunda linha na série temporal e/ou card comparando homi

## PresidioEscola
- **[alta/dado_subaproveitado]** A razão gasto-segurança ÷ gasto-educação é mostrada só no último ano, mas o banco tem 41 anos dessa série (1985→2025), com a razão subindo de 0,37 para o pico 0,72 em 2020
  - dado: despesa_funcao tem seguranca de 1985-2025 e educacao de 1965-2025 (ambas 27 UFs desde 1985). Agregando o Brasil por ano: razão seg/edu = 1985:0.368, 1995:0.404, 2005:0.545, 2015:0.619, 2020:0.719 (pico), 2024:0.548, 2025:0.559 — 41 anos com
  - mudanca: No handler presidio_escola (backend/app/api/violencia.py ~L425-520), que hoje lê só `ano`, adicionar uma query que soma seguranca e educacao por ano (GROUP BY ano) e devolve `serie_brasil: [{ano, seguranca, educacao, razao}]` (e opcionalmen

## PressaoSocial
- **[alta/indicador_raso]** A correlação carência × homicídio é publicada só no ano selecionado, escondendo que a pobreza passou a andar cada vez MAIS junto com a morte (r 0,44→0,76 de 2012 a 2021)
  - dado: indicadores_sociais tem série 2012-2025 (desemprego/gini/pobreza; analfabetismo 2016-2025) e violencia por UF cobre 1979-2024. Recalculando o Pearson da tela ano a ano: pobreza [(2012,0.438),(2015,0.621),(2019,0.727),(2021,0.764),(2023,0.56
  - mudanca: No handler pressao_social (backend/app/api/violencia.py ~L1402-1533) já se calcula _pearson por eixo para um ano; envolver o cálculo num loop sobre `disponiveis` (2012-2024) e devolver um campo `serie_correlacao: [{ano, <chave>: r}]` por ei

## QuemMorre
- **[alta/dado_subaproveitado]** A letalidade policial (intervenção legal) está no banco — 16.259 mortes, subindo, 84% pardos/pretos — e nenhuma tela da plataforma a expõe
  - dado: mortes_violentas tem tipo='intervencao_legal' (mortes decorrentes de ação policial, CID Y35), completamente ignorado por todos os endpoints. Provas: total 2015-2023 = 16.259; série por ano 933(2015)→1357→1833→2006→1443→2146→2254→2013→2274(2
  - mudanca: Backend (violencia.py, endpoint quem_morre ou novo bloco): agregar mortes_violentas tipo='intervencao_legal' por raca, uf e série anual; devolver com nota metodológica (é o dado do SIM, não conciliado com anuários de segurança). Frontend (f
- **[media/dado_subaproveitado]** QuemMorre abre por raça e por sexo×local, mas nunca usa a coluna faixa — a idade da vítima, dado central de 'quem morre', fica de fora
  - dado: mortes_violentas.faixa (0-14, 15-29, 30-44, 45-59, 60+) existe e o endpoint /quem-morre (violencia.py L1682) não a consulta em momento algum. Prova (homicídio 2022 por faixa): 15-29=21276 (43% do total), 30-44=14522, 45-59=5345, 60+=1954, 0
  - mudanca: Backend (violencia.py quem_morre): adicionar bloco por_faixa (GROUP BY faixa, tipo='homicidio', ano) com pct. Frontend (frontend/src/pages/violencia/QuemMorre.tsx): seção de pirâmide/idade ao lado de raça e sexo×local.

## Regioes
- **[alta/dado_subaproveitado]** O backend já calcula e devolve taxa_homicidio_mulheres por região, mas a tela nunca a exibe — a dimensão de feminicídio some entre os 4 indicadores mostrados
  - dado: Campo taxa_homicidio_mulheres computado em backend/app/domains/regioes/repository.py (linha ~156) e tipado em frontend/src/lib/api.ts (linha 841). Fonte: violencia.homicidios_mulheres por UF (2024: BA=414, SP=351, PE=271; total 3.642) ÷ pop
  - mudanca: Frontend frontend/src/pages/violencia/Regioes.tsx: adicionar 'taxa_homicidio_mulheres' ao array METRICAS (nova barra 'Homicídio de mulheres · por 100 mil mulheres') e à tabela. Zero mudança de backend — o número já vem no payload.
- **[media/dado_subaproveitado]** violencia tem homicidios_jovens e acidentes_transito por UF, nunca agregados por região — a narrativa 'homem jovem' da plataforma não tem recorte regional
  - dado: SELECT ano,sum(homicidios_jovens),sum(acidentes_transito) FROM violencia WHERE length(codigo_ibge)=2 GROUP BY ano → 2024: homicidios_jovens=19.801, acidentes_transito=38.253 (também acidentes_transito_jovens=10.388, homicidios_homens=38.882
  - mudanca: Backend regioes/repository.py._violencia_por_uf: incluir homicidios_jovens e acidentes_transito na soma por região e devolver taxa_homicidio_jovens / taxa_transito. Frontend: novas métricas no grid. Colunas já existem, sem ETL.

## RelacaoToxica
- **[media/dado_subaproveitado]** per_capita_litros (7,8 L de álcool puro) e binge (34,7%) chegam no payload e não aparecem em lugar nenhum
  - dado: toxica.py:89-90 injeta brasil.per_capita_litros=OMS_PER_CAPITA_LITROS (7.8) e brasil.binge=OMS_BINGE (0.347). O tipo já os declara: api.ts:891-892 (per_capita_litros, binge). Grep em RelacaoToxica.tsx: nenhuma referência a per_capita nem bi
  - mudanca: Em frontend/src/pages/violencia/RelacaoToxica.tsx acrescentar dois stats/linha de contexto: '7,8 litros de álcool puro por adulto/ano (OMS 2016)' e 'consumo em binge nos últimos 30 dias: 34,7% dos adultos'. Fonte OMS já está em d.fontes. Pu
- **[media/dado_subaproveitado]** consumo_alcool guarda o indicador 'frequente' (bebem regularmente) por UF e a tela de exposição só usa 'abusivo'
  - dado: SELECT indicador, COUNT(*) FROM consumo_alcool GROUP BY indicador -> abusivo=5592, frequente=28 (Brasil+27 UF, PNS 2019). Brasil frequente=0,264 (26,4% dos adultos) vs abusivo=0,171. Top UF frequente: RS=0,34, MS=0,313, SP=0,31. O repositór
  - mudanca: Adicionar repo.consumo('frequente') em backend/app/domains/alcool/toxica.py e incluir a taxa por UF no schema/tabela; em RelacaoToxica.tsx acrescentar a coluna 'bebem regularmente'. Sem ETL — os 28 registros já foram importados por sync-con

## SaudeMental
- **[media/indicador_raso]** A tabela por estado mostra leitos psiquiátricos em contagem BRUTA (SP=8.238) enquanto o mapa usa CAPS por 100 mil — o endpoint já devolve leitos_por_100mil e a tela o ignora, fazendo estado grande parecer bem servido
  - dado: O handler backend/app/api/violencia.py (linha ~1952) já computa 'leitos_por_100mil' e o tipo em frontend/src/lib/api.ts linha 1133 já o inclui. Raw distorce: SP 8.238 leitos brutos vs estados pequenos, mas por 100 mil hab a ordem muda. Colu
  - mudanca: Frontend frontend/src/pages/violencia/SaudeMental.tsx: na tabela 'Estado por estado' e no CSV, adicionar coluna 'Leitos por 100 mil' usando u.leitos_por_100mil (já no payload); opcionalmente um Stat Brasil de leitos/100mil. Zero backend.

## SegurancaContratos
- **[media/indicador_raso]** O ranking de UFs por valor de contratos de segurança é absoluto, dominado pelo tamanho do estado; falta o valor POR HABITANTE, que reordena completamente a lista
  - dado: violencia por UF traz populacao (ano 2024) e o handler já a carrega para pegar taxa_homicidios. Contratos plausíveis por UF: TOP absoluto = DF 3.142 mi, AL 2.140, SP 1.859, MG 1.523, RS 1.310, RJ 1.107. TOP por habitante = DF 1053, AL 665, 
  - mudanca: No handler seguranca_contratos (backend/app/api/violencia.py ~L695-844), o dict `taxas` (L764) já vem de Violencia por UF; incluir também populacao e no por_uf calcular `valor_por_habitante = valor / populacao`. No frontend (frontend/src/pa

## Senadores
- **[alta/dado_subaproveitado]** O endpoint da CEAPS já calcula o gasto por CATEGORIA e a tela joga fora — o senador não tem o "em que gasta" que o deputado tem
  - dado: backend/app/api/senadores.py já retorna `por_tipo` em /api/senadores/ceaps/resumo, mas frontend/src/pages/Senadores.tsx só declara `por_tipo` na interface (linha 28) e NUNCA o renderiza (grep confirma: única ocorrência é a tipagem). Query p
  - mudanca: Renderizar `por_tipo` como gráfico de barras horizontais 'Em que o Senado gasta a CEAPS' — espelhando o card 'Em que ele gasta' que DeputadoDetalhe.tsx já tem (opcaoTipos). Zero backend: o dado já vem no payload. Arquivo: frontend/src/pages
- **[media/indicador_raso]** A CEAPS tem 4 anos (2022–2025) com alta de +36%, mas a tela funde tudo num total só e ignora o parâmetro `ano` que o endpoint aceita
  - dado: backend/app/api/senadores.py: resumo_ceaps(ano: int | None) aceita filtro por ano. Senadores.tsx (linha 67) chama sem `ano`. Série real: SELECT ano,sum(valor) FROM despesas_senado GROUP BY ano → 2022 R$27,3M · 2023 R$29,0M · 2024 R$32,2M · 
  - mudanca: Adicionar seletor de ano (como DeputadoDetalhe.tsx já faz) passando `ano` ao endpoint, e/ou um stat 'variação vs ano anterior'. O total mensal já agrega por ano; basta agrupar/rotular por ano e mostrar a tendência de alta. Arquivos: fronten
- **[media/dado_subaproveitado]** Não há drilldown por senador — despesas_senado tem senador_nome+tipo+mês+fornecedor, permitindo a MESMA ficha que o deputado ganha, mas o card só linka para /fraude
  - dado: despesas_senado tem senador_nome, tipo, ano, mes, fornecedor_nome, fornecedor_cnpj por linha. Prova de que dá para montar a ficha de UM senador: SELECT tipo,sum(valor) FROM despesas_senado WHERE senador_nome LIKE '%RANDOLFE%' GROUP BY tipo 
  - mudanca: Dar a resumo_ceaps um parâmetro `senador` (filtro por senador_nome) reusando as agregações mensal/por_tipo/por_fornecedor já escritas, e criar SenadorDetalhe.tsx espelhando DeputadoDetalhe. Arquivos: backend/app/api/senadores.py (param), fr

## Suicidio
- **[alta/dado_subaproveitado]** A tela de suicídio mostra só o total agregado, mas o SIM tem sexo, faixa etária e meio — ignora que homem se mata ~3,5x mais que mulher e que a arma de fogo é a exceção (o oposto do homicídio)
  - dado: mortes_violentas (tipo='suicidio', 2015-2023) tem sexo/faixa/meio/raca por município. O endpoint /suicidio (violencia.py L1269) NÃO toca nessa tabela — usa só violencia.suicidios/taxa_suicidios agregados. Provas (2022): por sexo M=12893 vs 
  - mudanca: Backend (backend/app/api/violencia.py, endpoint suicidio): consultar mortes_violentas com tipo='suicidio' agregando por sexo, faixa e meio no ano; devolver blocos por_sexo (com taxa usando total-mulheres p/ homens e mulheres p/ elas), por_f