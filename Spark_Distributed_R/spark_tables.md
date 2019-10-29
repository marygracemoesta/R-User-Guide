## Working with Spark Tables in R

The `sparklyr` and `SparkR` packages provide R users access to the power
of Spark. However, for those who are new to distributed computing it can
be difficult to discover how to work with data that is partitioned
across a cluster. This guide is designed to quickly ramp up R users for
how to manipulate tables in Spark, and how the various APIs work with
eachother.

-----

#### Table of Contents

  - SparkSQL, `sparklyr`, and `dplyr`
  - Reading Files
  - Aggregations
  - Hive Metastore: Writes and Reads
  - New Columns with Mutate & Hive UDFs
  - SQL Translation and API Interoperability 


### SparkSQL, `sparklyr`, and `dplyr`

Tables created with SparkSQL act as a common layer for working with data
in `sparklyr`, `SparkR`, and SQL cells on Databricks. These tables can
be accessed via the *Data* tab in the left menu bar. Let’s begin by
creating a new table from a JSON file with `sparklyr`, then make our way
toward working with that data in `SparkR` and SQL cells. To learn more
about this topic, see the docs
[here](https://docs.databricks.com/user-guide/tables.html).

First load `sparklyr` and `dplyr`, then connect to Spark.

``` r
library(sparklyr)
library(dplyr)

## Connect to Spark
SparkR::sparkR.session()
```

    ## Java ref type org.apache.spark.sql.SparkSession id 1

``` r
sc <- spark_connect(method = "databricks")
```

    ## Warning in file.create(to[okay]): cannot create file '/usr/local/lib/R/
    ## site-library/sparklyr/java//sparklyr-2.4-2.11.jar', reason 'Permission
    ## denied'

    ## Warning in file.create(to[okay]): cannot create file '/usr/local/lib/R/
    ## site-library/sparklyr/java//sparklyr-2.2-2.11.jar', reason 'Permission
    ## denied'

    ## Warning in file.create(to[okay]): cannot create file '/usr/local/lib/R/
    ## site-library/sparklyr/java//sparklyr-2.1-2.11.jar', reason 'Permission
    ## denied'

#### Read in a JSON file from DBFS (blob storage) with `spark_read_json()`

This function takes a few parameters:

> *sc* - Spark Connection, as defined by `spark_connect()`
> 
> *name* - Optional name to register the table for SparkSQL
> 
> *path* - The path to the JSON file in your file system

There are other parameters as well, which you can find in the API
documentation here:
<https://spark.rstudio.com/reference/spark_read_json/>

``` r
## Read JSON file into Spark and name it for SparkSQL
jsonDF <- spark_read_json(sc, 
                          name = 'jsonTable', 
                          path = "/databricks-datasets-private/ML/working_with_spark_tables_in_r/books.json")

## Take a look at our DF
head(jsonDF)
```

    ## # Source: spark<?> [?? x 5]
    ##   author               genre    id    price title                
    ##   <chr>                <chr>    <chr> <dbl> <chr>                
    ## 1 Gambardella, Matthew Computer bk101 45    XML Developer's Guide
    ## 2 Ralls, Kim           Fantasy  bk102  5.95 Midnight Rain        
    ## 3 Corets, Eva          Fantasy  bk103  5.95 Maeve Ascendant      
    ## 4 Corets, Eva          Fantasy  bk104  5.95 Oberon's Legacy      
    ## 5 Corets, Eva          Fantasy  bk105  5.95 The Sundered Grail   
    ## 6 Randall, Cynthia     Romance  bk106  4.95 Lover Birds

#### Aggregate and Analyze

With `dplyr` syntax…

``` r
## Author counts, descending
group_by(jsonDF, author) %>% 
  count() %>%
  arrange(desc(n))
```

    ## # Source:     spark<?> [?? x 2]
    ## # Groups:     author
    ## # Ordered by: desc(n)
    ##   author                   n
    ##   <chr>                <dbl>
    ## 1 Corets, Eva              3
    ## 2 Gambardella, Matthew     1
    ## 3 Ralls, Kim               1
    ## 4 Randall, Cynthia         1

    %sql
    select author, count(*) as n from jsonTable
    GROUP BY author
    ORDER BY n DESC

#### Write a Table to Be Queried Later

These tables are stored by default as parquet files in
`/user/hive/warehouse/database.db/tablename` (all lowercase) on DBFS. If
your table isn’t associated with a database, it will simply be written
to `/user/hive/warehouse/tablename`.

``` r
## Write a table
## If you don't specify a database, it will write to the 'default' database, or in this case default.aggJSON
group_by(jsonDF, author) %>% 
  count() %>%
  arrange(desc(n)) %>%
  spark_write_table(name = 'json_books_agg', mode = "overwrite")
```

Verify the table location on
    DBFS:

``` r
system("ls /dbfs/user/hive/warehouse/json_books_agg", intern = T)
```

    ## [1] "_SUCCESS"                                                                                          
    ## [2] "_committed_1479134774032643852"                                                                    
    ## [3] "_started_1479134774032643852"                                                                      
    ## [4] "part-00000-tid-1479134774032643852-7c8a1d2f-63e9-46a4-8d21-07026dfc74de-6983-1-c000.snappy.parquet"
    ## [5] "part-00001-tid-1479134774032643852-7c8a1d2f-63e9-46a4-8d21-07026dfc74de-6984-1-c000.snappy.parquet"

#### Reading from a Table

``` r
## Read a table from the db
fromTable <- spark_read_table(sc, "json_books_agg") 

head(fromTable)
```

    ## # Source: spark<?> [?? x 2]
    ##   author                   n
    ##   <chr>                <dbl>
    ## 1 Gambardella, Matthew     1
    ## 2 Ralls, Kim               1
    ## 3 Randall, Cynthia         1
    ## 4 Corets, Eva              3

#### Data Type Conversions with `sparklyr`

`sparklyr` uses [Hive
UDFs](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)
for many data transformation operations. These are executed inside of
the `mutate()` function of `dplyr`.

Here’s an example of creating the current date in our table, then
extracting different datetime values from it. Then we will transform the
datetime value to a different format.

``` r
withDate <- jsonDF %>%
              mutate(today = current_timestamp())

head(withDate)
```

    ## # Source: spark<?> [?? x 6]
    ##   author           genre   id    price title            today              
    ##   <chr>            <chr>   <chr> <dbl> <chr>            <dttm>             
    ## 1 Gambardella, Ma… Comput… bk101 45    XML Developer's… 2019-10-29 16:57:11
    ## 2 Ralls, Kim       Fantasy bk102  5.95 Midnight Rain    2019-10-29 16:57:11
    ## 3 Corets, Eva      Fantasy bk103  5.95 Maeve Ascendant  2019-10-29 16:57:11
    ## 4 Corets, Eva      Fantasy bk104  5.95 Oberon's Legacy  2019-10-29 16:57:11
    ## 5 Corets, Eva      Fantasy bk105  5.95 The Sundered Gr… 2019-10-29 16:57:11
    ## 6 Randall, Cynthia Romance bk106  4.95 Lover Birds      2019-10-29 16:57:11

Let’s create two new columns, with the month and year, respectively.

``` r
withMMyyyy <- withDate %>%
                mutate(month = month(today), 
                       year = year(today))

head(withMMyyyy)
```

    ## # Source: spark<?> [?? x 8]
    ##   author      genre  id    price title      today               month  year
    ##   <chr>       <chr>  <chr> <dbl> <chr>      <dttm>              <int> <int>
    ## 1 Gambardell… Compu… bk101 45    XML Devel… 2019-10-29 16:57:12    10  2019
    ## 2 Ralls, Kim  Fanta… bk102  5.95 Midnight … 2019-10-29 16:57:12    10  2019
    ## 3 Corets, Eva Fanta… bk103  5.95 Maeve Asc… 2019-10-29 16:57:12    10  2019
    ## 4 Corets, Eva Fanta… bk104  5.95 Oberon's … 2019-10-29 16:57:12    10  2019
    ## 5 Corets, Eva Fanta… bk105  5.95 The Sunde… 2019-10-29 16:57:12    10  2019
    ## 6 Randall, C… Roman… bk106  4.95 Lover Bir… 2019-10-29 16:57:12    10  2019

Now let’s transform the `today` column to a different date format, then
extract the day of the month.

``` r
withUnixTimestamp <- withMMyyyy %>%
                      mutate(formatted_date = date_format(today, 'yyyy-MM-dd'),
                             day = dayofmonth(formatted_date))

## View on the columns we created
select(withUnixTimestamp, today, month, year, formatted_date, day)
```

    ## # Source: spark<?> [?? x 5]
    ##   today               month  year formatted_date   day
    ##   <dttm>              <int> <int> <chr>          <int>
    ## 1 2019-10-29 16:57:12    10  2019 2019-10-29        29
    ## 2 2019-10-29 16:57:12    10  2019 2019-10-29        29
    ## 3 2019-10-29 16:57:12    10  2019 2019-10-29        29
    ## 4 2019-10-29 16:57:12    10  2019 2019-10-29        29
    ## 5 2019-10-29 16:57:12    10  2019 2019-10-29        29
    ## 6 2019-10-29 16:57:12    10  2019 2019-10-29        29

#### Repeat in SQL

We can perform the same transformations in SQL. Assuming we have a table
registered in Spark (like `aggJSON` in this notebook), we can create new
tables from that. `sparklyr` doesn’t support this directly, but SparkR
has a `sql()` function, and in Databricks you can execute SQL cells.

Let’s do the SparkR `sql()` function
first.

``` r
## Run a nested query where we first create the timestamp, then extract the month
withTimestampDF <- SparkR::sql("SELECT *, date_format(today, 'MM') as month FROM
                                  (SELECT *, current_timestamp AS today FROM jsonTable)")

## Show the results
SparkR::collect(withTimestampDF)
```

    ##    array dict  int string               today month
    ## 1     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 2     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 3     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 4     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 5     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 6     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 7     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 8     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 9     NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 10    NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 11    NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 12    NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 13    NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 14    NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 15    NA   NA <NA>   <NA> 2019-10-29 16:57:12    10
    ## 16    NA   NA <NA>   <NA> 2019-10-29 16:57:12    10

Running it in a SQL cell would give you the same results.

    %sql
    SELECT *, date_format(today, 'MM') AS month FROM
      (SELECT *, current_timestamp AS today FROM jsonTable)

#### Picking Up a Transformed Table Downstream in `sparklyr`

In case you want to work with the output of SparkR or SQL cells, you can
access those transformed tables by creating temporary views.

    ## In SparkR
    SparkR::createOrReplaceTempView(withTimestampDF, "timestampTable")
    
    ## Now query with sparklyr
    spark_read_table(sc, "timestampTable") %>% collect()

``` 

# A tibble: 6 x 7
  author           genre   id    price title           today               month
  <chr>            <chr>   <chr> <dbl> <chr>           <dttm>              <chr>
1 Gambardella, Ma… Comput… bk101 45    XML Developer'… 2019-07-17 20:02:33 07   
2 Ralls, Kim       Fantasy bk102  5.95 Midnight Rain   2019-07-17 20:02:33 07   
3 Corets, Eva      Fantasy bk103  5.95 Maeve Ascendant 2019-07-17 20:02:33 07   
4 Corets, Eva      Fantasy bk104  5.95 Oberon's Legacy 2019-07-17 20:02:33 07   
5 Corets, Eva      Fantasy bk105  5.95 The Sundered G… 2019-07-17 20:02:33 07   
6 Randall, Cynthia Romance bk106  4.95 Lover Birds     2019-07-17 20:02:33 07 
```

Access the `timestampTable` in a SQL cell:

``` 
%sql
DROP TABLE IF EXISTS timestampTableSQL;

CREATE TABLE timestampTableSQL
  COMMENT 'This table is created with existing data'
  AS SELECT *, date_format(today, 'MM') AS month FROM
  (SELECT *, current_timestamp AS today FROM jsonTable)
  
```

``` r
## Now read that from sparklyr
spark_read_table(sc, "timestampTableSQL") %>% collect()
```

    ## # A tibble: 150 x 7
    ##    Petal.Length Petal.Width Sepal.Length Sepal.Width Species
    ##           <dbl>       <dbl>        <dbl>       <dbl> <chr>  
    ##  1          1.4         0.2          5.1         3.5 setosa 
    ##  2          1.4         0.2          4.9         3   setosa 
    ##  3          1.3         0.2          4.7         3.2 setosa 
    ##  4          1.5         0.2          4.6         3.1 setosa 
    ##  5          1.4         0.2          5           3.6 setosa 
    ##  6          1.7         0.4          5.4         3.9 setosa 
    ##  7          1.4         0.3          4.6         3.4 setosa 
    ##  8          1.5         0.2          5           3.4 setosa 
    ##  9          1.4         0.2          4.4         2.9 setosa 
    ## 10          1.5         0.1          4.9         3.1 setosa 
    ## # … with 140 more rows, and 2 more variables: today <dttm>, month <chr>

``` 
# A tibble: 6 x 7
  author           genre   id    price title           today               month
  <chr>            <chr>   <chr> <dbl> <chr>           <dttm>              <chr>
1 Gambardella, Ma… Comput… bk101 45    XML Developer'… 2019-07-17 20:02:33 07   
2 Ralls, Kim       Fantasy bk102  5.95 Midnight Rain   2019-07-17 20:02:33 07   
3 Corets, Eva      Fantasy bk103  5.95 Maeve Ascendant 2019-07-17 20:02:33 07   
4 Corets, Eva      Fantasy bk104  5.95 Oberon's Legacy 2019-07-17 20:02:33 07   
5 Corets, Eva      Fantasy bk105  5.95 The Sundered G… 2019-07-17 20:02:33 07   
6 Randall, Cynthia Romance bk106  4.95 Lover Birds     2019-07-17 20:02:33 07   
```

#### Hive UDAFs and SQL Translation from `sparklyr` to `SparkR`

``` r
## Create table in Spark SQL called 'iris'
irisDF <- sdf_copy_to(sc, iris, name = "iris", overwrite = T)

## Group it by Species and use the Hive UDAF `percentile_approx()` to get the quantiles, then view the results
quantileDF <- irisDF %>% group_by(Species)  %>% summarize(quantile_25th = percentile_approx(Sepal_Length, 0.25),
                                                          quantile_50th = percentile_approx(Sepal_Length, 0.50),
                                                          quantile_75th = percentile_approx(Sepal_Length, 0.75),
                                                          quantile_100th = percentile_approx(Sepal_Length, 1.0))
collect(quantileDF)
```

    ## # A tibble: 3 x 5
    ##   Species    quantile_25th quantile_50th quantile_75th quantile_100th
    ##   <chr>              <dbl>         <dbl>         <dbl>          <dbl>
    ## 1 virginica            6.2           6.5           6.9            7.9
    ## 2 versicolor           5.6           5.9           6.3            7  
    ## 3 setosa               4.8           5             5.2            5.8

``` r
## Can also get the query plan and run that in SparkR::sql()
query <- dbplyr::sql_render(irisDF %>% 
                                 group_by(Species) %>% 
                                 summarize(quantile_25th = percentile_approx(Sepal_Length, 0.25),
                                           quantile_50th = percentile_approx(Sepal_Length, 0.50),
                                           quantile_75th = percentile_approx(Sepal_Length, 0.75),
                                           quantile_100th = percentile_approx(Sepal_Length, 1.0)))

print(query)
```

    ## <SQL> SELECT `Species`, PERCENTILE_APPROX(`Sepal_Length`, 0.25) AS `quantile_25th`, PERCENTILE_APPROX(`Sepal_Length`, 0.5) AS `quantile_50th`, PERCENTILE_APPROX(`Sepal_Length`, 0.75) AS `quantile_75th`, PERCENTILE_APPROX(`Sepal_Length`, 1.0) AS `quantile_100th`
    ## FROM `iris`
    ## GROUP BY `Species`

    SparkR::sql("SELECT `Species`, 
    PERCENTILE_APPROX(`Sepal_Length`, 0.25) AS `quantile_25th`, 
    PERCENTILE_APPROX(`Sepal_Length`, 0.5) AS `quantile_50th`, 
    PERCENTILE_APPROX(`Sepal_Length`, 0.75) AS `quantile_75th`, 
    PERCENTILE_APPROX(`Sepal_Length`, 1.0) AS `quantile_100th`
    FROM `iris`
    GROUP BY `Species`") %>% SparkR::collect()

``` 
     Species quantile_25th quantile_50th quantile_75th quantile_100th
1 versicolor           5.6           5.9           6.3            7.0
2  virginica           6.2           6.5           6.9            7.9
3     setosa           4.8           5.0           5.2            5.8
```

##### Manipulating Column Names

``` r
## Convert to uppercase and add dollar sign
upper_names <- jsonDF %>% colnames() %>% toupper() %>% paste0("$")

jsonDF2 <- jsonDF %>% select(setNames(colnames(jsonDF), upper_names))

head(jsonDF2)
```

    ## # Source: spark<?> [?? x 5]
    ##   `AUTHOR$`            `GENRE$` `ID$` `PRICE$` `TITLE$`             
    ##   <chr>                <chr>    <chr>    <dbl> <chr>                
    ## 1 Gambardella, Matthew Computer bk101    45    XML Developer's Guide
    ## 2 Ralls, Kim           Fantasy  bk102     5.95 Midnight Rain        
    ## 3 Corets, Eva          Fantasy  bk103     5.95 Maeve Ascendant      
    ## 4 Corets, Eva          Fantasy  bk104     5.95 Oberon's Legacy      
    ## 5 Corets, Eva          Fantasy  bk105     5.95 The Sundered Grail   
    ## 6 Randall, Cynthia     Romance  bk106     4.95 Lover Birds

``` r
## Convert to lower case and remove dollar sign
clean_names <- jsonDF2 %>% colnames() %>% tolower()
clean_names <- gsub(pattern = "[$]", replacement = "_", x = clean_names)

jsonDF3 <- jsonDF2 %>% select(setNames(colnames(jsonDF2), clean_names))

head(jsonDF3)
```

    ## # Source: spark<?> [?? x 5]
    ##   author_              genre_   id_   price_ title_               
    ##   <chr>                <chr>    <chr>  <dbl> <chr>                
    ## 1 Gambardella, Matthew Computer bk101  45    XML Developer's Guide
    ## 2 Ralls, Kim           Fantasy  bk102   5.95 Midnight Rain        
    ## 3 Corets, Eva          Fantasy  bk103   5.95 Maeve Ascendant      
    ## 4 Corets, Eva          Fantasy  bk104   5.95 Oberon's Legacy      
    ## 5 Corets, Eva          Fantasy  bk105   5.95 The Sundered Grail   
    ## 6 Randall, Cynthia     Romance  bk106   4.95 Lover Birds

-----

At this point you should have a solid foundation of understanding how to
work with Spark tables in R on Databricks.
