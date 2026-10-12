# Upload files to a volume in Unity Catalog

## Steps to upload files

* Go to Catalog 
* Click on "Create" button
* Click on "Add Data"
* Select "Upload files to a volume"



## Retrieve data from a notebook

### SQL

```sql
%sql
SELECT * FROM csv.`/Volumes/workspace/default/worldcities.csv` --OPTIONS (header=>"true", inferSchema=>"true")
```



```sql
%sql
SELECT * FROM read_files(
  '/Volumes/workspace/default/worldcities.csv',
  format => 'csv',
  header => true,
  inferSchema => 'true'
)
```

### PySpark

```python
dataset_path = "/Volumes/workspace/default/telco-customer-churn-missing.csv"

schema_string = """
    customerID string,
    gender string,
    SeniorCitizen double,
    Partner string,
    Dependents string,
    tenure double,
    phoneService string,
    MultipleLines string,
    internetService string,
    OnlineSecurity string,
    OnlineBackup string,
    DeviceProtection string,
    TechSupport string,
    StreamingTV string,
    StreamingMovies string,
    Contract string,
    PaperlessBilling string,
    PaymentMethod string,
    MonthlyCharges double,
    TotalCharges double,
    Churn string
"""

telco_df = spark.read.csv(dataset_path, \
                          header=True,
                          schema=schema_string,
                          multiLine=True,
                          escape='"')

display(telco_df)
```

