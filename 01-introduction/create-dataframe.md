# Create dataframe
How to Create a PySpark DataFrame from a CSV File

The primary method for creating a PySpark DataFrame from a CSV file is the read.csv method of the SparkSession. This unified entry point, which encapsulates the older Spark Context for RDD operations, allows you to load a CSV file into a distributed DataFrame, with options to infer the schema or define a custom schema for type control. Here’s the basic syntax:
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("CreateDataFrameFromCSV").getOrCreate()
df = spark.read.csv("path/to/csv/file.csv", header=True, inferSchema=True)
```
It’s like turning your CSV file into a distributed table ready for Spark’s magic. Let’s try it with an employee CSV file, a common ETL scenario, with columns for employee IDs, names, ages, and salaries. Assume employees.csv contains:

```
employee_id,name,age,salary
E001,Alice,25,75000.00
E002,Bob,30,82000.50
E003,Cathy,28,90000.75
E004,David,35,100000.25
```


# How to Create a DataFrame from a Simple CSV File

A simple CSV file has uniform columns with basic types like strings, integers, and floats, perfect for straightforward ETL tasks like those in ETL Pipelines. Using the same employees.csv:
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("SimpleCSV").getOrCreate()

df_simple = spark.read.csv("employees.csv", header=True, inferSchema=True)
df_simple.show(truncate=False)
```
It’s like turning your CSV file into a distributed table ready for Spark’s magic. Let’s try it with an employee CSV file, a common ETL scenario, with columns for employee IDs, names, ages, and salaries. Assume employees.csv contains:

```
+-----------+-----+---+---------+
|employee_id|name |age|salary   |
+-----------+-----+---+---------+
|E001       |Alice|25 |75000.0  |
|E002       |Bob  |30 |82000.5  |
|E003       |Cathy|28 |90000.75 |
|E004       |David|35 |100000.25|
+-----------+-----+---+---------+
```


# References
* https://www.sparkcodehub.com/pyspark/dataframe/create-dataframe-from-csv