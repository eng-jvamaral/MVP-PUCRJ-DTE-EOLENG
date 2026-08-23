# MVP — Engenharia de Dados: Análise Operacional do Parque Eólico de Kelmarsh

**Aluno:** João Victor Amaral — Matrícula: 4052025002072
**Curso:** Pós-Graduação em Ciência de Dados e Analytics — PUC-Rio
**Sprint:** Engenharia de Dados
**Plataforma:** Databricks Free Edition (arquitetura Bronze → Silver → Gold)

## Objetivo
Construir um pipeline de dados na nuvem que transforme a telemetria operacional crua (SCADA) e os registros de eventos de um parque eólico real em Kelmarsh, Inglaterra, com 6 turbinas Senvion MM92 de 2,05 MW numa camada analítica em esquema estrela, capaz de responder perguntas de performance e manutenção das turbinas.

### Perguntas de negócio
1. **Curva de potência:** como a potência média se comporta em função do vento?
   Bate com o esperado? Alguma turbina fica sistematicamente abaixo (subperformance)?
2. **Disponibilidade:** qual a fração de tempo em cada status (normal, standby,
   aviso, parada) por turbina? Quais perdem mais tempo produtivo?
3. **Temperatura × operação:** como as temperaturas (mancal, gerador, multiplicadora)
   evoluem com carga/vento? Há temperatura anormal antes de eventos críticos?
4. **Eventos no tempo:** distribuição dos eventos do log por tipo e período — quais
   categorias são mais frequentes e mais custam em tempo parado?
5. **Qualidade/governança:** cobertura de cada sinal ao longo do tempo, % de nulos
   por sensor e leituras implausíveis (potência negativa, vento fora de faixa).

## Dados
- **Fonte:** Kelmarsh Wind Farm Data (Zenodo)
- **DOI:** 10.5281/zenodo.5841834
- **Licença:** CC-BY-4.0 — Cubico Sustainable Investments Ltd
- **Citação:** Plumley, C. (2022). *Kelmarsh wind farm data* [Data set]. Zenodo.
  https://doi.org/10.5281/zenodo.5841834
- **Escopo usado:** SCADA de 2018 e 2019 + cadastro estático das turbinas
  (`Kelmarsh_WT_static.csv`) + mapeamento de sinais (`Kelmarsh_WT_dataSignalMapping.csv`).
- **Observação:** os dados brutos de SCADA não estão versionados aqui, ficam
  persistidos em um Volume no Databricks. Este repositório guarda código,
  documentação, evidências e metadados de referência.

## Arquitetura do pipeline
Fonte (Zenodo) → Volume (Databricks) → Bronze (bruto em Delta) →
Silver (limpeza, tipagem, conciliação SCADA × eventos) →
Gold (esquema estrela: dimensões + fato) → Análises SQL + dashboards.

## Estrutura do repositório
- `notebooks/` — notebooks do pipeline (ingestão, silver, gold, análises)
- `sql/` — consultas de análise e de qualidade
- `docs/` — objetivo, catálogo de dados e relatório
- `evidencias/` — prints de resultados e dashboards
- `dados/referencia/` — cadastro e dicionário de sinais

## Status
🚧 Em construção — Sprint de Engenharia de Dados (entrega em 27/09).

## Contato

LinkedIn: https://www.linkedin.com/in/joaovictoramaral/
