# Como agir — proposta, justificativa e mensagem por seção eleitoral (E10)

Gerado por `eleitoral build-como-agir` em 2026-10-11. Camada **derivada e descritiva**: não infere intenção individual nem estima conversão de voto.

## 1. O que o bloco entrega

Para cada seção, o militante vê **o que falar** e **por que falar isso**:

1. **Ponto de campanha** — proposta confirmada pelos indicadores do território;
2. **Justificativa** — os indicadores que confirmam a proposta, com valor e referência;
3. **Ação territorial** — canal da proposta somado ao tom do diagnóstico de recuperação;
4. **Mensagem pronta** — frase curta com o dado do território.

## 2. Entradas

| Fonte | Uso |
| --- | --- |
| `snapshots/secoes_demanda.parquet` (E8) | proposta dominante, 10 scores e indicadores |
| `snapshots/secoes_recuperaveis.parquet` (E9) | `diagnostico`, `gap_p1`, `taxa_conversao`, `ausentes_2026`, `prioridade_recuperacao` |
| `guia_porta_a_porta_por_secao.csv` (E8) | conferência: proposta do guia × proposta confirmada |

Chave de cruzamento: `uf + municipality + zone + section` (`SG_UF`, `CD_MUNICIPIO`, `NR_ZONA`, `NR_SECAO` no desenho original; o identificador canônico é `section_id`). uma linha por section_id; 2917 repetições idênticas herdadas do E8 foram resolvidas na leitura.

## 3. Como a decisão é tomada

1. **Proposta dominante** — vem pronta do E8 (`proposta_dominante`, argmax dos 10 scores). O bloco **não** recalcula o argmax.
2. **Confirmação por indicadores** — cada proposta tem verificações com peso. Uma verificação confirma quando o indicador cruza a referência nacional na direção da demanda. A proposta só é confirmada se o indicador de **maior peso** confirmar e o total de confirmações atingir `min(2, nº de indicadores da proposta)`. Quando a proposta tem um único indicador, o mínimo cai para um — declarado nesta régua.
   - régua: confirmação por indicador acima do quantil 0.65 (demanda alta) ou abaixo do quantil 0.35 (demanda baixa), na direção do indicador; peso forte >= 0.4
   - se a dominante não confirma, vale a **proposta confirmada de maior score**;
   - se nenhuma proposta confirmar, a dominante é mantida e marcada como `sem_confirmacao`.
3. **Tom pelo diagnóstico (E9)** — `mobilização` lembra e apoia logística; `conversão` compara propostas e escuta; `preventiva` mantém presença e vínculo; `manutenção` consolida e agradece.
4. **Ação territorial** — `canal da proposta` + `tom do diagnóstico`.
5. **Mensagem pronta** — `impacto + "Aqui, " + dado do território + consequência`. São citados no máximo 2 indicadores por mensagem, para manter a frase curta.

## 4. Propostas e indicadores de confirmação

| Proposta | Indicadores (peso) | Canal territorial |
| --- | --- | --- |
| Fim da escala 6x1 / jornada de 40h | pct_jovens (0.7), renda_media (0.3) | Porta a porta com jovens trabalhadores |
| Salário mínimo acima da inflação | renda_media (1.0) | Panfletagem em feiras e terminais |
| Cesta básica sem imposto | renda_media (0.6), pct_criancas (0.4) | Mutirão em bairros periféricos |
| Pé-de-Meia universal | analfabetismo (0.4), pct_jovens (0.3), renda_media (0.3) | Roda com estudantes e portas de escola |
| Ensino de tempo integral | analfabetismo (0.5), pct_criancas (0.5) | Reunião com famílias e lideranças |
| Farmácia Popular ampliada | pct_idosos (0.6), mortalidade_geral (0.4) | Mutirão em UBS e filas de farmácia |
| Rede Brasil contra o Câncer | mortalidade_cancer (1.0) | Roda de saúde e agentes comunitários |
| Teleconsultas no SUS | ubs_por_10k (0.6), mortalidade_geral (0.4) | Porta a porta com apoio de ACS |
| Ministério da Segurança | homicidios (1.0) | Caminhada de rua e lideranças locais |
| Pacto Contra o Feminicídio | homicidios (0.6), pct_idosos (0.4) | Roda com mulheres e coletivos |

## 5. Referências nacionais medidas (quantis)

| Indicador | Direção da demanda | p65 (demanda alta) | p35 (demanda baixa) |
| --- | --- | ---: | ---: |
| pct_jovens — % da população de 15 a 29 anos | alto | 22.79 | 21.22 |
| pct_idosos — % da população com 60 anos ou mais | alto | 17.60 | 14.50 |
| pct_criancas — % da população de 0 a 14 anos | alto | 20.23 | 18.06 |
| renda_media — renda média do responsável | baixo | 3093.28 | 2203.06 |
| analfabetismo — taxa de analfabetismo (15 anos ou mais) | alto | 27.09 | 16.07 |
| mortalidade_geral — mortalidade geral por mil habitantes | alto | 8.09 | 7.01 |
| mortalidade_cancer — mortalidade por câncer por 100 mil habitantes | alto | 135.96 | 104.19 |
| homicidios — homicídios por 100 mil habitantes | alto | 24.08 | 11.33 |
| ubs_por_10k — UBS por 10 mil habitantes | baixo | 2.90 | 1.24 |

## 6. Proxies declarados

Indicadores pedidos no desenho original que **não existem** neste recorte, com o proxy efetivamente usado (mesma declaração do E8):

| Indicador pedido | Proxy usado | Nota |
| --- | --- | --- |
| ideb_municipal | analfabetismo | INEP/IDEB não carregado; alfabetização do Censo como proxy |
| evasao_em | analfabetismo | INEP/evasão não carregada; alfabetização do Censo como proxy |
| pct_criancas_tempo_parcial | pct_criancas | Censo não distingue turno; 0 a 14 anos como proxy |
| pct_emprego_formal | renda_media | CAGED/RAIS não carregado; renda do responsável como proxy |
| pct_renda_ate_2sm | renda_media | faixa de renda não publicada; renda média como proxy |
| pct_inseguranca_alimentar | renda_media | sem fonte local; renda média como proxy |
| icsap | mortalidade_geral | SIH/ICSAP não carregado; mortalidade geral como proxy |
| densidade_medicos | ubs_por_10k | CNES conta UBS, não médicos; UBS por 10 mil como proxy |
| taxa_feminicidio | homicidios | SIM sem causa de feminicídio; homicídios como proxy (E8) |
| pct_mulheres_chefes | pct_idosos | Censo não agregado por sexo do responsável neste recorte |

## 7. Resultados medidos (recorte atual)

- Seções processadas: **508618**.
- Proposta dominante **confirmada**: **374000** (73.5%).
- Dominante não confirmada, com **alternativa confirmada**: **20023** (3.9%).
- **Sem confirmação** por indicador (dominante mantida com aviso): **114595** (22.5%).
- **Fallback pela UF** (seção sem indicador socioeconômico): **0**.
- Confirmadas por **2 ou mais indicadores**: **147676**.
- Proposta **alterada** em relação ao E8: **20023**.

## 8. Distribuição das propostas finais

| Proposta | Seções |
| --- | ---: |
| Ministério da Segurança | 103170 |
| Rede Brasil contra o Câncer | 98065 |
| Teleconsultas no SUS | 82727 |
| Salário mínimo acima da inflação | 62230 |
| Farmácia Popular ampliada | 57102 |
| Ensino de tempo integral | 51664 |
| Fim da escala 6x1 / jornada de 40h | 36373 |
| Cesta básica sem imposto | 6986 |
| Pé-de-Meia universal | 6830 |
| Pacto Contra o Feminicídio | 3471 |

## 9. Distribuição das ações por diagnóstico

| Diagnóstico | Ação (canal) | Seções |
| --- | --- | ---: |
| conversão | Caminhada de rua e lideranças locais | 2482 |
| conversão | Panfletagem em feiras e terminais | 2111 |
| conversão | Roda de saúde e agentes comunitários | 1971 |
| conversão | Porta a porta com jovens trabalhadores | 1247 |
| conversão | Reunião com famílias e lideranças | 945 |
| conversão | Mutirão em UBS e filas de farmácia | 801 |
| conversão | Porta a porta com apoio de ACS | 222 |
| conversão | Roda com estudantes e portas de escola | 183 |
| conversão | Mutirão em bairros periféricos | 172 |
| conversão | Roda com mulheres e coletivos | 76 |
| manutenção | Caminhada de rua e lideranças locais | 12658 |
| manutenção | Porta a porta com apoio de ACS | 9346 |
| manutenção | Reunião com famílias e lideranças | 7177 |
| manutenção | Panfletagem em feiras e terminais | 6496 |
| manutenção | Roda de saúde e agentes comunitários | 6170 |
| manutenção | Porta a porta com jovens trabalhadores | 4597 |
| manutenção | Mutirão em UBS e filas de farmácia | 4191 |
| manutenção | Mutirão em bairros periféricos | 815 |
| manutenção | Roda com estudantes e portas de escola | 599 |
| manutenção | Roda com mulheres e coletivos | 236 |
| mobilização | Roda de saúde e agentes comunitários | 74695 |
| mobilização | Caminhada de rua e lideranças locais | 64011 |
| mobilização | Porta a porta com apoio de ACS | 45494 |
| mobilização | Panfletagem em feiras e terminais | 45436 |
| mobilização | Mutirão em UBS e filas de farmácia | 43756 |
| mobilização | Reunião com famílias e lideranças | 37994 |
| mobilização | Porta a porta com jovens trabalhadores | 26506 |
| mobilização | Mutirão em bairros periféricos | 5319 |
| mobilização | Roda com estudantes e portas de escola | 5183 |
| mobilização | Roda com mulheres e coletivos | 2602 |
| preventiva | Porta a porta com apoio de ACS | 27665 |
| preventiva | Caminhada de rua e lideranças locais | 24019 |
| preventiva | Roda de saúde e agentes comunitários | 15229 |
| preventiva | Mutirão em UBS e filas de farmácia | 8354 |
| preventiva | Panfletagem em feiras e terminais | 8187 |
| preventiva | Reunião com famílias e lideranças | 5548 |
| preventiva | Porta a porta com jovens trabalhadores | 4023 |
| preventiva | Roda com estudantes e portas de escola | 865 |
| preventiva | Mutirão em bairros periféricos | 680 |
| preventiva | Roda com mulheres e coletivos | 557 |

## 10. Proposta mais frequente por UF

| UF | Proposta | Seções na UF |
| --- | --- | ---: |
| AC | Ensino de tempo integral | 2315 |
| AL | Ministério da Segurança | 7200 |
| AM | Ministério da Segurança | 8263 |
| AP | Ministério da Segurança | 2338 |
| BA | Ministério da Segurança | 36325 |
| CE | Ministério da Segurança | 24253 |
| DF | Teleconsultas no SUS | 7049 |
| ES | Ministério da Segurança | 9903 |
| GO | Ensino de tempo integral | 15919 |
| MA | Ensino de tempo integral | 18836 |
| MG | Rede Brasil contra o Câncer | 52756 |
| MS | Ensino de tempo integral | 7210 |
| MT | Ensino de tempo integral | 8536 |
| PA | Ministério da Segurança | 21470 |
| PB | Salário mínimo acima da inflação | 10737 |
| PE | Ministério da Segurança | 21586 |
| PI | Salário mínimo acima da inflação | 10404 |
| PR | Rede Brasil contra o Câncer | 27483 |
| RJ | Teleconsultas no SUS | 37855 |
| RN | Ministério da Segurança | 8224 |
| RO | Ministério da Segurança | 4787 |
| RR | Ensino de tempo integral | 1605 |
| RS | Rede Brasil contra o Câncer | 27793 |
| SC | Rede Brasil contra o Câncer | 17739 |
| SE | Ministério da Segurança | 5984 |
| SP | Teleconsultas no SUS | 107653 |
| TO | Ensino de tempo integral | 4395 |

Distribuição completa por UF e proposta: `auditoria_como_agir.json`.


## 11. Cruzamento com o guia porta a porta (E8)

- Seções do guia encontradas no bloco: **41138**.
- Proposta do guia igual à dominante do E8: **41138**.
- Proposta do bloco **alterada** em relação ao guia: **971** (2.4%).
- Proposta confirmada entre as seções do guia: **36887**.
- Sem confirmação entre as seções do guia: **3280**.

## 12. Top 20 seções por gap vs. 2022

| UF | Município | Zona | Seção | Proposta | Motivo | Indicadores | Diagnóstico | Gap | Mensagem |
| --- | --- | --- | --- | --- | --- | ---: | --- | ---: | --- |
| AM | HUMAITÁ | 17 | 18 | Ensino de tempo integral | confirmada | 2 | mobilização | 117.29 | O Lula vai implementar o ensino de tempo integral em todas as escolas públicas. Aqui, a taxa de analfabetismo é de 42,3% e 28,1% da população tem até 14 anos. Isso significa mais aprendizado e mais oportunidades para as crianças. |
| RS | BARROS CASSAL | 54 | 214 | Ministério da Segurança | confirmada | 1 | mobilização | 107.21 | O Lula vai criar o Ministério da Segurança Pública e retomar territórios dominados pelo crime. Aqui, a taxa de homicídios é de 64,5 por 100 mil habitantes. Isso é proteção para quem trabalha e prisão para quem rouba. |
| PR | IMBAÚ | 111 | 276 | Fim da escala 6x1 / jornada de 40h | confirmada | 2 | mobilização | 98.83 | O Lula vai acabar com a escala 6x1 e reduzir a jornada para 40 horas sem cortar salário. Aqui, 24,2% da população tem entre 15 e 29 anos e a renda média do responsável é de R$ 2.093. Isso é mais tempo com a família, sem perder renda. |
| RS | LAGOÃO | 53 | 173 | Ministério da Segurança | confirmada | 1 | mobilização | 90.37 | O Lula vai criar o Ministério da Segurança Pública e retomar territórios dominados pelo crime. Aqui, a taxa de homicídios é de 56,2 por 100 mil habitantes. Isso é proteção para quem trabalha e prisão para quem rouba. |
| RS | CAMARGO | 62 | 190 | Rede Brasil contra o Câncer | confirmada | 1 | mobilização | 87.95 | O Lula vai criar a Rede Brasil contra o Câncer, a maior rede pública do mundo para tratamento da doença. Aqui, a mortalidade por câncer é de 201,3 por 100 mil habitantes. Isso é atendimento perto de casa. |
| MA | CENTRO NOVO DO MARANHÃO | 100 | 28 | Fim da escala 6x1 / jornada de 40h | confirmada | 2 | mobilização | 84.93 | O Lula vai acabar com a escala 6x1 e reduzir a jornada para 40 horas sem cortar salário. Aqui, 27,9% da população tem entre 15 e 29 anos e a renda média do responsável é de R$ 1.229. Isso é mais tempo com a família, sem perder renda. |
| SC | IRINEÓPOLIS | 25 | 148 | Ministério da Segurança | confirmada | 1 | mobilização | 83.50 | O Lula vai criar o Ministério da Segurança Pública e retomar territórios dominados pelo crime. Aqui, a taxa de homicídios é de 29,2 por 100 mil habitantes. Isso é proteção para quem trabalha e prisão para quem rouba. |
| MG | PERDÕES | 216 | 61 | Farmácia Popular ampliada | confirmada | 2 | mobilização | 82.89 | O Lula vai ampliar o Farmácia Popular e criar a Rede Brasil contra o Câncer. Aqui, 20,5% da população tem 60 anos ou mais e a mortalidade geral é de 8,6 por mil habitantes. Isso é remédio mais barato e atendimento perto de casa. |
| BA | VITÓRIA DA CONQUISTA | 41 | 310 | Pé-de-Meia universal | alternativa_confirmada | 2 | mobilização | 81.67 | O Lula vai universalizar o Pé-de-Meia para todos os alunos do ensino médio público. Aqui, a taxa de analfabetismo é de 27,9% e 23,9% da população tem entre 15 e 29 anos. Isso significa dinheiro no bolso da família e filho estudando. |
| GO | GUARANI DE GOIÁS | 47 | 78 | Ministério da Segurança | confirmada | 1 | mobilização | 79.50 | O Lula vai criar o Ministério da Segurança Pública e retomar territórios dominados pelo crime. Aqui, a taxa de homicídios é de 49,0 por 100 mil habitantes. Isso é proteção para quem trabalha e prisão para quem rouba. |
| PR | SÃO JOÃO DO TRIUNFO | 52 | 23 | Rede Brasil contra o Câncer | confirmada | 1 | mobilização | 78.29 | O Lula vai criar a Rede Brasil contra o Câncer, a maior rede pública do mundo para tratamento da doença. Aqui, a mortalidade por câncer é de 138,4 por 100 mil habitantes. Isso é atendimento perto de casa. |
| RS | HERVEIRAS | 40 | 349 | Farmácia Popular ampliada | confirmada | 2 | mobilização | 77.53 | O Lula vai ampliar o Farmácia Popular e criar a Rede Brasil contra o Câncer. Aqui, 19,6% da população tem 60 anos ou mais e a mortalidade geral é de 9,0 por mil habitantes. Isso é remédio mais barato e atendimento perto de casa. |
| PR | RESERVA DO IGUAÇU | 160 | 133 | Rede Brasil contra o Câncer | confirmada | 1 | mobilização | 77.37 | O Lula vai criar a Rede Brasil contra o Câncer, a maior rede pública do mundo para tratamento da doença. Aqui, a mortalidade por câncer é de 167,9 por 100 mil habitantes. Isso é atendimento perto de casa. |
| AM | HUMAITÁ | 17 | 67 | Ensino de tempo integral | confirmada | 2 | mobilização | 77.05 | O Lula vai implementar o ensino de tempo integral em todas as escolas públicas. Aqui, a taxa de analfabetismo é de 42,3% e 28,1% da população tem até 14 anos. Isso significa mais aprendizado e mais oportunidades para as crianças. |
| PA | TUCUMÃ | 74 | 183 | Ministério da Segurança | confirmada | 1 | mobilização | 76.64 | O Lula vai criar o Ministério da Segurança Pública e retomar territórios dominados pelo crime. Aqui, a taxa de homicídios é de 68,3 por 100 mil habitantes. Isso é proteção para quem trabalha e prisão para quem rouba. |
| BA | TREMEDAL | 177 | 64 | Salário mínimo acima da inflação | confirmada | 1 | mobilização | 76.47 | O Lula vai continuar valorizando o salário mínimo acima da inflação. Aqui, a renda média do responsável é de R$ 1.165. Isso é dinheiro no bolso todo mês. |
| AM | HUMAITÁ | 17 | 89 | Ensino de tempo integral | confirmada | 2 | mobilização | 76.43 | O Lula vai implementar o ensino de tempo integral em todas as escolas públicas. Aqui, a taxa de analfabetismo é de 42,3% e 28,1% da população tem até 14 anos. Isso significa mais aprendizado e mais oportunidades para as crianças. |
| RS | BOQUEIRÃO DO LEÃO | 93 | 173 | Rede Brasil contra o Câncer | confirmada | 1 | mobilização | 75.99 | O Lula vai criar a Rede Brasil contra o Câncer, a maior rede pública do mundo para tratamento da doença. Aqui, a mortalidade por câncer é de 256,1 por 100 mil habitantes. Isso é atendimento perto de casa. |
| GO | GUARANI DE GOIÁS | 47 | 71 | Ministério da Segurança | confirmada | 1 | mobilização | 75.50 | O Lula vai criar o Ministério da Segurança Pública e retomar territórios dominados pelo crime. Aqui, a taxa de homicídios é de 49,0 por 100 mil habitantes. Isso é proteção para quem trabalha e prisão para quem rouba. |
| PR | SÃO JOÃO DO TRIUNFO | 52 | 37 | Rede Brasil contra o Câncer | confirmada | 1 | mobilização | 75.14 | O Lula vai criar a Rede Brasil contra o Câncer, a maior rede pública do mundo para tratamento da doença. Aqui, a mortalidade por câncer é de 138,4 por 100 mil habitantes. Isso é atendimento perto de casa. |

## 13. Limitações metodológicas

- **Indicadores municipais, não de seção**: renda, idade, analfabetismo, mortalidade e cobertura de UBS são medidos no município e repetidos em todas as seções dele; a variação intraurbana não entra na confirmação.
- **Uma linha por seção**: o E8 publica `section_id` repetidos (linhas idênticas); o bloco resolve as repetições na leitura para manter a chave canônica e o endpoint devolver um único bloco por seção.
- **Proxies declarados**: IDEB/evasão, emprego formal, insegurança alimentar, ICSAP, densidade de médicos, feminicídio e chefia feminina não têm fonte carregada neste recorte (seção 6).
- **Referências quantílicas nacionais**: a confirmação usa p65/p35 do próprio recorte, não padrões absolutos. Se o recorte mudar, as referências mudam — e o JSON de auditoria registra os valores usados.
- **Pesos fixos e versionados**: os pesos das verificações são os mesmos do E8; mudá-los exige nova versão.
- **Frase pronta é sugestão**: a mensagem combina a frase de impacto do eixo nacional com o dado do território. Não substitui a escuta; o guia do app pede pergunta antes de resposta.
- **Sem confirmação é resposta legítima**: em territórios ricos, jovens e bem atendidos, nenhuma proposta cruza as referências — nesses casos o bloco mantém a dominante e diz explicitamente que não houve confirmação.
- **Fallback reduz variância**: onde falta indicador municipal, vale a média/modal da UF, declarada no campo `aviso_metodologico`.

## 14. Uso operacional

```powershell
uv run eleitoral build-como-agir
```

API: `GET /api/secao/{uf}/{municipio}/{zona}/{secao}/como-agir` e `GET /api/v1/como-agir/{uf}/{municipio}/{zona}/{secao}`. O bloco é **pré-calculado** em `snapshots/como_agir.parquet`; a requisição só lê a linha da seção.
