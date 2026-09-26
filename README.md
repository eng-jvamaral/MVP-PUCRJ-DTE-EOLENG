# MVP — Engenharia de Dados: Parque Eólico de Kelmarsh

**Pipeline analítico e diagnóstico operacional de sistemas mecânicos em turbinas eólicas**

Pós-Graduação em Ciência de Dados e Analytics — PUC-Rio · Sprint de Engenharia de Dados
Aluno: João Victor Amaral dos Santos · Matrícula: 4052025002072

**Relatório completo:** [`Relatorio_MVP_Engenharia_de_Dados_Joao_Victor_Amaral_4052025002072.pdf`](Relatorio_MVP_Engenharia_de_Dados_Joao_Victor_Amaral_4052025002072.pdf)

O relatório em PDF é o documento de entrega e contém todos os tópicos exigidos (contexto e perguntas, carga, modelagem e catálogo, pipeline, qualidade, análise e autoavaliação), com as evidências de execução no Databricks. Este README resume o projeto e orienta a navegação no repositório.



## Objetivo

Construir um pipeline de dados na nuvem que transforme a telemetria SCADA e o log de eventos do Parque Eólico de Kelmarsh (Reino Unido, 6 turbinas Senvion MM92 de 2.050 kW, ano de 2018) em uma camada analítica em esquema estrela, capaz de responder a perguntas de desempenho e manutenção das turbinas.

### Perguntas de negócio

1. **Curva de potência:** a curva observada corresponde à do fabricante? Alguma turbina fica sistematicamente abaixo das demais?
2. **Disponibilidade:** qual a fração do tempo em que cada turbina permanece parada?
3. **Temperatura × operação:** há anomalias térmicas que expliquem perdas de desempenho?
4. **Eventos no tempo:** quais eventos são mais frequentes e quais custam mais tempo?
5. **Qualidade e governança:** qual a cobertura, o percentual de nulos e a ocorrência de leituras implausíveis?

### Principais resultados

- A curva de potência do parque reproduz a do fabricante, o que valida o pipeline de ponta a ponta.
- A **turbina 5** produz cerca de 16% menos em vento forte, sem maior indisponibilidade.
- A investigação por eliminação de hipóteses apontou como causa-raiz provável um **controle de pitch descalibrado** (ângulo médio de 24,9° contra 12–14° das demais).
- Eventos *Warning* consomem cerca de 3× mais tempo que as paradas (*Stop*), embora sejam menos frequentes.
- A auditoria de qualidade identificou ruído no sensor de rotação do gerador (tratado em tabela curada) e uma lacuna sistêmica de coleta em outubro de 2018.

---

## Dados

| Item | Descrição |
|---|---|
| Fonte | *Kelmarsh Wind Farm Data* — Plumley (2022), Zenodo |
| DOI | [10.5281/zenodo.5841834](https://doi.org/10.5281/zenodo.5841834) |
| Licença | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — uso livre com atribuição |
| Escopo | SCADA (10 min) e log de status de 2018, 6 turbinas, mais cadastro das turbinas |
| Volume | 315.360 leituras × 299 colunas brutas e 69.963 eventos |

Os dados brutos **não** estão versionados neste repositório, conforme permitido pelo enunciado. Podem ser baixados diretamente no Zenodo (arquivos `Kelmarsh_SCADA_2018_3084.zip`, `Kelmarsh_WT_static.csv` e `Kelmarsh_WT_dataSignalMapping.csv`).

---

## Arquitetura

```
Zenodo ──► Volume kelmarsh_raw ──► Bronze ──► Silver ──► Gold (esquema estrela) ──► Análises SQL
           (Unity Catalog)         texto      tipagem    fato + 3 dimensões
                                   fiel       limpeza    + tabela curada
```

**Plataforma:** Databricks Free Edition (serverless) · Delta Lake · Unity Catalog · PySpark, pandas e SQL

### Modelo dimensional (camada Gold)

- `gold_fato_scada`: grão de 1 linha por turbina × instante de 10 minutos, com 13 medidas
- `gold_dim_turbina`: cadastro das 6 turbinas
- `gold_dim_tempo`: 52.560 instantes de 10 minutos de 2018 (UTC)
- `gold_dim_status`: 0 = Normal, 1 = Stop (obtido por *range join* com o log de eventos)
- `gold_fato_scada_curado`: versão curada do fato, usada nas análises

O diagrama ER está em [`docs/diagrama_er_final.pdf`](docs/diagrama_er_final.pdf) e o catálogo de dados em [`docs/catalogo.md`](docs/catalogo.md).

---

## Notebooks

Os notebooks estão em [`notebooks/`](notebooks/). As versões em HTML preservam as saídas de execução no Databricks e podem ser visualizadas diretamente no navegador:

| Notebook | Etapa | Código | Visualizar com saídas |
|---|---|---|---|
| 01 – Bronze | Ingestão dos CSVs em Delta | [ipynb](notebooks/01_bronze.ipynb) | [abrir](https://eng-jvamaral.github.io/MVP-PUCRJ-DTE-EOLENG/notebooks/html/01_bronze.html) |
| 02 – Silver | Tipagem, limpeza e padronização | [ipynb](notebooks/02_silver.ipynb) | [abrir](https://eng-jvamaral.github.io/MVP-PUCRJ-DTE-EOLENG/notebooks/html/02_silver.html) |
| 03 – Gold | Esquema estrela e reconciliação | [ipynb](notebooks/03_gold.ipynb) | [abrir](https://eng-jvamaral.github.io/MVP-PUCRJ-DTE-EOLENG/notebooks/html/03_gold.html) |
| 04 – Qualidade | Auditoria e curadoria | [ipynb](notebooks/04_qualidade.ipynb) | [abrir](https://eng-jvamaral.github.io/MVP-PUCRJ-DTE-EOLENG/notebooks/html/04_qualidade.html) |
| 05 – Análises | Respostas às perguntas (SQL) | [ipynb](notebooks/05_analises.ipynb) | [abrir](https://eng-jvamaral.github.io/MVP-PUCRJ-DTE-EOLENG/notebooks/html/05_analises.html) |

---

## Estrutura do repositório

```
MVP-PUCRJ-DTE-EOLENG/
├── README.md
├── Relatorio_MVP_Engenharia_de_Dados_Joao_Victor_Amaral_4052025002072.pdf   ← documento de entrega
├── notebooks/
│   ├── 01_bronze.ipynb … 05_analises.ipynb
│   └── html/                     ← mesmos notebooks com as saídas de execução
└── docs/
    ├── catalogo.md               ← catálogo de dados da camada Gold
    └── diagrama_er_final.pdf     ← diagrama do esquema estrela
```

## Como reproduzir

1. Crie uma conta no [Databricks Free Edition](https://www.databricks.com/learn/free-edition).
2. Em **Catalog → workspace → default**, crie o Volume `kelmarsh_raw` com os diretórios `scada_2018/` e `ref/`.
3. Faça o upload dos CSVs de 2018 (12 arquivos) em `scada_2018/` e dos 2 arquivos de referência em `ref/`.
4. Importe os notebooks e execute-os em ordem: `01_bronze` → `02_silver` → `03_gold` → `04_qualidade` → `05_analises`.

Todas as gravações usam o modo `overwrite`, então os notebooks podem ser reexecutados sem duplicar registros.

---

## Referência dos dados

PLUMLEY, C. **Kelmarsh wind farm data**. [S. l.]: Zenodo, 2022. DOI: 10.5281/zenodo.5841834.
