# 1. Contexto de Negócios e Perguntas (Etapa 2 e 4.1)

## Contexto do projeto

Este projeto tem como objetivo desenvolver um pipeline de dados em nuvem para analisar informações relacionadas à filmografia do ator Cary Grant, utilizando recursos do Databricks e uma arquitetura de dados organizada em camadas Bronze, Silver e Gold.

Cary Grant foi um ator britânico-americano com uma carreira cinematográfica desenvolvida principalmente entre as décadas de 1930 e 1960. A análise foi realizada a partir de uma amostra selecionada de 20 filmes de sua filmografia, contendo informações sobre título, ano de lançamento, gêneros cinematográficos, avaliação no IMDb e diretor.

O projeto utiliza dados relacionados a filmes como forma de demonstrar, em um contexto educacional, as principais etapas de um pipeline de dados: carregamento, armazenamento, transformação, organização, controle de qualidade e análise.

A fonte principal das informações sobre os filmes e suas avaliações é o IMDb, com referências de filmografia e direção também utilizadas para conferência dos dados. O conjunto de dados foi preparado especificamente para este MVP e representa um recorte da filmografia analisada, não correspondendo à totalidade dos filmes da carreira do ator.

## Objetivo

O objetivo do projeto é construir um pipeline de dados em nuvem capaz de organizar e transformar informações sobre filmes de Cary Grant, permitindo realizar análises sobre avaliações, gêneros cinematográficos e diretores.

A partir dos dados tratados, busca-se identificar padrões e informações relevantes sobre a filmografia analisada, demonstrando como um pipeline de dados pode transformar dados brutos em informações estruturadas para análise.

## Perguntas de negócio

O pipeline foi desenvolvido para responder às seguintes perguntas de negócio:

1. **Quais foram os filmes de Cary Grant com as melhores avaliações?**

2. **Como as avaliações dos filmes de Cary Grant se distribuem ao longo dos anos?**

3. **Quais gêneros cinematográficos aparecem com maior frequência em sua filmografia?**

4. **Quais foram as parcerias mais frequentes entre Cary Grant e diretores?**

5. **Quais diretores que trabalharam com Cary Grant tiveram os filmes com as melhores avaliações?**

## Variáveis analisadas

As principais variáveis utilizadas no projeto são:

| Variável | Descrição |
|---|---|
| `imdb_id` | Identificador único do filme no IMDb |
| `titulo` | Título do filme |
| `ano` | Ano de lançamento do filme |
| `generos` | Gêneros cinematográficos associados ao filme |
| `avaliacao_imdb` | Avaliação do filme no IMDb |
| `diretor` | Diretor responsável pelo filme |

## Escopo da análise

Para este MVP foi utilizada uma amostra de **20 filmes** da filmografia de Cary Grant. Dessa forma, os resultados apresentados nas análises refletem exclusivamente os filmes presentes nesse conjunto de dados e não devem ser interpretados como uma análise completa de toda a carreira cinematográfica do ator.

O projeto tem caráter acadêmico e tem como foco principal demonstrar a construção e o funcionamento de um pipeline de dados em nuvem, desde a ingestão dos dados brutos até a geração de informações organizadas para análise.


# 2. Carga dos Dados (Etapa 4.2)

## Fonte dos dados

Os dados utilizados neste projeto foram obtidos a partir de informações relacionadas à filmografia de Cary Grant, tendo o IMDb como fonte principal para títulos e avaliações dos filmes. Informações de filmografia e direção também foram utilizadas para conferência dos dados.

O conjunto de dados foi preparado especificamente para este MVP, contendo uma amostra de 20 filmes selecionados da filmografia de Cary Grant.

O IMDb é uma fonte de referência para informações cinematográficas e seus dados podem sofrer atualizações ao longo do tempo. Portanto, as avaliações utilizadas representam um retrato dos dados no momento da preparação do conjunto utilizado neste projeto.

## Armazenamento dos dados brutos

Para o armazenamento dos dados brutos foi utilizado um Volume no Databricks, localizado no Unity Catalog.

O Volume utilizado foi:

`/Volumes/workspace/default/cary_grant_raw`

O arquivo `cary_grant_filmes.csv` foi carregado nesse Volume e mantido em seu formato original para preservar os dados brutos antes das transformações.

## Estrutura do arquivo

O arquivo CSV contém as seguintes colunas:

| Coluna | Tipo | Descrição |
|---|---|---|
| `imdb_id` | String | Identificador do filme no IMDb |
| `titulo` | String | Título do filme |
| `ano` | Integer | Ano de lançamento |
| `generos` | String | Gêneros cinematográficos do filme |
| `avaliacao_imdb` | Double | Avaliação do filme no IMDb |
| `diretor` | String | Diretor do filme |

O conjunto de dados possui inicialmente 20 registros.

## Leitura dos dados - Camada Bronze

Após o carregamento do arquivo no Volume, os dados foram lidos no Databricks utilizando Apache Spark.

Foi utilizado o formato CSV, com a primeira linha do arquivo definida como cabeçalho e inferência automática dos tipos das colunas.

```python
FILE_PATH = f"{RAW_PATH}/cary_grant_filmes.csv"

df_bronze = (
    spark.read
    .format("csv")
    .option("header", True)
    .option("inferSchema", True)
    .option("sep", ",")
    .load(FILE_PATH)
)

display(df_bronze)

# 3. Modelagem e Catálogo de Dados (Etapa 4.3)

## Arquitetura de dados

O projeto utiliza uma arquitetura em camadas baseada no conceito de Medallion Architecture, organizando os dados nas camadas Bronze, Silver e Gold.

Essa estrutura permite separar os dados brutos das etapas de tratamento e das informações preparadas para análise.

### Camada Bronze

A camada Bronze representa os dados após sua ingestão no ambiente de nuvem.

Nesta etapa, o arquivo CSV é carregado a partir do Volume do Databricks, mantendo sua estrutura original para preservar os dados brutos.

A leitura dos dados foi realizada por meio do Apache Spark, gerando o DataFrame `df_bronze`.

### Camada Silver

A camada Silver contém os dados tratados e padronizados.

Nesta etapa foram realizadas transformações como:

- remoção de espaços desnecessários nos campos textuais;
- conversão dos tipos de dados;
- padronização das colunas;
- remoção de registros duplicados utilizando o identificador IMDb;
- manutenção de uma estrutura adequada para as análises posteriores.

O resultado foi armazenado como uma tabela Delta no catálogo do Databricks:

`workspace.default.cary_grant_silver`

### Camada Gold

A camada Gold contém dados preparados especificamente para responder às perguntas de negócio definidas no projeto.

Foram criadas três tabelas Gold:

| Tabela | Finalidade |
|---|---|
| `workspace.default.cary_grant_gold_avaliacoes` | Organização dos filmes de acordo com suas avaliações no IMDb |
| `workspace.default.cary_grant_gold_generos` | Análise da frequência dos gêneros e de suas avaliações médias |
| `workspace.default.cary_grant_gold_diretores` | Análise da quantidade de filmes por diretor, avaliação média e melhor avaliação |

## Catálogo de Dados

As tabelas do projeto foram armazenadas no Unity Catalog utilizando o namespace:

`workspace.default`

A organização das tabelas permite identificar de forma estruturada os dados tratados e os resultados preparados para análise.

### Tabelas criadas

**Silver**

`workspace.default.cary_grant_silver`

**Gold**

`workspace.default.cary_grant_gold_avaliacoes`

`workspace.default.cary_grant_gold_generos`

`workspace.default.cary_grant_gold_diretores`

## Formato de armazenamento

As tabelas Silver e Gold foram armazenadas no formato **Delta Lake**.

O uso do formato Delta permite trabalhar com tabelas estruturadas no ambiente do Databricks e manter os dados organizados para as etapas posteriores do pipeline.

Dessa forma, o projeto apresenta uma separação entre os dados brutos, os dados tratados e os dados preparados para análise, seguindo a lógica das camadas Bronze, Silver e Gold.
