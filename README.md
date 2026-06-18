#  Análise de Precificação em Planos de Saúde (ANS) com PySpark

##  Visão Geral
Este projeto analisa os Valores Comerciais de Mensalidades (VCM) praticados pelas operadoras de planos de saúde no Brasil entre 2004 e 2026. O objetivo principal é investigar **como a faixa etária influencia o valor das mensalidades ao longo do tempo**, validando os limites regulatórios impostos pela Lei nº 9.656/98.

##  Impacto do Projeto (Metodologia XYZ)
> **"Analisei a influência da faixa etária na precificação de planos de saúde brasileiros (X), processando de forma distribuída mais de 3,5 milhões de registros históricos cobrindo 22 anos de atuação da Agência Nacional de Saúde Suplementar (Y), construindo um pipeline completo de Big Data no Google Colab com PySpark contemplando ETL, Análise Exploratória (Spark SQL) e Machine Learning (Z)."**

##  Os Dados
- **Fonte:** Dados Abertos Governamentais da ANS (Agência Nacional de Saúde Suplementar).
- **Volume:** ~3.500.000 registros distribuídos em 23 arquivos anuais.
- **Tamanho:** ~400 MB (descomprimido).
- **Características:** Base regulatória oficial contendo os preços homologados para comercialização, segmentados por faixas etárias e abrangência.

##  Tecnologias Utilizadas
- **Processamento Distribuído:** Apache Spark (PySpark), Spark SQL
- **Linguagem:** Python 3
- **Análise e Visualização:** Pandas, Matplotlib, Seaborn, WordCloud
- **Machine Learning:** Scikit-Learn (K-Means), Mlxtend (Apriori)
- **Ambiente:** Google Colab / Jupyter Notebook

##  Arquitetura e Pipeline de Dados
O projeto segue as melhores práticas de Engenharia de Dados em um pipeline de 4 fases:

1. **Ingestão (ETL - Extract & Load):** Extração de arquivos `.zip` em lotes e consolidação de dataframes particionados preservando a origem temporal (`ano_origem`).
2. **Transformação e Limpeza:** Tratamento de nulos, tipagem estrita via *schema enforcement*, e eliminação de inconsistências monetárias (VCM_MINIMO ≤ Mensalidade ≤ VCM_MAXIMO).
3. **Análise Exploratória (EDA):** Análise estatística univariada/bivariada e consultas distribuídas avançadas via `Spark SQL`.
4. **Mineração e Modelagem (ML):**
   - **K-Means:** Clusterização não-supervisionada para encontrar perfis intrínsecos de operadoras e precificação.
   - **Apriori:** Geração de regras de associação para descobrir padrões não evidentes (Lift e Confidence) no mercado suplementar.

##  Principais Insights
* *(Adicione aqui seu principal achado do projeto, ex: "Constatou-se que planos para a faixa de 59+ anos chegam consistentemente ao limite regulatório de 6x o valor de faixas jovens nos últimos 5 anos.")*
* *(Adicione um insight dos modelos, ex: "O algoritmo de clusterização revelou um grupo isolado de operadoras premium, focadas em abrangência nacional com tickets médios descolados do mercado.")*

##  Como Executar
1. Faça um clone deste repositório:
   ```bash
   git clone https://github.com/SEU-USUARIO/ans-big-data-pyspark.git
   ```
2. Abra o arquivo `Projeto_Big_Data_ANS.ipynb` no **Google Colab**.
3. Adicione o arquivo de dados original e o dicionário de dados na mesma estrutura do seu Google Drive.
4. Execute as células em sequência.

##  Autor
Feito por **Marcos Davi**
- [LinkedIn](https://linkedin.com/in/marcosdavidev/)
