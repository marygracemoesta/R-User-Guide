
## Apache Arrow

* [What is Apache Arrow?](#what-is-apache-arrow)
* [Installing Arrow](#installing-arrow)
* [Arrow Benchmarks](#arrow-benchmarks)
___

#### What is Apache Arrow?

[Apache Arrow](https://arrow.apache.org/) is a project that aims to improve analytics processing performance by representing data in-memory in columnar format and taking advantage of modern hardware.  To better understand the problem Arrow is trying to solve, check out the following image from the project homepage.

<img src="https://github.com/kurlare/misc-assets/blob/master/apache-arrow.png?raw=true">

Without Arrow, passing data from one system too another involves serialization and deserialization operations between in-memory formats.  This can be especially limiting when working with user defined functions in Spark, or when simply want to collect some data back to the driver node.  

Instead of copying and converting data from Java to Python or from Pandas to Spark, the in-memory representation of data remains consistent regardless of the system use for processing.  This eliminates the cost and memory expansion of converting from one format to another, and can seriously boost performance when working with UDFs in Spark.  One customer saw a SparkR job that took 50 minutes wihout Arrow successfully execute in **40 seconds** with it!

These performance benefits are realized through a form of data skipping and hardware optimization.  Another helpful graphic from the project homepage illustrates the difference between row-based formatting vs. columnar.

<img src="https://arrow.apache.org/img/simd.png">


#### Installing Arrow
In order for Arrow to work properly, complete the following steps:

1. Attach the [initialization script for Apache Arrow](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/Customizing.md#apache-arrow-installation) to your cluster. 
2. Create a cluster with Databricks Runtime version 5.5 or greater.
3. Install `arrow` from CRAN in the Cluster UI.
4. Install `sparklyr` from CRAN in the Cluster UI. 

#### Arrow Benchmarks
The code for these benchmarks was taken [from RStudio](https://blog.rstudio.com/2019/03/15/sparklyr-1-0/) and run on Databricks Runtime 5.5 LTS.  They are divided into three tasks:

- Copying data from R to Spark
- Collecting data from Spark to R
- User defined functions with `sparklyr::spark_apply()`

**Note:** They do take 20-60 minutes to run.

##### First Benchmark:  Copying data from R to Spark

```r
bench::press(rows = c(10^7), {
  bench::mark(
    arrow_on = {
      library(arrow)
      sparklyr_df <<- copy_to(sc, data.frame(y = 1:rows), overwrite = T)
    },
    arrow_off = if (rows <= 10^7) {
      if ("arrow" %in% .packages()) detach("package:arrow")
      sparklyr_df <<- copy_to(sc, data.frame(y = 1:rows), overwrite = T)
    } else NULL, iterations = 4, check = FALSE)
})
```

_Results:_

| condition  | rows       | time    |
|------------|------------|---------|
| arrow_on   | 10,000,000 | 5.19sec |
| arrow_off  | 10,000,000 | 3.95min |

##### Second Benchmark:  Collecting data from Spark to R

```r
bench::press(rows = c(5 * 10^7), {
  bench::mark(
    arrow_on = {
      library(arrow)
      collected <- sdf_len(sc, rows) %>% collect()
    },
    arrow_off = if (rows <= 5 * 10^7) {
      if ("arrow" %in% .packages()) detach("package:arrow")
      collected <- sdf_len(sc, rows) %>% collect()
    } else NULL, iterations = 4, check = FALSE)
})
```

_Results:_

| condition  | rows       | time    |
|------------|------------|---------|
| arrow_on   | 50,000,000 | 15.5sec |
| arrow_off  | 50,000,000 | 13.8sec |

Interestingly, it didn't seem to make a difference in this benchmark.  Worth trying again at a later date.

##### Third Benchmark: User Defined Functions with `spark_apply()`

```r
## spark_apply, then count and collect
bench::press(rows = c(10^6), {
  bench::mark(
    arrow_on = {
      library(arrow)
      sdf_len(sc, rows) %>% spark_apply(~ .x / 2) %>% dplyr::count() %>% collect
    },
    arrow_off = if (rows <= 10^6) {
      if ("arrow" %in% .packages()) detach("package:arrow")
      sdf_len(sc, rows) %>% spark_apply(~ .x / 2) %>% dplyr::count() %>% collect
    } else NULL, iterations = 4, check = FALSE)
})
```

_Results:_

| condition  | rows       | time    |
|------------|------------|---------|
| arrow_on   | 100,000    | 4.16sec |
| arrow_off  | 100,000    | 1.06min |
| arrow_on   | 1,000,000  | 4.77sec |
| arrow_off  | 1,000,000  | 10.77min|

It's worth noting that Databricks offers similar optimizations out of the box for SparkR.  See [here](https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/DB_Runtime.md#sparkr-optimization) for more details.
