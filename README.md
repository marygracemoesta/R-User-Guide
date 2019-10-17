# R on Databricks: A User Guide

R is widely used by data teams in every sector of the economy.  It enjoys a vibrant ecosystem of open source packages including two APIs for working with Apache Spark.  This opens up the tantalizing prospect of scaling R programs to hundreds of machines executing in parallel.   

As Databricks grows in popularity and usage across market segments - especially Enterprise -  R users face a somewhat steep learning curve for both Spark and working in the cloud generally.  This user guide is designed to facilitate a smooth transition to productivity for R developers using Databricks.  Each section has examples in R or discusses a particular feature as it relates to R. 

The flow of a section will progress from basic concepts through to advanced tips and functionality.  As such, this guide can be used as a reference and will complement the [official Databricks documentation](https://docs.databricks.com).  Please reach out to us with questions or suggestions for improvement.  We hope it enables you to unlock the full potential of R on Databricks!  

____

##### Contents

* [**Getting Started**](https://github.com/marygracemoesta/R-User-Guide/tree/master/Getting_Started)
  * [Databricks Architecture Overview](https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/Architecture.md)
  * [Databricks Runtimes](https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/DB_Runtime.md)
  * [Databricks File System (DBFS)](https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/DBFS.md)
* [**Developing on Databricks**](https://github.com/marygracemoesta/R-User-Guide/tree/master/Developing_on_Databricks)
  * [Package Management](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/package_management.md)
  * [Databricks Notebooks](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/db_notebooks.md)
  * [RStudio Integration](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/RStudio_integrations.md)
  * [Customizing With Init Scripts](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/Customizing.md)
* [**Spark: Distributed R**](https://github.com/marygracemoesta/R-User-Guide/tree/master/Spark_Distributed_R)
  * [Single Node vs. Distributed R](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/single_v_distributed.md)
  * [`sparklyr` vs. `SparkR`](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/sparklyr_v_sparkR.md)
  * [Streaming](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/streaming.md)
  * [User-Defined Functions (UDFS)](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/udfs.md)
  * [Apache Arrow](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/arrow.md)
* [**R in Production**](https://github.com/marygracemoesta/R-User-Guide/tree/master/R_in_Production)
  * Jobs
  * Notebook Workflows
  * REST API
  * Migrating Existing Workloads
* [**MLflow**](https://github.com/marygracemoesta/R-User-Guide/tree/master/MLflow)
  * [Tracking](https://github.com/marygracemoesta/R-User-Guide/blob/master/MLflow/tracking.md)
  * [Models & Model Deployment](https://github.com/marygracemoesta/R-User-Guide/blob/master/MLflow/models.md)
  * [Projects](https://github.com/marygracemoesta/R-User-Guide/blob/master/MLflow/projects.md)
* [**Delta Lake**](https://github.com/marygracemoesta/R-User-Guide/blob/master/Delta_Lake/deltaLake.md)
  * Time Travel
  
  ___
