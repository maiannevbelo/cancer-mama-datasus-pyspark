# Análise de Registros de Câncer de Mama no DATASUS com PySpark

## Identificação

**Aluna:** Maianne Vidal Belo  
**Matrícula:** 72501114  

## Descrição do projeto

Este projeto foi desenvolvido como atividade da disciplina de Ciência de Dados II e tem como objetivo aplicar as etapas do processo de descoberta de conhecimento em dados (KDD) utilizando ferramentas de Big Data e Machine Learning.

Foram analisados registros de APAC de quimioterapia do Sistema de Informações Ambulatoriais do SUS (SIA/SUS), referentes ao estado da Bahia durante o ano de 2025.

A análise foi direcionada aos registros cujo diagnóstico principal (`AP_CIDPRI`) inicia por `C50`, correspondente às neoplasias malignas da mama.

Após a integração dos 12 arquivos mensais, a base inicial apresentou **283.432 registros e 74 variáveis**. Após o filtro para C50, foram selecionados **135.115 registros**, atendendo ao critério de volume superior a 100 mil registros estabelecido para o projeto.

> **Importante:** os dados correspondem a registros administrativos de APAC e não devem ser interpretados diretamente como quantidade de pacientes. Um mesmo indivíduo pode apresentar múltiplos registros ao longo do tratamento.

## Fonte dos dados

Os dados utilizados são provenientes do **DATASUS**, por meio dos arquivos de produção ambulatorial do SIA/SUS — APAC de Quimioterapia (AQ).

**Período analisado:** janeiro a dezembro de 2025  
**Unidade da Federação:** Bahia (BA)  
**Grupo:** APAC de Quimioterapia (AQ)  
**Diagnóstico principal selecionado:** CID-10 iniciado por C50  

### Links oficiais

- **Sistema de Informações Ambulatoriais do SUS (SIA/SUS):** https://sia.datasus.gov.br/principal/index.php
- **Documentação e arquivos da APAC — DATASUS:** https://sia.datasus.gov.br/documentos/listar_ftp_apac.php

Os arquivos foram obtidos programaticamente por meio da biblioteca **PySUS** e processados utilizando **PySpark**.

## Tecnologias utilizadas

- Python
- Google Colab
- PySpark
- Spark SQL
- PySUS
- Matplotlib
- Machine Learning com Spark MLlib
- GitHub

## Etapas desenvolvidas

### 1. Ingestão e pré-processamento

Os 12 arquivos mensais de 2025 foram integrados utilizando PySpark.

Foram realizadas etapas de:

- avaliação de valores ausentes;
- verificação de duplicidades;
- conversão de tipos de dados;
- tratamento da variável idade;
- identificação de possíveis outliers;
- criação de variáveis derivadas;
- seleção das variáveis utilizadas nas análises.

Após o filtro para diagnóstico principal iniciado por C50, foram obtidos **135.115 registros**.

### 2. Análise Exploratória com Spark SQL

O DataFrame foi registrado como uma visão temporária do Spark SQL e utilizado para responder perguntas relacionadas ao perfil dos registros.

Entre os principais resultados:

- **99,15%** dos registros foram classificados como sexo feminino;
- a faixa etária com maior frequência foi de **50 a 59 anos**, com 38.890 registros;
- os estádios **2 e 3** apresentaram as maiores frequências;
- a categoria **parda** correspondeu a 77,06% dos registros na variável raça/cor;
- a categoria de continuidade do tratamento `S` apresentou maior frequência em todos os estadiamentos analisados.

### 3. Modelagem Preditiva

A variável administrativa `AQ_CONTTR`, referente à continuidade do tratamento registrada na APAC, foi utilizada como variável-alvo para um problema de classificação binária:

- `1` = Sim;
- `0` = Não.

Foram utilizados dois algoritmos:

#### Regressão Logística

- **Acurácia:** 70,48%
- **F1:** 0,6085
- **AUC:** 0,7066

#### Random Forest

- **Acurácia:** 70,76%
- **F1:** 0,5864
- **AUC:** 0,7184

Os resultados dos dois algoritmos foram próximos. A Random Forest apresentou maior AUC e acurácia, enquanto a Regressão Logística apresentou maior F1.

Devido à maior frequência da classe `S` na base, a acurácia não foi interpretada isoladamente.

> A variável `AQ_CONTTR` possui natureza administrativa. A categoria `N` não deve ser interpretada diretamente como abandono do tratamento ou interrupção clínica do acompanhamento.

### 4. Modelagem Descritiva — K-Means

Para a análise de agrupamento foram testadas soluções com 2, 3 e 4 clusters.

Resultados da métrica Silhouette:

- **K = 2:** 0,7346
- **K = 3:** 0,6908
- **K = 4:** 0,6594

A solução com **K = 2** apresentou o maior valor de Silhouette e foi selecionada.

O modelo final identificou:

- **Cluster 0:** 56.636 registros — idade média aproximada de 69,6 anos;
- **Cluster 1:** 78.438 registros — idade média aproximada de 48,7 anos.

A idade foi a característica que mais diferenciou os dois agrupamentos.

## Principais limitações

A base analisada contém registros administrativos de APAC e não indivíduos únicos.

Os dados também não representam todos os casos de câncer de mama diagnosticados na Bahia, mas o recorte de registros de APAC de quimioterapia do SUS analisado.

Na análise de clustering, as características utilizadas não foram previamente padronizadas. Dessa forma, a diferença de escala entre as variáveis pode ter aumentado a influência da idade na formação dos clusters.

## Como executar o projeto

O notebook foi desenvolvido no **Google Colab**.

Para reproduzir a análise:

1. Abra o arquivo `Projeto_Cancer_Mama_DATASUS.ipynb`.
2. Utilize a opção **Open in Colab**.
3. Execute as células na ordem apresentada.
4. As bibliotecas PySUS e PySpark são instaladas no início do notebook.
5. Os arquivos mensais são obtidos programaticamente e posteriormente processados pelo PySpark.

## Arquivos do repositório

- `Projeto_Cancer_Mama_DATASUS.ipynb` — notebook contendo o pipeline completo de análise.
- `README.md` — descrição e instruções do projeto.
- `RELATORIO.md` — relatório estruturado segundo as etapas do processo KDD.

## Autora

**Maianne Vidal Belo**  
Matrícula: **72501114**
