Databricks Table Schema Version

Databricks does not support retrieving the schema of a specific historical version of a table directly via SQL commands like `DESCRIBE TABLE` or `DESCRIBE DETAIL`. These commands only return the **current** schema and metadata of the table.

To view the schema at a specific point iDatabricks does not support a direct SQL command to describe the table schema **from a specific historical version** (time travel) using the `DESCRIBE TABLE` syntax. The `DESCRIBE TABLE` command only returns the **current** schema of the table.

To retrieve the schema as it existed at a specific version, you must use **time travel** syntax to query the table at that version and inspect its schema:

1.  **Using PySpark/Scala/Java:**
    Read the table at the specific version and call `printSchema()`.
    ```python
    df = spark.read.option("versionAsOf", <version_number>).table("schema_name.table_name")
    df.printSchema()
    ```

2.  **Using SQL:**
    You can select from the table at a specific version and describe the result, or use the `DESCRIBE DETAIL` command to get metadata about that specific version if supported by your runtime, but the most reliable method is to treat the time-travelled table as a temporary view:
    ```sql
    CREATE TEMPORARY VIEW temp_schema AS
    SELECT * FROM schema_name.table_name VERSION AS OF <version_number>;
    
    DESCRIBE TABLE temp_schema;
    ```

**Key Facts:**
*   **DESCRIBE TABLE** always returns the **current** schema.
*   **DESCRIBE DETAIL** provides metadata like `minReaderVersion`, `minWriterVersion`, and `tableFeatures`, but not the column-level schema history.
*   **Time Travel** (`VERSION AS OF` or `TIMESTAMP AS OF`) allows you to access data and schema as they existed at a past point.
*   For **Unity Catalog**, you can also use `INFORMATION_SCHEMA.COLUMNS` on the current state, but it does not provide historical schema versions.