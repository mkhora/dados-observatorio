# Publicação — contrato para E6

## Geração e atualização

Execute ingestão e `build-marts` fora do serviço público; depois `validate` e
`publish-snapshot --output snapshots/<nova-versao>` via `uv run eleitoral`.
Somente SOURCE é exportado, sem demonstrações, staging, CSV, banco, caminhos
locais, nomes de arquivos originais ou dados individuais de eleitores.
Nomes de candidatos são informações eleitorais públicas, não nomes de eleitores.

A leitura transacional fixa uma visão do banco; a exportação é comparada
integralmente ao mart em ambas as direções, incluindo NULLs e duplicatas.
Os sete marts são obrigatórios. Resultados/participação/agregados vazios bloqueiam
publicação. Histórico vazio ou sem contraparte é indisponibilidade, não erro.
Novas importações após o build exigem reconstrução.

Um temporário irmão é renomeado apenas depois de completar as conferências.
Destino existente é recusado. Lock exclusivo evita publicadores cooperantes
concorrentes. Erros removem temporário/lock; interrupção abrupta pode deixá-los:
confirme ausência de processo ativo antes de limpar `.pending-*` e `.publish.lock`.
Não publique em diretório gravável por terceiros. Não há promoção automática de
`current`; selecione a versão completa e conserve a anterior para rollback.

Arquivos recebem permissão/atributo somente leitura; no Windows isso não é uma
ACL imutável. O proprietário pode alterar permissões. Monte volume somente
leitura no consumidor com conta sem escrita. Hashes detectam alteração, não
autenticam o produtor; obtenha o manifesto por canal confiável. Não há assinatura.

## Arquivos e contrato

- `manifest.json`: versão 1.0.0, fontes, hashes de origem, datas de importação,
  build/publicação UTC, software, cobertura, verificações e limitações.
  Geração original desconhecida é `null`; fontes são `LOCAL_NOT_AUTHENTICATED`.
- `manifest.sha256`: hash SHA-256 do manifesto, sem dependência circular.
- `schema.json`: cópia de `src/eleitoral/schemas/public-v1.json`, contrato 1.0.0
  com nomes, tipos DuckDB e nulabilidade de cada coluna exportada.
- Sete `<mart>.parquet`: hashes, contagens e tamanhos no manifesto.
  `data_version` deriva dos hashes/metadados; não se promete binário determinístico
  entre execuções ou versões do DuckDB.

Mudança incompatível de significado/coluna exige nova versão principal. Códigos
são texto e preservam zeros; município usa código TSE. `HUGEINT` é inteiro de
128 bits: a futura API deve evitar perda de precisão JSON/JavaScript. `DOUBLE`
representa taxas em porcentagem; diferenças de taxas são pontos percentuais (pp).
`NULL` é indisponível, nunca zero.

| Mart | Granularidade |
| --- | --- |
| `mart_section_results` | candidato por seção/eleição/turno/cargo |
| `mart_section_turnout` | seção/eleição/turno/cargo |
| `mart_section_historical_comparison` | candidato nominal por par 2022/2026, turno/cargo |
| `mart_zone_results` | candidato por zona/eleição/turno/cargo |
| `mart_municipality_results` | candidato por município/eleição/turno/cargo |
| `mart_state_results` | candidato por UF/eleição/turno/cargo |
| `mart_national_results` | candidato por eleição/turno/cargo na base importada |

Indicadores: votos, aptos, comparecimento, abstenções, válidos, brancos, nulos;
taxas de participação/brancos/nulos; percentuais do candidato entre válidos/aptos;
diferenças históricas absolutas/em pp; contagens e cobertura de seções.
Fórmulas e limites em `docs/analitica-e3.md`.
Não some denominadores entre candidatos/cargos. Para participação territorial
de cargo sem candidatos, agregue `mart_section_turnout` separadamente por
eleição/turno/cargo, com soma estrita: qualquer ausência mantém NULL.

## Disponibilidade

- Arquivo: `AVAILABLE` com linhas; `UNAVAILABLE` se vazio.
- Valor: `OBSERVED`, `CONFIRMED_ZERO`, `MISSING`; denominador ausente/zero
  torna percentual indisponível.
- Histórico: `UNMATCHED`, `UNKNOWN`, `EXACT_KEY_UNVERIFIED`,
  `VERIFIED_CONTINUITY`, `REORGANIZED`. Linhas podem existir sem comparação possível;
  examine contrapartes e `descriptive_classification`.
- `loaded=PASS` significa fontes carregadas; `structural_integrity.status=PASS`
  significa verificações internas aprovadas, com possíveis avisos;
  `export_integrity=PASS` significa exportação preservada. Não são reconciliação oficial.
- `official_reconciliation=NOT_RUN`: o comando não consulta fontes externas.
  A conferência manual parcial E5 não certifica todos os indicadores/cobertura.

`coverage` conta seções e denominadores não nulos por eleição/ano/turno/cargo/UF;
o denominador de cobertura é `sections` do recorte. `historical_coverage` separa
pares de candidatos e seções distintas. Fração pareada é
`matched_sections_2022 / sections_2022` quando positivo; não mede continuidade
comprovada nem cobertura sobre universo oficial.

## Consumo sem ingestão

A futura API deve abrir manifesto/schema/Parquets, conferir versões e hashes,
filtrar recortes e expor ausências/limitações. DuckDB em memória pode consultar
`read_parquet` de arquivo específico. Nunca use glob de todas as versões nem
conecte o banco de trabalho. Não execute ingestão/build no serviço público.
E5 não implementa endpoints, autenticação nem frontend React.