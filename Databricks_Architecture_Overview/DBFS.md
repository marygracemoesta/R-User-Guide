# Using the Databricks File System 
The databricks file system, or DBFS, is an abstraction that sits on top of any Blob storage. You can read more about the
details of DBFS here:  https://docs.databricks.com/user-guide/databricks-file-system.html#databricks-file-system

 
Amongst many uses, DBFS can be helpful in pulling local files onto Databricks.
```bash
# Using the DBFS cli to copy a local file 
databricks fs cp /local/file/directory dbfs:/your/directory/file.csv
```
Distributed R can be used to pull files from DBFS 
```R
## Reading in your data from DBFS using SparkR
df <- read.df('dbfs:/your/directory/file.csv', 'csv')

## Reading in your data from DBFS using sparklyr
df<-spark_read_csv( sc, 'df','dbfs:/your/directory/file.csv')
```
You can read more about the SparkR and sparklyr data types in the `Spark - Distributed R sections` under `SparkR vs. sparklyr`.

# Using DBFS resources for Deep Learning 
Within DBFS there is a `/ml` directory. This directory was designed with an optimized FUSE mount specifically for deep learning
and processing image data. 
