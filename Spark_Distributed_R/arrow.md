
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
2. Install `arrow` from CRAN in the Cluster UI.
3. Install `sparklyr` from CRAN in the Cluster UI. 


#### Arrow Benchmarks
