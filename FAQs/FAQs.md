# Using Spark with R - Some common questions & answers


## I'm new to Spark. Should I use SparkR or sparklyr?

Spark on R is different to the Python and Scala APIs in that users have to choose between two distinct APIs when writing Spark code. SparkR is the API set that is part of the main Apache Spark project, whilst sparklyr was originated and continues to be maintained by the team behind RStudio.

Your choice of API should be informed by what you are trying to do and you're existing skill set:
### SparkR
- Good for data engineering tasks - has the full suite of Spark's data engineering APIs available.
- In Databricks, SparkR benefits from the custom enhancements made to Spark in the Databricks Runtime.
- Lacks full support of Spark MLib - in particular missing the feature engineering functions. 

### sparklyr
- Good support for data science use cases; full set of feature engineering and distributed ML algorithms. 
- Compatibility with dplyr syntax reduces learning curve and makes it easy to switch between writing distributed queries on Spark and vanilla queries on R dataframes.
- Lacks full support of Spark APIs without extension. For example, SparkR's `to_json()` has no equivalent in sparklyr, whilst dealing with nested data requires the package `sparklyr.nested`.

In summary, choosing whether to use sparklyr or SparkR should depend on what you are trying to do and may call for flexibility even within the same project. Both projects are open source allowing users to inspect and contribute to the code base, both have good documentation and user communities for seeking help.

## Is there a performance difference between sparklyr and SparkR?
On Databricks there is an out of the box performance optimization when transferring data between SparkR and R. This works when collecting data, pushing data to Spark, or running UDFs.

With sparklyr you can get the same benefits by installing Apache Arrow [link to install guide](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/arrow.md#installing-arrow). This has been observed to give considerable performance benefits, typically of an order of magnitude, i.e. 10minutes without arrow to 5 seconds with arrow. Please see the previous link for some benchmarks.

Anecdotally, we have observed some performance issues when using windowing functions with sparklyr compared to SparkR. We've also observed that UDFs are more scalable and stable with SparkR.

## Can Spark make everything faster? How do I parellelise workloads?
Spark is not magic - taking a long running piece of code and executing it on Spark without any changes will result in a long running piece of code running on Spark. There are 2 things to think about when parallelising workloads: running actual Spark code (through SparkR or sparklyr) or using Spark as a distribution engine for spreading calculations across a cluster. Writing Spark code is already parallelised and can be optimised by the developer, but does not require anything in particular to parallelise code. 

Using Spark as a distribution engine allows developers to parallelise their existing workloads through functions such as `sparklyr::spark_apply()` or `SparkR::gapply()`. These functions allow you to pass vanilla R code for Spark to execute on its worker nodes, effectively running many processes at once. This is known as using a UDF - a User Defined Function. For more details see [here](https://spark.apache.org/docs/latest/api/R/index.html) for SparkR, and [here](https://spark.rstudio.com/guides/distributed-r.html) for sparklyr. Please also see [this article](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/udfs.md) in this repo for further guidance.

Not all workloads are suitable for this parallelisation however. The characteristic of a task that is well suited to paralellisation is that it can be broken down into independent subtasks. Imagine training a model on a 100 different subsets of data - you could train one model at a time, or 100 all at once and still get the same result. Workloads with complex dependencies are harder to parallelise. Consider any iterative algorithm where the input of task `n` is the result of task `n-1`. Such algorithms require reformulating before they can be parallelised.

## How do I troubleshoot?
If you are working with SparkR in a Databricks notebook, error messages will be printed as they occur. This makes debugging R on Spark much the same as debugging R usually; read the error message and figure out what is going wrong. If using sparklyr, to find the error message requires investigating the worker logs. The worker logs can be found by navigating to the Executors tab after clicking through to the details of the failed job. If using RStudio, you'll need to navigate to the SparkUI tab of the cluster's page in Databricks and find the executor's logs.

Troubleshooting is most difficult when using Spark as a distribution engine. Because each parallel run happens in an ephemeral R session on the worker nodes, the root cause of an 
error sometimes fails to make it to the main driver logs, which requires digging into the worker logs to understand why your code is failing. If working in a notebook, the worker logs can be found by navigating to the Executors tab after clicking through to the details of the failed job. If using RStudio, you'll need to navigate to the SparkUI tab of the cluster's page in Databricks and find the executor's logs.


## Best practice - things to try and do
- Do all of your data engineering work with SparkR/sparklyr, and only drop data back into an R dataframe when you need to use packages from the R ecosystem on a small dataset that will fit into a single computer's memory. 
- When developing a UDF, start small and continually test. Start with a simple function and add complexity, continually running and testing your function as you go. When you introduce a bug, this will make it easy to know which line of code is responsible, rather than debugging a whole script. 
- Use native SparkR or sparklyr functionality when possible - it will run faster due to the optimisation engine. Avoid UDFs unless necessary. 
- If using UDFs, turn off Adaptive Query Execution.
	- sparklyr: `spark_session_config(sc, "spark.databricks.optimizer.enabled", F)`
	- SparkR: `sparkR.session(sparkConfig = list("spark.databricks.optimizer.adaptive.enabled" = F))`

## Worst practice - things to try and avoid
- The most common error is an Out of Memory (OOM) error. This happens when you try to put to too much data into a single process. Below are some examples that can cause OOM, but if this occurs to you, think in general terms about the movement of data through your code and if you are attempting to put too much in one place.
	- Using `collect()` on large datasets. Do you *really* need all the data on the driver? This loses all the benefits of Spark. A leading cause of Out of Memory errors.
	- When performing aggregations, consider the amount of data involved and what the operation is. For example, if using the `collect_list()` function, be aware that it will attempt to build a list that exists on a single worker. If you try to build a list of an entire 10GB table's column, this will most likely cause an Out of Memory error. 