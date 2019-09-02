# Where does R sit in the Spark Architecture?
It is important to understand the underlying architecture before jumping into R on Databricks. In SparkR there are two
main components:
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