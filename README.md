# 🚀 Sympla Data Pipeline, ML Forecasting & BI Strategy

Este repositório contém o pipeline completo de dados e a camada de visualização desenvolvidos para o case de Data Analytics da Sympla. O objetivo principal do projeto foi construir uma fundação sólida de dados em nuvem (Data Warehouse), aplicar Inteligência Artificial para prever vendas e desenvolver um painel executivo para guiar a estratégia de expansão do CEO para o ano de 2034.

## 📊 O Dataset
A base de dados utilizada simula o ambiente transacional da Sympla, contendo o histórico de vendas de ingressos, cadastro de produtores, eventos, canais de aquisição e localidades (estados). O cenário de negócios estabelece que o mês atual de análise é **Fevereiro de 2034**, exigindo análises comparativas contra o mês anterior (MoM) e o mesmo período do ano passado (YoY), além da necessidade de prever o comportamento do mercado até o fim do ano.

## 🏗️ Pipeline de Dados: Arquitetura Medallion
O pipeline de engenharia foi desenvolvido em Python estruturado em um Jupyter Notebook (`pipeline_sympla_case.ipynb`), adotando as melhores práticas da Arquitetura Medallion no Google Cloud Storage e BigQuery:

### 1. Camada RAW (Bronze)
* **Ingestão:** Extração dos dados transacionais brutos (`.csv`) diretamente do Data Lake.
* **Objetivo:** Manter uma cópia imutável e exata da fonte de origem para auditoria e reprocessamento histórico.

### 2. Camada TRUSTED (Silver) - *Data Quality*
* **Limpeza e Tipagem:** Aplicação de regras rigorosas de qualidade de dados. Conversão de strings de datas para objetos `datetime`, remoção de espaços e padronização em caixa alta para chaves de cruzamento.
* **Precisão Financeira:** Conversão das métricas monetárias e de ingressos, tratando formatos locais (vírgulas) e transformando em números de precisão `Decimal`, evitando perdas de arredondamento por ponto flutuante.
* **Armazenamento:** Os dados limpos foram salvos em formato colunar `.parquet` via engine `pyarrow`, garantindo compressão e máxima performance de leitura.

### 3. Camada REFINED (Gold) - *Star Schema*
* **Modelagem Dimensional:** A base normalizada foi transformada em um modelo *Star Schema* otimizado para análises de BI em milissegundos.
  * **Dimensões (`dim_localidade`, `dim_produtor`, `dim_evento`):** Geradas via extração de Surrogate Keys (SKs) únicas do histórico.
  * **`dim_tempo`:** Expandida cirurgicamente até Dezembro de 2034 para suportar as linhas do futuro geradas pelo modelo de IA.
  * **`fato_vendas`:** Tabela central consolidando as métricas de receita e volume.
* O modelo final foi ingerido via job nativo do Google BigQuery (`WRITE_TRUNCATE`), democratizando o acesso aos dados para as ferramentas analíticas.

## 🤖 Machine Learning & Forecasting (A Fato Final)
Para responder à pergunta do CEO sobre o "futuro" da expansão, implementou-se um algoritmo de Séries Temporais em Machine Learning consumindo os dados da camada Refined.

* **O Algoritmo:** Treinamos um `RandomForestRegressor` (Scikit-Learn). O modelo aprendeu a sazonalidade e o padrão histórico de faturamento cruzando o calendário com o peso de cada Estado, Produtor e Evento, atingindo um **Score R² de 0.85** no teste.
* **A Projeção em Granularidade Total:** Em vez de prever um faturamento global genérico, o pipeline utiliza *Cross Join* no Pandas para gerar uma malha com todas as combinações ativas de mercado. O algoritmo previu o comportamento *individual* de cada nicho (ex: *Evento de Tecnologia em SP do Produtor X*) de Março a Dezembro de 2034.
* **A Fato Consolidada (Governança):** O output da IA foi concatenado com o histórico real da `fato_vendas`. Foi criada a coluna `flag_previsao` (0 = Realizado; 1 = Projetado). A tabela foi injetada no BigQuery, permitindo que os executivos e o dashboard separem analiticamente o faturamento real da expectativa estatística sem quebrar o modelo de dados.

## 📈 Produto de Dados: Dashboard & Matriz BCG Dinâmica
O consumo final dos dados (Refined + Forecast) foi materializado em um **Cockpit Executivo no Power BI**, desenhado para responder exatamente onde concentrar, manter ou reduzir esforços de expansão de vendas.

O grande diferencial do dashboard é a implementação de uma **Matriz BCG Canônica e Dinâmica** construída inteiramente com DAX Avançado (utilizando `ALLSELECTED`, `ISINSCOPE` e Transição de Contexto):
* **Eixo X (Share/Proporção):** Calcula em tempo real o % de participação financeira do item frente ao total filtrado.
* **Eixo Y (Crescimento YoY):** Mede a aceleração das vendas comparado a Fevereiro de 2033.
* **Inteligência de Recortes:** Através de parâmetros de campo, o CEO pode alterar a visão do gráfico dinamicamente (por Estado, Tamanho do Produtor, Canal de Aquisição ou Categoria do Evento). O DAX recalcula as médias do eixo e reclassifica os pontos em tempo real nos 4 quadrantes estratégicos:
  * ⭐ **Estrelas:** (Alto Share, Alto Crescimento) - Foco principal de expansão.
  * ❓ **Interrogações:** (Baixo Share, Alto Crescimento) - Apostas para alocação de marketing.
  * 🐄 **Vacas Leiteiras:** (Alto Share, Baixo Crescimento) - Manutenção de base de caixa.
  * 🍍 **Abacaxis:** (Baixo Share, Baixo Crescimento) - Avaliação de desinvestimento.

## 🛠️ Stack Tecnológica
* **Linguagem & Engenharia:** Python (Pandas, Scikit-Learn, PyArrow)
* **Data Lake:** Google Cloud Storage
* **Data Warehouse:** Google BigQuery
* **Dataviz & Business Intelligence:** Microsoft Power BI (Modelagem Star Schema e DAX Avançado)

---
*Projeto desenvolvido para demonstração de arquitetura de dados ponta a ponta: da ingestão do dado bruto à entrega de valor e decisão C-Level.*
