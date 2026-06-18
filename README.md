#  Análise de Precificação em Planos de Saúde (ANS) com PySpark

##  Visão Geral
Este projeto analisa os Valores Comerciais de Mensalidades (VCM) praticados pelas operadoras de planos de saúde no Brasil entre 2004 e 2026. O objetivo principal é investigar **como a faixa etária influencia o valor das mensalidades ao longo do tempo**, validando os limites regulatórios impostos pela Lei nº 9.656/98.

##  Impacto do Projeto 
> **"Analisei a influência da faixa etária na precificação de planos de saúde brasileiros, processando de forma distribuída mais de 3,5 milhões de registros históricos cobrindo 22 anos de atuação da Agência Nacional de Saúde Suplementar, construindo um pipeline completo de Big Data no Google Colab com PySpark contemplando ETL, Análise Exploratória (Spark SQL) e Machine Learning."**

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
* **Curva de Precificação e Limite Legal:** Comprovou-se quantitativamente que a precificação segue rigorosamente o risco assistencial. A faixa de "59 anos ou mais" apresenta mensalidade média muito próxima de 6 vezes o valor da faixa "0 a 18 anos", demonstrando que as operadoras utilizam o limite máximo de variação permitido pela Lei nº 9.656/98 para repassar custos.
* **Evolução Temporal e Inflação Médica:** A série histórica de 22 anos (2004–2026) evidencia um crescimento acelerado dos Valores Comerciais (VCM), com quebras de tendência (aumentos mais acentuados) nos períodos pós-pandemia, tracionados pelo aumento exponencial da despesa assistencial.
* **Agrupamentos de Mercado (K-Means):** A clusterização não-supervisionada dividiu o mercado em perfis claros: o cluster de "Acesso Popular" (alto volume, custo contido) e o cluster "Premium" (mensalidades atípicas voltadas à alta renda, com despesas assistenciais descoladas da média do mercado).
* *Padrões Ocultos (Apriori):** O algoritmo de regras de associação revelou uma fortíssima relação estatística (alto lift e confidence) entre mensalidades menores e planos de abrangência "Regionalizada", validando a estratégia das operadoras de restringir a rede de atendimento para baratear o plano.

##  Como Executar
1. Faça um clone deste repositório:
   ```bash
   git clone https://github.com/SEU-USUARIO/ans-big-data-pyspark.git
   ```
2. Abra o arquivo `Projeto_Big_Data_ANS.ipynb` no **Google Colab**.
3. Adicione o arquivo de dados original e o dicionário de dados na mesma estrutura do seu Google Drive.
4. Execute as células em sequência.

## 📚 Fontes e Referências
1. **Apache Spark:** [Documentação Oficial](https://spark.apache.org/docs/latest/)
2. **PySpark:** [API de Referência](https://spark.apache.org/docs/latest/api/python/)
3. **Scikit-Learn:** [Modelos de Machine Learning](https://scikit-learn.org/stable/documentation.html)
4. **Mlxtend:** [Mineração de Regras de Associação](http://rasbt.github.io/mlxtend/)
5. **Pandas:** [Estruturas de Dados e Análise](https://pandas.pydata.org/docs/)
6. **NumPy:** [Computação Científica](https://numpy.org/doc/stable/)
7. **Matplotlib:** [Visualização de Dados](https://matplotlib.org/stable/users/index.html)
8. **Seaborn:** [Visualização Estatística de Dados](https://seaborn.pydata.org/)
9. **WordCloud:** [Geração de Nuvens de Palavras](https://amueller.github.io/word_cloud/)
10. **ANS (Agência Nacional de Saúde Suplementar):** Dicionário de Dados da ANS (`dicionario_valor_comercial.pdf`)
11. **Legislação Brasileira:** Lei nº 9.656 de 3 de junho de 1998 (Regulação de Planos de Saúde e reajuste por faixa etária)

##  Autor
Feito por **Marcos Davi**
- [LinkedIn](https://linkedin.com/in/marcosdavidev/)
