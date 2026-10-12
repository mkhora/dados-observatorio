# Auditoria final — Observatório Popular (E7)

Gerado por `eleitoral audit-final`. Valores lidos do warehouse local e do artefato derivado; não substituem reconciliação oficial.

## Cobertura e integridade (Etapa 1)

- UFs presentes: 2022 = 27, 2026 = 27.
- ZZ (exterior), à parte: 2022 = 1064 seções, 2026 = 1351 seções.
- Seções: 2022 = 472075, 2026 = 499248.
- Seções extintas: 10812; novas: 37985; comparáveis: 461263.
- Duplicatas de chave: 0.
- Inconsistências de soma: 0.
- Lula 2022 (warehouse): 57259504; Lula 2026 (warehouse): 53879538.
- Snapshot 2022 reconciliado com o warehouse: SIM.

## Distribuição de prioridade (seções comparáveis)

| Prioridade | Seções | % |
| --- | ---: | ---: |
| ALTA | 46128 | 10.0% |
| MÉDIA | 138377 | 30.0% |
| BAIXA | 276758 | 60.0% |

Limites calibrados: ALTA >= 35.6577 pts; MÉDIA >= 23.9721 pts (quantis 0,90 e 0,60).

## Checklist

- [x] Consistência de totais e chaves (validate).
- [x] Sanidade de scores: ALTA ≈ 10% (obtido 10.0%).
- [x] Seções novas e extintas fora do ranking por padrão.
- [x] API responde somente leitura, com rate limit e timeout por consulta.
- [ ] Performance < 500 ms: conferir por endpoint em produção.
- [ ] Acessibilidade: contraste, labels e teclado (revisão manual pendente).
- [x] LGPD: nenhum dado individual; apenas agregados por seção.

## Alertas

- Nenhum alerta.
