# Searching the Entire Database (Metadata Search)
If you are trying to find which tables or columns contain a specific word in their name, you need to query the system catalogs.


## All tables
```sql
-- Search for columns across the information schema
SELECT table_name, column_name 
FROM my_catalog.information_schema.columns 
WHERE column_name ILIKE '%apple%';
```
## Specific databases and tables
```sql
-- Search for tables schemas and tables across the information schema
SELECT table_schema, table_name
FROM my_catalog.information_schema.columns 
WHERE table_schema LIKE '%fruits%'
AND table_name LIKE '%apple%'  
```

## Specific tables and columns
```sql
-- Search for tables_schemas, tables and columns across the information schema
SELECT table_schema, table_name, column_name 
FROM my_catalog.information_schema.columns 
WHERE table_schema LIKE '%fruits%'
AND table_name LIKE '%apple%'  
AND column_name  ILIKE 'quality%'
```

```python
db = spark.sql("SHOW SCHEMAS")
for db_row in db.collect():
    tb = spark.sql(f"SHOW TABLES IN {db_row.databaseName}")
    for tb_row in tb.collect():
        df = spark.table(f"{db_row.databaseName}.{tb_row.tableName}")
        filter = "1=0"
        for col in df.colunns:
            filter += f" OR {col} LIKE '%unicorn%'"
        df = df.filter(filter)
        if (df.count() > 0):
            print(f"{db_row.databaseName}.{tb_row.tableName}")
            print(df.collect())
```

# Searching Across ALL Columns in a Table

```sql
SELECT * FROM my_table 
WHERE concat_ws(' ', *) ILIKE '%apple%';
```



# References
* [Reddit - Search for a word in databricks table or database](https://www.reddit.com/r/SQL/comments/15by9ep/search_for_a_word_in_databricks_table_or_database/)