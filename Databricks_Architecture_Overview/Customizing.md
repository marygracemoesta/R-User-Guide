
# Customizing with Init Scripts

In addition to the various flavors of Databricks Runtime that can be selected at cluster creation, users can supply initialization scripts to bootstrap the environment in various ways.  This document will serve as a repository to easily access scripts that perform various tasks. 


##### Contents

* [Creating & Attaching Init Scripts](#creating-&-attaching-init-scripts)
* [Example Scripts](#example-scripts)
  * [RStudio Server Installation](#rstudio-server-installation)
  * [Apache Arrow Installation](#apache-arrow-installation)
  * [Library Installation](#library-installation)
  * [Running an R Script](#running-an-r-script)

____

### Creating & Attaching Init Scripts

An initialization script is simply a bash script on DBFS that is executed at cluster startup.  These are fairly flexible, and you can learn more in the official documentation [here](https://docs.databricks.com/clusters/init-scripts.html#init-script-types).  You can upload the file to DBFS through the Databricks CLI, or you can write the file to DBFS using `dbutils` in a Databricks Notebook.  The examples below will be using `dbutils` in Python. 

Once you have the script and are ready to attach it to a cluster, you'll need to edit the Advanced Options section of the cluster UI.  Under the _Init Scripts_ tab, you'll be able to specify a path to the script on DBFS.  

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Databricks_Architecture_Overview/images/init_script_example.png?raw=true" height=600 width=500>

Now the cluster will run the script every time it is restarted.

### Example Scripts

#### RStudio Server Installation

To work with RStudio on Databricks, you'll need to run the following code in a **Python** cell in a notebook. 

```python
%python
script = """
  if [[ $DB_IS_DRIVER = "TRUE" ]]; then
    sudo apt-get update
    sudo apt-get install -y gdebi-core alien
    cd /tmp
    sudo wget https://download2.rstudio.org/rstudio-server-1.1.453-amd64.deb
    sudo gdebi -n rstudio-server-1.1.453-amd64.deb
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
