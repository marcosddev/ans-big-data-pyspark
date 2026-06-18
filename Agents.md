# PROMPT TÉCNICO COMPLETO — PROJETO BIG DATA ANS (GOOGLE COLAB)

---

## SEU PAPEL

Você é um engenheiro de dados sênior encarregado de implementar um projeto acadêmico completo em Google Colab. Seu trabalho é escrever código real, funcional e comentado — não pseudocódigo, não explicações teóricas soltas. Cada célula deve ser autossuficiente, executável em sequência e bem documentada com comentários em português.

**Padrão de qualidade exigido:** cada célula deve começar com um bloco de comentários em Markdown (`%%markdown` ou célula de texto) explicando o objetivo da etapa, e o código deve ser limpo, com nomes de variáveis descritivos e comentários inline nas partes críticas.

---

## AMBIENTE TÉCNICO OBRIGATÓRIO

- **Plataforma:** Google Colab
- **Linguagem:** Python 3
- **Processamento distribuído:** PySpark (SparkSession local)
- **Bibliotecas adicionais:** Pandas, Matplotlib, Seaborn, WordCloud, Mlxtend, Scikit-Learn
- **Armazenamento:** Google Drive (montado via `drive.mount`)
- **Saída de dados:** `.csv` e `.xlsx` no Google Drive

Sempre instale pacotes que não vêm por padrão no Colab com `!pip install` no início da célula correspondente.

---

## DATASET PRINCIPAL

### Arquivo ZIP
**Nome:** `nota_tecnica_vcm_faixa_etaria.zip`
**Localização:** Google Drive (path a ser definido pelo usuário)

Contém **23 arquivos CSV anuais**, um por ano, de **2004 a 2026**:
```
NOTA_TECNICA_VCM_FAIXA_ETARIA_2004.csv
NOTA_TECNICA_VCM_FAIXA_ETARIA_2005.csv
...
NOTA_TECNICA_VCM_FAIXA_ETARIA_2026.csv
```

**Volume total aproximado:** 3,5 milhões de linhas  
**Separador:** `;`  
**Encoding:** tentar `latin1` primeiro, depois `utf-8` e `windows-1252`

### Colunas (schema idêntico em todos os CSVs)

| Coluna | Tipo esperado | Descrição |
|---|---|---|
| `CD_OPERADORA` | string/int | Código da operadora |
| `ID_PLANO` | string/int | Identificador do plano |
| `CD_NOTA` | string | Código da nota técnica |
| `DT_NTRP` | date | Data de início de vigência da nota |
| `ID_ABRG` | string | Tipo de abrangência do plano |
| `FAIXA_ETARIA` | string | Faixa etária (10 categorias) |
| `VL_COMERCIAL_MENSALIDADE` | float | Valor comercial da mensalidade |
| `VL_DESP_ASSISTENCIAL` | float | Valor da despesa assistencial |
| `VCM_MINIMO` | float | Valor mínimo comercial permitido |
| `VCM_MAXIMO` | float | Valor máximo comercial permitido |
| `DT_ATUALIZACAO` | date | Data de atualização do registro |

### Faixas etárias presentes
```
00 a 18 anos | 19 a 23 anos | 24 a 28 anos | 29 a 33 anos | 34 a 38 anos
39 a 43 anos | 44 a 48 anos | 49 a 53 anos | 54 a 58 anos | 59 anos ou mais
```

### Arquivo de apoio
**Nome:** `dicionario_valor_comercial.pdf`  
Use como referência conceitual para interpretar os campos. Cite-o nas células de texto/Markdown como "Dicionário de Dados da ANS" ao explicar as variáveis. O PDF não é dado — é documentação semântica.

---

## PERGUNTA CENTRAL DO PROJETO

> **"Como a faixa etária influencia o valor das mensalidades dos planos de saúde ao longo do tempo?"**

**Toda análise, gráfico, consulta SQL e modelo de ML deve estar conectado a essa pergunta.** O projeto não é uma coleção de gráficos — é uma narrativa analítica coerente. Cada etapa deve contribuir para responder essa pergunta central.

---

## ESTRUTURA DO NOTEBOOK — 19 CÉLULAS OBRIGATÓRIAS

O professor exige células separadas. Não concentre tudo em uma célula. A seguir está a especificação exata de cada célula.

---

### CÉLULA 1 — Imports, SparkSession e Montagem do Drive

**Objetivo:** preparar todo o ambiente de execução.

Implemente:
1. `drive.mount('/content/drive')` com `force_remount=True`
2. Definição da variável `BASE_PATH` apontando para a pasta do Drive onde estão os arquivos (deixe como variável configurável no topo da célula, com instrução ao usuário)
3. Importação de todos os módulos necessários:
   - `pyspark`, `SparkSession`, `functions as F`, `types`
   - `pandas`, `numpy`
   - `matplotlib.pyplot`, `seaborn`
   - `wordcloud`
   - `mlxtend.frequent_patterns`, `mlxtend.preprocessing`
   - `sklearn.preprocessing`, `sklearn.cluster`
   - `zipfile`, `os`, `re`, `io`
4. Criação da `SparkSession` com configurações adequadas para Colab (memória, serialização)
5. Print de confirmação com versão do Spark

**Comentário de abertura (célula Markdown antes do código):**  
Explique que esta célula inicializa o ambiente distribuído com PySpark no Google Colab, monta o Drive e importa todas as dependências do projeto.

---

### CÉLULA 2 — Leitura e Consolidação do ZIP (ETL de Ingestão)

**Objetivo:** extrair o ZIP, ler todos os CSVs anuais e consolidar em um único DataFrame Spark.

Implemente:
1. Extração do ZIP para um diretório temporário (`/tmp/ans_data/`)
2. Listagem de todos os arquivos `.csv` extraídos ordenados por ano
3. Loop de leitura de cada CSV com:
   - Tentativa de separador `;`
   - Tentativa de encodings: `latin1` → `utf-8` → `windows-1252`
   - Extração do ano do nome do arquivo via regex: `re.search(r'(\d{4})', filename)`
   - Adição da coluna `ano_origem` com o ano extraído
   - Padronização dos nomes de colunas (`.strip()`, `.upper()`)
4. Union de todos os DataFrames parciais em `df_spark`
5. Print do schema e contagem total de registros
6. Exibição das primeiras 5 linhas com `.show(5, truncate=False)`

**Tratamento de erro:** se um CSV falhar, registre o erro e continue para o próximo — não interrompa o loop.

**Comentário de abertura:**  
Explique a estratégia de leitura longitudinal: o ZIP contém 23 arquivos anuais (2004–2026) que representam uma única base histórica. A coluna `ano_origem` é adicionada para preservar a referência temporal de cada registro.

---

### CÉLULA 3 — Visualização Inicial do Dataset

**Objetivo:** fornecer uma visão geral da base consolidada.

Implemente:
1. Exibição formatada de:
   - Total de registros (`df_spark.count()`)
   - Total de colunas
   - Lista de colunas com tipo de dado
   - Range de `ano_origem` (mínimo e máximo)
   - Distribuição de registros por ano (`.groupBy('ano_origem').count().orderBy('ano_origem')`)
2. Exibição estilizada de amostra de 10 linhas usando Pandas (`.limit(10).toPandas()`) com `display()` ou `style`
3. Verificação prévia de nulos por coluna (`.isnull().sum()` via Pandas ou `F.count` via Spark)

**Limite de exibição:** nunca faça `.toPandas()` sem `.limit()` — a base tem 3,5M linhas.

---

### CÉLULA 4 — Renomeação e Padronização de Colunas

**Objetivo:** padronizar os nomes das colunas para uso em SQL e análises.

Implemente mapeamento explícito:
```python
rename_map = {
    'CD_OPERADORA': 'cd_operadora',
    'ID_PLANO': 'id_plano',
    'CD_NOTA': 'cd_nota',
    'DT_NTRP': 'dt_ntrp',
    'ID_ABRG': 'id_abrg',
    'FAIXA_ETARIA': 'faixa_etaria',
    'VL_COMERCIAL_MENSALIDADE': 'vl_mensalidade',
    'VL_DESP_ASSISTENCIAL': 'vl_desp_assistencial',
    'VCM_MINIMO': 'vcm_minimo',
    'VCM_MAXIMO': 'vcm_maximo',
    'DT_ATUALIZACAO': 'dt_atualizacao',
    'ano_origem': 'ano_origem'
}
```

Use `.withColumnRenamed()` em loop ou `reduce`. Após renomear, exiba o schema atualizado.

---

### CÉLULA 5 — Limpeza Real dos Dados

**Objetivo:** garantir consistência e integridade da base antes das análises.

Implemente sequencialmente:

1. **Contagem inicial** → salve em `registros_antes`
2. **Conversão de tipos:**
   - `vl_mensalidade`, `vl_desp_assistencial`, `vcm_minimo`, `vcm_maximo` → `DoubleType()`
   - `dt_ntrp`, `dt_atualizacao` → `DateType()` via `F.to_date()` (tentar formatos `yyyy-MM-dd` e `dd/MM/yyyy`)
   - `cd_operadora`, `id_plano` → string
3. **Remoção de duplicados:** `.dropDuplicates()`
4. **Tratamento de nulos:**
   - Registros com `vl_mensalidade` nula ou ≤ 0 → remover
   - Registros com `faixa_etaria` nula → remover
5. **Verificação de consistência monetária:**
   ```python
   # Identifica registros onde vcm_minimo > vl_mensalidade OU vl_mensalidade > vcm_maximo
   df_inconsistentes = df_spark.filter(
       (F.col('vcm_minimo') > F.col('vl_mensalidade')) |
       (F.col('vl_mensalidade') > F.col('vcm_maximo'))
   )
   ```
   Registre a quantidade, mas **não remova** — apenas sinalize.
6. **Padronização da coluna `faixa_etaria`:** trim e upper para garantir consistência no groupBy

---

### CÉLULA 6 — Relatório de Qualidade dos Dados

**Objetivo:** documentar formalmente o processo de limpeza.

Gere um relatório estruturado (pode ser via `print` ou DataFrame Pandas estilizado) contendo:

| Métrica | Valor |
|---|---|
| Registros antes da limpeza | N |
| Registros após a limpeza | N |
| Registros removidos | N |
| Percentual removido | N% |
| Duplicados encontrados | N |
| Nulos em vl_mensalidade | N |
| Nulos em faixa_etaria | N |
| Inconsistências min/max | N |
| Anos cobertos | 2004–2026 |
| Faixas etárias únicas | N |

**Comentário de abertura:**  
Explique que a base da ANS passa por validação de integridade seguindo boas práticas de engenharia de dados: os dados de planos de saúde regulados exigem consistência monetária (VCM mínimo ≤ mensalidade ≤ VCM máximo).

---

### CÉLULA 7 — Estatísticas Descritivas

**Objetivo:** compreender a distribuição estatística das variáveis monetárias.

Implemente:
1. `.describe()` via Spark para `vl_mensalidade`, `vl_desp_assistencial`, `vcm_minimo`, `vcm_maximo`
2. Cálculo explícito de **quartis** (Q1, mediana/Q2, Q3) usando `approxQuantile()`:
   ```python
   quantiles = df_spark.approxQuantile('vl_mensalidade', [0.25, 0.5, 0.75], 0.01)
   ```
3. Tabela comparativa entre as 4 variáveis monetárias (média, mediana, desvio padrão, mín, máx)
4. **Cálculo adicional:** razão média entre `vl_mensalidade` de "59 anos ou mais" vs "00 a 18 anos" — isso é a evidência quantitativa central do projeto

**Comentário de abertura:**  
Contextualize que a Lei 9.656/98 permite que operadoras cobrem até 6x mais para a faixa de 59+ em relação à faixa 0-18. As estatísticas devem revelar se essa prática se reflete nos dados reais da ANS.

---

### CÉLULA 8 — Group By Principal e Spark SQL

**Objetivo:** análise central — variação de mensalidade por faixa etária.

**Parte A — Group By PySpark:**
```python
df_por_faixa = df_spark.groupBy('faixa_etaria').agg(
    F.mean('vl_mensalidade').alias('media_mensalidade'),
    F.min('vl_mensalidade').alias('min_mensalidade'),
    F.max('vl_mensalidade').alias('max_mensalidade'),
    F.stddev('vl_mensalidade').alias('desvio_mensalidade'),
    F.count('*').alias('qtd_registros')
).orderBy('faixa_etaria')
```
Exiba o resultado.

**Parte B — Spark SQL:**
Crie uma view e execute pelo menos **5 consultas SQL distintas**:
```sql
-- 1. Ranking de faixas etárias por mensalidade média
SELECT faixa_etaria, ROUND(AVG(vl_mensalidade), 2) as media
FROM vw_ans GROUP BY faixa_etaria ORDER BY media DESC

-- 2. Top 10 operadoras com maior mensalidade média (apenas faixa 59+)
SELECT cd_operadora, ROUND(AVG(vl_mensalidade), 2) as media_senior
FROM vw_ans WHERE faixa_etaria = '59 anos ou mais'
GROUP BY cd_operadora ORDER BY media_senior DESC LIMIT 10

-- 3. Evolução anual da mensalidade média
SELECT ano_origem, ROUND(AVG(vl_mensalidade), 2) as media_anual
FROM vw_ans GROUP BY ano_origem ORDER BY ano_origem

-- 4. Quantidade de planos únicos por faixa etária
SELECT faixa_etaria, COUNT(DISTINCT id_plano) as planos_unicos
FROM vw_ans GROUP BY faixa_etaria ORDER BY faixa_etaria

-- 5. Relação despesa assistencial vs mensalidade por faixa
SELECT faixa_etaria,
  ROUND(AVG(vl_mensalidade), 2) as media_mensalidade,
  ROUND(AVG(vl_desp_assistencial), 2) as media_despesa,
  ROUND(AVG(vl_desp_assistencial) / AVG(vl_mensalidade) * 100, 1) as pct_despesa
FROM vw_ans GROUP BY faixa_etaria ORDER BY faixa_etaria
```

**Comentário de abertura:**  
Explique que o Group By é a técnica central de agregação distribuída do Spark. As consultas SQL são executadas via Spark SQL, que compila as queries em um plano de execução distribuído equivalente ao Map-Reduce.

---

### CÉLULA 9 — Histograma de Distribuição das Mensalidades

**Objetivo:** mostrar como os preços estão distribuídos na base.

Implemente com Matplotlib/Seaborn:
1. Converter amostra para Pandas: `.sample(fraction=0.05, seed=42).toPandas()` (5% = ~175k linhas, suficiente)
2. Histograma de `vl_mensalidade` com:
   - `bins=50`
   - Escala X limitada ao percentil 99 (excluir outliers extremos da visualização)
   - Linha vertical na média e na mediana com legenda
   - Título: "Distribuição das Mensalidades dos Planos de Saúde (ANS)"
   - Eixos rotulados em português
   - Paleta neutra (azul)

**Análise textual obrigatória:** após o gráfico, imprima 3–5 linhas interpretando a forma da distribuição (assimetria positiva esperada, onde se concentra a maioria dos planos).

---

### CÉLULA 10 — Gráfico de Densidade (KDE)

**Objetivo:** visualizar a distribuição probabilística por faixa etária.

Implemente:
1. Uma curva KDE separada para cada faixa etária (ou pelo menos para 3 faixas representativas: "00 a 18 anos", "34 a 38 anos", "59 anos ou mais")
2. Use `seaborn.kdeplot()` com `fill=True, alpha=0.3`
3. Legendas claras por faixa
4. Título: "Densidade de Probabilidade — Mensalidade por Faixa Etária"

**Análise textual:** comente o deslocamento das curvas à direita para faixas mais velhas — isso é a evidência visual da relação faixa etária × preço.

---

### CÉLULA 11 — Gráfico de Pizza

**Objetivo:** mostrar a composição percentual por tipo de abrangência (`id_abrg`).

Implemente:
1. GroupBy em `id_abrg` com contagem
2. Pie chart com:
   - Labels descritivos (se `id_abrg` for código, crie um dicionário de mapeamento: ex. `{'N': 'Nacional', 'G': 'Grupo de Municípios', ...}`)
   - Percentuais exibidos nas fatias (`autopct='%1.1f%%'`)
   - Fatias menores que 2% agrupadas em "Outros"
   - Tamanho: `figsize=(9, 9)`
   - `startangle=90`

**Análise textual:** interprete o que a distribuição de abrangência revela sobre o mercado de planos de saúde brasileiro.

---

### CÉLULA 12 — Gráfico de Barras: Mensalidade Média por Faixa Etária

**Esta é a visualização central do projeto.**

Implemente:
1. Use o resultado do GroupBy da Célula 8 (`df_por_faixa`)
2. Barras horizontais ordenadas por faixa etária (da mais jovem à mais velha)
3. Cada barra deve exibir o valor médio formatado como `R$ XXX,XX`
4. Linha de referência horizontal na média geral
5. Coloração gradiente (mais escuro = mais caro)
6. Título: "Mensalidade Média por Faixa Etária — Planos de Saúde (ANS, 2004–2026)"
7. `figsize=(12, 8)`, fonte mínima 12

**Análise textual obrigatória:** calcule e exiba explicitamente a razão entre a mensalidade média de "59 anos ou mais" e "00 a 18 anos". Compare com o limite legal de 6x da Lei 9.656/98.

---

### CÉLULA 13 — Gráfico de Evolução Temporal

**Objetivo:** analisar como as mensalidades evoluíram de 2004 a 2026.

Implemente:
1. Use a consulta SQL da evolução anual (Célula 8) como base
2. Gráfico de linha com:
   - Eixo X: `ano_origem`
   - Eixo Y: mensalidade média
   - Marcadores em cada ponto (`marker='o'`)
   - Anotação nos pontos de maior variação ano a ano
   - Grade (`grid=True`)
3. **Versão adicional:** linhas separadas para 3 faixas etárias ("00 a 18", "34 a 38", "59+") no mesmo gráfico — isso mostra a divergência temporal entre faixas

**Análise textual:** identifique anos com maiores altas e relacione com contexto econômico brasileiro quando possível (ex: 2015–2016 crise, 2020–2021 COVID).

---

### CÉLULA 14 — Análise de Outliers (IQR)

**Objetivo:** identificar valores extremos de mensalidade.

Implemente:
1. Cálculo via `approxQuantile()`:
   ```python
   Q1, Q3 = df_spark.approxQuantile('vl_mensalidade', [0.25, 0.75], 0.01)
   IQR = Q3 - Q1
   limite_inferior = Q1 - 1.5 * IQR
   limite_superior = Q3 + 1.5 * IQR
   ```
2. Contagem e percentual de outliers
3. Boxplot (via Pandas em amostra) com destaque dos outliers
4. Tabela com exemplos dos 10 maiores outliers (operadora, faixa etária, valor)
5. **Análise por faixa etária:** verifique se outliers estão concentrados em faixas específicas

**Exiba explicitamente as fórmulas e valores calculados** (Q1, Q3, IQR, limites) — o professor avalia se o aluno entende o método.

---

### CÉLULA 15 — Nuvem de Palavras

**Objetivo:** visualizar os termos textuais predominantes.

Como a base ANS tem poucas colunas textuais, implemente da seguinte forma:

1. **Opção preferida:** use `faixa_etaria` + `id_abrg` + `cd_nota` para criar texto composto
2. Concatene os valores como texto, replique cada registro pela sua mensalidade média (frequência ponderada)
3. Limpe stopwords (números isolados, caracteres especiais)
4. Gere WordCloud com:
   - `background_color='white'`
   - `colormap='Blues'`
   - `max_words=80`
   - `width=1200, height=600`

**Comentário de abertura:** explique que a nuvem de palavras é uma técnica de análise de frequência textual. Neste contexto, ela revela as categorias mais frequentes na base ANS.

---

### CÉLULA 16 — Mineração de Dados (Apriori + Regras de Associação)

**Objetivo:** descobrir padrões de associação entre categorias de dados.

Como a base ANS é numérica/categórica (não transacional), aplique a seguinte estratégia de discretização:

1. **Prepare o dataset transacional:**
   - Crie colunas binárias para: faixa etária (10 colunas), tipo de abrangência (`id_abrg`, N colunas), e faixa de preço (discretize `vl_mensalidade` em: "barato", "moderado", "caro", "muito_caro" por quartis)
   - Use amostra de 50.000 registros (`.sample(fraction=0.015, seed=42).toPandas()`)
   - Encode como matriz binária (0/1) usando `pd.get_dummies()` ou `TransactionEncoder`

2. **Apriori:**
   ```python
   from mlxtend.frequent_patterns import apriori, association_rules
   frequent_itemsets = apriori(df_transacional, min_support=0.05, use_colnames=True)
   rules = association_rules(frequent_itemsets, metric='confidence', min_threshold=0.5)
   rules = rules.sort_values('lift', ascending=False)
   ```

3. **Exiba:**
   - Top 10 itemsets frequentes com suporte
   - Top 10 regras por lift com colunas: antecedente, consequente, suporte, confiança, lift
   - Interpretação de pelo menos 3 regras em linguagem natural

4. **Explique as métricas:**
   - Suporte: frequência do padrão na base
   - Confiança: P(consequente | antecedente)
   - Lift: quanto a regra supera a aleatoriedade (lift > 1 = associação positiva)

**Comentário de abertura:** explique o algoritmo Apriori como técnica de mineração de dados baseada no princípio da anti-monotonicidade, e relacione com o conteúdo da disciplina.

---

### CÉLULA 17 — Inteligência Artificial: Clusterização K-Means

**Objetivo:** agrupar perfis de planos com comportamento semelhante.

Implemente o pipeline completo:

1. **Seleção de features:**
   ```python
   features = ['vl_mensalidade', 'vl_desp_assistencial', 'vcm_minimo', 'vcm_maximo', 'ano_origem']
   ```

2. **Preparação dos dados:**
   - Amostra: `df_spark.sample(fraction=0.03, seed=42).toPandas()` (~105k linhas)
   - Remover nulos nas features selecionadas
   - Padronizar com `StandardScaler()`

3. **Escolha do K — Método do Cotovelo:**
   ```python
   inertias = []
   K_range = range(2, 11)
   for k in K_range:
       km = KMeans(n_clusters=k, random_state=42, n_init=10)
       km.fit(X_scaled)
       inertias.append(km.inertia_)
   # Plote o gráfico do cotovelo
   ```

4. **Aplicação do K-Means** com o K escolhido (provavelmente 3 ou 4 para este dataset)

5. **Visualização dos clusters:**
   - Scatter plot 2D (redução via PCA para 2 componentes)
   - Coloração por cluster
   - Centros dos clusters marcados

6. **Interpretação:** calcule e exiba a média de `vl_mensalidade` e `faixa_etaria` mais frequente por cluster. Explique o que cada cluster representa (ex: "Cluster 0: planos baratos para jovens", etc.)

**Comentário de abertura:** contextualize K-Means como aprendizado não supervisionado. Explique que o objetivo é descobrir agrupamentos naturais nos dados sem rótulos pré-definidos, e relacione com a pergunta central do projeto.

---

### CÉLULA 18 — Exportação de Dados

**Objetivo:** exportar os resultados para compartilhamento.

Implemente:

1. **Dataset consolidado tratado** → CSV no Drive:
   ```python
   df_spark.coalesce(1).write.option('header', True).option('sep', ';').mode('overwrite').csv(path_output_csv)
   ```

2. **Principais resultados** → Excel com múltiplas abas:
   - Aba 1: Estatísticas descritivas por faixa etária
   - Aba 2: Evolução temporal (média anual)
   - Aba 3: Outliers identificados
   - Aba 4: Regras de associação (top 20)
   - Aba 5: Perfil dos clusters K-Means

   Use `pandas.ExcelWriter` com `engine='openpyxl'`

3. Exiba mensagem de confirmação com os caminhos dos arquivos gerados.

---

### CÉLULA 19 — Conclusão Final

**Objetivo:** sintetizar toda a narrativa analítica do projeto.

Escreva (como célula de texto Markdown + prints formatados) uma conclusão estruturada que responda explicitamente:

1. **Qualidade dos dados:** a base ANS estava em bom estado? Quantos registros foram perdidos na limpeza? Havia inconsistências?

2. **Distribuição das mensalidades:** como os preços estão distribuídos? Há assimetria? Onde se concentra a maioria?

3. **Impacto da faixa etária:** qual a diferença percentual entre a faixa mais barata e a mais cara? A razão observada é superior ou inferior ao limite de 6x da Lei 9.656/98?

4. **Evolução temporal:** as mensalidades subiram ao longo de 2004–2026? Em quais períodos houve maior crescimento?

5. **Outliers:** os valores extremos se concentram em faixas etárias específicas ou operadoras específicas?

6. **Mineração de dados:** quais as 3 associações mais relevantes descobertas pelo Apriori? O que elas revelam sobre o mercado?

7. **Machine Learning:** quantos clusters foram encontrados? O que diferencia cada cluster? Os agrupamentos fazem sentido analítico?

8. **Resposta à pergunta central:** conclua com um parágrafo diretamente respondendo "Como a faixa etária influencia o valor das mensalidades dos planos de saúde ao longo do tempo?"

---

## REGRAS TÉCNICAS INEGOCIÁVEIS

### Regra 1 — Nunca use `.toPandas()` sem `.limit()` ou `.sample()`
A base tem 3,5M de linhas. Converter tudo para Pandas mata o Colab. Use sempre amostragem ou limite explícito.

### Regra 2 — Sempre exiba contagens antes e depois de transformações
Toda filtragem ou limpeza deve ser acompanhada de print mostrando quantos registros existiam antes e quantos restaram.

### Regra 3 — Comentários em português
Todo comentário inline e célula Markdown deve ser em português. O código em si segue convenções Python (inglês para variáveis e funções é aceitável).

### Regra 4 — Cada célula deve ser autoexplicativa
Uma pessoa que lê a célula sem contexto deve entender o que ela faz, por que ela existe e o que ela produz — isso via a célula Markdown de abertura.

### Regra 5 — Tratar erros graciosamente
Envolva operações críticas (leitura de CSV, conversão de tipo, operações Spark) em `try/except` com mensagens de erro descritivas.

### Regra 6 — Preservar o ano de origem
A coluna `ano_origem` deve ser preservada em todos os DataFrames derivados. Ela é essencial para a análise temporal.

### Regra 7 — Gráficos com qualidade de apresentação
Todo gráfico deve ter: título descritivo, eixos rotulados em português, fonte mínima 12, tamanho mínimo `(10, 6)`, e `plt.tight_layout()`. Nenhum gráfico sem rótulos.

### Regra 8 — A narrativa deve ser coerente
Cada célula deve mencionar brevemente como ela se conecta à pergunta central. Não são análises isoladas — são etapas de uma mesma investigação.

---

## REFERÊNCIAS QUE DEVEM SER CITADAS NA CONCLUSÃO

- Documentação oficial do Apache Spark: https://spark.apache.org/docs/latest/
- Documentação do PySpark: https://spark.apache.org/docs/latest/api/python/
- Documentação do Scikit-Learn: https://scikit-learn.org/stable/documentation.html
- Documentação do Mlxtend: http://rasbt.github.io/mlxtend/
- Documentação do Pandas: https://pandas.pydata.org/docs/
- Dicionário de Dados da ANS: `dicionario_valor_comercial.pdf` (material fornecido)
- Lei 9.656/98 — Planos de Saúde (para contextualizar a faixa de 6x)
- Materiais e slides disponibilizados pelo professor

---

## CHECKLIST FINAL DO PROJETO

Antes de considerar o notebook completo, confirme que estão presentes:

- [ ] Célula 1: Imports + SparkSession + Drive
- [ ] Célula 2: Leitura ZIP + consolidação + coluna `ano_origem`
- [ ] Célula 3: Visualização inicial
- [ ] Célula 4: Renomeação de colunas
- [ ] Célula 5: Limpeza completa (duplicados, nulos, tipos, consistência)
- [ ] Célula 6: Relatório de qualidade
- [ ] Célula 7: Estatísticas descritivas + quartis + razão 59+/0-18
- [ ] Célula 8: GroupBy por faixa etária + 5 consultas Spark SQL
- [ ] Célula 9: Histograma de mensalidades
- [ ] Célula 10: Gráfico de densidade KDE
- [ ] Célula 11: Gráfico de pizza (abrangência)
- [ ] Célula 12: Gráfico de barras (faixa etária × mensalidade)
- [ ] Célula 13: Evolução temporal (linha por ano + por faixa)
- [ ] Célula 14: Outliers IQR com fórmulas explícitas
- [ ] Célula 15: Nuvem de palavras
- [ ] Célula 16: Apriori + regras de associação interpretadas
- [ ] Célula 17: K-Means + cotovelo + clusters interpretados
- [ ] Célula 18: Exportação CSV + Excel multi-abas
- [ ] Célula 19: Conclusão respondendo a pergunta central
- [ ] Referências citadas
- [ ] Todos os gráficos com título e eixos rotulados
- [ ] Nenhum `.toPandas()` sem limite/sample

---

*Este prompt define o projeto completo. Implemente célula por célula, na ordem especificada, garantindo que cada etapa seja executável de forma independente e que a narrativa do projeto seja coerente do início ao fim.*
