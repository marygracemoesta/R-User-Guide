
## Single Node R vs. Distributed R

**Contents**

* [Single Node R on Databricks](#single-node-r-on-databricks)
* [Cluster Computing with Spark and R](#cluster-computing-with-spark-and-r)
  * [Spark Architecture](#spark-architecture)
  * [R on Spark](#r-on-spark)

___

## Single Node R on Databricks
If your data set will fit in memory on a single reasonably sized machine, you may not need to use Spark.  In these cases you can provision a 'cluster' with 0 workers and one driver.  This configuration provides a single node to develop and execute R programs on Databricks. 

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Databricks_Architecture_Overview/images/single_node_cluster.png?raw=true">

The architecture here is quite simple - a single virtual machine in the cloud with [Databricks Runtime](DBRlink).  While Spark is inaccessible in this architecture, you can still use many of the benefits of Databricks: RStudio, Notebooks, Libraries, and [DBFS](https://github.com/marygracemoesta/R-User-Guide/blob/master/Databricks_Architecture_Overview/DBFS.md#dbfs) as the persistent storage layer.  

## Cluster Computing with Spark and R 
When the size of your data will no longer fit in memory on a single node, it's time to turn to Spark.  Spark is a distributed, in memory processing engine with a rich functionality for data engineering and data science that can scale to petabytes of data.  Luckily for R users, there are two APIs for accessing Spark - `SparkR` and `sparklyr`.  For now we will focus on the common architecture between the two, but if you want more detail on the differences between them see [this section](linktocome).

### Spark Architecture

The diagram below shows an example Apache Spark cluster, consisting of one `Driver` node and four `Executor` or worker nodes. Each of these Executor nodes have `slots` which are logical equivalent to individual execution cores of a single machine.

<img src="http://training.databricks.com/databricks_guide/gentle_introduction/videoss_logo.png" width=850 height=400> <br><br>

In a Spark application, the driver sends **tasks** to the empty slots on the executors when there is work to be completed:

<img src="http://training.databricks.com/databricks_guide/gentle_introduction/spark_cluster_tasks.png" width=850 height=300><br>

Driver programs access Spark through a `SparkSession` object, which is already instantiated for you in Databricks Notebooks.  In a sense, you can think of the notebook environment as the driver.  When working in RStudio (either via Databricks Connect or R Studio Server Integration), you'll have to establish the connection to the SparkSession before submitting work to the cluster.

The details of your Apache Spark application can be viewed in the `Spark Web UI` when working in either Databricks notebooks or R Studio.  The web UI is accessible in Databricks by going to "Clusters" and then clicking on the "View Spark UI" link. Alternatively, you can click at the top left of a notebook where you would select the cluster to view its Spark Web UI.

### Spark with R

Both `SparkR` and `sparklyr` are APIs to access the `SparkContext` and execute commands across a cluster from R.  Each package contains wrappers around the original Scala classes and functions that Spark was written in.  This means that when you use the APIs you are not actually running any R code on the worker nodes.  The R session on the driver communicates with the JVM to send commands out to the JVM processes on each worker.  
1. R-JVM bridge in the driver - this bridge allows for an R user to submit commands to Spark.  
2. Launching R processes on executors - the process is key to achieve parallelism in SparkR and sparklyr

<p align="center">
<img src="https://databricks-yong-star.s3.amazonaws.com/graphics/SparkR_Architecture.png" width=800 height=350>
</p>

Beyond that SparkR and sparklyr are just packages that provide an interface to interact with Spark in an R process


## Single Node R vs. Distributed R
The distinguishing difference between single node R and distributed R is parallelism. Single node R (i.e. anything written in base R, tidyverse, etc.) is run on a single machine (sometimes referred to as a single driver), whether that be in the locally or in the cloud.

Distributed R is run not only on a driver, but also on several other instances called workers. The driver sends several 
tasks to the workers in parallel. Because these tasks are happening in parallel, often times performance is increased 
when using distributed R rather than single node R. Databricks supports the use of **both** single node and distributed R. 


___
[Back to table of contents](https://github.com/marygracemoesta/R-User-Guide#contents)

