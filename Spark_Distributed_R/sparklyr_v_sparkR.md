# `sparklyr` vs. `SparkR`

R users find themselves in the unique position of having to choose between two APIs for Spark.  In this section we will detail some of the commonalities and differences between them.

##### Stewardship
A key difference between the two packages lies in their origin and authorship.  

`SparkR` is the 'official' package and is documented at [spark.apache.org](https://spark.apache.org/docs/latest/sparkr.html).  It was built by the Spark community and developers from Databricks.  As such it adheres closely to Scala classes and the DataFrame API.

`sparklyr`, on the other hand, originated from RStudio and is largely maintained by them. Its documentation is also hosted by RStudio at [spark.rstudio.com](https://spark.rstudio.com/).  Given its origin in RStudio, it comes as no surprise that `sparklyr` is tightly integrated into the tidyverse, both in programming style and through API interoperability with `dplyr`.

Both libraries are highly capable of working with big data in R - as of Oct. 2019 their feature sets are more or less at parity.

##### API Differences

Let's take an example to begin understanding the differences between the two APIs more deeply.  In this case we'll read CSV files into Spark using both `sparklyr` and `SparkR`, then compare the classes of the two.  **Note:**  In these examples we will explicitly reference the package used for each function in order to avoid confusion.

```r
## Read airlines dataset from 2008
airlinesDF <- SparkR::read.df("/databricks-datasets/asa/airlines/2008.csv", source = "csv", inferSchema = "true", header = "true")

## Read airlines dataset from 2007
airlines_sdf <- sparklyr::spark_read_csv(sc, name = 'airlines', path = "/databricks-datasets/asa/airlines/2007.csv")

## Check the class of each loaded dataset
cat(c("Class of SparkR object:\n", class(airlinesDF), "\n\n"))
cat(c("Class of sparklyr object:\n", class(airlines_sdf)))

```
```bash
Class of SparkR object:
  SparkDataFrame 

Class of sparklyr object:
 tbl_spark tbl_sql tbl_lazy tbl
```
