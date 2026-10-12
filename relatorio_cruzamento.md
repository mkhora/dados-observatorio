# Cruzamento socioeleitoral — geolocalização × bases públicas (E8)

Gerado por `eleitoral crosswalk` em 2026-10-11. Camada **derivada e descritiva**: não infere intenção individual nem estima conversão de voto.

## 1. Fontes utilizadas

- **IBGE — Censo 2022, agregados por setor censitário** (renda do responsável, demografia, características do domicílio, alfabetização). Acesso em 2026-10-11. Consolidado municipal: `warehouse/publicas/censo_municipio.parquet`.
- **DATASUS — SIM, Mortalidade Geral 2022** (causa básica CID-10 e município de residência). Acesso em 2026-10-11. Consolidado municipal: `warehouse/publicas/sim_municipal_2022.parquet`.
- **DATASUS — CNES, Unidades Básicas de Saúde** (API de dados abertos). Acesso em 2026-10-11. Consolidado municipal: `warehouse/publicas/ubs_municipal.parquet`.
- **Camada de prioridade eleitoral (E7)**: `snapshots/secoes_prioritarias.parquet`.
- **Geolocalização das seções (E7)**: `snapshots/locais_votacao_geo.parquet`.
- **SNIS**: representado pelo saneamento do Censo 2022 (mesma unidade municipal).
- **INEP (IDEB/evasão)** e **CAGED/RAIS**: sem fonte direta estável no ambiente; a educação é aproximada pela taxa de alfabetização do Censo e o emprego formal pela renda do responsável. Ver limitações.

## 2. Seções processadas e cobertura de matching

- Seções no recorte: **511535**.
- Com Censo municipal casado: **510658** (99.8%).
- Com mortalidade do SIM casada: **510658** (99.8%).
- Com cobertura de UBS casada: **507565** (99.2%).
- Seções sem indicador municipal recebem a **média da UF** como fallback declarado.

- Zonas de virada (prioridade ALTA e score da proposta dominante ≥ 0.7): **41138** seções.

## 3. Distribuição de propostas dominantes por UF

| UF | Proposta dominante | Seções |
| --- | --- | ---: |
| AC | Ensino de tempo integral | 92 |
| AC | Ministério da Segurança | 13 |
| AL | Salário mínimo acima da inflação | 178 |
| AL | Ensino de tempo integral | 132 |
| AL | Ministério da Segurança | 98 |
| AL | Cesta básica sem imposto | 33 |
| AL | Fim da escala 6x1 / jornada de 40h | 16 |
| AL | Pé-de-Meia universal | 2 |
| AL | Rede Brasil contra o Câncer | 2 |
| AM | Ensino de tempo integral | 307 |
| AM | Fim da escala 6x1 / jornada de 40h | 289 |
| AM | Ministério da Segurança | 149 |
| AM | Cesta básica sem imposto | 54 |
| AP | Ensino de tempo integral | 59 |
| AP | Fim da escala 6x1 / jornada de 40h | 22 |
| AP | Ministério da Segurança | 15 |
| BA | Ministério da Segurança | 3646 |
| BA | Salário mínimo acima da inflação | 2154 |
| BA | Fim da escala 6x1 / jornada de 40h | 259 |
| BA | Ensino de tempo integral | 240 |
| BA | Farmácia Popular ampliada | 76 |
| BA | Rede Brasil contra o Câncer | 66 |
| BA | Cesta básica sem imposto | 60 |
| BA | Pé-de-Meia universal | 24 |
| CE | Salário mínimo acima da inflação | 1697 |
| CE | Ministério da Segurança | 793 |
| CE | Fim da escala 6x1 / jornada de 40h | 371 |
| CE | Pé-de-Meia universal | 98 |
| CE | Ensino de tempo integral | 91 |
| CE | Rede Brasil contra o Câncer | 39 |
| CE | Cesta básica sem imposto | 28 |
| CE | Farmácia Popular ampliada | 25 |
| CE | Teleconsultas no SUS | 5 |
| ES | Ministério da Segurança | 321 |
| ES | Farmácia Popular ampliada | 56 |
| ES | Pacto Contra o Feminicídio | 54 |
| ES | Rede Brasil contra o Câncer | 22 |
| ES | Cesta básica sem imposto | 20 |
| ES | Salário mínimo acima da inflação | 17 |
| ES | Ensino de tempo integral | 10 |
| ES | Pé-de-Meia universal | 1 |
| GO | Fim da escala 6x1 / jornada de 40h | 192 |
| GO | Ensino de tempo integral | 186 |
| GO | Ministério da Segurança | 146 |
| GO | Farmácia Popular ampliada | 142 |
| GO | Rede Brasil contra o Câncer | 87 |
| GO | Salário mínimo acima da inflação | 35 |
| GO | Pacto Contra o Feminicídio | 16 |
| MA | Ensino de tempo integral | 1060 |
| MA | Salário mínimo acima da inflação | 731 |
| MA | Ministério da Segurança | 306 |
| MA | Fim da escala 6x1 / jornada de 40h | 295 |
| MA | Cesta básica sem imposto | 242 |
| MA | Pé-de-Meia universal | 13 |
| MG | Farmácia Popular ampliada | 1180 |
| MG | Rede Brasil contra o Câncer | 1138 |
| MG | Salário mínimo acima da inflação | 703 |
| MG | Fim da escala 6x1 / jornada de 40h | 274 |
| MG | Ensino de tempo integral | 157 |
| MG | Ministério da Segurança | 147 |
| MG | Pé-de-Meia universal | 132 |
| MG | Pacto Contra o Feminicídio | 115 |
| MG | Cesta básica sem imposto | 35 |
| MG | Teleconsultas no SUS | 21 |
| MS | Ensino de tempo integral | 482 |
| MS | Ministério da Segurança | 59 |
| MS | Rede Brasil contra o Câncer | 56 |
| MS | Farmácia Popular ampliada | 30 |
| MS | Fim da escala 6x1 / jornada de 40h | 6 |
| MS | Pacto Contra o Feminicídio | 4 |
| MT | Ensino de tempo integral | 239 |
| MT | Ministério da Segurança | 80 |
| MT | Rede Brasil contra o Câncer | 15 |
| MT | Pacto Contra o Feminicídio | 15 |
| MT | Cesta básica sem imposto | 14 |
| MT | Farmácia Popular ampliada | 12 |
| MT | Salário mínimo acima da inflação | 4 |
| PA | Fim da escala 6x1 / jornada de 40h | 604 |
| PA | Salário mínimo acima da inflação | 303 |
| PA | Ensino de tempo integral | 282 |
| PA | Ministério da Segurança | 204 |
| PA | Cesta básica sem imposto | 67 |
| PA | Pé-de-Meia universal | 17 |
| PB | Salário mínimo acima da inflação | 796 |
| PB | Ministério da Segurança | 269 |
| PB | Ensino de tempo integral | 73 |
| PB | Rede Brasil contra o Câncer | 47 |
| PB | Farmácia Popular ampliada | 35 |
| PB | Cesta básica sem imposto | 11 |
| PB | Pé-de-Meia universal | 5 |
| PE | Ministério da Segurança | 1247 |
| PE | Salário mínimo acima da inflação | 1143 |
| PE | Ensino de tempo integral | 383 |
| PE | Fim da escala 6x1 / jornada de 40h | 150 |
| PE | Cesta básica sem imposto | 62 |
| PE | Pé-de-Meia universal | 21 |
| PE | Rede Brasil contra o Câncer | 21 |
| PE | Farmácia Popular ampliada | 19 |
| PE | Teleconsultas no SUS | 2 |
| PI | Salário mínimo acima da inflação | 1355 |
| PI | Ministério da Segurança | 210 |
| PI | Fim da escala 6x1 / jornada de 40h | 105 |
| PI | Ensino de tempo integral | 100 |
| PI | Pé-de-Meia universal | 68 |
| PI | Farmácia Popular ampliada | 66 |
| PI | Rede Brasil contra o Câncer | 33 |
| PI | Cesta básica sem imposto | 32 |
| PR | Rede Brasil contra o Câncer | 482 |
| PR | Farmácia Popular ampliada | 232 |
| PR | Ministério da Segurança | 157 |
| PR | Ensino de tempo integral | 76 |
| PR | Fim da escala 6x1 / jornada de 40h | 38 |
| PR | Salário mínimo acima da inflação | 36 |
| PR | Cesta básica sem imposto | 13 |
| PR | Teleconsultas no SUS | 8 |
| PR | Pacto Contra o Feminicídio | 7 |
| RJ | Teleconsultas no SUS | 787 |
| RJ | Farmácia Popular ampliada | 323 |
| RJ | Rede Brasil contra o Câncer | 79 |
| RJ | Salário mínimo acima da inflação | 69 |
| RJ | Ministério da Segurança | 56 |
| RN | Salário mínimo acima da inflação | 471 |
| RN | Ministério da Segurança | 458 |
| RN | Ensino de tempo integral | 196 |
| RN | Fim da escala 6x1 / jornada de 40h | 131 |
| RN | Rede Brasil contra o Câncer | 102 |
| RN | Pé-de-Meia universal | 76 |
| RN | Farmácia Popular ampliada | 24 |
| RN | Cesta básica sem imposto | 19 |
| RO | Ministério da Segurança | 38 |
| RO | Ensino de tempo integral | 7 |
| RO | Fim da escala 6x1 / jornada de 40h | 5 |
| RO | Rede Brasil contra o Câncer | 1 |
| RO | Farmácia Popular ampliada | 1 |
| RR | Ensino de tempo integral | 36 |
| RR | Fim da escala 6x1 / jornada de 40h | 25 |
| RR | Ministério da Segurança | 3 |
| RR | Cesta básica sem imposto | 1 |
| RS | Rede Brasil contra o Câncer | 1853 |
| RS | Farmácia Popular ampliada | 525 |
| RS | Ministério da Segurança | 85 |
| RS | Teleconsultas no SUS | 15 |
| RS | Ensino de tempo integral | 10 |
| RS | Cesta básica sem imposto | 7 |
| RS | Pacto Contra o Feminicídio | 6 |
| RS | Fim da escala 6x1 / jornada de 40h | 2 |
| SC | Rede Brasil contra o Câncer | 328 |
| SC | Farmácia Popular ampliada | 43 |
| SC | Fim da escala 6x1 / jornada de 40h | 15 |
| SC | Ensino de tempo integral | 14 |
| SC | Ministério da Segurança | 9 |
| SC | Teleconsultas no SUS | 4 |
| SC | Cesta básica sem imposto | 2 |
| SE | Salário mínimo acima da inflação | 210 |
| SE | Ministério da Segurança | 186 |
| SE | Ensino de tempo integral | 85 |
| SE | Pé-de-Meia universal | 62 |
| SE | Cesta básica sem imposto | 9 |
| SP | Teleconsultas no SUS | 3436 |
| SP | Rede Brasil contra o Câncer | 843 |
| SP | Fim da escala 6x1 / jornada de 40h | 579 |
| SP | Farmácia Popular ampliada | 494 |
| SP | Ensino de tempo integral | 218 |
| SP | Salário mínimo acima da inflação | 15 |
| SP | Ministério da Segurança | 9 |
| SP | Cesta básica sem imposto | 7 |
| SP | Pacto Contra o Feminicídio | 1 |
| SP | Pé-de-Meia universal | 1 |
| TO | Ensino de tempo integral | 218 |
| TO | Ministério da Segurança | 80 |
| TO | Cesta básica sem imposto | 43 |
| TO | Fim da escala 6x1 / jornada de 40h | 11 |
| TO | Farmácia Popular ampliada | 9 |
| TO | Rede Brasil contra o Câncer | 8 |
| TO | Salário mínimo acima da inflação | 6 |

## 4. Top 20 seções prioritárias

| Seção | UF | Município | Zona | Seção | Proposta dominante | Score | Votos recuperáveis | Mensagem sugerida |
| --- | --- | --- | --- | --- | --- | ---: | ---: | --- |
| SOURCE|GO|96253|49|213 | GO | TRINDADE | 49 | 213 | Ministério da Segurança | 0.75 | 300 | Ministério da Segurança une as forças para reduzir a violência que atinge a comunidade. |
| SOURCE|MG|41238|39|168 | MG | BELO HORIZONTE | 39 | 168 | Rede Brasil contra o Câncer | 0.81 | 237 | Rede Brasil contra o Câncer encurta a fila e leva diagnóstico e tratamento a quem espera. |
| SOURCE|MG|41238|39|310 | MG | BELO HORIZONTE | 39 | 310 | Rede Brasil contra o Câncer | 0.81 | 227 | Rede Brasil contra o Câncer encurta a fila e leva diagnóstico e tratamento a quem espera. |
| SOURCE|MG|42285|177|143 | MG | JENIPAPO DE MINAS | 177 | 143 | Salário mínimo acima da inflação | 0.86 | 227 | Com renda apertada, cada real do salário mínimo acima da inflação chega na mesa da família. |
| SOURCE|RJ|60011|238|249 | RJ | RIO DE JANEIRO | 238 | 249 | Teleconsultas no SUS | 0.88 | 226 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|176|178 | RJ | RIO DE JANEIRO | 176 | 178 | Teleconsultas no SUS | 0.88 | 218 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|218|114 | RJ | RIO DE JANEIRO | 218 | 114 | Teleconsultas no SUS | 0.88 | 217 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|25|389 | RJ | RIO DE JANEIRO | 25 | 389 | Teleconsultas no SUS | 0.88 | 217 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|188|258 | RJ | RIO DE JANEIRO | 188 | 258 | Teleconsultas no SUS | 0.88 | 213 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|AM|2550|31|484 | AM | MANAUS | 31 | 484 | Ministério da Segurança | 0.94 | 212 | Ministério da Segurança une as forças para reduzir a violência que atinge a comunidade. |
| SOURCE|RJ|60011|188|321 | RJ | RIO DE JANEIRO | 188 | 321 | Teleconsultas no SUS | 0.88 | 211 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|BA|38490|18|178 | BA | SALVADOR | 18 | 178 | Ministério da Segurança | 0.91 | 211 | Ministério da Segurança une as forças para reduzir a violência que atinge a comunidade. |
| SOURCE|RJ|60011|24|597 | RJ | RIO DE JANEIRO | 24 | 597 | Teleconsultas no SUS | 0.88 | 210 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|10|61 | RJ | RIO DE JANEIRO | 10 | 61 | Teleconsultas no SUS | 0.88 | 210 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|125|321 | RJ | RIO DE JANEIRO | 125 | 321 | Teleconsultas no SUS | 0.88 | 209 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|14|404 | RJ | RIO DE JANEIRO | 14 | 404 | Teleconsultas no SUS | 0.88 | 208 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|8|72 | RJ | RIO DE JANEIRO | 8 | 72 | Teleconsultas no SUS | 0.88 | 207 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|167|36 | RJ | RIO DE JANEIRO | 167 | 36 | Teleconsultas no SUS | 0.88 | 207 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|SP|71072|350|1 | SP | SÃO PAULO | 350 | 1 | Teleconsultas no SUS | 0.78 | 207 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |
| SOURCE|RJ|60011|167|24 | RJ | RIO DE JANEIRO | 167 | 24 | Teleconsultas no SUS | 0.88 | 205 | Teleconsultas no SUS aproximam o médico de quem tem dificuldade de acesso. |

## 5. Limitações metodológicas

- **Indicadores municipais vs. seção**: renda, idade, saneamento, mortalidade e cobertura de saúde são medidos no **município** e atribuídos a cada seção pelo nome normalizado do município. A variação intraurbana não entra nos indicadores.
- **Matching espacial imperfeito**: a malha de setores por UF (Voronoi) exige download de centenas de MB por UF; neste recorte foi substituída pela agregação municipal.
- **SIM sem causa feminicídio dedicada**: o Pacto Contra o Feminicídio usa a taxa de homicídios como proxy declarada.
- **Alfabetização com supressão**: o agregado por setor suprime valores (X), deprimindo a taxa absoluta; por isso entra apenas como **posto percentual** (rank).
- **Emprego formal**: aproximado pela renda do responsável (CAGED/RAIS não carregado).
- **Pesos fixos**: os pesos por proposta são documentados e versionados.
- **Fallback de UF**: onde o município não casa, usa-se a média da UF; reduz a variância.

## 6. Recomendações para a campanha

- Priorizar as **zonas de virada** do `guia_porta_a_porta_por_secao.csv`, ordenadas por votos recuperáveis e score da proposta dominante.
- Concentrar material por proposta: cards e áudios do guia correspondem ao indicador que mais pesa em cada território.
- Revalidar o casamento TSE↔IBGE dos municípios com mais seções sem matching antes de publicar números.
- Rodar a etapa de Voronoi por UF em máquina com espaço para substituir a aproximação municipal por indicadores de setor censitário.
