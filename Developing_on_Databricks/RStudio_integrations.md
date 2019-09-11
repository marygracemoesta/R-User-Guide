# Rstudio Integrations
Witin Databricks there are 3 different ways to write R:
 - Within Databricks notebooks
 - Integration withe Rstudio Server
 - Using Databricks connect to integrate with Rstudio desktop 
 
 ## Writing R in Databricks Notebooks
 Databricks notebooks allows for a variety of defualt languages:
 - R
 - SQL
 - Python
 - Scala
 Selecting R as the default language upon notebook creation is one way to devlop R code on Databricks. 
 <img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/R_default.png?" raw = true)>
 
You can also use Databricks [magic commands](https://docs.databricks.com/user-guide/notebooks/notebook-use.html#mix-languages) to write R code in a notebook where the default language is not R.
<img src = "https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/R_magic_command.png?" raw = true>

## Integration With Rstudio Server
Databricks has a native integration with Rstudio Server. Details for the Rstudio server itegration setup can be found [here](https://docs.databricks.com/spark/latest/sparkr/rstudio.html#get-started-with-rstudio-server-open-source). Please note that the setup varies by Tier (i.e. setup for Rstudio Server Open Source varies from setup with Rstudio Server Pro). 

