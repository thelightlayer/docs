# SQL Join Strategy and Performance Optimization Guide

- Types of JOINs and when to use them
- Performance issues of Nested Loop Join and how to identify them
- Using `EXPLAIN ANALYZE` to understand PostgreSQL execution plans
- Practical examples and optimization strategies

---

## 📘 Detailed Comparison of JOIN Types

| JOIN Type         | Working Principle                                           | Time Complexity | Suitable Scenario                        | Advantages                                  | Disadvantages                              |
|------------------|-------------------------------------------------------------|------------------|-------------------------------------------|---------------------------------------------|---------------------------------------------|
| **Nested Loop**  | Each row in outer table is scanned against all inner rows    | O(N × M)         | Small table join large table, complex join condition | Index usable, supports all join conditions  | Very slow with large datasets              |
| **Hash Join**    | Build a hash table on one table and probe with the other     | O(N + M)         | Equality join, medium to large tables     | Fast, linear time, efficient in-memory      | High memory use, no non-equi support       |
| **Merge Join**   | Sort both sides then perform merge-like row-by-row join      | O(N + M)         | Sorted input or indexed columns           | Great for sorted data, supports streaming   | Needs sorting/indexing, fragile on unsorted data |

---

## ✅ How PostgreSQL Decides Join Strategy

PostgreSQL chooses join strategies based on:

- Table size (from `ANALYZE` statistics)
- Index availability
- Whether join is equality (=)
- If input is already sorted or index-sortable

---

## 🔍 PostgreSQL Join Strategy Examples

### 🌀 1. Nested Loop Join Example

```sql
SELECT o.id, c.name
FROM orders o
JOIN customers c ON o.customer_id = c.id;
```

#### 💡 Hypothetical Conditions:

- `orders` has 10 rows (small)
- `customers` has 1,000,000 rows
- No index on `customer_id`, or join condition is complex

#### 🧠 Execution Plan:

```
Nested Loop
  -> Seq Scan on orders
  -> Seq Scan on customers (per order row)
```

✅ Best when joining small table to large table, or with `LIMIT` queries

---

### ⚡ 2. Hash Join Example

```sql
SELECT e.name, d.name AS department
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

#### 💡 Hypothetical Conditions:

- `employees` has 500,000 rows
- `departments` has 100 rows
- Simple equality join

#### 🧠 Execution Plan:

```
Hash Join
  -> Seq Scan on employees
  -> Hash on departments.id
```

✅ Efficient for large joins with equality conditions

---

### 🔗 3. Merge Join Example

```sql
SELECT s.id, p.name
FROM sales s
JOIN products p ON s.product_id = p.id
ORDER BY s.product_id;
```

#### 💡 Hypothetical Conditions:

- Both `sales` and `products` are sorted (or have index on `product_id`)
- Equality join condition

#### 🧠 Execution Plan:

```
Merge Join
  -> Index Scan on sales
  -> Index Scan on products
```

✅ Excellent for pre-sorted data, interval joins, or when used with window functions

---

## 🧠 Practical Join Strategy Suggestions

| Scenario                                  | Recommended Join Type            |
|------------------------------------------|----------------------------------|
| Small table joins large table            | ✅ Nested Loop or Index Nested Loop |
| Large tables with equality condition     | ✅ Hash Join                     |
| Indexed or pre-sorted columns            | ✅ Merge Join                    |
| Non-equality joins (`<`, `BETWEEN`, etc) | ✅ Nested Loop (only supported method) |

---

## 🧪 SQL Optimization Tips

- Use `EXPLAIN ANALYZE` to understand which join type is used
- Force PostgreSQL join type using settings (e.g. `SET enable_hashjoin = off;`)
- Create appropriate indexes to avoid Nested Loop on large tables
- Replace double `GROUP BY + JOIN` with `ROW_NUMBER()` window function

---

Generated: 2025-04-16
