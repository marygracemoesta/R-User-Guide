# SparkR vs. sparklyr
While both packages provide an interfact with Spark, the largest distinction between SparkR and sparklyr is that SparkR 
was built by the Spark community and sparklyr was build by theR community. Beyond that fundamental difference, there are various
datatypes that differ between the two - the most notable difference being dataframes. SparkR uses the Spark DataFrame API,
while sparklyr uses tbl_spark. The important takeaway from this is that you can't pass a `tbl_spark` 
