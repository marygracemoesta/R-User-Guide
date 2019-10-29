Faster R Package Loads on Databricks
================
Rafi Kurlansik
10/29/2019

##### Faster R Package Loads on Databricks

*you will be fastest if you avoid doing the work in the first
place*[\[1\]](http://dirk.eddelbuettel.com/blog/2017/11/27/#011_faster_package_installation_one)

In R, you typically download and install a variety of *packages* from
CRAN. These packages are installed into a *library*, which is a path on
the file system. You can check the what directories are recognized by R
as libraries with
    `.libPaths()`

``` r
.libPaths()
```

    ## [1] "/home/rafi.kurlansik@databricks.com"                                  
    ## [2] "/databricks/spark/R/lib"                                              
    ## [3] "/usr/local/lib/R/site-library"                                        
    ## [4] "/usr/lib/R/site-library"                                              
    ## [5] "/usr/lib/R/library"                                                   
    ## [6] "/home/rafi.kurlansik@databricks.com/R/x86_64-pc-linux-gnu-library/3.6"

When you call `library(dplyr)` R will search for the package in the
libraries listed under `.libPaths()`, starting at the first path and if
the package isn’t found continue searching through the rest of the
directories in order. You can add and remove paths from `.libPaths()`,
telling R to look for packages in the library of your choice.

As an example, let’s start by adding the current working directory to
the list of libraries.

``` r
## Add current working directory to library paths
.libPaths(c(getwd(), .libPaths()))
```

There it is\!

``` r
## List library paths currently recognized by R
.libPaths()
```

    ## [1] "/home/rafi.kurlansik@databricks.com"                                  
    ## [2] "/databricks/spark/R/lib"                                              
    ## [3] "/usr/local/lib/R/site-library"                                        
    ## [4] "/usr/lib/R/site-library"                                              
    ## [5] "/usr/lib/R/library"                                                   
    ## [6] "/home/rafi.kurlansik@databricks.com/R/x86_64-pc-linux-gnu-library/3.6"

Now let’s run a little experiment. Let’s install a few different types
of packages into our custom library:

  - The latest version of some packages from CRAN
  - An older version of a package from CRAN
  - A custom package that we built locally

This way we can validate that a pre-compiled library spanning new, old,
and custom can be built and made available to the cluster. **Note the
first line in the output of the following cell. We are installing into
the first library on the search path - the directory we created moments
ago.**

``` r
## The latest versions from CRAN
install.packages(c('quantreg', 'broom', 'noncensus'), repos = "http://cran.us.r-project.org")
```

    ## Installing packages into '/home/rafi.kurlansik@databricks.com'
    ## (as 'lib' is unspecified)

``` r
## Older version from source
install.packages("https://cran.r-project.org/src/contrib/Archive/jsonlite/jsonlite_1.5.tar.gz", type = "source", dependencies = T)
```

    ## Installing package into '/home/rafi.kurlansik@databricks.com'
    ## (as 'lib' is unspecified)

    ## inferring 'repos = NULL' from 'pkgs'

``` r
## Custom/Local package from source
install.packages("/dbfs/rk/r-projects/minimal/renv/library/Local_lib/hosta/hosta_1.0.tar.gz", type = "source", repos = NULL)
```

    ## Installing package into '/home/rafi.kurlansik@databricks.com'
    ## (as 'lib' is unspecified)

Now if we look at the contents of our library, we’ll see the packages
listed.

``` r
dir("/home/rafi.kurlansik@databricks.com")
```

    ##  [1] "R"           "broom"       "evaluate"    "highr"       "hosta"      
    ##  [6] "jsonlite"    "magrittr"    "markdown"    "noncensus"   "quantreg"   
    ## [11] "rmarkdown"   "testing.Rmd" "testing.md"

We can see the conventional structure of the installed `broom` package
by listing the contents of its
    directory.

``` r
dir("/home/rafi.kurlansik@databricks.com/broom")
```

    ##  [1] "DESCRIPTION" "INDEX"       "LICENSE"     "Meta"        "NAMESPACE"  
    ##  [6] "NEWS.md"     "R"           "data"        "doc"         "extdata"    
    ## [11] "help"        "html"

When this cluster is terminated, this custom library and all of our work
will be too. To persist it, copy to DBFS.

``` r
## Copy from driver to DBFS
system("cp -R /home/rafi.kurlansik@databricks.com /dbfs/rk/lib")
```

There it
    is\!

``` r
dir("/dbfs/rk/lib/")
```

    ## [1] "proj-lib-test"                 "rafi.kurlansik@databricks.com"

Since R is aware of DBFS on Databricks, we can add this path to
`.libPaths()`.

``` r
.libPaths("/dbfs/rk/lib/rafi.kurlansik@databricks.com")
.libPaths()
```

    ## [1] "/dbfs/rk/lib/rafi.kurlansik@databricks.com"
    ## [2] "/usr/local/lib/R/site-library"             
    ## [3] "/usr/lib/R/site-library"                   
    ## [4] "/usr/lib/R/library"

Now all that’s left to do is load the packages into memory from our
library on DBFS. We use the `lib.loc =` parameter to specify the exact
library to look in. This prevents R from searching other libraries in
the search path, and will throw an error if the package isn’t available.
This would be unnecessary in a case where there’s only one library on
the search paths of `.libPaths()`.

``` r
## Load a custom package from DBFS:
library(hosta, lib.loc = .libPaths()[1])

## Load an old version from DBFS:
library(jsonlite, lib.loc = .libPaths()[1])

## Verify older version:
packageVersion("jsonlite")
```

    ## [1] '1.5'

Functions work, yo\!

``` r
jsonlite::toJSON(mtcars[1, ], pretty = TRUE)
```

    ## [
    ##   {
    ##     "mpg": 21,
    ##     "cyl": 6,
    ##     "disp": 160,
    ##     "hp": 110,
    ##     "drat": 3.9,
    ##     "wt": 2.62,
    ##     "qsec": 16.46,
    ##     "vs": 0,
    ##     "am": 1,
    ##     "gear": 4,
    ##     "carb": 4,
    ##     "_row": "Mazda RX4"
    ##   }
    ## ]
    
 ___
    
*Happily knitted with RMarkdown*
