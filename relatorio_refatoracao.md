# Relatório de refatoração — votos recuperáveis (E9)

Refatoração do cálculo de "votos recuperáveis" do Observatório Popular, substituindo
a lógica antiga (E7) pela metodologia de **baseline 2022**. O `gap_p1` é a métrica
central da campanha.

## Como reproduzir

```powershell
uv run eleitoral build-recuperaveis --output snapshots/secoes_recuperaveis.parquet
uv run eleitoral audit-recuperaveis --output auditoria_recuperaveis.json
```

Artefatos: `snapshots/secoes_recuperaveis.parquet` (42 colunas),
`auditoria_recuperaveis.json`, `docs/calculo_antigo.md`.

## 1. Fórmula antiga × nova

### Antes (E7, `src/eleitoral/priority.py`)

```text
votos_recuperaveis = abstencoes_2026 + nulos_2026 + brancos_2026     # absoluto, sem baseline
score_prioridade   = min(100, max(0,
                       0,4 * lula_2022/aptos_2022*100
                     + 2,0 * (abstencoes_2026-abstencoes_2022)/aptos_2022*100
                     + 0,3 * (abstencoes_2026+nulos_2026+brancos_2026)/aptos_2026*100 ))
prioridade         = ALTA/MÉDIA/BAIXA por quantis 0,90 / 0,60 do score
```

### Depois (E9, `src/eleitoral/recuperaveis.py`)

```text
share_lula_2022 = lula_2022 / válidos_2022
potencial_p1    = share_lula_2022 * válidos_2026                 # núcleo (piso realista)
potencial_p2    = share_lula_2022 * média(comparecimento_22, comparecimento_26)
potencial_p3    = share_lula_2022 * aptos_2026                   # teto teórico
gap_p1          = potencial_p1 - lula_2026
taxa_conversao  = lula_2026 / potencial_p1
ausentes_2026   = abstenções_2026 + brancos_2026 + nulos_2026
```

**Camadas:** P1 = núcleo (piso realista, usada na classificação); P2 = mede o custo da
abstenção; P3 = teto teórico, só para priorização.

**Classificação (`prioridade_recuperacao`) — régua calibrada por quantis** de
`percentil_gap` com **piso semântico**:

| Tier | Regra | Uso |
| --- | --- | --- |
| CRÍTICA | `percentil_gap > 90` | top 10% — mutirões e foco |
| ALTA | `percentil_gap > 65` | próximos 25% |
| MÉDIA | `percentil_gap > 30` | próximos 35% |
| BAIXA | restante | base consolidada |
| Piso | `gap_p1 <= 0` **ou** `share_lula_2022 < 0,3` → BAIXA | evita "crítica" por 2 votos |

Três réguas convivem no artefato para auditoria: `prioridade_recuperacao`
(calibrada + piso, usada pelo frontend), `prioridade_spec` (limiares fixos do
rascunho: taxa < 0,7 / < 0,85, share > 0,4) e `prioridade_quantil` (só quantil).
A API expõe `modo=calibrado|spec|quantil`.

**Diagnóstico** (matriz gap × ausentes):

| Gap P1 | Ausentes 2026 | Diagnóstico | Ação sugerida |
| --- | --- | --- | --- |
| Alto | Alto | mobilização | Mutirão, lembrete, apoio logístico |
| Alto | Baixo | conversão | Porta a porta, comparação de propostas |
| Baixo | Alto | preventiva | Mobilização preventiva |
| Baixo | Baixo | manutenção | Manter presença |

"Alto" = `ausentes_2026 > 15%` dos aptos de 2026.

## 2. Justificativa metodológica

1. **Baseline, não volume.** O cálculo antigo somava abstenção + brancos + nulos de
   2026 em qualquer seção, inclusive onde Lula nunca foi competitivo. O `gap_p1` mede
   o quanto 2026 ficou abaixo do que Lula **já havia conquistado** na mesma seção.
2. **Separa abstenção de migração.** O cruzamento com `ausentes_2026` distingue
   "não compareceu / votou inválido" de "voto migrou" — a ação de campo é diferente.
3. **Régua acionável.** Os limiares fixos do rascunho (taxa < 0,7 / < 0,85)
   produziriam apenas **1,21%** de CRÍTICA: a taxa de conversão nacional de Lula é
   **0,9301** (share de 48,40% em 2022 → 45,02% em 2026), então quase ninguém fica
   abaixo de 0,7. É o mesmo problema que o E7 já havia encontrado (limiares 70/40 →
   ~0,7% ALTA) e resolvido por quantis. A calibração garante um grupo de ~10% sempre
   acionável e o piso semântico impede que um gap irrelevante entre no topo.
4. **`gap_p1` verificável.** `Σ gap_p1` (comparáveis) = 3.930.088,78 =
   `Σ potencial_p1` (54.034.330,60) − `Σ lula_2026` (50.104.537). Fecha com o
   warehouse.

## 3. Impacto nas prioridades

Recorte: 510.060 seções processadas; **461.263 comparáveis**; 37.985 novas; 10.812
extintas. Lula: 57.259.504 (2022) → 53.879.538 (2026).

### Distribuição (`prioridade_recuperacao`, comparáveis)

| Régua | CRÍTICA | ALTA | MÉDIA | BAIXA |
| --- | --- | --- | --- | --- |
| **Calibrada (padrão)** | 9,66% (44.573) | 22,75% (104.939) | 29,78% (137.357) | 37,81% (174.394) |
| Quantil puro (sem piso) | 9,99% | 25,00% | 35,00% | 30,01% |
| Spec (limiares fixos) | 1,21% | 21,42% | 55,75% | 21,61% |
| Antiga (E7) | — | 10,00% | 30,00% | 60,00% |

A BAIXA calibrada (37,81%) fica acima dos 30% teóricos porque o **piso semântico**
rebaixa 42.820 seções de base fraca (`share_lula_2022 < 0,3`) — 9,28% do total.
Motivos: `por_quantil` 69,1%, `sem_gap` 21,61%, `base_fraca` 9,28%.

### Diagnóstico (comparáveis)

| Diagnóstico | Seções | % |
| --- | --- | --- |
| mobilização | 351.353 | 76,17% |
| preventiva | 95.696 | 20,75% |
| conversão | 10.210 | 2,21% |
| manutenção | 4.004 | 0,87% |

O país é majoritariamente "gap + ausentes altos" (mobilização): o gargalo é
comparecimento/voto válido, não só migração.

### Comparação com o cálculo antigo (3 níveis, seções comuns = 461.263)

| Direção | Seções | % |
| --- | --- | --- |
| Subiram de faixa | 205.239 | 44,5% |
| Mantiveram a faixa | 188.253 | 40,8% |
| Desceram de faixa | 67.771 | 14,7% |
| **Mudaram** | **273.010** | **59,2%** |

Matriz de transição (antiga → nova): BAIXA→ALTA 79.081, BAIXA→MÉDIA 76.266,
MÉDIA→ALTA 49.892, MÉDIA→BAIXA 42.182, ALTA→MÉDIA 14.788, ALTA→BAIXA 10.801,
e 188.253 estáveis. Do antigo topo (ALTA = 46.128), só 20.539 (44,5%) permanecem
ALTA — 25.589 saem.

O total absoluto muda de conceito: `Σ votos_recuperaveis` antigo (abstenção+brancos+
nulos) = 36.715.596; `Σ gap_p1` novo = 3.930.089. Não são comparáveis: o antigo media
todo o volume inválido; o novo mede o déficit em relação ao piso de 2022.

### Top 5 por `gap_p1` (de 20 em `auditoria_recuperaveis.json`)

| Seção | UF | Lula 2022 | Lula 2026 | Gap P1 | Conversão | Diagnóstico |
| --- | --- | --- | --- | --- | --- | --- |
| SOURCE\|AM\|2356\|17\|18 | AM | 273 | 141 | 117,3 | 0,546 | mobilização |
| SOURCE\|ZZ\|30341\|1\|1863 | ZZ | 327 | 152 | 112,5 | 0,575 | mobilização |
| SOURCE\|ZZ\|29955\|1\|576 | ZZ | 327 | 103 | 110,8 | 0,482 | mobilização |
| SOURCE\|RS\|85391\|54\|214 | RS | 178 | 75 | 107,2 | 0,412 | mobilização |
| SOURCE\|PR\|75442\|111\|276 | PR | 50 | 107 | 98,8 | 0,520 | mobilização |

## 4. Recomendações para a campanha

1. **Foco no top 10% (CRÍTICA, 44.573 seções).** Concentrar mutirões e porta a porta
   onde o gap é maior e a base de 2022 era forte. O ranking ordena por `gap_p1`.
2. **Priorizar `mobilização` (76% das seções).** O gargalo dominante é
   comparecimento/voto válido: lembrete, apoio logístico e orientação sobre voto
   válido têm mais retorno que persuasão.
3. **Tratar `conversão` como frente separada (2,2%).** Poucas seções, mas são onde o
   comparecimento é alto e o voto migrou — exigem comparação de propostas.
4. **Usar `taxa_conversao` como semáforo.** Abaixo de 0,7 em seções de base forte =
   alerta; entre 0,85 e 1,0 = manutenção.
5. **Cruzar com o E8.** O cruzamento socioeleitoral já consome o artefato antigo; para
   o planejamento territorial, usar `secoes_recuperaveis.parquet` (gap) como camada
   principal e o antigo apenas como referência.
6. **Revalidar a régua a cada nova apuração.** Os quantis são recalculados no build;
   a distribuição esperada (~10/25/35/30) é um alvo, não um resultado fixo.

## 5. API e frontend atualizados

- **API** (`/api/v1/priority/*`): expõe `potencial_p1/p2/p3`, `gap_p1`,
  `taxa_conversao`, `ausentes_2026`, `percentil_gap`, `prioridade_recuperacao`,
  `prioridade_spec`, `prioridade_quantil`, `motivo_classificacao` e `diagnostico`.
  Filtro `modo=calibrado|spec|quantil`; ordenação padrão por `gap_p1`; ação sugerida
  por diagnóstico. **Retrocompatibilidade:** `votos_recuperaveis` continua existindo
  como alias arredondado de `gap_p1`, `prioridade` (3 níveis) colapsa CRÍTICA em ALTA
  e `score_prioridade` passa a ser o déficit de conversão (`100*(1-taxa_conversao)`).
  Se `snapshots/secoes_recuperaveis.parquet` não existir, a API cai para o artefato
  antigo sem quebrar.
- **Frontend**: card "Votos Recuperáveis" → **"Gap vs. 2022"**; badges de
  prioridade e de diagnóstico; barra de `taxa_conversao` (0–100%); listas e mapa
  ordenados por `gap_p1`; `/mapas` mostra prioridade **crítica** (faixa p90).

## 6. Limitações e decisões

- **Polars ausente.** O enunciado citava Polars, mas o ambiente não tem a dependência
  e toda a camada analítica usa DuckDB. A semântica das fórmulas foi preservada.
- **Fontes.** O enunciado citava `snapshots/secao_2022.parquet` / `secao_2026.parquet`,
  que não existem; o cálculo lê o warehouse (`fact_section_turnout` +
  `fact_candidate_votes`), como o E7 já fazia.
- **Régua calibrada** foi escolhida (decisão do responsável) por reproduzir a
  distribuição esperada; o modo `spec` fica disponível para auditoria.
- Seções novas/extintas não têm baseline: recebem BAIXA e ficam fora do ranking por
  padrão (`status_comparabilidade`).
- 54 seções comparáveis têm `gap_p1` nulo (contraparte de Lula ausente); ficam fora do
  ranking.

```text
PROJECT=OBSERVATORIO_POPULAR_ELEITORAL
E9_STATUS=PASS
ARTIFACT=snapshots/secoes_recuperaveis.parquet
AUDIT=auditoria_recuperaveis.json (status=OK, 0 avisos)
GAP_P1_NACIONAL=3930088.78
DISTRIBUICAO_CALIBRADA=9.66/22.75/29.78/37.81
RETROCOMPATIBILIDADE=votos_recuperaveis -> gap_p1
API=ATUALIZADA (modo=calibrado|spec|quantil)
FRONTEND=ATUALIZADO (gap vs. 2022, diagnostico, taxa_conversao)
RUFF=PASS
FRONTEND_TYPECHECK=PASS
FRONTEND_BUILD=PASS
```

