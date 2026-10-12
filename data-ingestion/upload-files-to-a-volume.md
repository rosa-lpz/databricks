# Upload files to a volume in Unity Catalog

## Steps to upload files

* Go to Catalog 
* Click on "Create" button
* Click on "Add Data"
* Select "Upload files to a volume"



## Retrieve data from a notebook

```sql
%sql
SELECT * FROM csv.`/Volumes/workspace/default/databricks/spatial/worldcities.csv` --OPTIONS (header=>"true", inferSchema=>"true")
```



```sql
%sql
SELECT * FROM read_files(
  '/Volumes/workspace/default/databricks/spatial/worldcities.csv',
  format => 'csv',
  header => true,
  inferSchema => 'true'
)
```

