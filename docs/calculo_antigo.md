# Cálculo antigo de "votos recuperáveis" (mapeamento — E9, Etapa 1)

Documento de **auditoria** do cálculo que está sendo substituído pela metodologia
de baseline 2022. Nada foi removido: `src/eleitoral/priority.py`,
`snapshots/secoes_prioritarias.parquet` e o contrato E7 continuam no repositório.
Este texto só **mapeia** o que existia e por que é insuficiente.

## Onde o cálculo antigo vive

| Camada | Arquivo | Papel |
| --- | --- | --- |
| Cálculo | `src/eleitoral/priority.py` (`build_priority`, `PRIORITY_BASE_SQL`, `_SCORE`) | Gera o Parquet derivado a partir do warehouse DuckDB |
| CLI | `src/eleitoral/cli.py` (`build-priority`, `priority-summary`) | `uv run eleitoral build-priority` |
| Cache/API | `src/eleitoral/api/priority.py` (`Priority`) | Carrega o Parquet em memória e serve consultas |
| Endpoints | `src/eleitoral/api/app.py` (`/api/v1/priority/*`) | Expõe `votos_recuperaveis`, `score_prioridade`, `prioridade` |
| Frontend | `PriorityActions.tsx`, `Explore.tsx`, `Section.tsx`, `Home.tsx`, `SectionsMap.tsx` | Cards, listas e mapa |
| Contrato | `docs/entregas/E7.md` | Documentação original da camada |

Artefato produzido: **`snapshots/secoes_prioritarias.parquet`** (28 colunas; ver
`src/eleitoral/api/priority.py::COLUMNS`). Recorte fixo: Lula (`candidate_number='13'`),
presidente (`office='1'`), 1º turno (`round=1`), anos 2022 e 2026, `dataset='SOURCE'`.

## Fórmulas antigas

Duas métricas eram publicadas por seção:

```text
votos_recuperaveis = abstencoes_2026 + nulos_2026 + brancos_2026        -- absoluto, sem baseline

score_prioridade   = min(100, max(0,
                       base        = lula_2022 / aptos_2022 * 100 * 0,4
                     + delta       = (abstencoes_2026 - abstencoes_2022) / aptos_2022 * 100 * 2,0
                     + recuperavel = (abstencoes_2026 + nulos_2026 + brancos_2026) / aptos_2026 * 100 * 0,3
                     ))
```

- **Pesos** (`WEIGHTS`): `base_lula_2022 = 0,4`, `delta_abstencao = 2,0`, `recuperaveis = 0,3`.
- **Limiares** (`THRESHOLDS`): referência fixa 70/40, mas a produção **calibra por
  quantis** entre seções comparáveis — `ALTA = quantil 0,90`, `MÉDIA = quantil 0,60`
  (`calibrate()`), para respeitar a sanidade 10/30/60.
- **`status_comparabilidade`**: `comparável` / `nova` (ausente em 2022) / `extinta`
  (ausente em 2026). Novas e extintas ficam fora do ranking por padrão.
- `delta_abstention_pp` é a variação de abstenção **sobre os aptos de 2022** (pode
  passar de 100).

## Problemas do cálculo antigo (motivação da refatoração)

1. **`votos_recuperaveis` não tem baseline.** Soma abstenção + brancos + nulos de 2026
   em qualquer seção, inclusive nas onde Lula nunca foi competitivo. Não mede
   "quanto falta em relação ao que Lula já teve".
2. **Não distingue abstenção de migração de voto.** Todo o volume inválido é tratado
   como recuperável, mesmo quando o voto simplesmente foi para outro candidato.
3. **Score com pesos arbitrários e saturado.** Os pesos 0,4/2,0/0,3 são uma escolha
   discricionária; o `min(100, …)` satura em dezenas de milhares de seções, o que
   obriga a usar `votos_recuperaveis` como desempate (README) e enfraquece o score.
4. **Escala não comparável entre métricas.** `base` e `recuperavel` são percentuais
   sobre bases diferentes (aptos de anos distintos), somados como se fossem a mesma
   unidade.
5. **Ordenação por volume absoluto** favorece seções grandes, não onde há maior
   **gap** relativo ao piso de 2022.

## O que a nova metodologia muda

A votação de Lula em 2022 passa a ser o **piso relativo por seção**
(`share_lula_2022 = lula_2022 / validos_2022`). O gap mede o quanto o desempenho real
de 2026 ficou abaixo desse piso projetado sobre os válidos de 2026. Ver
`relatorio_refatoracao.md` para o comparativo completo e
`src/eleitoral/recuperaveis.py` para a implementação.

O artefato antigo **não é apagado**: permanece como referência e insumo do
cruzamento socioeleitoral (E8, `src/eleitoral/crosswalk.py`).
