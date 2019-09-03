# Package Management on Databricks

Databricks supports a variety of options for installing and managing new, old, and custom R packages.  In this chapter we'll begin by providing examples of the basic approaches, then progress into more advanced options.

## Installing Packages

_Notebook Scope_

At the most basic level, you can install R packages in your notebooks and RStudio scripts using the familiar `install.packages()` function. 

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/installpackages.png?raw=true">

This will install the package on the driver node **only**.  

_Cluster Scope_

Under the 'Libraries' tab in the Clusters UI you can attach packages to the cluster.  

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/attach_library_clusters_ui.png?raw=true">

Each time the cluster is started these packages will be installed on *both* driver and worker nodes.  This is important for when you want to perform [user defined functions](insertlink) with `SparkR` or `sparklyr`.

_Older Package Versions_

At the notebook or script level there are [several ways](https://support.rstudio.com/hc/en-us/articles/219949047-Installing-older-versions-of-packages) to install older versions of packages.  For example,

```R
require(devtools)
install_version("ggplot2", version = "0.9.1", repos = "http://cran.us.r-project.org")
```

To install an older package at the cluster scope, you will have to specify the repository it can be found in in the Cluster UI.  Typically this will be a snapshot from the [Microsoft R Application Network](https://mran.microsoft.com/) (MRAN) corresponding to the date when your package version was the latest on CRAN.  

For instance, if you wanted the version of `dplyr` available on 12/19/2015, specify `https://cran.microsoft.com/snapshot/2015-12-19/` as the package repository.

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/install_version.png?raw=true">

Checking the package version on our cluster we get 

##### Faster Package Loads



##### Databricks Container Services
