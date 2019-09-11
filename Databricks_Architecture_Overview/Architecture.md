# Databricks Architecture Overview
As a foundation to working with R on Databricks, it's important to understand where R fits into the overall architecture of a Databricks cluster.  In this section we'll review core concepts of Spark clusters, as well as the case of single node data science with R.

**Contents**

* [Cluster Computing with Spark](#cluster-computing-with-spark)
  * [Spark Architecture](#spark-architecture)
  * [R on Spark](#r-on-spark)
* [Single Node R on Databricks](#single-node-r-on-databricks)


___

### Cluster Computing with Spark and R 
When the size of your data will no longer fit in memory on a single node, it's time to turn to Spark.  Spark is a distributed, in memory processing engine with a rich functionality for data engineering and data science that can scale to petabytes of data.  Luckily for R users, there are two APIs for accessing Spark - `SparkR` and `sparklyr`.  For now we will focus on the common architecture between the two, but if you want more detail on the differences between the two see [this section](linktocome).

## Spark Architecture

The diagram below shows an example Apache Spark cluster, consisting of one `Driver` node and four `Executor` nodes. Each of these Executor nodes have `slots` which are logical equivalent to individual execution cores of a single machine.

<img src="http://training.databricks.com/databricks_guide/gentle_introduction/videoss_logo.png" width=850 height=480> <br><br>

The Driver sends **Tasks** to the empty slots on the Executors when there are remaining tasks to be completed:

<img src="http://training.databricks.com/databricks_guide/gentle_introduction/spark_cluster_tasks.png" width=850 height=400><br>

At a high level, every Apache Spark application consists of a driver process that launches various parallel operations on executor JVMs.  These executors can run either on multiple machines in a cluster or on the same machine locally.  Driver programs access Apache Spark through a `SparkSession` object regardless of deployment model.  In Databricks, the **Notebook environment** is essentially the `driver` program.  This driver program contains the main loop for the current Spark application and creates distributed datasets on the cluster, then applies operations (transformations & actions) to those datasets.

The details of your Apache Spark application can be viewed in the `Spark Web UI`.  The web UI is accessible in Databricks by going to "Clusters" and then clicking on the "View Spark UI" link. Alternatively, you can click at the top left of a notebook where you would select the cluster to view its Spark Web UI.

## R on Spark

At a high level, here is how an R session interacts with a Spark cluster:

1. R-JVM bridge in the driver - this bridge allows for R jobs to be submitted to a Databricks cluster
2. Launching R processes on executors - the process is key to achieve parallelism in SparkR and sparklyr

<p align="center">
<img src="https://databricks-yong-star.s3.amazonaws.com/graphics/SparkR_Architecture.png" width=500 height=300>
</p>

Beyond that SparkR and sparklyr are just packages that provide an interface to interact with Spark in an R process


### Single Node R on Databricks
If your data set will fit in memory on a single reasonably sized machine, you may not need to use Spark.  In these cases you can provision a 'cluster' with 0 workers and one driver.  This configuration provides a single node to develop and execute R programs on Databricks. 

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Databricks_Architecture_Overview/images/single_node_cluster.png?raw=true">

The architecture here is quite simple - a single virtual machine in the cloud with [Databricks Runtime](DBRlink).  While Spark is inaccessible in this architecture, you can still use many of the benefits of Databricks: RStudio, Notebooks, Libraries, and DBFS as the persistent storage layer. 

# Single Node R vs. Distributed R
The distinguishing difference between single node R and distributed R is parallelism. Single node R (i.e. anything written in base R, 
tidyverse, etc.) is run on a single machine (sometimes referred to as a single driver), whether that be in the locally or in the cloud.
Distributed R is run not only on a driver, but also on several other instances called workers. The driver sends several 
tasks to the workers in parallel. Because these tasks are happening in parallel, often times performance is increased 
when using distributed R rather than single node R. Databricks supports the use of **both** single node and distributed R.  
