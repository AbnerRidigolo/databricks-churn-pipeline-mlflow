# 🚀 Projeto de Pipeline End-to-End: Previsão de Churn com Databricks e MLOps

Este repositório contém o código de um projeto de portfólio que demonstra um pipeline completo de Engenharia de Dados e Machine Learning (MLOps) na plataforma Databricks, utilizando o Unity Catalog.

**Objetivo:** Prever a probabilidade de *churn* (cancelamento) de clientes de uma empresa de telecomunicações, construindo um pipeline que vai desde a ingestão dos dados brutos até a "produção" de um modelo de IA.

---

## 🛠️ Ferramentas e Conceitos Aplicados

* **Plataforma:** Databricks (com Unity Catalog)
* **Processamento de Dados:** Apache Spark (PySpark)
* **Armazenamento (Lakehouse):** Delta Lake
* **MLOps (Ciclo de Vida de ML):** MLflow (Tracking e Model Registry)
* **Modelagem (IA):** Scikit-learn (Logistic Regression, Random Forest)
* **Linguagem:** Python

---

## 🏗️ Arquitetura do Pipeline

O projeto é executado em um único notebook Databricks (`churn_pipeline.py`) e é dividido em três fases distintas que simulam um ambiente de produção real:

### Fase 1: Engenharia de Dados (ETL com Arquitetura Bronze/Silver)

O objetivo desta fase é pegar dados brutos e "sujos" e transformá-los em um *produto de dados* limpo, confiável e pronto para análise.

1.  **Camada Bronze:** Os dados são lidos de um arquivo CSV bruto (`WA_Fn-UseC_-Telco-Customer-Churn.csv`) e carregados em um DataFrame Spark.
2.  **Camada Prata:** O DataFrame passa por um processo de limpeza e transformação:
    * Tratamento de valores nulos (ex: `TotalCharges` com " " -> 0).
    * Padronização dos nomes das colunas (ex: `customerID` -> `id_cliente`).
    * Remoção de colunas irrelevantes (`id_cliente`).
3.  **Entrega:** O DataFrame limpo (`df_prata`) é salvo em formato **Delta Lake** (`/Volumes/workspace/default/up/churn_prata`), servindo como a fonte única da verdade (Single Source of Truth) para a próxima fase.

### Fase 2: Ciência de Dados e MLOps

Com os dados limpos, o "chapéu" de Cientista de Dados é assumido para treinar e gerenciar os modelos de IA.

1.  **Carregamento de Features:** Os dados são lidos da tabela Delta `churn_prata`.
2.  **Pré-processamento:** É criado um `ColumnTransformer` (`preprocessor`) do Scikit-learn para converter dados categóricos (One-Hot Encoding) e normalizar dados numéricos (Standard Scaler).
3.  **Rastreamento de Experimentos (MLflow):**
    * O `mlflow.sklearn.autolog()` é ativado para rastrear automaticamente métricas e parâmetros.
    * Dois modelos são treinados dentro de "runs" do MLflow para comparação:
        * `Baseline_RegressaoLogistica_v2` (Modelo Simples)
        * `Challenger_RandomForest_v2` (Modelo Complexo)
    * **Crucial:** O artefato `preprocessor` é salvo manualmente (`mlflow.sklearn.log_model(preprocessor, "preprocessor")`) junto com o modelo, garantindo que o pipeline de inferência possa replicar as transformações.
4.  **Registro de Modelos (MLOps):** O melhor modelo (`Challenger_RandomForest_v2`) é registrado no **Unity Catalog** com o nome de 3 partes: `workspace.default.modelo_churn`.

### Fase 3: Inferência em Lote (Simulação de Produção)

Esta fase simula um *job* agendado que usa o modelo em produção para gerar previsões.

1.  **Carregamento de Artefatos:** O pipeline carrega os artefatos diretamente do MLflow, garantindo que a versão correta seja usada:
    * O `preprocessor` é carregado pelo seu `run_id`.
    * O modelo de IA é carregado pelo seu nome e versão registrados: `models:/workspace.default.modelo_churn/1`.
2.  **Geração de Previsões:** O modelo é usado para prever o *churn* em dados de teste (simulando novos dados).
3.  **Camada Ouro:** As previsões (`churn_previsto`) são unidas aos dados originais e salvas em uma tabela Delta final (`/Volumes/workspace/default/up/churn_predicoes`). Esta tabela "Ouro" é o produto final, pronto para ser consumido por um dashboard de BI ou analista.

---

## 📈 Resultados

#### Rastreamento de Experimentos no MLflow
https://github.com/AbnerRidigolo/databricks-churn-pipeline-mlflow/issues/1#issue-3583457449

#### Tabela de Previsões Finais (Camada Ouro)
![Image](https://github.com/user-attachments/assets/c32e86cd-ad62-473f-8faf-15056745248f)

---

## 🚀 Como Executar o Projeto

1.  **Setup:** Crie uma conta de Free Trial no Databricks.
2.  **Upload do CSV:** Baixe o dataset "Telco Customer Churn" do Kaggle. Faça o upload do arquivo CSV para um Volume no Databricks (ex: `/Volumes/workspace/default/up/`).
3.  **Criar Notebook:** Crie um novo notebook e anexe-o a um cluster.
4.  **Importar Código:** Importe o arquivo `churn_pipeline.py` para o notebook (ou copie e cole o código).
5.  **Atualizar Caminhos:**
    * **Célula 1:** Atualize `caminho_csv` para o local onde você salvou o CSV.
    * **Célula 3:** Atualize `caminho_delta` para onde você deseja salvar a tabela Prata.
6.  **Executar Células 1-7:** Rode as células em ordem.
7.  **Atualizar Célula 8:** Após rodar a Célula 7, vá na UI do MLflow, copie o **"Run ID"** do experimento `Challenger_RandomForest_v2` e cole-o na variável `run_id` da Célula 8.
8.  **Registrar Modelo:** Siga os passos na Célula 7 (comentários) para registrar o modelo no Unity Catalog com o nome `workspace.default.modelo_churn`.
9.  **Executar Célula 8:** Execute a célula final para gerar as previsões.
