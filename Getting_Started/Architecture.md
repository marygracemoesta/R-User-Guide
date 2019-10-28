# Databricks Architecture Overview

To be most productive on Databricks, it's important to have a basic understanding of how data flows through the platform and where R fits into the picture.
___

## What Even is Databricks?

Databricks is a cloud native **Unified Data Analytics Platform**, bringing together the work of data engineering, data science, and data analysts to realize business value.  In the following graphic we see a logical and conceptual representation of the platform and how it can be used to develop data products. 

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/images/ref_arch.png?raw=true">

*Storage and Compute*

Infrastructure is managed by Databricks - you should never have to spend much time configuring or debugging VMs in the cloud.  We make it easy to provision large Spark clusters, single nodes, or GPU ready instances.  To optimize performance and ease of use these machines are loaded with one of the [Databricks Runtimes](https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/DB_Runtime.md).  

For storage, Databricks will create a default blob storage bucket that can be used to support your Data Lake, model artifacts, or any other arbitrary files.  No need to make API calls to S3 or ADLS, [DBFS](https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/DBFS.md) lets you relate to this storage as a path on the local file system. 

*ETL & The Data Lake*

Streaming and static data sources can be extracted and ingested into the platform using the petabyte scalability of Spark.  Land **all** of your raw data in reliable and performant [Delta Lake](https://github.com/marygracemoesta/R-User-Guide/blob/master/Delta_Lake/deltaLake.md), then progressively refine and enrich it until you have features ready for analytics and machine learning.  

*Data Science & Machine Learning*

For exploratory data analysis, Databricks Notebooks support the vast open source ecosystem of packages.  Data Analysts can enjoy creating entire notebooks out of SQL, or continue using their existing BI applications and consume from Delta Lake via JDBC/ODBC connectors.  To manage the ML lifecycle, MLflow offers a set of APIs and a UI to track, deploy and monitor models regardless of the underlying library used for training.  

*Developer Experience*

If you prefer working in an IDE (such as RStudio), you can [launch a hosted instance of RStudio](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/RStudio_integrations.md) or configure a secure connection to Databricks through RStudio on your desktop.  Most everything on Databricks can be controlled via REST API or through the CLI.  

*Production Jobs*

Once you have your notebook or script ready for production, it can be scheduled to run on a regular or ad hoc basis.  [Jobs](https://docs.databricks.com/jobs.html#jobs) support retries, parameterization, log delivery, alerts, and more.  

To summarize - the Unified Data Analytics Platform provides the software and infrastructure to bring together and extend the skills of engineers, scientists, and analysts.


