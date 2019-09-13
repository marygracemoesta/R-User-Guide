# Using a R to read and write to a Delta Lake
R supports reading and writing to a [Delta Lake](https://docs.databricks.com/delta/index.html#delta-guide) 

```r
df <- read.df(path = 'dbfs:/delta_table", source = "delta")
```

```r
write.df(sparkdf, source = "delta", path = "dbfs:/delta_table/)
```

As of September 2019, sparklyr doesn't directly support reading and writing to Delta tables. 
