# Observatório Popular de Dados Eleitorais — base pública de dados

Repositório **somente de dados**: reúne os dados eleitorais **extraídos** das fontes
oficiais e as camadas **tratadas/derivadas** geradas pelo pipeline do Observatório
Popular. Não contém código de aplicação, banco de trabalho, credenciais, dados
pessoais de eleitores nem exports intermediários.

Este README descreve o conteúdo, a proveniência, os limites de leitura e o catálogo
de arquivos da base.

## Aviso de leitura (obrigatório)

- **`dataset = SOURCE`** identifica a base efetivamente importada. Ela **pode ser um
  recorte**: "nacional" agrega a base importada, não necessariamente o universo
  oficial completo.
- **Proveniência local.** Os arquivos de origem foram importados localmente:
  `provenance_status = LOCAL_NOT_AUTHENTICATED`. Os `sha256` do manifesto detectam
  alteração, mas **não autenticam** o produtor nem a procedência TSE.
- **`NULL` significa indisponível, nunca zero.** Não substitua ausência por zero em
  somas ou médias.
- **`official_reconciliation = NOT_RUN`.** Estes dados não passaram por reconciliação
  oficial automática contra as fontes do TSE. Não os apresente como resultado
  certificado.
- **LGPD.** Somente agregados por seção eleitoral; nenhum dado individual de eleitor.
  Nomes de candidatos são informação eleitoral pública.
- **Camadas derivadas (E7–E10) são descritivas.** Prioridade de mobilização, scores
  por proposta, guia porta a porta e bloco "Como agir" **não** inferem intenção
  individual, **não** estimam conversão de voto e **não** certificam identidade de
  eleitor.

## Estrutura do repositório

```text
.
├── README.md                       # este arquivo
├── LICENSE                         # MIT (software) — ver "Licença e atribuição"
├── .gitignore
├── data/
│   ├── priority.parquet            # insumo da API (cópia de snapshots/secoes_recuperaveis.parquet)
│   └── marts/current.json          # recibo do último build analítico
├── snapshots/
│   ├── e6-2022-2026-v1/            # snapshot público de resultados (2022 + 2026)
│   ├── secoes_*.parquet            # camadas derivadas por seção (E7–E9)
│   ├── zonas_virada_prioritarias.parquet
│   ├── como_agir.parquet           # bloco "Como agir" (E10)
│   └── locais_votacao_geo.parquet  # coordenadas por seção (geocode)
├── warehouse/
│   └── publicas/                   # bases públicas consolidadas (IBGE, DATASUS)
├── schema/public-v1.json           # contrato canônico das colunas (1.0.0)
├── guia_porta_a_porta_por_secao.csv
├── auditoria*.json / auditoria_final.md
├── relatorio_*.md
└── docs/                           # metodologia e contratos
```

## Sobre a versão `e5-2022-v1`

Este repositório contém **apenas o snapshot `e6-2022-2026-v1`**. O snapshot anterior
`e5-2022-v1` (somente 2022) **não** foi incluído, porque:

- `e6-2022-2026-v1` é **superconjunto**: contém 2022 **e** 2026, incluindo todos os
  marts que o E5 publicava, com o mesmo contrato (`schema 1.0.0`);
- nada neste repositório consome o E5 — a leitura é sempre pela versão ativa (E6).

O E5 tem valor apenas como **versão histórica publicada** (rollback/proveniência).
Para adicioná-lo, copie a pasta `snapshots/e5-2022-v1/` (manifesto, `manifest.sha256`,
`schema.json` e os sete Parquets) preservando o diretório intacto; **não** misture
arquivos de versões diferentes.

## Catálogo de arquivos

### A. Snapshot público de resultados — `snapshots/e6-2022-2026-v1/`

Contrato de 7 marts + manifesto + schema (ver `docs/publicacao.md`). É a versão
ativa consumida pela API e pelo frontend.

| Arquivo | Granularidade / conteúdo | Tamanho | Hospedagem |
| --- | --- | ---: | --- |
| `manifest.json` | fontes, hashes, datas, cobertura, verificações, limitações | ~41 KB | repo |
| `manifest.sha256` | hash do manifesto | 81 B | repo |
| `schema.json` | contrato 1.0.0 (nomes, tipos DuckDB, nulabilidade) | ~25 KB | repo |
| `mart_section_results.parquet` | candidato por seção/eleição/turno/cargo | ~174 MiB | **LFS/Release** |
| `mart_section_historical_comparison.parquet` | candidato por par 2022/2026 | ~156 MiB | **LFS/Release** |
| `mart_section_turnout.parquet` | participação por seção/eleição/turno/cargo | ~37 MiB | repo |
| `mart_zone_results.parquet` | candidato por zona | ~6,6 MiB | repo |
| `mart_municipality_results.parquet` | candidato por município | ~5,6 MiB | repo |
| `mart_state_results.parquet` | candidato por UF | ~40 KB | repo |
| `mart_national_results.parquet` | candidato por eleição/turno/cargo | ~7,2 KB | repo |

`data_version` desta versão: `fe3f416614dfa18e1a3e5dda9292dbbca66d5aa1377020934cb6bf332044140d`
(`published_at`: 2026-10-10T22:56:49Z).

### B. Camadas derivadas por seção — `snapshots/*.parquet`, `data/priority.parquet`

Camadas **derivadas e descritivas** geradas pelo pipeline (E7–E10). Chave canônica:
`section_id` (formato `SOURCE|UF|município|zona|seção`).

| Arquivo | Conteúdo | Tamanho |
| --- | --- | ---: |
| `secoes_prioritarias.parquet` | prioridade de mobilização por seção (E7) | ~20,9 MiB |
| `secoes_recuperaveis.parquet` | votos recuperáveis, gap, taxa de conversão, diagnóstico (E9) | ~34,3 MiB |
| `secoes_demanda.parquet` | 10 scores por proposta + `proposta_dominante` (E8) | ~14,3 MiB |
| `secoes_ibge.parquet` | Censo 2022 por seção (renda, idade, saneamento, alfabetização, coordenada) | ~19,0 MiB |
| `secoes_socio.parquet` | mortalidade (SIM), homicídios e cobertura de UBS (CNES) | ~10,6 MiB |
| `zonas_virada_prioritarias.parquet` | prioridade ALTA e score dominante ≥ corte (E8) | ~2,4 MiB |
| `como_agir.parquet` | proposta, justificativa, ação e mensagem por seção (E10) | ~16,0 MiB |
| `locais_votacao_geo.parquet` | coordenadas por seção (geocode) | ~10,9 MiB |
| `data/priority.parquet` | insumo da API (cópia de `secoes_recuperaveis.parquet`) | ~34,3 MiB |
| `guia_porta_a_porta_por_secao.csv` | mensagem, ação e material por seção prioritária (E8) | ~11,0 MiB |

### C. Bases públicas consolidadas — `warehouse/publicas/`

Bases oficiais já tratadas/agregadas por município (IBGE, DATASUS). Pequenas e
recomendadas para versionar.

| Arquivo | Conteúdo | Tamanho |
| --- | --- | ---: |
| `censo_municipio.parquet` | Censo 2022 consolidado por município | ~287 KB |
| `sim_municipal_2022.parquet` | SIM/DATASUS 2022 por município (câncer, homicídios) | ~45 KB |
| `ubs_municipal.parquet` | cobertura de UBS (CNES/DATASUS) por município | ~21 KB |
| `censo2022/mun_alfabetizacao.parquet` | alfabetização por município | ~30 KB |
| `censo2022/mun_basico.parquet` | população, domicílios, área por município | ~80 KB |
| `censo2022/mun_demografia.parquet` | jovens, idosos, crianças por município | ~72 KB |
| `censo2022/mun_domicilio2.parquet` | água, esgoto, lixo por município | ~57 KB |
| `censo2022/mun_renda.parquet` | renda média/mediana por município | ~111 KB |

Os arquivos brutos baixados dessas fontes (`warehouse/publicas/censo2022/*.zip`,
`warehouse/publicas/sim/sim_2022.csv|zip`) são grandes e reobtíveis: **não** estão aqui.

### D. Metadados, contratos e recibos

| Arquivo | Conteúdo |
| --- | --- |
| `snapshots/e6-2022-2026-v1/manifest.json` | proveniência, hashes, cobertura, limitações |
| `snapshots/e6-2022-2026-v1/manifest.sha256` | hash do manifesto |
| `snapshots/e6-2022-2026-v1/schema.json` | contrato de colunas da versão publicada |
| `schema/public-v1.json` | contrato canônico 1.0.0 (fonte do `schema.json`) |
| `data/marts/current.json` | recibo do último build analítico (run, contagens, avisos) |

### E. Relatórios e auditorias

| Arquivo | Conteúdo |
| --- | --- |
| `auditoria.json` | cobertura, integridade e reconciliação do snapshot 2022 (E7) |
| `auditoria_recuperaveis.json` | medições da camada de votos recuperáveis (E9) |
| `auditoria_como_agir.json` | régua, limiares e distribuição do bloco "Como agir" (E10) |
| `auditoria_otimizacao.json` | medições de otimização do banco |
| `auditoria_final.md` | checklist final e alertas (E7) |
| `relatorio_cruzamento.md` | fontes, cobertura, distribuição por UF e top 20 (E8) |
| `relatorio_como_agir.md` | contrato, régua e limitações do bloco "Como agir" (E10) |
| `relatorio_refatoracao.md` | comparativo da metodologia antiga × nova (E9) |
| `relatorio_otimizacao.md` | notas de otimização |

### F. Documentação metodológica — `docs/`

Sem estes documentos os números ficam sem definição de fórmula, denominador e
cobertura.

| Arquivo | Conteúdo |
| --- | --- |
| `docs/analitica-e3.md` | fórmulas, denominadores, chaves históricas e limites estatísticos |
| `docs/publicacao.md` | contrato dos marts, manifesto, schema e disponibilidade |
| `docs/calculo_antigo.md` | metodologia de prioridade anterior (E7) para comparação |
| `docs/entregas/E5.md` | snapshot 2022 anterior (histórico) |
| `docs/entregas/E6.md` | snapshot 2022 + 2026 e interface pública |
| `docs/entregas/E7.md` | prioridade de mobilização por seção |
| `docs/entregas/E8.md` | cruzamento socioeleitoral (IBGE/DATASUS) |

## Tamanho dos arquivos e limites do GitHub

O GitHub impõe **100 MiB por arquivo** (bloqueio rígido), alerta a partir de **50 MiB**
e recomenda repositórios abaixo de **1 GiB**. O conjunto atual soma **~554 MiB** em
47 arquivos. Por isso:

- **Versionar direto no repo** (cabe): todo o grupo B, o grupo C, os grupos D, E e F,
  os Parquets pequenos do snapshot e a documentação. Total aproximado: ~140 MiB.
- **Git LFS** (`git lfs track "*.parquet"`): `mart_section_results.parquet` (~174 MiB)
  e `mart_section_historical_comparison.parquet` (~156 MiB) do snapshot E6.
- **GitHub Releases / storage externo** (Zenodo, S3, R2): os CSVs de origem do TSE
  (`data/raw/`) e os brutos de `warehouse/publicas/`. São grandes e **reobtíveis no
  canal oficial** (TSE; IBGE; DATASUS): referencie-os por URL + `sha256` do manifesto
  em vez de carregá-los.
- **Nunca versionar**: `*.duckdb`, `*.wal`, `*.sqlite3`, `*.zip`, staging, temporários,
  caches de geocode e segredos.

### `.gitignore`

```gitignore
# Bancos de trabalho e temporários
*.duckdb
*.duckdb.wal
*.wal
*.sqlite3*
tmp/
.tmp/
*.pending
*.publish.lock

# Brutos grandes (reobtíveis nas fontes oficiais)
*.zip
warehouse/publicas/sim/
warehouse/geocode-cache.csv
data/raw/*.csv

# Segredos
.env
.env.*
!*.env.example

# Caches
__pycache__/
.ruff_cache/
.venv/
node_modules/
```

## Como consumir

Os arquivos são Parquet/CSV. Qualquer leitor de Parquet serve; com DuckDB:

```sql
-- Lula por seção no snapshot E6 (office/cargo é o código TSE: '1' = presidente)
SELECT uf, municipality, zone, section, candidate_name, votes
FROM read_parquet('snapshots/e6-2022-2026-v1/mart_section_results.parquet')
WHERE year = 2026 AND round = 1 AND office = '1' AND candidate_number = '13'
ORDER BY votes DESC
LIMIT 10;

-- Participação nacional 2026 (não some denominadores entre cargos)
SELECT year, round, office, sum(eligible) aptos, sum(abstentions) abstencoes
FROM read_parquet('snapshots/e6-2022-2026-v1/mart_section_turnout.parquet')
WHERE year = 2026 AND office = '1'
GROUP BY ALL;

-- Camada derivada: seções prioritárias (chave canônica section_id)
SELECT section_id, uf, municipality_name, zone, section, prioridade_recuperacao, gap_p1
FROM read_parquet('snapshots/secoes_recuperaveis.parquet')
WHERE prioridade_recuperacao = 'CRÍTICA'
ORDER BY gap_p1 DESC
LIMIT 20;
```

> `office` guarda o **código TSE** do cargo (`'1'` = presidente), não o nome. O nome
> do candidato aparece em `candidate_name` (Lula = `'LUIZ INÁCIO LULA DA SILVA'`,
> número `'13'`). Valores de `prioridade_recuperacao`: `CRÍTICA`, `ALTA`, `MÉDIA`, `BAIXA`.

Regras de leitura: `NULL` é indisponível (não é zero); **não** some denominadores
entre candidatos nem entre cargos; para participação de cargo sem candidatos use
`mart_section_turnout` separadamente; trate `UNMATCHED`/`UNKNOWN` como ausência de
comparação, nunca como zero.

## Versões e integridade

- Cada snapshot é **imutável**. Uma atualização gera uma **nova versão**
  (`snapshots/e6-2022-2026-v2`, ...); nunca sobrescreva uma versão publicada.
- `manifest.json` traz `data_version`, `published_at`, `built_at`, `software_version`,
  `sources` (com `sha256` e `reference_url`) e `files` (hash, bytes e contagem por
  arquivo). `manifest.sha256` é o hash do manifesto.
- `schema.json` descreve nomes, tipos DuckDB e nulabilidade de cada coluna.
- Confira `sha256` dos Parquets contra o manifesto antes de usar. Hash detecta
  alteração, mas não autentica a origem.

## Licença e atribuição

- **Software** deste projeto: MIT (`LICENSE`), sujeito à confirmação do responsável.
- **Dados**: **não** são relicenciados por este repositório. Cada fonte mantém suas
  condições:
  - **TSE** — Resultados (dados abertos): `https://dadosabertos.tse.jus.br/dataset/resultados-2022`
    e `.../resultados-2026`.
  - **IBGE** — Censo 2022, agregados por setores (`ftp.ibge.gov.br`).
  - **DATASUS** — SIM (Mortalidade Geral 2022) e CNES (UBS).
- Mantenha a **atribuição** às fontes em qualquer redistribuição ou análise publicada.

## Como citar

```text
Observatório Popular de Dados Eleitorais. Base pública de dados eleitorais
(snapshots e camadas derivadas). Versão: fe3f416614dfa18e1a3e5dda9292dbbca66d5aa1377020934cb6bf332044140d.
Disponível em: <URL do repositório>. Acesso em: <data>.
```

> **Nota:** os valores acima descrevem a base importada e as camadas derivadas.
> Não constituem reconciliação oficial nem certificação do resultado eleitoral.

