# Package Management on Databricks

Databricks supports a variety of options for installing and managing new, old, and custom R packages.  In this chapter we'll begin by providing examples of the basic approaches, then progress into more advanced options.

## Installing Packages

#### Notebook Scope

At the most basic level, you can install R packages in your notebooks and RStudio scripts using the familiar `install.packages()` function. 

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/installpackages.png?raw=true">

This will install the package on the driver node **only**.  

#### Cluster Scope

Under the 'Libraries' tab in the Clusters UI you can attach packages to the cluster.  

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/attach_library_clusters_ui.png?raw=true">

Each time the cluster is started these packages will be installed on *both* driver and worker nodes.  This is important for when you want to perform [user defined functions](insertlink) with `SparkR` or `sparklyr`.

#### Older Package Versions

Each release of Databricks Runtime includes a set of pre-installed popular R packages.  These are typically the latest stable versions but sometimes installing the latest version of a package can break your code.  

For instance, in [DBR 5.5, the version of `dplyr` is 0.8.0.1](https://docs.databricks.com/release-notes/runtime/5.5.html#installed-r-libraries) but what if your code won't run unless the installed version is 0.7.4?  How do you go about installing an older version of a package on Databricks?

At the notebook or script level there are [several ways](https://support.rstudio.com/hc/en-us/articles/219949047-Installing-older-versions-of-packages) to install older versions of packages.  One quick way is the `install_version()` function from `devtools`.

```R
require(devtools)
install_version("dplyr", version = "0.7.4", repos = "http://cran.us.r-project.org")
```

To install an older package at the cluster scope, use a snapshot from the [Microsoft R Application Network](https://mran.microsoft.com/) (MRAN).  MRAN saves the contents of CRAN on a daily basis and stores them as snapshots.  Packages pulled from a specific date will contain the latest version of the package available on that date.  For version 0.7.4 of `dplyr`, we would have to go back to the snapshot from December 19, 2015 and use that URL as the repository in the Cluster UI.

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/install_version.png?raw=true">

Checking the package version on our cluster after installing from this MRAN snapshot, we see the correct version:

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/install_dplyr.png?raw=true">

In this way we can achieve greater customization of the packages on our cluster, even overwriting the versions pre-installed with Databricks Runtime.

##### Custom Packages

* Upload to DBFS through CLI
* Install from source

##### Faster Package Loads

_...you will be fastest if you avoid doing the work in the first place._ [[1]](http://dirk.eddelbuettel.com/blog/2017/11/27/#011_faster_package_installation_one)

* Problem statement
* How CRAN/MRAN works for windows/mac/linux
* How R looks for libraries
* Building a repo on DBFS
* Setting the library path for faster package loads


##### Databricks Container Services

* Idea of DCS
* Examples of R Dockerfiles 
