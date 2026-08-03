# Delta Lake Advanced Engineering & Optimization

🌎 **Language:** English | [Português](README-PT.md)

## Performance Engineering Project using Databricks and Delta Lake

## Overview

This project demonstrates advanced data engineering techniques applied to Delta Lake within the Databricks platform.

The objective is to evaluate the impact of optimization strategies on Delta tables, analyzing common challenges found in large-scale data environments, such as:

- Small Files Problem
- Large number of files to scan
- Unnecessary data processing
- Need for optimization in analytical workloads

Through comparative benchmarks before and after optimization techniques, this project demonstrates the impact of Delta Lake capabilities:

- OPTIMIZE
- ZORDER
- Data Skipping
- Delta Time Travel
- Historical Version Restore

---

# Solution Architecture

```
Data Generation
        |
        |
        ▼
Delta Table
performance.fato_vendas
        |
        |
        ├── Initial Benchmark
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

A synthetic dataset was created to simulate a corporate sales table.

Delta Table:

```
performance.fato_vendas
```

Main attributes:

| Field | Description |
|---|---|
| id_venda | Sales identifier |
| data_venda | Sale date |
| id_cliente | Customer identifier |
| id_produto | Product identifier |
| id_loja | Store identifier |
| quantidade | Quantity sold |
| valor_unitario | Unit price |
| valor_total | Total sales amount |

---

# Initial Scenario

Before applying optimization techniques, the Delta table presented the **Small Files Problem**.

## Delta Table

```
numFiles: 200

sizeInBytes:
428.32 MB
```

This scenario represents a common challenge in Lakehouse environments, where frequent ingestion processes can generate many small files, increasing read overhead and negatively impacting query performance.

---

# Initial Benchmark

## Scenario 1 - Date Range Query

Query:

```sql
SELECT
    COUNT(*) AS total_vendas,
    SUM(valor_total) AS faturamento
FROM performance.fato_vendas
WHERE data_venda BETWEEN '2025-01-01'
AND '2025-03-31';
```

Result:

```
Execution Time:
3.427s

Read Files:
200

Data Processed:
227.60 MB
```

---

## Scenario 2 - Customer Filter Query

Query:

```sql
SELECT *
FROM performance.fato_vendas
WHERE id_cliente = 79;
```

Result:

```
Execution Time:
3.517s

Read Files:
200

Data Processed:
453.25 MB
```

---

## Scenario 3 - Store Aggregation Query

Query:

```sql
SELECT
    id_loja,
    SUM(valor_total) AS faturamento
FROM performance.fato_vendas
GROUP BY id_loja;
```

Result:

```
Execution Time:
2.005s

Read Files:
200

Data Processed:
210.95 MB
```

---

# Delta Lake Optimization - OPTIMIZE

File compaction was performed using:

```sql
OPTIMIZE performance.fato_vendas;
```

The goal was to reduce the number of small files and improve read efficiency.

---

# OPTIMIZE Results

## Before

```
Files:
200

Size:
428.32 MB
```

## After

```
Files:
7

Size:
398.05 MB
```

Result:

```
96.5% reduction in the number of Delta files
```

---

# Benchmark After OPTIMIZE

## Scenario 1 - Date Range Query

Query:

```sql
SELECT
    COUNT(*) AS total_vendas,
    SUM(valor_total) AS faturamento
FROM performance.fato_vendas
WHERE data_venda BETWEEN '2025-01-01'
AND '2025-03-31';
```

Result:

```
Execution Time:
1.792s

Read Files:
7

Data Processed:
157.88 MB
```

Comparison:

```
Before:
3.427s

After:
1.792s
```

Approximate Improvement:

```
48% reduction in execution time

30% reduction in processed data volume
```

---

## Scenario 2 - Customer Filter Query

Query:

```sql
SELECT *
FROM performance.fato_vendas
WHERE id_cliente = 79;
```

Result:

```
Execution Time:
2.458s

Read Files:
7

Data Processed:
381.35 MB
```

Comparison:

```
Before:
3.517s

After:
2.458s
```

Approximate Improvement:

```
30% reduction in execution time

16% reduction in processed data volume
```

---

## Scenario 3 - Store Aggregation Query

Query:

```sql
SELECT
    id_loja,
    SUM(valor_total) AS faturamento
FROM performance.fato_vendas
GROUP BY id_loja;
```

Result:

```
Execution Time:
1.633s

Read Files:
7

Data Processed:
139.98 MB
```

Comparison:

```
Before:
2.005s

After:
1.633s
```

Approximate Improvement:

```
19% reduction in execution time

34% reduction in processed data volume
```

---

# Delta Lake Optimization - ZORDER

To optimize queries using specific filters, ZORDER was applied using the column:

```
id_cliente
```

Command:

```sql
OPTIMIZE performance.fato_vendas
ZORDER BY (id_cliente);
```

The objective was to improve **Data Skipping**, allowing Delta Lake to identify which files contain relevant data before performing the scan.

---

# Benchmark After OPTIMIZE + ZORDER

Query:

```sql
SELECT *
FROM performance.fato_vendas
WHERE id_cliente = 79;
```

Result:

```
Execution Time:
2.402s

Read Files:
1

Data Processed:
87.02 MB
```

Comparison:

Before:

```
453.25 MB processed
```

After:

```
87.02 MB processed
```

Result:

```
Approximately 80% reduction in processed data volume
```

---

# Delta Lake Time Travel

Delta Lake maintains a history of changes performed on tables, enabling auditing, previous version queries, and data recovery.

---

# Scenario

An incorrect update changed the value of an existing sales record.

---

## Initial Version

Historical query:

```sql
SELECT *
FROM performance.fato_vendas
VERSION AS OF 1
WHERE id_venda = 771;
```

Result:

| id_venda | data_venda | id_cliente | id_produto | id_loja | quantidade | valor_unitario | valor_total |
|---|---|---|---|---|---|---|---|
|771|2025-07-27|79|35|24|6|49.73|298.38|

---

## After Incorrect Update

Current table query:

```sql
SELECT *
FROM performance.fato_vendas
WHERE id_venda = 771;
```

Result:

| id_venda | data_venda | id_cliente | id_produto | id_loja | quantidade | valor_unitario | valor_total |
|---|---|---|---|---|---|---|---|
|771|2025-07-27|79|35|24|6|49.73|0.00|

---

## Restore Previous Version

The table was restored using:

```sql
RESTORE TABLE performance.fato_vendas
TO VERSION AS OF 1;
```

After restore:

| id_venda | data_venda | id_cliente | id_produto | id_loja | quantidade | valor_unitario | valor_total |
|---|---|---|---|---|---|---|---|
|771|2025-07-27|79|35|24|6|49.73|298.38|

---

# Technologies Used

## Platform

- Databricks
- Apache Spark

## Storage

- Delta Lake

## Languages

- Python
- PySpark
- Spark SQL

---

# Key Concepts Demonstrated

- Delta Lake Optimization
- Small Files Problem
- OPTIMIZE
- ZORDER
- Data Skipping
- Performance Benchmarking
- Delta Time Travel
- Historical Version Restore
- Lakehouse Performance Engineering

---

# Results Achieved

This project demonstrated how Delta Lake optimization techniques can improve analytical query efficiency through:

- Significant reduction in Delta file count
- Lower amount of processed data
- Improved query execution performance
- Better utilization of Data Skipping
- Data auditing and recovery capabilities

---

# Author

**Felipe Porfirio**

Senior Data Engineer

Portfolio:

https://felipeporfiriolima.github.io/

LinkedIn:

https://www.linkedin.com/in/felipe-porfirio-lima/
