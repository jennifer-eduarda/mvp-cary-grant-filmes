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
