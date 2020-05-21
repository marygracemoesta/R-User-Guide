## Distributed R: User Defined Functions in Spark

**Contents**

* [Understanding UDFs](#understanding-udfs)
* [Distributed `apply`](#distributed-apply)
  * `spark_apply`
  * `dapply` & `gapply`
  * `spark.lapply`
* [Leveraging Packages in Distributed R](#leveraging-packages-in-distributed-r)
* [Apache Arrow](#apache-arrow)

___

#### Understanding UDFs

Both `SparkR` and `sparklyr` support user-defined functions (UDFs) in R which allow you to execute arbitrary R code across a cluster.  The advantage here is the ability to distribute the computation of functions included in R's massive ecosystem of 3rd party packages.  In particular, you may want to use a domain-specific package for machine learning or apply a specific statisical transformation that is not available through the Spark API.  Running in-house custom R libraries on larger data sets would be another place to use this family of functions.

How do these functions work?  The R process on the driver has to communicate with R processes on the worker nodes through a series of serialize/deserialize operations through the JVMs.  The following illustration walks through the steps required to run arbitrary R code across a cluster.

<img src="https://github.com/kurlare/misc-assets/blob/master/DistributedR_ControlFlow.png?raw=true" width="800" />

Looks great, but what's the catch?  

* **You have to reason about your program carefully and understand how exactly these functions are being, *ahem*, applied across your cluster.**  

* **R processes on worker nodes are ephemeral.  When the function being applied finishes execution the process is shut down and all state is lost.**

* **As a result, you have to pass any contextual data and libraries along with your function to each worker for your job to be successful.**

* **There is overhead related to creating the R process and ser/de operations in each worker.**  

Don't be surprised if using these functions runs slower than expected, especially on the first pass if you have to install the packages in each worker.  One of the benefits of running distributed R on Databricks is that you can install libraries at the cluster scope.  This makes them available on each worker and you do not have to pay this performance penalty every time you spin up a cluster.

The general best practice is to leverage the Spark API first and foremost, then if there is no way to implement your logic except in R you can turn to UDFs and get the job done.  This is echoed [here](https://therinspark.com/distributed.html) by one of the authors of `sparklyr` who is currently at RStudio.

___

#### Distributed `apply`

Between `sparklyr` and `SparkR` there are a number of options for how you can distribute your R code across a cluster with Spark.  Functions can be applied to each *group* or each *partition* of a Spark DataFrame, or to a list of elements in R.  In the following table you can see the whole family of distributed `apply` functions:

|  Package | Function Name |     Applied To     |   Input  |    Output    |
|:--------:|:-------------:|:------------------:|:--------:|:-----------:|
| sparklyr |  spark_apply  | partition or group | Spark DF |   Spark DF  |
|  SparkR  |     dapply    |      partition     | Spark DF |   Spark DF  |
|  SparkR  | dapplyCollect |      partition     | Spark DF | R dataframe |
|  SparkR  |     gapply    |        group       | Spark DF |   Spark DF  |
|  SparkR  | gapplyCollect |        group       | Spark DF | R dataframe |
|  SparkR  |  spark.lapply |    list element    |   list   |     list    |

Let's work through these different functions one by one.

##### `spark_apply`

For the first example, we'll use **`spark_apply()`**.

`spark_apply` takes a Spark DataFrame as input and must return a Spark DataFrame as well.  By default it will execute the function against each partition of the data, but we will change this by specifying a 'group by' column in the function call.  `spark_apply()` will also distribute all of the contents of your local `.libPaths()` to each worker when you call it for the first time unless you set the `packages` parameter to `FALSE`.  For more details see the [Official Documentation](https://spark.rstudio.com/guides/distributed-r/).  Let's test this by loading some airlines data into Spark and creating a new column for each Unique Carrier inside the R process on the workers:

**Note:** To get the best performance, we specify the schema of the expected output DataFrame to `spark_apply`.  This is optional, but if we don't supply the schema Spark will need to sample the output to infer it.  This can be quite costly on longer running UDFs.

```r
# Read data into Spark
sc <- sparklyr::spark_connect(method = "databricks")

sparklyAirlines <- sparklyr::spark_read_csv(sc, 
                                            name = 'airlines', 
                                            path = "/databricks-datasets/asa/airlines/2007.csv")

## Take a subset of the columns
subsetDF <- dplyr::select(sparklyAirlines, UniqueCarrier, Month, DayofMonth, Origin, Dest, DepDelay, ArrDelay) 

## Focus on the month of December, Christmas Eve
holidayTravelDF <- dplyr::filter(subsetDF, Month == 12, DayOfMonth == 24)

## Add a new column for each group and return the results
resultsDF <- sparklyr::spark_apply(holidayTravelDF,
                                  group_by = "UniqueCarrier",
                                  function(e){
                                    # 'e' is a data.frame containing all the rows for each distinct UniqueCarrier
                                    one_carrier_df <- data.frame(newcol = paste0(unique(e$UniqueCarrier), "_new"))
                                    one_carrier_df
                                  }, 
                                   # Specify schema
                                   columns = list(
                                   UniqueCarrier = "string",
                                   newcol = "string"),
                                   # Do not copy packages to each worker
                                   packages = F)
head(resultsDF)
```
```
# Source: spark<?> [?? x 2]
  UniqueCarrier newcol
  <chr>         <chr> 
1 UA            UA_new
2 AA            AA_new
3 NW            NW_new
4 EV            EV_new
5 B6            B6_new
6 DL            DL_new
```


##### `dapply` & `gapply`

In `SparkR`, there are separate functions depending on whether you want to run R code on each partition of a Spark DataFrame (`dapply`), or each group (`gapply`).  With these functions you **must** supply the schema ahead of time.  In the next example we will recreate the first but use `gapply` instead.

```r
library(SparkR)

# Get data into SparkR DF
airlinesDF <- SparkR::sql("SELECT * FROM airlines")

# Define schema
schema <- structType(structField("UniqueCarrier", "string"),
                     structField("newcol", "string"))

resultsDF <- gapply(airlinesDF,
                   cols = "UniqueCarrier",
                   function(key, e){
                     
                     one_carrier_df <- data.frame(
                       UniqueCarrier = unique(e$UniqueCarrier), 
                       newcol = paste0(unique(e$UniqueCarrier), "_new")
                     )
                     
                     one_carrier_df
                   },
                   schema = schema)

head(resultsDF)
```
```
  UniqueCarrier newcol
1            AA AA_new
2            B6 B6_new
3            OO OO_new
4            YV YV_new
5            HA HA_new
6            XE XE_new
```

##### `spark.lapply`

This final function is also from SparkR.  It accepts a list and then uses Spark to apply R code to each element in the list across the cluster.  As [the docs](https://spark.apache.org/docs/latest/api/R/spark.lapply.html) state, it is conceptually similar to `lapply` in base R, so it will return a **list** back to the driver.  

For this example we'll take a list of strings and manipulate them in parallel, somewhat similar to the examples we've seen so far.

```r
# Create list of strings
carriers <- list("UA", "AA", "NW", "EV", "B6", "DL",
                 "OO", "F9", "YV", "AQ", "US", "MQ",
                 "OH", "HA", "XE", "AS", "CO", "FL",
                 "WN", "9E")

list_of_dfs <- spark.lapply(carriers, 
                            function(e) {
                              data.frame(UniqueCarrier = e,
                                         newcol = paste0(e, "_new"))
                            })

# Convert the list of small data.frames into a tidy single data.frame
tidied <- data.frame(t(sapply(list_of_dfs, unlist)))

head(tidied)
```
```
  UniqueCarrier newcol
1            AA AA_new
2            B6 B6_new
3            OO OO_new
4            YV YV_new
5            HA HA_new
6            XE XE_new
```

%md
___

### Leveraging Packages in Distributed R

As stated above, everything required for your function in `spark_apply` needs to be passed along with it.  On Databricks you can install packages on the cluster and they will automatically be installed on each worker.  This saves you time and gives you two options to use libraries in `spark_apply` on Databricks:
<br><br>

* Load the entire library - `library(broom)`
* Reference a specific function from the library namespace - `broom::tidy()`

In a less trivial example of `spark_apply`, we can train a model on each partition. We begin by grouping the input data by `Origin`, then specify our function to apply.  This will be a simple model where our dependent variable is Arrival Delay (`ArrDelay`) and our independent variable is Departure Delay (`DepDelay`) from the month of December.  Furthermore, we can use the `broom` package to tidy up the output of our linear model.  The results will be a Spark DataFrame with different coefficients for each group.

**Note:** Make sure you attach the `broom` package to your cluster before you run the next cell.

```r
library(broom)

## Input columns we need for December only
featuresDF <- dplyr::filter(subsetDF, Month == 12) %>%
                dplyr::select(Origin, DepDelay, ArrDelay)

## Group the flights data by Origin, then estimate the Arrival Delay based on the Departure Delay
coefDF <- sparklyr::spark_apply(featuresDF,
                               group_by = "Origin",
                               function(e){
                                 # e = one_origin_df
                                 # e = an R dataframe associated with ONE distinct origin.
                                 e$ArrDelay <- as.numeric(e$ArrDelay)
                                 e$DepDelay <- as.numeric(e$DepDelay)
                                 broom::tidy(lm(ArrDelay ~ DepDelay, data = na.omit(e)))
                               },
                               packages = F)

head(coefDF)
```
```
# Source: spark<?> [?? x 6]
  Origin term        estimate std_error statistic  p_value
  <chr>  <chr>          <dbl>     <dbl>     <dbl>    <dbl>
1 BGM    (Intercept)    1.59    2.90        0.550 5.85e- 1
2 BGM    DepDelay       0.991   0.0194     51.0   8.40e-43
3 PSE    (Intercept)   -0.456   1.05       -0.433 6.66e- 1
4 PSE    DepDelay       0.962   0.0294     32.8   2.48e-62
5 MSY    (Intercept)   -1.23    0.241      -5.11  3.36e- 7
6 MSY    DepDelay       0.981   0.00663   148.    0.      
```

## Apache Arrow

[Apache Arrow](https://arrow.apache.org/) is a project that aims to improve analytics processing performance by representing data in-memory in columnar format and taking advantage of modern hardware.  The main purpose and benefit of the project can be summed up in the following image, taken from the homepage of the project.

<img src="https://github.com/kurlare/misc-assets/blob/master/apache-arrow.png?raw=true">

Arrow is highly effective at speeding up data transfers.  It's worth mentioning that [Databricks Runtime offers a similar optimization](https://databricks.com/blog/2018/08/15/100x-faster-bridge-between-spark-and-r-with-user-defined-functions-on-databricks.html) out of the box with SparkR.  This is not currently available for `sparklyr`, so if you want to use that API it's recommended to [install Arrow](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/arrow.md#installing-arrow).

___

This concludes the lesson on UDFs with Spark in R.  If you want to learn more, here are additional resources about distributed R with Spark.

1. [100x Faster Bridge Between R and Spark on Databricks](https://databricks.com/blog/2018/08/15/100x-faster-bridge-between-spark-and-r-with-user-defined-functions-on-databricks.html)
2. [Shell Oil: Parallelizing Large Simulations using SparkR on Databricks](https://databricks.com/blog/2017/06/23/parallelizing-large-simulations-apache-sparkr-databricks.html)
3. [Distributed R Chapter from 'The R in Spark'](https://therinspark.com/distributed.html)

*** 
