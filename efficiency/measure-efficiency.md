## Measuring Query Efficiency in Databricks

Here are the main ways to evaluate and analyze query performance:

------

### 1. **Query Profile (Spark UI)**

The most powerful tool — access it via:

- **Databricks UI** → Go to your cluster → **Spark UI** → **SQL / Jobs tab**
- Or click the **"View"** link next to a completed query in the notebook

Key metrics to look at:

- **Duration** – total execution time
- **Rows read/written** – data volume processed
- **Spill** – if data spills to disk, your query is memory-inefficient
- **Shuffle** – high shuffle = expensive joins or aggregations

------

### 2. **EXPLAIN Command**

Shows the execution plan without running the query:

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

-- For more detail:
EXPLAIN EXTENDED SELECT * FROM orders WHERE customer_id = 123;

-- For cost-based optimizer info:
EXPLAIN COST SELECT * FROM orders WHERE customer_id = 123;
```

Look for:

- **FileScan** with partition filters → good (predicate pushdown working)
- **BroadcastHashJoin** → efficient for small tables
- **SortMergeJoin** → more expensive, used for large tables
- **Exchange** nodes → indicates shuffles (costly)

------

### 3. **Query History (SQL Warehouse)**

If using **Databricks SQL**:

- Go to **SQL → Query History**
- See duration, rows returned, data scanned, and execution details per query

------

### 4. **Delta Table Stats**

Check if your table is optimized:

```sql
-- Check table details and statistics
DESCRIBE DETAIL my_table;

-- Check partition info
SHOW PARTITIONS my_table;

-- Analyze table to update statistics
ANALYZE TABLE my_table COMPUTE STATISTICS;
ANALYZE TABLE my_table COMPUTE STATISTICS FOR COLUMNS col1, col2;
```

------

### 5. **Runtime Metrics in Notebooks**

Add timing around your queries:

```python
import time

start = time.time()
df = spark.sql("SELECT * FROM orders WHERE status = 'open'")
df.count()  # trigger action
print(f"Duration: {time.time() - start:.2f}s")
```

Or use the `%timeit` magic:

```python
%timeit df.filter("status = 'open'").count()
```

------

### 6. **Key Inefficiency Red Flags**

| Signal                | Problem                               |
| --------------------- | ------------------------------------- |
| Full table scan       | Missing partitioning or Z-ordering    |
| High shuffle bytes    | Poor join strategy or no bucketing    |
| Disk spill            | Insufficient memory / skewed data     |
| Many small files      | Need to run `OPTIMIZE` on Delta table |
| Low partition pruning | Filters not on partition columns      |

------

### 7. **Optimization Commands**

```sql
-- Compact small files and Z-order for faster reads
OPTIMIZE my_table ZORDER BY (customer_id, order_date);

-- Remove old/deleted data files
VACUUM my_table RETAIN 168 HOURS;
```

------

**Quick tip:** Start with `EXPLAIN` to catch plan issues before running, then use the **Spark UI Query Profile** after execution to pinpoint bottlenecks like skew, spill, or excessive shuffle.