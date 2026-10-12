# Base analítica E3

## Estado de publicação E5

Este guia é a referência metodológica. As conferências atuais e divergências
estão em `docs/entregas/E5.md`; o contrato público está em `docs/publicacao.md`.
Consistência estrutural não certifica autenticidade nem reconciliação oficial.
Não há resultados reais de 2026 carregados no catálogo conferido; não se presume
indisponibilidade desses dados nas fontes oficiais.

## Operação

Na raiz `.`:

```powershell
uv run eleitoral build-marts
uv run eleitoral validate
uv run eleitoral summary --year 2026 --round 1 --level uf
uv run eleitoral summary --year 2022 --round 2 --level nacional --candidate Lula
uv run eleitoral summary --year 2022 --round 1 --level secao --uf AC --candidate Lula
uv run eleitoral compare --candidate Lula --round 1
uv run eleitoral compare --dataset DEMONSTRATION
```

`summary` aceita `secao`, `zona`, `municipio`, `uf`, `nacional`, filtros territoriais,
`--office` e `--limit`. O padrão é `SOURCE`; exemplos fictícios exigem
`--dataset DEMONSTRATION`. `ALL` mantém namespaces em linhas separadas.
Nenhum registro significa indisponibilidade, não votação zero.

O manifesto seleciona somente importações `CONCLUIDA`, sem glob de bancos ou diretórios.
Sobreposição de chaves entre arquivos bloqueia publicação, sem deduplicação silenciosa.
O build transacional preserva a versão DuckDB anterior em caso de falha SQL/integridade.
Os sete Parquets ficam em snapshots UUID em
`data/marts`.
`current.json` aponta para a publicação atual; `analytical_builds` guarda proveniência.
DuckDB e filesystem não têm transação distribuída: em interrupção entre commit e ponteiro,
consulte o catálogo de builds e execute novamente. Não consuma todos os UUIDs juntos.
Snapshots anteriores não são apagados automaticamente. Origens CSV/Parquet não são escritas.

## Modelo e marts

Chave dos fatos: dataset, eleição, ano, turno, cargo, UF, município, zona e seção.
Votos por candidato acrescentam `candidate_id`. Zeros à esquerda são preservados:
identificadores textualmente diferentes não são automaticamente equiparados.

- `dim_election`: eleição/turno/cargo/abrangência; cargo ausente é `UNKNOWN`.
- `dim_candidate`: candidatura em uma eleição/turno/cargo, com sequência quando disponível.
- `dim_geography`: UF/município/zona/seção; código municipal é do TSE, não do IBGE.
- `fact_candidate_votes`: candidato por seção; exclui brancos/nulos e números de legenda.
- `fact_ballot_votes`: todas as linhas normalizadas da votação, inclusive categorias especiais,
  com importação e hash; staging mantém as colunas originais.
- `fact_section_turnout`: participação por seção/eleição/turno/cargo, nunca por candidato.
  Quando falta detalhe, conserva brancos/nulos explicitamente registrados nos números 95/96,
  deixando aptos, comparecimento e válidos indisponíveis.
- `mart_section_turnout`: participação e taxas, uma linha por chave de seção/cargo.
- `mart_section_results`: candidato na seção, votos, denominadores e percentuais.
- `mart_section_historical_comparison`: pares 2022/2026, mesmo turno/cargo, candidato nominal,
  votos, diferenças, percentuais, variações em pp, comparecimento e abstenção.
- `mart_zone_results`, `mart_municipality_results`, `mart_state_results`,
  `mart_national_results`: candidato por eleição/turno/cargo e território correspondente.

Denominadores territoriais são somados antes do join de candidatos, considerando todas as
seções existentes nos fatos do território, não apenas as linhas do candidato.
Qualquer ausência de denominador mantém a soma publicada como NULL.
`abstentions` segue a soma estrita (NULL quando alguma seção não instalada não
reporta o valor). A **taxa** `abstention_pct` da API pública é publicada mesmo
nesses recortes: usa `sum(abstentions)` sobre as seções que reportam e
`sum(eligible)` restrito a essas mesmas seções, sem inferir abstenção para quem
não reporta. No Brasil de 2026 (1º turno, presidente), a taxa é ~21,08% sobre
499.207 das 499.248 seções.
`sections`, `candidate_sections`, `eligible_sections`, `attendance_sections` e
`valid_vote_sections` explicitam cobertura **dos registros importados**, não certificam
cobertura oficial total. O mart nacional significa agregado da base importada, que pode
ser um recorte. Não some aptos ou comparecimento entre candidatos ou cargos.
SQL de desenvolvimento:
`sql/marts`.
Os scripts também são incluídos no wheel.

## Dicionário de indicadores

| Campo | Definição/cálculo |
| --- | --- |
| `votes` | QT_VOTOS do candidato; ausência nunca é zero |
| `eligible` | QT_APTOS da seção/cargo |
| `attendance` | QT_COMPARECIMENTO |
| `abstentions` | QT_ABSTENCOES (alias QT_ABSTENCAO aceito em E2) |
| `valid_votes` | QT_VOTOS_VALIDOS; se ausente, QT_VOTOS_NOMINAIS + QT_VOTOS_LEGENDA, somente com ambos disponíveis |
| `nominal_votes`, `party_votes` | Votos nominais e de legenda, preservados separadamente |
| `blank_votes`, `null_votes` | QT_VOTOS_BRANCOS e QT_VOTOS_NULOS ou registro explícito 95/96 na ausência de detalhe |
| `separately_annulled_votes` | QT_VOTOS_ANULADOS_APU_SEP; não somado automaticamente aos válidos |
| `abstention_pct` | 100 × abstenções / aptos |
| `attendance_pct` | 100 × comparecimento / aptos |
| `blank_pct`, `null_pct` | 100 × categoria / oportunidades de voto no cargo |
| `valid_vote_pct` | 100 × votos do candidato / votos válidos no cargo |
| `eligible_vote_pct` | 100 × votos do candidato / eleitores aptos |
| `vote_difference` | votos 2026 − votos 2022 |
| `*_pp_change` | percentual 2026 − percentual 2022, em pontos percentuais |

Divisões usam NULLIF: denominador zero ou ausente produz NULL. Percentuais vão de 0 a 100
para indicadores usuais. Para senador em 2026 existem dois votos por eleitor: oportunidades
de voto = 2 × comparecimento; em 2022 = comparecimento. A participação nos válidos conta
votos, enquanto a participação entre aptos é uma proporção observada por candidato.
Esta última não mede persuadibilidade, conversão política nem probabilidade individual.
Nenhuma causa é atribuída a abstenções, brancos ou nulos.

Fontes oficiais consultadas: TSE, “Votos brancos e nulos não anulam eleição e são
considerados apenas para fins estatísticos” (setembro de 2021); TSE, “100% das seções
totalizadas: confira como ficou o quadro eleitoral após o 2º turno” (31/10/2022);
TRE-CE, tabela “RESULTADO GERAL - 2º TURNO” (geração 04/11/2022); TRE-SP,
“Eleições 2026: conheça a ordem dos 6 votos na urna” (junho de 2026).
Conferência externa não autentica os arquivos locais.

## Comparabilidade histórica

O pareamento automático usa namespace, UF, município, zona, seção, turno, cargo e nome
normalizado do candidato. Número de urna isolado não identifica uma pessoa entre eleições.
Nome igual também não prova identidade: homônimos e alterações de nome exigem revisão.
Não há inferência de continuidade de eleitorado nem de transferência de votos individuais.

- `VERIFIED_CONTINUITY`: vínculo explícito importado com fonte da evidência.
- `EXACT_KEY_UNVERIFIED`: chaves exatas e candidato nominal coincidente, sem continuidade verificada.
- `REORGANIZED`: reorganização documentada na interface de vínculos.
- `UNMATCHED`: candidato/seção sem contraparte; não significa seção extinta.
- `UNKNOWN`: metadados insuficientes, inclusive cargo desconhecido.

Reorganizações podem produzir vários pares. **Não some diferenças entre pares** de vínculos
um-para-muitos; elas não constituem redistribuição estimada do eleitorado. Classificações:
`POSITIVE_TO_CONFIRMED_ZERO`, `VOTE_GROWTH`, `VOTE_REDUCTION`, `UNCHANGED`,
`COMPARISON_UNAVAILABLE`. Somente zero explicitamente registrado autoriza a primeira.

## Geografia e contexto futuro

`dim_polling_place` conserva número, endereço, município, UF e fontes de importação;
`bridge_section_polling_place` liga seção/eleição/cargo a local. Endereço sem coordenadas
fica `ADDRESS_UNVERIFIED`. Nenhuma geocodificação é executada. Local de votação não é
necessariamente residência dos eleitores. Coordenadas `VERIFIED` significam declaração
do responsável pela importação, não auditoria automática da fonte.

```powershell
uv run eleitoral import-context --kind coordinates --file C:/dados/coordenadas.csv
uv run eleitoral import-context --kind continuity --file C:/dados/continuidade.csv
uv run eleitoral import-context --kind municipality-mapping --file C:/dados/tse-ibge.csv
uv run eleitoral import-context --kind census --file C:/dados/censo.csv
uv run eleitoral build-marts
```

CSV UTF-8 com separador reconhecido pelo DuckDB e cabeçalhos exatos; todos os campos
obrigatórios, duplicação rejeitada, carga transacional. Execute primeiro build-marts.
Cabeçalhos por tipo:

- coordinates: `polling_place_id,latitude,longitude,location_source,verification_status`.
  Status obrigatório `VERIFIED`, latitude −90..90 e longitude −180..180.
- continuity: `dataset,uf,municipality,zone_2022,section_2022,zone_2026,section_2026,status,evidence_source`.
  Status VERIFIED_CONTINUITY, REORGANIZED ou UNKNOWN; só vínculos intramunicipais nesta versão.
- municipality-mapping: `uf,municipality,ibge_municipality,evidence_source,verification_status`.
  Status VERIFIED; nenhuma equivalência TSE/IBGE é presumida.
- census: `ibge_municipality,census_year,indicator,value,unit,source`.
  Interface de agregados municipais canônicos; não é leitor de qualquer layout bruto do IBGE.

`municipality_socioeconomic_context` faz associação somente por mapeamento verificado.
Não há fonte censitária real disponível: integração substantiva pendente.
`descriptive_clusters` é apenas contrato de armazenamento (município, run_id, rótulo, método,
parâmetros JSON e fonte). Nenhum algoritmo executado nem inferência individual/persuasão.

## Conferência pública

`validate` verifica contagens negativas, chaves duplicadas, território/ano/turno obrigatório,
coerência de comparecimento/aptos/abstenções, limites por cargo, pares históricos ambíguos
e reconciliação de votos em todos os níveis. Incompletude e cargo ausente geram avisos.
Avisos não significam validação de autenticidade nem ausência de todos os possíveis erros.
Não há testes, CI ou dependências adicionais nesta etapa.