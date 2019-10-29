## `sparklyr` vs. `SparkR`

R users find themselves in the unique position of having to choose between two APIs for Spark.  In this section we will detail some of the commonalities and differences between them.  Tempting as it may be to pick and choose function calls from both into one script, it makes code more difficult to read and maintain.  Therefore it is recommended as a best practice to choose one API to develop your Spark application in R.  

_Contents_
* [Stewardship](#stewardship)
* [API Differences](#api-differences)
* [API Interoperability](#api-interoperability)

#### Stewardship
A key difference between the two packages lies in their origin and authorship.  

`SparkR` is the 'official' package and is documented at [spark.apache.org](https://spark.apache.org/docs/latest/sparkr.html).  It was built by the Spark community and developers from Databricks.  As such it adheres closely to Scala classes and the DataFrame API.

`sparklyr`, on the other hand, originated from RStudio and is largely maintained by them. Its documentation is also hosted by RStudio at [spark.rstudio.com](https://spark.rstudio.com/).  Given its origin in RStudio, it comes as no surprise that `sparklyr` is tightly integrated into the tidyverse, both in programming style and through API interoperability with `dplyr`.

Both libraries are highly capable of working with big data in R - as of Oct. 2019 their feature sets are more or less at parity.

#### API Differences

Let's take an example to begin understanding the differences between the two APIs more deeply.  In this case we'll read CSV files into Spark using both `sparklyr` and `SparkR`, then compare the classes of the two.  **Note:**  In these examples we will explicitly reference the package used for each function in order to avoid confusion.

```r
## Read airlines dataset from 2008
airlinesDF <- SparkR::read.df("/databricks-datasets/asa/airlines/2008.csv", 
                               source = "csv", 
                               inferSchema = "true", 
                               header = "true")

## Read airlines dataset from 2007
airlines_sdf <- sparklyr::spark_read_csv(sc, name = 'airlines', 
                                         path = "/databricks-datasets/asa/airlines/2007.csv")

## Check the class of each loaded dataset
cat(c("Class of SparkR object:\n", class(airlinesDF), "\n\n"))
cat(c("Class of sparklyr object:\n", class(airlines_sdf)))


# output:
Class of SparkR object:
  SparkDataFrame 

Class of sparklyr object:
 tbl_spark tbl_sql tbl_lazy tbl
```

Two distinct classes.  Now watch what happens when we run a `sparklyr` command on a SparkDataFrame and a `SparkR` command on a `tbl_spark`.

```r
## Function from sparklyr on SparkR object
sparklyr::sdf_pivot(airlinesDF, DepDelay ~ UniqueCarrier)


# output:
Error : Unable to retrieve a Spark DataFrame from object of class SparkDataFrame 
(NOTE: If you wish to use SparkR, import it by calling 'library(SparkR)'.)
```
```r
## Function from SparkR on sparklyr object
SparkR::arrange(sparklyAirlines, "DepDelay")


# output:
Error in (function (classes, fdef, mtable)  : 
  unable to find an inherited method for function ‘arrange’ for signature ‘"tbl_spark", "character"’
```

As you might expect, calling `SparkR` functions on `sparklyr` objects and vice versa can lead to unexpected -- and undesirable -- behavior. Why is this?

`sparklyr` translates `dplyr` functions like `arrange()` into a SQL query plan that is used by SparkSQL.  This is not the case with `SparkR`, which has functions for SparkSQL tables and Spark DataFrames.  At the end of the day DataFrame operations are translated into a query plan for SparkSQL, but the classes used to build those plans are different in each package.  This limits API interoperability and is one of the reasons why we don't recommended going back and forth between them in the same job.

#### API Interoperability
At this point you may be wondering if there is any space in the APIs where they can work together?  There is, and that overlap in the venn diagram of these two packages is SparkSQL.

Recall that when we loaded the airlines data from 2007 into a `tbl_spark`, we specified the table name _airlines_. This table is registered with SparkSQL and can be referenced using the `sql()` function from `SparkR`. Executing SQL queries this way will return a Spark DataFrame:

```r
## Use SparkR to query the 'airlines' table loaded into SparkSQL through sparklyr
top10delaysDF <- SparkR::sql("SELECT 
                              UniqueCarrier, 
                              DepDelay, 
                              Origin 
                              FROM 
                              airlines 
                              WHERE 
                              DepDelay NOT LIKE 'NA' 
                              ORDER BY DepDelay 
                              DESC LIMIT 10")

## Check class of result
cat(c("Class of top10delaysDF: ", class(top10delaysDF), "\n\n"))

## Inspect the results
cat("Top 10 Airline Delays for 2007:\n")
head(top10delaysDF, 10)


# output:
Class of top10delaysDF:  SparkDataFrame 

Top 10 Airline Delays for 2007:
   UniqueCarrier DepDelay Origin
1             NW      999    EWR
2             AA      999    RNO
3             AA      999    PHL
4             MQ      998    RST
5             9E      997    SWF
6             AA      996    DFW
7             NW      996    DEN
8             MQ      995    IND
9             MQ      994    SJT
10            AA      993    MSY
```

#### Further Reading
To learn more about working with these tables, see the [Working With Spark Tables](linktocome) section. Still, we don't recommend taking this approach. Make your life simple and build your job around one API!

___
[Back to table of contents](https://github.com/marygracemoesta/R-User-Guide#contents)
