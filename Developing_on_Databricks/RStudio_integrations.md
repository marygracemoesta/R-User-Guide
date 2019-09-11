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

Once the intit script has been installed on the cluster, login information can be found in the `Apps` tab of the cluster
<img src ="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/Rstudio_integration.png?" raw = true>

## Rstudio Desktop Integration
Databricks also supports integration with Rstudio Desktop using Databricks Connect. Please refer to the [PDF](https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/DB%20Connect%20with%20RStudio%20Dekstop.pdf) For step by step instructions. 

## Differences in Integrations
When it comes to the two different integrations with Rstudio - the distinction between the two become prevelenat when looking at the architecture. In the Rstudio Server integration, Rstudio lives inside the driver. 
<img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/rstudioServerarchitecture.png?" raw = true>

Where as Rstudio Desktop + DB Connect uses the local machine and the drive and submit queries to the nodes managed by the Databricks cluster. 

## Gotchas 
Something important to note when using the Rstudio integrations:
- Loss of notebook functionality: magic commands that work in Databricks notebooks, do not work within Rstudio
- As of September 2019, `sparklyr` is *not* supported using Rstudio Desktop + DB Connect 
