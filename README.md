# **Pipeline de Controle de Estoque Comercial**
![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![BigQuery](https://img.shields.io/badge/BigQuery-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)


Pipeline ETL para controle de equipamentos alocados a colaboradores do setor comercial com integração de dados de RH.

## 📋 Visão Geral
O script é estruturado seguindo as etapas:
* Instalação de Depedências
* Integração com banco com o google Drive e captura de dados do DE/PARA de itens contido em planilha google sheets
* Encaminhamento para tabela DeltaLake Databricks
* Processamento em e SPARK/SQL para ETL dos dados
* Classificação dos itens do estoque com base em suas respectivas classes
* Integração com dados de RH
* Integração com o BigQuery para atualização do banco de dados
* Alimenta dashboard em Looker Studio 



## 🔄 Fluxo do Processo
```mermaid
flowchart TD
    A[Google Sheets] -->|Captura De/Para| B[Databricks]
    B -->|Delta Lake| C[Catálogo de Itens]
    C -->|Spark SQL| D[Processamento ETL]
    D -->|Transformação| E[Classificação de Itens]
    E -->|Pandas| F[BigQuery]
    F -->|Visualização| G[Looker Studio]
```

## ⚙️ Pré-requisitos
- Python 3.8+
- Databricks Runtime
- Conta de serviço Google Cloud
- Acesso ao BigQuery

## 📁 Estrutura do código

## 🔧 Instalação

```bash
# No notebook Databricks:
%pip install google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client gspread drive pandas_gbq gspread_dataframe
unidecode numpy datetime packaging==23.2 -q

dbutils.library.restartPython()

```
**Finalidade das bibliotecas**
* Realiza as Autenticações Google/bigquery (OAuth2)
* Manipulação de planilhas (gspread)
* Integração BigQuery (pandas_gbq)
* Processamento de dados (Pandas/NumPy)
* Reinicialização do ambiente Python
  
## 🌐 Integração com Google Sheets

```bash

SCOPES = ["https://www.googleapis.com/auth/spreadsheets", "https://www.googleapis.comauth/drive"]
SERVICE_ACCOUNT_FILE = '/Workspace/Users/.../gsa_dados_drive.json'
credentials = Credentials.from_service_account_file(SERVICE_ACCOUNT_FILE, scopes=SCOPES)
client = gspread.authorize(credentials)

```
* Os SCOPES definem as permissões de leitura/escrita que serão utilizadas para acessar o ambiente google
* Autenticação via conta de serviço do google drive para manioulação das planilhas
* Criação do cliente via objeto de credencial gerado pelo uso de arquivo JSON da conta de serviço.
  
**Após a integração o código abaixo:**

* Acessa a planilha pelo ID definido no parâmetro open_by_key
* Seleciona a aba específica
* Converte os dados para DataFrame Pandas

```bash
spreadsheet = client.open_by_key('ID_PLANILHA')
worksheet = spreadsheet.worksheet("nome_aba_planilha")
ss = pd.DataFrame(worksheet.get_all_records())
```
**Encaminhamento da tabela para o Delta Lake Catalog/Databricks via spark**
```bash
spark.createDataFrame(ss).write.format("delta")
    .mode("overwrite")
    .saveAsTable("database.schema.depara_itens")
```
* Cria tabela Delta Lake versionada
* Estrutura: database.schema.table_name
* Modo overwrite para atualizações completas
  
## 🧑‍💻 ETL principal: processamento em SPARK/SQL (EM DEV)

