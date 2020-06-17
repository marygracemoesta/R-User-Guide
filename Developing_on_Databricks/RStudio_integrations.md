## Rstudio Integration

RStudio offers a polished experience for developing R programs that is impossible to reproduce in a notebook.  Databricks offers two choices for sticking with your favorite IDE - hosted in the cloud or on your local machine.  In this section we'll go over setup and best practices for both options.

**Contents**

* [Hosted RStudio Server](#hosted-rstudio-server)
  * [Accessing Spark](#accessing-spark)
  * [Persisting Work](#persisting-work)
  * [Git Integration](#git-integration)
* [RStudio Desktop with Databricks Connect](#rstudio-desktop-with-databricks-connect)
  * [Setup](#connecting-to-spark)
  * [Limitations](#limitations)
 
 ___
 
### Hosted RStudio Server
The simplest way to use RStudio with Databricks is to use Databricks Runtime (DBR) for Machine Learning 7.0+.  The open source version of Rstudio Server will automatically installed on the driver node of a cluster:

<img src="https://spark.rstudio.com/images/deployment/databricks/rstudio-databricks-local.png">
  
___

You can then launch hosted RStudio Server from the 'Apps' tab in the Cluster UI:

<img src ="https://docs.databricks.com/_images/rstudio-apps-ui.png" height = 275 width = 1000>

For earlier versions of DBR you can attach the [RStudio Server installation script](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/Customizing.md#rstudio-server-installation) to your cluster.  Full details for setting up hosted Rstudio Server can be found in the official docs [here](https://docs.databricks.com/spark/latest/sparkr/rstudio.html#get-started-with-rstudio-server-open-source).  If you have a license for RStudio Server Pro you can [set that up](https://docs.databricks.com/spark/latest/sparkr/rstudio.html#install-rstudio-server-pro) as well.

After the init script has been installed on the cluster, login information can be found in the `Apps` tab of the cluster UI as before.

The hosted RStudio experience should feel nearly identical to your desktop experience.  In fact, you can also customize the environment further by supplying an additional init script to [modify `Rprofile.site`](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/Customizing.md#modifying-rprofile-in-rstudio).  

#### Accessing Spark

_(This section is taken from [Databricks documentation](https://docs.databricks.com/spark/latest/sparkr/rstudio.html#get-started-with-rstudio-server-open-source).)_

From the RStudio UI, you can import the SparkR package and set up a SparkR session to launch Spark jobs on your cluster.

```r
library(SparkR)
sparkR.session()
```

You can also attach the sparklyr package and set up a Spark connection.

```r
SparkR::sparkR.session()
library(sparklyr)
sc <- spark_connect(method = "databricks")
```

This will display the tables registered in the metastore.  

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/sparklyr_tables_ui_view.png?raw=true" height=300 width=480>


Tables from the `default` database will be shown, but you can switch the database using `sparklyr::tbl_change_db()`.

```r
## Change from default database to non-default database
tbl_change_db(sc, "not_a_default_db")
```

#### Persisting Work

Databricks clusters are treated as ephemeral computational resources - the default configuration is for the cluster to automatically terminate after 120 minutes of inactivity. This applies to your hosted instance of RStudio as well.  To persist your work you'll need to use [DBFS](https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/DBFS.md) or integrate with version control.  

By default, the working directory in RStudio will be on the driver node.  To change it to a path on DBFS, simply set it with `setwd()`.

```r
setwd("/dbfs/my_folder_that_will_persist")
```

You can also access DBFS in the File Explorer.  Click on the `...` all the way to the right and enter `/dbfs/` at the prompt.

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/rstudio-gotofolder.png" height = 250 width = 450>

Then the contents of DBFS will be available:

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/file_explorer_rstudio_dbfs.png" height=200 width=450>

You can store RStudio Projects on DBFS and any other arbitrary file.  When your cluster is terminated at the end of your session, the work will be there for you when you return.

#### Git Integration

The first step is to disable websockets from within RStudio's options:

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/disable_websockets.png?raw=true" width = 350 height = 350>

Once that is complete, a GitHub repo can be connected by creating a new project from the Project dropdown menu at the top right of RStudio.  Select *Version Control*, and on the next window select the git repo that you want to work with on Databricks.  When you click *Create Project*, the repo will be cloned to the subdirectory you chose on the driver node and git integration will be visible from RStudio.

At this point you can resume your usual workflow of checking out branches, committing new code, and pushing changes to the remote repo.  

#### Persisting R Project Files in DBFS

Instead of GitHub, you can also use the Databricks File System (DBFS) to persist files associated with the R project.  Since DBFS enables users to treat buckets in object storage as local storage by prepending the write path with `/dbfs/`, this is very easy to do with the RStudio terminal window or with `system()` commands.

For example, an entire R project can be copied into DBFS via a single `cp` command.

```r
system("cp -r /driver/my_r_project /dbfs/my_r_project")
```
___

### RStudio Desktop with Databricks Connect
Databricks Connect is a library that allows users to remotely access Spark on Databricks clusters from their local machine.  At a high level the architecture looks like this:

<img src="https://spark.rstudio.com/images/deployment/databricks/rstudio-databricks-remote.png">

##### Setup
Documentation can be found [here](https://docs.databricks.com/dev-tools/databricks-connect.html#databricks-connect), but essentially you will install the client library locally, configure the cluster on Databricks, and then authenticate with a token.  At that point you'll be able to connect to Spark on Databricks  from your local RStudio instance and develop freely.

##### Gotchas 
Something important to note when using the Rstudio integrations:
- Loss of notebook functionality: magic commands that work in Databricks notebooks, do not work within Rstudio
- `dbutils` is not supported
___
[Back to table of contents](https://github.com/marygracemoesta/R-User-Guide#contents)
