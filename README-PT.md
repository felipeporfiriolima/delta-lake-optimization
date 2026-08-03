# Delta Lake Advanced Engineering & Optimization

## Projeto de Engenharia de Performance utilizando Databricks e Delta Lake

## Visão Geral

Este projeto demonstra técnicas avançadas de engenharia de dados aplicadas ao Delta Lake dentro da plataforma Databricks.

O objetivo é avaliar o impacto de estratégias de otimização em tabelas Delta, analisando problemas comuns encontrados em ambientes de dados em escala, como:

- Small Files Problem
- Grande quantidade de arquivos para leitura
- Processamento desnecessário de dados
- Necessidade de otimização para consultas analíticas

Através de benchmarks comparativos antes e depois das otimizações, o projeto demonstra o impacto das funcionalidades:

- OPTIMIZE
- ZORDER
- Data Skipping
- Delta Time Travel
- Restore de versões históricas

---

# Arquitetura da Solução

```
Geração de Dados
        |
        |
        ▼
Delta Table
performance.fato_vendas
        |
        |
        ├── Benchmark Inicial
        |
        |
        ├── OPTIMIZE
        |
        |
        ├── ZORDER
        |
        |
        └── Time Travel / Restore
```

---

# Dataset

Foi criado um conjunto de dados simulando uma tabela de vendas corporativa.

Tabela Delta:

```
performance.fato_vendas
```

Principais atributos:

| Campo | Descrição |
|---|---|
| id_venda | Identificador da venda |
| data_venda | Data da venda |
| id_cliente | Cliente |
| id_produto | Produto |
| id_loja | Loja |
| quantidade | Quantidade vendida |
| valor_unitario | Valor unitário |
| valor_total | Valor total da venda |

---

# Cenário Inicial

Antes das otimizações, a tabela Delta apresentava o problema conhecido como **Small Files Problem**.

## Delta Table

```
numFiles: 200

sizeInBytes:
428.32 MB
```

Esse cenário representa uma situação comum em ambientes Lakehouse, onde cargas frequentes podem gerar muitos arquivos pequenos, aumentando o overhead de leitura e prejudicando a performance das consultas.

---

# Benchmark Inicial

## Cenário 1 - Consulta por período

Consulta:

```sql
SELECT
    COUNT(*) AS total_vendas,
    SUM(valor_total) AS faturamento
FROM performance.fato_vendas
WHERE data_venda BETWEEN '2025-01-01'
AND '2025-03-31';
```

Resultado:

```
Tempo:
3.427s

Read Files:
200

Dados processados:
227.60 MB
```

---

## Cenário 2 - Consulta por cliente

Consulta:

```sql
SELECT *
FROM performance.fato_vendas
WHERE id_cliente = 79;
```

Resultado:

```
Tempo:
3.517s

Read Files:
200

Dados processados:
453.25 MB
```

---

## Cenário 3 - Agregação por loja

Consulta:

```sql
SELECT
    id_loja,
    SUM(valor_total) AS faturamento
FROM performance.fato_vendas
GROUP BY id_loja;
```

Resultado:

```
Tempo:
2.005s

Read Files:
200

Dados processados:
210.95 MB
```

---

# Otimização Delta Lake - OPTIMIZE

Foi executada a compactação dos arquivos Delta utilizando:

```sql
OPTIMIZE performance.fato_vendas;
```

O objetivo foi reduzir a quantidade de arquivos pequenos e melhorar a eficiência de leitura.

---

# Resultado do OPTIMIZE

## Antes

```
Arquivos:
200

Tamanho:
428.32 MB
```

## Depois

```
Arquivos:
7

Tamanho:
398.05 MB
```

Resultado:

```
Redução de 96,5% na quantidade de arquivos Delta
```

---

# Benchmark após OPTIMIZE

## Cenário 1 - Consulta por período

Consulta:

```sql
SELECT
    COUNT(*) AS total_vendas,
    SUM(valor_total) AS faturamento
FROM performance.fato_vendas
WHERE data_venda BETWEEN '2025-01-01'
AND '2025-03-31';
```

Resultado:

```
Tempo:
1.792s

Read Files:
7

Dados processados:
157.88 MB
```

Comparação:

```
Antes:
3.427s

Depois:
1.792s
```

Melhoria aproximada:

```
48% redução no tempo de execução
```

---

## Cenário 2 - Consulta por cliente

Consulta:

```sql
SELECT *
FROM performance.fato_vendas
WHERE id_cliente = 79;
```

Resultado:

```
Tempo:
2.458s

Read Files:
7

Dados processados:
381.35 MB
```

Comparação:

```
Antes:
2.458s

Depois:
2.458s
```

Melhoria aproximada:

```
Redução de 30% no tempo de execução
```

---

## Cenário 3 - Agregação por loja

Consulta:

```sql
SELECT
    id_loja,
    SUM(valor_total) AS faturamento
FROM performance.fato_vendas
GROUP BY id_loja;
```

Resultado:

```
Tempo:
1.633s

Read Files:
7

Dados processados:
139.98 MB
```

Comparação:

```
Antes:
2.005s

Depois:
1.633s
```

Melhoria aproximada:

```
Redução de 19% no tempo de execução
```

---

# Otimização Delta Lake - ZORDER

Para otimizar consultas utilizando filtros específicos, foi aplicado ZORDER utilizando a coluna:

```
id_cliente
```

Comando:

```sql
OPTIMIZE performance.fato_vendas
ZORDER BY (id_cliente);
```

O objetivo é melhorar o **Data Skipping**, permitindo que o Delta Lake identifique quais arquivos possuem dados relevantes antes de realizar a leitura.

---

# Benchmark após OPTIMIZE + ZORDER

Consulta:

```sql
SELECT *
FROM performance.fato_vendas
WHERE id_cliente = 79;
```

Resultado:

```
Tempo:
2.402s

Read Files:
1

Dados processados:
87.02 MB
```

Comparação:

Antes:

```
453.25 MB processados
```

Depois:

```
87.02 MB processados
```

Resultado:

```
Redução aproximada de 80% no volume de dados processados
```

---

# Delta Lake Time Travel

O Delta Lake mantém histórico das alterações realizadas nas tabelas, permitindo auditoria, consulta de versões anteriores e recuperação de dados.

---

# Cenário

Uma atualização incorreta alterou o valor de uma venda existente.

---

## Versão Inicial

Consulta utilizando histórico:

```sql
SELECT *
FROM performance.fato_vendas
VERSION AS OF 1
WHERE id_venda = 771;
```

Resultado:

| id_venda | data_venda | id_cliente | id_produto | id_loja | quantidade | valor_unitario | valor_total |
|---|---|---|---|---|---|---|---|
|771|2025-07-27|79|35|24|6|49.73|298.38|

---

## Após alteração incorreta

Consulta atual:

```sql
SELECT *
FROM performance.fato_vendas
WHERE id_venda = 771;
```

Resultado:

| id_venda | data_venda | id_cliente | id_produto | id_loja | quantidade | valor_unitario | valor_total |
|---|---|---|---|---|---|---|---|
|771|2025-07-27|79|35|24|6|49.73|0.00|

---

## Recuperação da versão anterior

Foi realizada a restauração da tabela:

```sql
RESTORE TABLE performance.fato_vendas
TO VERSION AS OF 1;
```

Após o restore:

| id_venda | data_venda | id_cliente | id_produto | id_loja | quantidade | valor_unitario | valor_total |
|---|---|---|---|---|---|---|---|
|771|2025-07-27|79|35|24|6|49.73|298.38|

---

# Tecnologias Utilizadas

## Plataforma

- Databricks
- Apache Spark

## Armazenamento

- Delta Lake

## Linguagens

- Python
- PySpark
- Spark SQL

---

# Principais Conceitos Demonstrados

- Delta Lake Optimization
- Small Files Problem
- OPTIMIZE
- ZORDER
- Data Skipping
- Performance Benchmarking
- Delta Time Travel
- Restore de versões históricas
- Engenharia de Performance em Lakehouse

---

# Resultados Obtidos

O projeto demonstrou como técnicas de otimização Delta Lake podem melhorar a eficiência de consultas analíticas através de:

- Redução significativa da quantidade de arquivos Delta
- Menor volume de dados processados
- Melhoria no tempo de execução das consultas
- Melhor aproveitamento do Data Skipping
- Capacidade de auditoria e recuperação de dados

---

# Author

**Felipe Porfirio**

Senior Data Engineer

Portfolio:

https://felipeporfiriolima.github.io/

LinkedIn:

https://www.linkedin.com/in/felipe-porfirio-lima/
