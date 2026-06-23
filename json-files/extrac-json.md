# Extract values from JSON column

In **Databricks (Spark SQL)**, the most common ways to extract values from a JSON column are:

### Sample data

Suppose `json_col` contains:

```json
{
  "name": "John",
  "age": 30,
  "address": {
    "city": "New York"
  }
}
```

### 1. Use `get_json_object()`

Extract a field using a JSONPath expression:

```sql
SELECT
    get_json_object(json_col, '$.name') AS name,
    get_json_object(json_col, '$.age') AS age,
    get_json_object(json_col, '$.address.city') AS city
FROM my_table;
```

### 2. Parse JSON into a struct with `from_json()`

This is preferred when you'll access multiple fields.

```sql
SELECT
    parsed.name,
    parsed.age,
    parsed.address.city
FROM (
    SELECT from_json(
        json_col,
        'name STRING, age INT, address STRUCT<city: STRING>'
    ) AS parsed
    FROM my_table
);
```

### 3. If the column is already a struct

Sometimes Databricks has already parsed the JSON into a struct column.

```sql
SELECT
    json_col.name,
    json_col.age,
    json_col.address.city
FROM my_table;
```

### 4. Extract array elements

JSON:

```json
{
  "items": [
    {"id": 1},
    {"id": 2}
  ]
}
```

Get the first item's id:

```sql
SELECT get_json_object(json_col, '$.items[0].id') AS first_id
FROM my_table;
```

Or parse and access directly:

```sql
SELECT parsed.items[0].id
FROM (
    SELECT from_json(
        json_col,
        'items ARRAY<STRUCT<id: INT>>'
    ) AS parsed
    FROM my_table
);
```

### Check the column type first

```sql
DESCRIBE TABLE my_table;
```

or

```sql
SELECT typeof(json_col)
FROM my_table
LIMIT 1;
```

If you share a sample JSON value and the column type (`STRING` or `STRUCT`), I can write the exact Databricks SQL expression.
