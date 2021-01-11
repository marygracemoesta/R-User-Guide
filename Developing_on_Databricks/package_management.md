# Package Management on Databricks

Databricks supports a variety of options for installing and managing new, old, and custom R packages.  In this chapter we'll begin by providing examples of the basic approaches, then progress into more advanced options.

**Contents**

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

If the package source is located on github (or other version control systems), you can make use of the `devtools` package to install directly from the source repository:

```R
require(devtools)
install_github("tidyverse/dplyr")
```
[See the devtools documentation for more details](https://www.rdocumentation.org/packages/devtools)

If you want to install the custom package on each node in the cluster you will need to use an [init script](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/Customizing.md#library-installation).

## Faster Package Loads

_...you will be fastest if you avoid doing the work in the first place._ [[1]](http://dirk.eddelbuettel.com/blog/2017/11/27/#011_faster_package_installation_one)

Attaching dozens of packages can significantly extend the time it takes for your cluster to come online or for your job to complete.  To understand why this happens, we need to take a closer look at where packages come from.  

### What is slowing us down?

CRAN stores packages in 3 different formats: Mac, Windows, and source.  

The default behavior of `install.packages()` is to download the package binaries for your operating system, _if they are available_.  If they aren't available R will instead download the package source files from CRAN in `packageName.tar.gz`format.  

Binaries can be installed into your library directly, while source files need to be compiled first.  Windows and Mac users will usually be able to skip compiling, shortening overall installation time.  Linux users will almost always have to compile from source.  This extra time spent compiling adds up quickly when installing many packages.

There are therefore two problems to overcome with regard to performance.  First, since Databricks Runtime uses Linux you are always installing packages on CRAN from source.  Second, a Databricks cluster terminates when not actively in use, taking all of the installed packages down with it.  Every time we spin those machines up, we start from scratch and perform the work of downloading, compiling, and installing all over again. 

### Getting the Best Performance on Databricks

To get better performance we need to avoid doing all that work in the first place!  We can accomplish this by ultimately persisting our installed packages in a library on DBFS. 

### Building a Library on DBFS

All packages are installed into a _library_, which is located on a path in the file system.  You can check the what directories are recognized by R as libraries with `.libPaths()`.  

<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/defaul_libpaths.png?raw=true"> 

When you call `library(dplyr)` R will search for the package in the libraries listed under `.libPaths()`, starting at the first path and if the package isn't found continue searching through the rest of the directories in order.  You can add and remove paths from `.libPaths()`, effectively telling R where to look for packages.   Let's create a directory on the driver to install packages into and add it to the list of libraries.



```bash
%sh
mkdir /usr/lib/R/proj-lib-test
```

```R
## Append our library to the list
.libPaths(c("/usr/lib/R/proj-lib-test", .libPaths()))
```

Now every package we install will write to that directory on the driver.  Once you have the packages and versions you want in that directory, **copy them to DBFS to persist.**

```R
## Copy to DBFS
system("cp -R /usr/lib/R/proj-lib-test /dbfs/path_to_r_library", intern = T)
```
Now when the cluster is terminated, the packages will remain.  

See [this page](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/notebooks/faster_package_loads.md) for a more thorough example. 

### Setting the Library Path

Assuming you have installed your desired packages into a library on DBFS, you can begin using them with one command. 

```R
## Add library to search path
.libPaths(c("/dbfs/path_to_r_library", .libPaths()))

## Load package, no need to install again!
library(custom_package)
```

If you need these libraries to be available to each worker, you can use an [init script](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/Customizing.md#add-a-library-at-startup) or simply include it in your closure when running [user defined functions](linktocome).

## Databricks Container Services

[Databricks Container Services](https://docs.databricks.com/clusters/custom-containers.html) (DCS) lets you specify a Docker image to run across your cluster.  This can be built off of the base Databricks Runtime image, or it can be entirely customized to your specifications.  DCS will give you the ultimate control and flexibility when it comes to locking down packages and their versions in an execution environment.  

Here is an example R Dockerfile pulled from [the official git repo for DCS](https://github.com/databricks/containers) that grabs the latest version of R from RStudio and also installs RStudio Server:

```
FROM databricksruntime/minimal:latest

# Ubuntu 16.04.3 LTS installs R version 3.2.3 by default. This is fairly out dated.
# We add RStudio's debian source to install the latest r-base version (3.6.0)
# We are using the more secure long form of pgp key ID of marutter@gmail.com
# based on these instructions (avoiding firewall issue for some users):
# https://cran.rstudio.com/bin/linux/ubuntu/#secure-apt
RUN apt-get update \
  && apt-get install --yes software-properties-common apt-transport-https \
  && gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 \
  && gpg -a --export E298A3A825C0D65DFD57CBB651716619E084DAB9 | sudo apt-key add - \
  && add-apt-repository 'deb [arch=amd64,i386] https://cran.rstudio.com/bin/linux/ubuntu xenial-cran35/' \
  && apt-get update \
  && apt-get install --yes \
    libssl-dev \
    r-base \
    r-base-dev \
  && add-apt-repository -r 'deb [arch=amd64,i386] https://cran.rstudio.com/bin/linux/ubuntu xenial-cran35/' \
  && apt-key del E298A3A825C0D65DFD57CBB651716619E084DAB9 \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# hwriterPlus is used by Databricks to display output in notebook cells
# Rserve allows Spark to communicate with a local R process to run R code
RUN R -e "install.packages(c('hwriterPlus'), repos='https://mran.revolutionanalytics.com/snapshot/2017-02-26')" \
 && R -e "install.packages(c('htmltools'), repos='https://cran.microsoft.com/')" \
 && R -e "install.packages('Rserve', repos='http://rforge.net/')"

# Additional instructions to setup rstudio. If you dont need rstudio, you can 
# omit the below commands in your docker file. Even after this you need to use
# an init script to start the RStudio daemon (See README.md for details.)

# Databricks configuration for RStudio sessions.
COPY Rprofile.site /usr/lib/R/etc/Rprofile.site

# Rstudio installation.
RUN apt-get update \
 # Installation of rstudio in databricks needs /usr/bin/python.
 && apt-get install -y python \
 # Install gdebi-core.
 && apt-get install -y gdebi-core \
 # Download rstudio 1.2 package for ubuntu 16.04 and install it.
 && apt-get install -y wget \
 && wget https://download2.rstudio.org/server/trusty/amd64/rstudio-server-1.2.5042-amd64.deb -O rstudio-server.deb \
 && gdebi -n rstudio-server.deb \
 && rm rstudio-server.deb
 ```
 
Notice how the middle section illustrates how to incorporate package installation in the image, including setting the version of the package.
___
[Back to table of contents](https://github.com/marygracemoesta/R-User-Guide#contents)

