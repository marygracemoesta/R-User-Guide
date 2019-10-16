
# Customizing with Init Scripts

In addition to the various flavors of Databricks Runtime that can be selected at cluster creation, users can supply initialization scripts to bootstrap the environment in various ways.  This document will serve as a repository to easily access scripts that perform various tasks. 


##### Contents

* [Attaching Init Scripts](#attaching-init-scripts)
* [Example Scripts](#example-scripts)
  * [RStudio Server Installation](#rstudio-server-installation)
  * [Apache Arrow Installation](#apache-arrow-installation)
  * [Library Installation](#library-installation)
  * [Running an R Script](#running-an-r-script)

____

### Attaching Init Scripts

To attach an init script to a cluster, you'll need to edit the advanced options section of the Cluster UI.  Under the _Init Scripts_ tab, you'll be able to specify a path to the script on DBFS.  

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Databricks_Architecture_Overview/images/init_script_example.png?raw=true" height=600 width=500>

Upon startup, the cluster will run any scripts at that location.  

### Example Scripts

#### RStudio Server Installation

To work with RStudio on Databricks, you'll need to run the follow
