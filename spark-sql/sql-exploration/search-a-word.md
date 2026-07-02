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
-- Search for columns across the information schema
SELECT table_schema, table_name
FROM my_catalog.information_schema.columns 
WHERE table_schema LIKE '%fruits%'
AND table_name LIKE '%apple%'  
```

## Specific tables and columns
```sql
-- Search for columns across the information schema
SELECT table_schema, table_name, column_name 
FROM my_catalog.information_schema.columns 
WHERE table_schema LIKE '%fruits%'
AND table_name LIKE '%apple%'  
AND column_name  ILIKE 'quality%'
```


# Searching Across ALL Columns in a Table

```sql
SELECT * FROM my_table 
WHERE concat_ws(' ', *) ILIKE '%apple%';
```