# Relatório do Projeto — Ciência de Dados II

## Análise dos registros de APAC de quimioterapia relacionados ao câncer de mama na Bahia em 2025

**Aluna:** Maianne Vidal Belo  
**Matrícula:** 72501114  

---

## 1. Problema e conjunto de dados

O presente projeto teve como objetivo analisar registros de tratamento quimioterápico relacionados ao câncer de mama no Sistema Único de Saúde (SUS), utilizando técnicas de processamento de Big Data, análise exploratória e Machine Learning.

Foram utilizados dados do Sistema de Informações Ambulatoriais do SUS (SIA/SUS), especificamente registros de APAC de Quimioterapia (AQ) referentes ao estado da Bahia, abrangendo as 12 competências mensais do ano de 2025.

Os arquivos foram obtidos programaticamente utilizando a biblioteca PySUS e posteriormente processados com PySpark.

Após a integração dos 12 arquivos mensais, foram identificados **283.432 registros e 74 variáveis**. Como a base de quimioterapia contém registros referentes a diferentes neoplasias, foi aplicado um filtro sobre o diagnóstico principal (`AP_CIDPRI`), selecionando os códigos iniciados por `C50`, relacionados às neoplasias malignas da mama.

Após esse filtro, o conjunto principal utilizado no projeto apresentou **135.115 registros**, correspondendo a 47,67% dos registros de quimioterapia inicialmente analisados.

O volume final atende ao requisito do projeto de utilização de conjunto de dados com pelo menos 100 mil registros.

É importante destacar que a unidade analisada é o **registro administrativo de APAC**, e não o paciente individual. Dessa forma, um mesmo indivíduo pode apresentar mais de um registro ao longo do tratamento.

---

## 2. Seleção, pré-processamento e transformação dos dados

O processamento principal foi realizado utilizando PySpark DataFrames.

Inicialmente, foi realizada a avaliação da qualidade dos dados, incluindo a identificação de valores nulos, duplicidades, tipos das variáveis e possíveis valores inconsistentes.

Não foram identificados registros completamente duplicados, permanecendo os **135.115 registros** após a verificação.

As variáveis encontravam-se originalmente armazenadas predominantemente como texto (`string`). Dessa forma, foram realizadas conversões necessárias para permitir as análises quantitativas.

### Tratamento da idade

A idade foi analisada utilizando as variáveis disponíveis na base e foi criada a variável derivada `IDADE_ANOS`.

Foram obtidos **135.074 registros com idade válida em anos**. Outros 41 registros apresentaram codificação que não permitiu interpretação segura e, portanto, foram mantidos na base original, mas tratados como ausentes na variável derivada.

A idade válida variou entre 21 e 99 anos.

A análise pelo método do Intervalo Interquartil (IQR) identificou 346 registros entre 94 e 99 anos como possíveis valores extremos. Esses registros foram preservados, pois correspondem a idades biologicamente possíveis e não constituem, isoladamente, evidência de erro.

### Valor da APAC

A variável referente ao valor registrado na APAC foi convertida para formato numérico, originando `VALOR_APAC`.

Foram identificados valores entre R$ 0,00 e R$ 3.699,40, com média aproximada de R$ 360,11.

Os registros com valor zero foram preservados, pois não foi estabelecido que esse valor representasse necessariamente erro ou ausência de informação.

---

## 3. Análise Exploratória de Dados com Spark SQL

O DataFrame preparado foi registrado como uma visão temporária, permitindo a realização de consultas por meio do Spark SQL.

Foram elaboradas cinco perguntas exploratórias.

### 3.1 Qual a distribuição dos registros segundo o sexo?

Foram identificados:

- **133.973 registros classificados como sexo feminino (99,15%)**;
- **1.142 registros classificados como sexo masculino (0,85%)**.

Observou-se forte predominância dos registros classificados como sexo feminino, embora também tenham sido encontrados registros classificados como sexo masculino.

### 3.2 Como os registros estão distribuídos por faixa etária?

A distribuição encontrada foi:

- Até 39 anos: 8.626 registros;
- 40 a 49 anos: 30.922;
- 50 a 59 anos: 38.890;
- 60 a 69 anos: 32.762;
- 70 a 79 anos: 16.912;
- 80 anos ou mais: 6.962.

A maior frequência foi observada entre **50 e 59 anos**, e houve concentração importante dos registros entre 40 e 69 anos.

### 3.3 Como os registros estão distribuídos segundo o estadiamento?

Foram encontrados:

- Estádio 0: 2.167 registros;
- Estádio 1: 25.601;
- Estádio 2: 47.332;
- Estádio 3: 45.204;
- Estádio 4: 14.811.

Os estádios **2 e 3** concentraram as maiores frequências.

### 3.4 Qual a distribuição segundo raça/cor?

A categoria parda apresentou a maior frequência, com **104.113 registros (77,06%)**, seguida pelas categorias preta, branca, amarela e indígena.

Os resultados representam a distribuição da variável raça/cor nos registros administrativos analisados e não devem ser generalizados diretamente para todas as pessoas com câncer de mama na Bahia.

### 3.5 Como a continuidade do tratamento está distribuída segundo o estadiamento?

Em todos os estadiamentos, a categoria de continuidade `S` apresentou maior frequência que a categoria `N`.

Entretanto, por se tratar de uma variável administrativa da APAC, esse resultado não deve ser interpretado diretamente como medida de adesão ou abandono do tratamento.

---

## 4. Modelagem Preditiva

Foi desenvolvido um problema de classificação utilizando como variável-alvo `AQ_CONTTR`, referente à continuidade do tratamento registrada na APAC.

A variável foi transformada em:

- `1` = Sim;
- `0` = Não.

Após o tratamento da idade, foram utilizados **135.074 registros** para a modelagem.

A base foi dividida aproximadamente em:

- **80% para treinamento:** 107.973 registros;
- **20% para teste:** 27.101 registros.

Foram utilizados dois algoritmos de famílias diferentes: **Regressão Logística** e **Random Forest**, sendo o Random Forest um método ensemble.

### 4.1 Regressão Logística

A Regressão Logística apresentou:

- **Acurácia:** 70,48%;
- **F1:** 0,6085;
- **AUC:** 0,7066.

Como aproximadamente 70% dos registros pertenciam à classe `S`, a acurácia foi interpretada com cautela e analisada juntamente com F1 e AUC.

### 4.2 Random Forest

A Random Forest foi configurada com 50 árvores e apresentou:

- **Acurácia:** 70,76%;
- **F1:** 0,5864;
- **AUC:** 0,7184.

### 4.3 Comparação dos modelos

Os dois algoritmos apresentaram resultados próximos.

A Random Forest apresentou maior acurácia e AUC, enquanto a Regressão Logística apresentou maior F1. Portanto, as métricas utilizadas não demonstraram superioridade uniforme de um único algoritmo em todos os critérios avaliados.

Os resultados indicam alguma capacidade das variáveis utilizadas de discriminar as categorias da variável administrativa de continuidade do tratamento, embora o desempenho observado seja moderado.

---

## 5. Modelagem Descritiva — K-Means

Para identificar agrupamentos de registros com características semelhantes, foi aplicado o algoritmo K-Means.

Foram utilizadas características relacionadas à idade, estadiamento e informação sobre tratamento anterior.

Para justificar o número de clusters, foram testados três valores de K:

- **K = 2:** Silhouette = 0,7346;
- **K = 3:** Silhouette = 0,6908;
- **K = 4:** Silhouette = 0,6594.

Entre as alternativas avaliadas, **K = 2 apresentou o maior valor de Silhouette**, sendo selecionado para o modelo final.

### 5.1 Caracterização dos clusters

O modelo final distribuiu os 135.074 registros em:

- **Cluster 0:** 56.636 registros;
- **Cluster 1:** 78.438 registros.

O Cluster 0 apresentou idade média de aproximadamente **69,6 anos** e estadiamento médio de **2,24**.

O Cluster 1 apresentou idade média de aproximadamente **48,7 anos** e estadiamento médio de **2,40**.

A principal diferença observada entre os agrupamentos foi a idade. O estadiamento médio e a distribuição da informação sobre tratamento anterior apresentaram diferenças menores.

Os clusters representam agrupamentos estatísticos e não devem ser interpretados como categorias clínicas ou prognósticas.

---

## 6. Descoberta de Conhecimento — KDD

A aplicação das diferentes etapas permitiu identificar características relevantes nos registros de APAC de quimioterapia relacionados ao câncer de mama na Bahia em 2025.

Entre os principais achados destacam-se:

- predominância dos registros classificados como sexo feminino;
- maior concentração dos registros entre 40 e 69 anos;
- maior frequência dos estádios 2 e 3;
- predominância da categoria parda na variável raça/cor;
- maior frequência da categoria de continuidade `S`;
- desempenho moderado dos modelos preditivos;
- identificação de dois agrupamentos estatísticos diferenciados principalmente pela idade.

Do ponto de vista da análise de dados, os resultados demonstram como ferramentas de Big Data e Machine Learning podem ser utilizadas para explorar grandes conjuntos de registros administrativos de saúde e identificar padrões presentes nesses dados.

---

## 7. Limitações

Este estudo apresenta limitações importantes.

Os dados correspondem a **registros administrativos de APAC**, não sendo possível interpretar o número de registros como número de pacientes.

O conjunto analisado também não representa todos os casos de câncer de mama diagnosticados na Bahia, pois corresponde ao recorte de registros de APAC de quimioterapia do SUS utilizado no estudo.

A variável `AQ_CONTTR` possui natureza administrativa e sua categoria `N` não deve ser interpretada diretamente como abandono do tratamento.

Na análise de clustering, as variáveis utilizadas apresentavam escalas diferentes e não foram previamente padronizadas. Consequentemente, a idade pode ter exercido maior influência na formação dos agrupamentos.

Também existem limitações relacionadas à qualidade e ao preenchimento das informações presentes em bases administrativas.

---

## 8. Conclusão e próximos passos

O projeto permitiu aplicar um fluxo de descoberta de conhecimento utilizando dados públicos do DATASUS e processamento com PySpark.

Foram realizadas as etapas de ingestão, pré-processamento, análise exploratória com Spark SQL, modelagem preditiva com dois algoritmos, comparação de métricas e modelagem descritiva com K-Means.

O uso do PySpark possibilitou processar mais de 100 mil registros, enquanto Spark SQL permitiu responder questões exploratórias relevantes. A Regressão Logística e a Random Forest possibilitaram avaliar a classificação da variável administrativa de continuidade do tratamento, e o K-Means permitiu identificar agrupamentos estatísticos nos registros.

Como trabalhos futuros, recomenda-se avaliar outras variáveis disponíveis no SIA/SUS, padronizar as características utilizadas no clustering, explorar estratégias para lidar com o desequilíbrio entre classes e considerar a integração com outras fontes de dados do DATASUS.

Os resultados devem ser compreendidos como uma análise exploratória e acadêmica dos registros administrativos estudados, respeitando as limitações da fonte de dados e do desenho metodológico adotado.
