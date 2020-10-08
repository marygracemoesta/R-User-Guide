
# Customizing with Init Scripts

In addition to the various flavors of [Databricks Runtime](https://github.com/marygracemoesta/R-User-Guide/blob/master/Databricks_Architecture_Overview/DB_Runtime.md#databricks-runtimes) that can be selected at cluster creation, users can supply initialization scripts to bootstrap the environment in various ways.  This document will serve as a repository to easily access scripts that perform various tasks. 


##### Contents

* [Creating & Attaching Init Scripts](#creating-&-attaching-init-scripts)
* [Example Scripts](#example-scripts)
  * [RStudio Server Installation](#rstudio-server-installation)
  * [Apache Arrow Installation](#apache-arrow-installation)
  * [Library Installation](#library-installation)
  * [Running an R Script](#running-an-r-script)
  * [Modifying Rprofile in RStudio](#modifying-rprofile-in-rstudio)
  * [Add a Library at Startup](#add-a-library-at-startup)

____

### Creating & Attaching Init Scripts

An initialization script is simply a bash script on DBFS that is executed at cluster startup.  These are fairly flexible, and you can learn more in the official documentation [here](https://docs.databricks.com/clusters/init-scripts.html#init-script-types).  You can upload the file to DBFS through the Databricks CLI, or you can write the file to DBFS using `dbutils` in a Databricks Notebook.  The examples below will be using `dbutils` in Python. 

Once you have the script and are ready to attach it to a cluster, you'll need to edit the Advanced Options section of the cluster UI.  Under the _Init Scripts_ tab, you'll be able to specify a path to the script on DBFS.  

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Getting_Started/images/init_script_example.png?raw=true" height=600 width=500>

Now the cluster will run the script every time it is restarted.

### Example Scripts

#### RStudio Server Installation

To work with RStudio on Databricks, you'll need to run the following code in a **Python** cell in a notebook. 

```python
%python

## Define the contents of the script
script = """
  if [[ $DB_IS_DRIVER = "TRUE" ]]; then
    sudo apt-get update
    sudo apt-get install -y gdebi-core alien
    cd /tmp
    sudo wget https://www.rstudio.org/download/latest/stable/server/${UBUNTU_CODENAME}/rstudio-server-latest-amd64.deb
    sudo gdebi -n rstudio-server-*-amd64.deb
    sudo rstudio-server restart
    exit 0
  else
    exit 0
  fi
"""

## Create directory to save the script in
dbutils.fs.mkdirs("/databricks/rstudio")

## Save the script to DBFS
dbutils.fs.put("/databricks/rstudio/rstudio-install.sh", script, True)
```

Note the path is `/databricks/rstudio/rstudio-install.sh`.  This is what we will add to the _Init Script_ path in the Advanced Options section of the cluster UI.


#### Apache Arrow Installation

Apache Arrow is an open source project that provides a common in-memory format between different processes.  This is especially useful when moving data back and forth between Spark and R, as you would with [user defined functions](linktocome).  **To enable Arrow with R on Databricks Runtimes prior to 7.0**, the first step is to attach an init script to a cluster.  Using a Python cell in a Databricks Notebook, run the following cell:

```python
%python

## Define contents of the script
script = """
#!/bin/bash
git clone https://github.com/apache/arrow
cd arrow/r
R CMD INSTALL .
sudo su - -c "R -e 'arrow::install_arrow()'"
"""

## Create directory to save the script in
dbutils.fs.mkdirs("/databricks/arrow")

## Save the script to DBFS
dbutils.fs.put("/databricks/arrow/arrow-install.sh", script, True)
```

Note the path is `/databricks/arrow/arrow-install.sh`. This is what we will add to the Init Script path in the Advanced Options section of the cluster UI.  After the init script completes simply include `library(arrow)` at the top of your R code and you are good to go.  

**To enable Arrow for Databricks Runtime 7.0+** attach the Arrow package to the cluster through the Cluster UI then set the following Spark config under 'Advanced Options' in the Cluster UI:  

`spark.sql.execution.arrow.sparkr.enabled true`

For more details please see the [Apache Arrow with R](https://github.com/marygracemoesta/R-User-Guide/blob/master/Spark_Distributed_R/arrow.md) section.

#### Library Installation
One quick way to install a list of packages on a cluster is through an init script.  At a certain point you may see long cluster startup times as R has to download, compile, and install the packages.  See the [Faster Package Loads](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/package_management.md#faster-package-loads) section for an alternative solution.

Using a Python cell in a Databricks Notebook run the following code, making sure to add your list of packages using `install.packages()` as shown.

```python
%python

## Define contents of script 
script = """
#!/bin/bash
R --vanilla <<EOF 
install.packages('forecast', repos="https://cran.microsoft.com/snapshot/2017-09-28/")
install.packages('sparklyr', dependencies=TRUE, repos="https://cran.microsoft.com/snapshot/2017-09-28/")
q()
EOF
"""

## Create directory to save the script in
dbutils.fs.mkdirs("/databricks/rlibs")

## Save the script to DBFS
dbutils.fs.put("/databricks/rlibs/r-library-install.sh", script, True)
```

Note the path is `/databricks/rlibs/r-library-install.sh`.  This is what we will add to the _Init Script_ path in the Advanced Options section of the cluster UI.

#### Running an R Script
If you want to run an arbitrary R script on cluster startup, you can do that too. Using a Python cell in a Databricks Notebook, run the following code, replacing `<INSERT R SCRIPT>` with ... your R script!

```python
%python

## Define contents of script 
script = """
#!/bin/bash
R --vanilla <<EOF 
<INSERT R SCRIPT>
q()
EOF
"""

## Create directory to save the script in
dbutils.fs.mkdirs("/databricks/rscripts")

## Save the script to DBFS
dbutils.fs.put("/databricks/rscripts/my-r-script.sh", script, True)
```

Note the path is `/databricks/rscripts/my-r-script.sh`.  This is what we will add to the _Init Script_ path in the Advanced Options section of the cluster UI.

#### Modifying Rprofile in RStudio
Customizing your Rprofile is great way to enhance your R experience.  You can recreate this experience on Databricks by leveraging an init script to modify your `Rprofile.site` file.  Essentially you will pipe an R script into the `Rprofile.site` file, appending changes to it.  **Note:**  This will only work on RStudio, not in an R Notebook.  

As a prerequisite, upload a R file containing the modifications you'd like to make to DBFS.  In this example the file is called `r_profile_changes.R`  Then run the following code in Databricks Notebook using Python:

```python
%python

## Define contents of script
script = """
#!/bin/bash
if [[ $DB_IS_DRIVER = "TRUE" ]]; then
echo 'source("/dbfs/R/scripts/r_profile_changes.R")' >> /usr/lib/R/etc/Rprofile.site
fi
"""

## Create directory to save the script in
dbutils.fs.mkdirs("/databricks/rscripts")

## Save the script to DBFS
dbutils.fs.put("/databricks/rscripts/modify-rprofile.sh", script, True)
```
Note the path is `/databricks/rscripts/modify-rprofile.sh`.  This is what we will add to the _Init Script_ path in the Advanced Options section of the cluster UI.

#### Add a Library at Startup

If you are using the [Faster Package Loads](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/package_management.md#faster-package-loads) method, you may find yourself adding extra lines of code for each script to set the `.libPaths()` to include your directory on DBFS.  To avoid adding these lines in each script or UDF, you can use an init script to copy over your R packages to the default library path for R - `/databricks/spark/R/lib`.

```python
%python

## Define contents of script 
script = """
#!/bin/bash
R --vanilla <<EOF 
system("cp -R /dbfs/path_to_r_library /databricks/spark/R/lib/", intern = T)
q()
EOF
"""

## Create directory to save the script in
dbutils.fs.mkdirs("/databricks/rscripts")

## Save the script to DBFS
dbutils.fs.put("/databricks/rscripts/copy_library.sh", script, True)
```

You can use any of the four default paths for R packages in Databricks Runtime, but the first - `/databricks/sparkR/lib` - is nice because the only package in it is SparkR.  This essentially gives you an empty directory to copy your pre-compiled packages to, removing the need to overwrite any other packages such as those found in the other paths.

___
[Back to table of contents](https://github.com/marygracemoesta/R-User-Guide#contents)
