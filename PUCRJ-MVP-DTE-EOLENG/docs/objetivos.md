PUC-RJ

MVP Engenharia de Dados

Aluno: João Victor Amaral dos Santos (4052025002072)

Formação: Engenheiro Mecânico

# Tratamento e análise de telemetria de um parque eólico em Kelmarsh - Inglaterra

Objetivo: construir um pipeline de dados na nuvem que transforme a telemetria operacional crua (SCADA) e os registros de eventos de um parque eólico real em Kelmarsh, Inglaterra, contendo 6 turbinas eólicas Senvion MM92 numa camada analítica em esquema estrela, capaz de responder a perguntas de performance e manutenção das turbinas.

Escopo: apenas dados coletados entre os anos de 2018 e 2019.

## Perguntas a serem respondidas:

1. Curva de potência: como a potência média se comporta em função do vento? Bate com o esperado (corte ~3 m/s, nominal ~2,05 MW)? Alguma turbina fica sistematicamente abaixo das outras (subperformance)?

2. Disponibilidade: qual a fração de tempo em cada status (normal, standby, aviso, parada) por turbina? Quais perdem mais tempo produtivo?

3. Temperatura × operação: como as temperaturas (mancal, gerador, multiplicadora) evoluem com carga/vento? Aparece temperatura anormal antes de eventos críticos?

4. Eventos no tempo: distribuição dos eventos do log por tipo e por período — quais categorias são mais frequentes e quais mais custam em tempo parado?

5. Qualidade/governança: cobertura de cada sinal ao longo do tempo, % de nulos por sensor e leituras implausíveis (potência negativa, vento fora de faixa).

