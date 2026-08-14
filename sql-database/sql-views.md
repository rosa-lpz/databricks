# SQL views in Databricks

# Create View
In Databricks SQL, you can create a temporary table using a Temporary View (or TEMP VIEW). Databricks SQL treats temporary tables as session-scoped temporary views.

Here are the standard ways to create them depending on your use case:



## From an Existing Table or Select Query
The most common approach is using CREATE TEMPORARY VIEW with a SELECT statement:
```SQL
CREATE OR REPLACE TEMPORARY VIEW temp_high_sales AS
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```
### Example
```SQL
CREATE OR REPLACE TEMPORARY VIEW temp_high_sales AS
SELECT 
    customer_id,
    SUM(amount) AS total_spent
FROM 
    sales_data
WHERE 
    transaction_date >= '2026-01-01'
GROUP BY 
    customer_id
HAVING 
    total_spent > 1000;

```

## Manual Data Insertion (Using VALUES)
If you need a quick temporary table with mock data or inline values:

### Example
```SQL
CREATE OR REPLACE TEMPORARY VIEW temp_categories AS
SELECT * FROM VALUES 
    (1, 'Electronics'),
    (2, 'Apparel'),
    (3, 'Home & Kitchen')
AS t(category_id, category_name);
```


## Using Global Temporary Views (Cross-Session Scope)
If you need the temporary data to be accessible across different notebooks or jobs attached to the same cluster:

```SQL
CREATE OR REPLACE GLOBAL TEMPORARY VIEW global_temp_sales AS
SELECT * FROM sales_data WHERE status = 'PENDING';

-- To query a global temporary view, you MUST prefix it with the `global_temp` database:
SELECT * FROM global_temp.global_temp_sales;
```

Key Differences & Scope
TypeScopeAccess SyntaxLifecycleTemporary View (TEMP VIEW)Current Session / Notebook onlySELECT * FROM temp_view_nameDropped when session endsGlobal Temp View (GLOBAL TEMP VIEW)All sessions on the clusterSELECT * FROM global_temp.temp_view_nameDropped when cluster stops/restarts




Note: If you explicitly want to remove a temporary table before your session ends, use:
```SQL
DROP VIEW IF EXISTS temp_high_sales;
```