# Persisting Work on Databricks
Databricks clusters are treated as ephemeral computational resources - the default configuration is for the cluster to automatically terminate after 120 minutes of inactivity.  While this saves you a bundle on your cloud costs, it means you'll need to figure out where to persist your work long term.  

#### DBFS
The Databricks file system, or [DBFS](https://docs.databricks.com/user-guide/databricks-file-system.html#databricks-file-system), is an abstraction that sits on top of any blob storage such as S3 or ADLS. It allows you to treat files in cloud storage as though they reside on the local file system of your laptop.  You can use the [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#dbfs-cli) to interact with DBFS or you can use R.  The following examples will use R. 

#### Working with DBFS in R

By default, your working directory in an R session on Databricks will be on the driver node, **not** DBFS.  DBFS is accessible to all nodes in a cluster through the `/dbfs/` path.  Simply include it in the path to your files when using R.

##### Unix Operations
```R
## List files on DBFS
system("ls /dbfs/", intern = T)

## Copy from one directory to another
system("cp /dbfs/directory_A /dbfs/directory_B

## Persist from driver to DBFS
system("cp /databricks/driver/file.txt /dbfs/file.txt")
```

##### Reading Data

**Note:** When reading into Spark, use the `dbfs:/` syntax.

```R
## Read a csv into local R dataframe
df <- read.csv("/dbfs/your/directory/file.csv")

## Reading in your data from DBFS using SparkR
df <- read.df('dbfs:/your/directory/file.csv', 'csv')

## Reading in your data from DBFS using sparklyr
df<-spark_read_csv( sc, 'df','dbfs:/your/directory/file.csv')
```

##### Saving & Loading Objects

```R
## Save a linear model to DBFS
model <- lm(mpg ~ ., data = mtcars)
saveRDS(model, file = "/dbfs/your/directory/model.RDS")

## Load back into memory
model <- readRDS(file = "/dbfs/your/directory/model.RDS")
```

You can read more about the SparkR and sparklyr data types in the `Spark - Distributed R sections` under `SparkR vs. sparklyr`.  We'll also talk more about DBFS in the package management section of this guide.

# Using DBFS resources for Deep Learning 
Within DBFS there is a `/ml` directory. This directory was designed with an optimized FUSE mount specifically for deep learning and processing image data. To learn more, see the [high performance local APIs](https://docs.databricks.com/user-guide/databricks-file-system.html#high-performance-local-apis) section of the DBFS documentation.
