# Package Management on Databricks

Databricks supports a variety of options for installing and managing new, old, and custom R packages.  In this chapter we'll begin by providing examples of the basic approaches, then progress into more advanced options.

Contents

* [Installing Packages](#installing-packages)
  * [Notebook Scope](#notebook-scope)
  * [Cluster Scope](#cluster-scope)
  * [Older Package Versions](#older-package-versions)
  * [Custom Packages](#custom-packages)
* [Faster Package Loads](#faster-package-loads)
* [Databricks Container Services](#databricks-container-services)

## Installing Packages

### Notebook Scope

At the most basic level, you can install R packages in your notebooks and RStudio scripts using the familiar `install.packages()` function. 

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/installpackages.png?raw=true">

This will install the package on the driver node **only**.  

### Cluster Scope

Under the 'Libraries' tab in the Clusters UI you can attach packages to the cluster.  

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/attach_library_clusters_ui.png?raw=true">

Each time the cluster is started these packages will be installed on *both* driver and worker nodes.  This is important for when you want to perform [user defined functions](insertlink) with `SparkR` or `sparklyr`.

### Older Package Versions

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

### Custom Packages

To install a custom package on Databricks, first [build](https://kbroman.org/pkg_primer/pages/build.html) your package from the command line locally or [using RStudio](http://r-pkgs.had.co.nz/package.html).  Next, use the [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html) to copy the file to DBFS:

```bash
databricks fs cp /local_path_to_package/custom_package.tar.gz dbfs:/path_to_package/
```

Once you have the `tar.gz` file on DBFS, you can install the package using `install.packages()`.

```R
## In R
install.packages("/dbfs/path_to_package/custom_package.tar.gz", type = "source", repos = NULL)
```
This can also be done in a `%sh` cell:

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/install_custom_sh.png?raw=true">

If you want to install the custom package on each node in the cluster you will need to use an [init script](linktocome).

## Faster Package Loads

_...you will be fastest if you avoid doing the work in the first place._ [[1]](http://dirk.eddelbuettel.com/blog/2017/11/27/#011_faster_package_installation_one)

Attaching dozens of packages can significantly extend the time it takes for your cluster to come online or for your job to complete.  To understand why this happens, we need to take a closer look at where packages come from.  

#### What is slowing us down?

CRAN stores packages in 3 different formats: Mac, Windows, and source.  

The default behavior of `install.packages()` is to download the package binaries for your operating system, _if they are available_.  If they aren't available R will instead download the package source files from CRAN in `packageName.tar.gz`format.  

Binaries can be installed into your library directly, while source files need to be compiled first.  Windows and Mac users will usually be able to skip compiling, shortening overall installation time.  Linux users will almost always have to compile from source.  This extra time spent compiling adds up quickly when installing many packages.

There are therefore two problems to overcome with regard to performance.  First, since Databricks Runtime uses Linux you are always installing packages on CRAN from source.  Second, a Databricks cluster terminates when not actively in use, taking all of the installed packages down with it.  Every time we spin those machines up, we start from scratch and perform the work of downloading, compiling, and installing all over again. 

#### Getting the Best Performance on Databricks

To avoid repeating this work every time we turn on a cluster we will need to persist the installed packages to a directory on DBFS.  This will be a one time operation, and afterwards we can simply instruct R to look for the packages on that path. 

##### Installing Packages on DBFS



* How R looks for libraries
* Building a repo on DBFS
* Setting the library path for faster package loads


## Databricks Container Services

* Idea of DCS
* Examples of R Dockerfiles 
