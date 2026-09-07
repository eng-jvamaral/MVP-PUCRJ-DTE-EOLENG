# Catálogo de Dados

Camada analítica (Gold) em esquema estrela, construída a partir do dataset **Kelmarsh Wind Farm** (Zenodo, DOI 10.5281/zenodo.5841834, licença CC-BY-4.0 Cubico Sustainable Investments).

Os tipos indicados são os **de destino no Gold**, na camada Bronze todas as colunas estão como texto (`string`) e são convertidas na Silver. A coluna *origem* aponta o nome na camada anterior. Os exemplos do fato vêm da leitura real de `2018-11-16 09:20:00` (turbina 1); os das dimensões, da turbina 1.

> **Nota sobre fuso:** os timestamps estão em **UTC** (`+00:00`), não em horário local do Reino Unido. Considerar na análise de padrões por hora do dia.

## gold_fato_scada

Grão: uma linha por **turbina × instante de 10 minutos**.

| coluna (Gold) | tipo | descrição | unidade | origem | exemplo |
|---|---|---|---|---|---|
| turbina_id | int | identificador da turbina (FK) | — | silver_scada.turbina_id | 1 |
| tempo_id | timestamp | instante da leitura (FK) | — | silver_scada.tempo | 2018-11-16 09:20:00 |
| status_id | int | status operacional vigente (FK, definido na reconciliação da S5) | — | reconciliação silver_scada × silver_status | — |
| wind_speed_m_s | double | velocidade média do vento | m/s | silver_scada.wind_speed_m_s | 3.01 |
| wind_speed_std_m_s | double | desvio-padrão do vento no intervalo | m/s | silver_scada.wind_speed_std_m_s | 0.41 |
| wind_speed_min_m_s | double | vento mínimo no intervalo | m/s | silver_scada.wind_speed_min_m_s | 2.00 |
| wind_speed_max_m_s | double | vento máximo no intervalo | m/s | silver_scada.wind_speed_max_m_s | 3.66 |
| power_kw | double | potência ativa média | kW | silver_scada.power_kw | 20.98 |
| rotor_speed_rpm | double | rotação do rotor | rpm | silver_scada.rotor_speed_rpm | 9.00 |
| generator_rpm | double | rotação do gerador | rpm | silver_scada.generator_rpm | 1066.44 |
| blade_pitch_a_deg | double | ângulo de passo da pá A | graus | silver_scada.blade_pitch_a_deg | 1.49 |
| front_bearing_temp_c | double | temperatura do mancal dianteiro | °C | silver_scada.front_bearing_temp_c | 47.34 |
| rear_bearing_temp_c | double | temperatura do mancal traseiro | °C | silver_scada.rear_bearing_temp_c | 52.48 |
| gear_oil_temp_c | double | temperatura do óleo da multiplicadora | °C | silver_scada.gear_oil_temp_c | 44.40 |
| generator_bearing_front_temp_c | double | temperatura do mancal dianteiro do gerador | °C | silver_scada.generator_bearing_front_temp_c | 40.89 |
| nacelle_ambient_temp_c | double | temperatura ambiente na nacele (referência) | °C | silver_scada.nacelle_ambient_temp_c | 11.69 |

## gold_dim_turbina

Origem: `Kelmarsh_WT_static.csv` → `silver_turbina`.

| coluna (Gold) | tipo | descrição | unidade | exemplo |
|---|---|---|---|---|
| turbina_id | int | identificador da turbina (PK) | — | 1 |
| nome | string | nome/rótulo da turbina | — | Kelmarsh 1 |
| latitude | double | latitude da turbina | graus | 52.400604 |
| longitude | double | longitude da turbina | graus | -0.947133 |
| potencia_nominal_kw | int | potência nominal | kW | 2050 |
| diametro_rotor_m | double | diâmetro do rotor | m | 92 |
| altura_cubo_m | double | altura do cubo | m | 78.5 |

> Observação: a altura do cubo varia entre turbinas (68,5 m nas turbinas 3 e 6; 78,5 m nas demais) — potencial fator explicativo em diferenças de vento/potência.

## gold_dim_tempo

Origem: coluna `tempo` da `silver_scada`.

| coluna (Gold) | tipo | descrição | unidade | exemplo |
|---|---|---|---|---|
| tempo_id | timestamp | instante da leitura (PK) | — | 2018-11-16 09:20:00 |
| data | date | data (sem hora) | — | 2018-11-16 |
| ano | int | ano | — | 2018 |
| mes | int | mês | — | 11 |
| dia | int | dia do mês | — | 16 |
| hora | int | hora | — | 9 |
| minuto | int | minuto | — | 20 |
| dia_semana | string | dia da semana | — | sexta-feira |

## gold_dim_status

Origem: `silver_status` (apenas eventos de duração — Stop, Warning, Communication; eventos Informational são instantâneos e não entram na reconciliação).

| coluna (Gold) | tipo | descrição | unidade | origem | exemplo |
|---|---|---|---|---|---|
| status_id | int | identificador do status (PK) | — | gerado | 1 |
| status | string | tipo do evento | — | silver_status.status | Stop |
| categoria_iec | string | categoria IEC do evento | — | silver_status.categoria_iec | Forced outage |
| categoria_contrato | string | categoria de contrato de serviço | — | silver_status.categoria_contrato | Full Performance |

---

## Linhagem (resumo do pipeline)

`Fonte (Zenodo)` → `Volume (Databricks)` → **Bronze** (`bronze_scada`, `bronze_status`: cópia fiel em texto) → **Silver** (`silver_scada`, `silver_status`, `silver_turbina`: tipagem, limpeza de nomes, padronização de chaves) → **Gold** (`gold_dim_turbina`, `gold_dim_tempo`, `gold_dim_status`, `gold_fato_scada`: esquema estrela com status reconciliado) → **Análises SQL + dashboards**.
