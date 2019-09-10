# Databricks Architecture Overview
As a foundation to working with R on Databricks, it's important to understand where R fits into the overall architecture of a Databricks cluster.  In this section we'll review core concepts of Spark clusters, as well as the case of single node data science with R.

**Contents**

* Cluster Computing with Spark and R
* Single Node Data Science with R

### Cluster Computing with Spark and R 
When the size of your data will no longer fit in memory on a single node, it's time to turn to Spark.  Spark is a distributed, in memory processing engine with a rich functionality for data engineering and data science that can scale to petabytes of data.  Luckily for R users, there are two APIs for accessing Spark - `SparkR` and `sparklyr`.  For now we will focus on the common architecture between the two, but if you want more detail on the differences between the two see [this section](linktocome).

At a high level, here is how an R session interacts with a Spark cluster:

1. R-JVM bridge in the driver - this bridge allows for R jobs to be submitted to a Databricks cluster
2. Launching R processes on executors - the process is key to achieve parallelism in SparkR and sparklyr

<p align="center">
<img src="https://databricks-yong-star.s3.amazonaws.com/graphics/SparkR_Architecture.png" width=400 height=300>
</p>

Beyond that SparkR and sparklyr are just packages that provide an interface to interact with Spark in an R process

# Single Node R vs. Distributed R
The distinguishing difference between single node R and distributed R is parallelism. Single node R (i.e. anything written in base R, 
tidyverse, etc.) is run on a single machine (sometimes referred to as a single driver), whether that be in the locally or in the cloud.
Distributed R is run not only on a driver, but also on several other instances called workers. The driver sends several 
tasks to the workers in parallel. Because these tasks are happening in parallel, often times performance is increased 
when using distributed R rather than single node R. Databricks supports the use of **both** single node and distributed R.  
