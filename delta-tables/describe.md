


# Trace Delta Table Notebook
In Databricks, there are several ways to trace which notebook created or last modified a Delta table:
* Delta Table History
* Query the History via Python/PySpark
* Check `_delta_log` Directly
* Table Properties (if set manually)
* Unity Catalog Lineage (if enabled)

## Delta Table History
The most direct method — Delta logs every operation with metadata:

```sql
DESCRIBE HISTORY your_table_name;
```

Look at the `notebook` column in the results. It contains:
- `notebookId` — the ID of the notebook that ran the operation
- `notebookPath` — the full workspace path of the notebook

You can also filter for specific operations:
```sql
DESCRIBE HISTORY your_table_name LIMIT 10;
```

---

## Query the History via Python/PySpark

```python
from delta.tables import DeltaTable

dt = DeltaTable.forName(spark, "your_table_name")
history_df = dt.history()

# Show notebook info
history_df.select("version", "timestamp", "operation", "notebook") \
          .show(truncate=False)
```

Or if using a path:
```python
dt = DeltaTable.forPath(spark, "/path/to/delta/table")
```

---

## Check `_delta_log` Directly

Each Delta table has a `_delta_log/` folder. You can read the JSON commit files to see notebook metadata:

```python
import json

log_path = "/path/to/your/delta/table/_delta_log/00000000000000000001.json"
with open(log_path) as f:
    for line in f:
        entry = json.loads(line)
        if "commitInfo" in entry:
            print(entry["commitInfo"].get("notebook"))
```

---

## Table Properties (if set manually)

If the team set lineage properties at write time:
```sql
DESCRIBE EXTENDED your_table_name;
```
Look under **Table Properties** for any custom tags like `source_notebook`.

---

## Unity Catalog Lineage (if enabled)

If your workspace uses **Unity Catalog**, you get automatic lineage tracking:

- Go to **Catalog Explorer → your table → Lineage tab**
- It visually shows which notebooks, jobs, and dashboards read/wrote the table — no SQL needed.

---

## Quick Summary

| Method | What you get |
|---|---|
| `DESCRIBE HISTORY` | Full audit log with notebook path/ID |
| Delta Python API | Same, but programmatic |
| `_delta_log` JSON | Raw commit metadata |
| Unity Catalog Lineage | Visual lineage graph |

The **`DESCRIBE HISTORY`** approach is usually the fastest starting point.