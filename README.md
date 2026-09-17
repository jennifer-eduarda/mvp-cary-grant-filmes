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


display(df_bronze)

```
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

# 4. Pipeline de Dados (Etapa 4.4)

## Fluxo do pipeline

O pipeline desenvolvido neste projeto segue o fluxo:

**Fonte dos dados → Volume do Databricks → Bronze → Silver → Gold → Análise**

O processo foi implementado utilizando Apache Spark no ambiente Databricks.

### Etapa 1 - Ingestão dos dados

Os dados foram disponibilizados em um arquivo CSV e carregados no Volume do Databricks:

`/Volumes/workspace/default/cary_grant_raw`

O arquivo utilizado foi:

`cary_grant_filmes.csv`

A camada Bronze foi criada a partir da leitura desse arquivo utilizando Spark.

### Etapa 2 - Camada Bronze

Na camada Bronze, os dados são carregados mantendo sua estrutura original.

O arquivo CSV é lido utilizando o seguinte processo:

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
```
# 5. Qualidade de Dados (Etapa 4.5)

## Verificações realizadas

Após a transformação dos dados para a camada Silver, foram realizadas verificações para avaliar a qualidade do conjunto de dados.

Foram verificados:

- quantidade total de registros;
- quantidade de identificadores IMDb distintos;
- existência de valores nulos;
- faixa de valores das avaliações IMDb;
- existência de registros duplicados.

## Resultados

Após o tratamento dos dados, foram obtidos os seguintes resultados:

| Indicador | Resultado |
|---|---:|
| Total de registros | 20 |
| IDs IMDb distintos | 20 |
| IDs nulos | 0 |
| Títulos nulos | 0 |
| Anos nulos | 0 |
| Gêneros nulos | 0 |
| Avaliações nulas | 0 |
| Diretores nulos | 0 |
| Menor avaliação IMDb | 5.9 |
| Maior avaliação IMDb | 8.3 |

A quantidade de identificadores IMDb distintos é igual à quantidade total de registros, indicando que não foram identificadas duplicidades com base no identificador do filme.

Também não foram encontrados valores nulos nas principais colunas utilizadas nas análises.

As avaliações presentes no conjunto analisado variam de **5.9 a 8.3**.

## Tratamentos aplicados

Na camada Silver foram aplicados tratamentos para melhorar a qualidade e a padronização dos dados.

Entre eles estão:

- remoção de espaços desnecessários dos campos textuais;
- conversão dos tipos de dados;
- padronização das informações;
- remoção de registros duplicados utilizando `imdb_id`.

Essas etapas foram realizadas antes da criação das tabelas Gold, garantindo que os dados utilizados nas análises passassem por uma etapa de validação e tratamento.

## Considerações sobre a qualidade

Os resultados das verificações indicam que o conjunto de dados utilizado no MVP apresenta consistência para as análises propostas.

Entretanto, como o conjunto possui apenas 20 filmes selecionados, os resultados devem ser interpretados dentro do escopo dessa amostra e não como uma representação estatística de toda a filmografia de Cary Grant.

# 6. Análise de Dados (Etapa 4.5)

## Objetivo da análise

A etapa de análise tem como objetivo utilizar os dados tratados e organizados nas camadas Silver e Gold para responder às perguntas de negócio definidas no início do projeto.

As análises foram realizadas sobre uma amostra selecionada de 20 filmes da filmografia de Cary Grant, considerando informações sobre título, ano de lançamento, gêneros, avaliação no IMDb e diretor.

## Pergunta 1 – Quais foram os filmes de Cary Grant com as melhores avaliações?

A primeira análise ordena os filmes de acordo com sua avaliação no IMDb, permitindo identificar os títulos com maiores avaliações dentro da amostra analisada.

Os cinco primeiros resultados foram:

| Filme | Ano | Avaliação IMDb |
|---|---:|---:|
| North by Northwest | 1959 | 8.3 |
| Arsenic and Old Lace | 1944 | 7.9 |
| Notorious | 1946 | 7.9 |
| The Philadelphia Story | 1940 | 7.8 |
| Bringing Up Baby | 1938 | 7.8 |

Também aparecem com avaliação 7.8 os filmes His Girl Friday, Charade e Gunga Din.

A análise demonstra que, dentro da amostra selecionada, North by Northwest apresentou a maior avaliação no IMDb, com 8.3.

## Pergunta 2 – Como as avaliações dos filmes de Cary Grant se distribuem ao longo dos anos?

A segunda análise relaciona o ano de lançamento dos filmes com suas respectivas avaliações no IMDb.

| Ano | Quantidade de filmes | Avaliação média |
|---:|---:|---:|
| 1937 | 1 | 7.70 |
| 1938 | 1 | 7.80 |
| 1939 | 2 | 7.70 |
| 1940 | 2 | 7.80 |
| 1941 | 1 | 7.30 |
| 1944 | 1 | 7.90 |
| 1946 | 1 | 7.90 |
| 1947 | 1 | 7.60 |
| 1952 | 2 | 7.05 |
| 1953 | 1 | 5.90 |
| 1955 | 1 | 7.40 |
| 1957 | 1 | 7.40 |
| 1959 | 2 | 7.75 |
| 1962 | 1 | 6.60 |
| 1963 | 1 | 7.80 |
| 1966 | 1 | 6.60 |

Na amostra analisada, as maiores médias de avaliação ocorreram em 1944 e 1946, ambas com 7.9. O ano de 1953 apresentou a menor média, com 5.9.

Como vários anos possuem apenas um filme na amostra, essas médias devem ser interpretadas considerando a quantidade de registros disponível em cada ano.

## Pergunta 3 – Quais gêneros cinematográficos aparecem com maior frequência em sua filmografia?

Para responder à terceira pergunta, os gêneros foram separados individualmente e contabilizados a partir dos filmes analisados.

| Gênero | Quantidade de filmes |
|---|---:|
| Romance | 18 |
| Comedy | 14 |
| Drama | 6 |
| Thriller | 5 |
| Mystery | 4 |
| Adventure | 3 |
| Film-Noir | 2 |
| War | 2 |
| Crime | 1 |
| Fantasy | 1 |
| Sci-Fi | 1 |

O gênero mais frequente na amostra foi Romance, presente em 18 dos 20 filmes analisados. Comedy aparece em segundo lugar, presente em 14 filmes.

A análise mostra que os filmes selecionados apresentam forte presença de gêneros relacionados a romance e comédia, embora também apareçam categorias como drama, suspense, aventura e guerra.

## Pergunta 4 – Quais foram as parcerias mais frequentes entre Cary Grant e diretores?

A quarta análise contabiliza a quantidade de filmes da amostra associados a cada diretor.

| Diretor | Quantidade de filmes | Avaliação média |
|---|---:|---:|
| Alfred Hitchcock | 4 | 7.73 |
| Howard Hawks | 4 | 7.53 |
| Leo McCarey | 2 | 7.55 |
| George Cukor | 1 | 7.80 |
| Stanley Donen | 1 | 7.80 |
| Frank Capra | 1 | 7.90 |
| George Stevens | 1 | 7.80 |
| Henry Koster | 1 | 7.60 |
| Blake Edwards | 1 | 7.20 |
| Norman Taurog | 1 | 7.20 |
| Delbert Mann | 1 | 6.60 |
| Sidney Sheldon | 1 | 5.90 |
| Charles Walters | 1 | 6.60 |

Na amostra selecionada, as parcerias mais frequentes foram com Alfred Hitchcock e Howard Hawks, com quatro filmes cada. Leo McCarey aparece em seguida, com dois filmes.

## Pergunta 5 – Quais diretores que trabalharam com Cary Grant tiveram os filmes com as melhores avaliações?

Para essa análise, foi considerada a maior avaliação IMDb registrada entre os filmes de cada diretor na amostra.

| Diretor | Quantidade de filmes | Avaliação média | Melhor avaliação |
|---|---:|---:|---:|
| Alfred Hitchcock | 4 | 7.73 | 8.3 |
| Frank Capra | 1 | 7.90 | 7.9 |
| George Cukor | 1 | 7.80 | 7.8 |
| Howard Hawks | 4 | 7.53 | 7.8 |
| Stanley Donen | 1 | 7.80 | 7.8 |
| George Stevens | 1 | 7.80 | 7.8 |
| Leo McCarey | 2 | 7.55 | 7.7 |
| Henry Koster | 1 | 7.60 | 7.6 |

Na amostra analisada, o maior valor individual de avaliação está associado a um filme dirigido por Alfred Hitchcock, com 8.3.

A quantidade de filmes também foi considerada na tabela para contextualizar os resultados, já que alguns diretores aparecem apenas uma vez na amostra.

## Síntese das análises

As cinco análises permitiram explorar diferentes aspectos da amostra de filmes de Cary Grant, incluindo avaliações, evolução temporal, frequência de gêneros e recorrência de diretores.

Os resultados mostram que North by Northwest apresentou a maior avaliação IMDb da amostra, enquanto Romance foi o gênero mais frequente. Alfred Hitchcock e Howard Hawks foram os diretores com maior número de filmes entre os títulos selecionados.

Os resultados apresentados são referentes exclusivamente aos 20 filmes selecionados para este MVP e não representam necessariamente toda a filmografia de Cary Grant.

# 7. Autoavaliação

Este projeto foi uma oportunidade de aplicar, de forma prática, conceitos relacionados à construção de um pipeline de dados em ambiente de nuvem.

Durante o desenvolvimento foram trabalhadas etapas de ingestão, armazenamento, transformação, modelagem, controle de qualidade e análise dos dados utilizando Databricks e Apache Spark.

Esta foi minha primeira experiência utilizando o Databricks para a construção de um pipeline de dados e também minha primeira experiência estruturando a documentação de um projeto por meio de um arquivo README no GitHub. Por esse motivo, algumas etapas apresentaram desafios e exigiram maior atenção, principalmente para compreender a organização das camadas, o armazenamento dos dados e a estruturação da documentação.

No início do projeto, também existia uma preocupação em relação ao tamanho dos dados e à possibilidade de o conjunto utilizado ficar muito pesado para ser processado no ambiente de nuvem. Por isso, foi escolhido inicialmente um conjunto de dados mais enxuto, contendo 20 filmes selecionados, permitindo compreender o funcionamento do pipeline e realizar as etapas de tratamento e análise de forma mais controlada.

Ao longo do desenvolvimento, foi possível compreender melhor como os dados podem ser organizados em diferentes camadas e como o Databricks pode ser utilizado para realizar o processamento e armazenamento dessas informações.

O projeto foi estruturado seguindo a lógica de camadas Bronze, Silver e Gold, permitindo separar os dados brutos dos dados tratados e das informações preparadas para análise.

Entre os principais aprendizados estão a utilização de Volumes no Databricks, a leitura de arquivos CSV com Spark, a transformação de DataFrames, a criação de tabelas em Delta Lake, a utilização do catálogo para organização dos dados e a documentação de um projeto técnico no GitHub.

Como limitação, o projeto utiliza uma amostra selecionada de 20 filmes de Cary Grant. Dessa forma, os resultados das análises devem ser interpretados dentro do escopo desse conjunto de dados e não como uma análise completa de toda a filmografia do ator.

Como evolução futura, o pipeline poderia ser ampliado para trabalhar com um conjunto maior de filmes e incluir novas informações, como número de votos, elenco, roteiristas e outras características dos títulos, permitindo análises mais abrangentes.

Considero que o desenvolvimento deste MVP contribuiu para ampliar minha compreensão sobre pipelines de dados em nuvem e proporcionou uma primeira experiência prática com ferramentas e conceitos de Engenharia de Dados que ainda não havia utilizado anteriormente.
