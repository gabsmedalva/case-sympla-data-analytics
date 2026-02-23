# 🚀 Sympla Data Pipeline & ML Forecasting

Este repositório contém o pipeline completo de dados desenvolvido para o case de Data Analytics da Sympla. O objetivo principal do projeto foi construir uma fundação sólida de dados em nuvem (Data Warehouse) e aplicar Inteligência Artificial para guiar a estratégia de expansão do CEO para o ano de 2034.

## Arquitetura e Racional do Projeto

O pipeline foi estruturado em um Jupyter Notebook (`pipeline_sympla_case.ipynb`) executando as seguintes fases lógicas:

### 1. ETL e Data Quality (Raw ➔ Trusted)
* **Extração:** Leitura dos dados transacionais brutos do Data Lake (Google Cloud Storage).
* **Transformação Sênior:** * Normalização rigorosa de tipagem: Conversão de valores financeiros para `Decimal` (evitando perdas de ponto flutuante), tratamento de strings nulas e normalização de datas.
  * Padronização de strings para o cruzamento de dimensões (Caixa alta, remoção de espaços em branco).
* **Carga (Trusted):** Armazenamento do formato colunar `.parquet` utilizando a engine `pyarrow` para máxima performance de I/O.

### 2. Modelagem Dimensional (Star Schema)
Para garantir que o Dashboard final e o Copiloto de IA tivessem um tempo de resposta de milissegundos na ponta, a base transacional foi convertida em um modelo de **Star Schema**:
* **`dim_tempo`:** Expandida cirurgicamente até Dez/2034 para suportar as projeções de Machine Learning.
* **`dim_localidade`, `dim_produtor`, `dim_evento`:** Dimensões geradas extraindo as chaves únicas (Surrogate Keys) do histórico consolidado.
* **`fato_vendas`:** Tabela central contendo as chaves numéricas de relacionamento e métricas agregadas.

O modelo foi então ingerido via job nativo do Google BigQuery (`WRITE_TRUNCATE`), garantindo a escalabilidade na nuvem.

### 3. Modelo Preditivo (Machine Learning Forecasting)
Para responder à pergunta do CEO sobre o "futuro" da expansão, implementou-se um algoritmo preditivo sobre os dados do BigQuery.
* **O Algoritmo:** `RandomForestRegressor` (Scikit-Learn). Escolhido por sua excelência em lidar com sazonalidades complexas e relações não-lineares, atingindo um **Score R² de 0.85** no treino.
* **A Granularidade (O Diferencial de Negócio):** O modelo não prevê o faturamento global da empresa de forma genérica. Utilizando a técnica de *Cross Join* no Pandas, foi gerada uma malha com todas as combinações ativas de Estados, Produtores e Eventos. O algoritmo previu o comportamento *individual* de cada nicho de Março a Dezembro de 2034.
* **Governança:** A tabela final injetada no BigQuery possui a trava de segurança `flag_previsao`, permitindo que os executivos separem analiticamente o faturamento real do caixa projetado.

## Stack Tecnológica
* **Linguagem:** Python (Pandas, Scikit-Learn, PyArrow)
* **Data Warehouse:** Google BigQuery
* **Data Lake:** Google Cloud Storage
* **Dataviz & IA:** Microsoft Power BI (DAX Avançado) e Gemini 2.5 Pro (Text-to-SQL API)

---
*Projeto desenvolvido para demonstração de arquitetura ponta a ponta: do dado bruto à decisão C-Level.*
