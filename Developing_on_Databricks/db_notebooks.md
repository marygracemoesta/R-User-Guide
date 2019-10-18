
 ## Databricks Notebooks
 
 Databricks notebooks allows for a variety of defualt languages:
 - R
 - SQL
 - Python
 - Scala
 Selecting R as the default language upon notebook creation is one way to devlop R code on Databricks. 
 <img src="https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/R_default.png?" raw = true)>
 
You can also use Databricks [magic commands](https://docs.databricks.com/user-guide/notebooks/notebook-use.html#mix-languages) to write R code in a notebook where the default language is not R.
<img src = "https://github.com/marygracemoesta/R-User-Guide/blob/master/Developing_on_Databricks/images/R_magic_command.png?" raw = true>

# Databricks Utility Functions
Databricks utiltiy functions, often called dbutils, are a set of utility functions that live inside Databricks notebooks to
better the developer/data science experience. An example using `display` is below:

Beyond display, there are several other useful Databricks utility functions:
- File system utilities 
    - Utilities to access the Databricks File System
- Notebook workflow utilities 
    - Utilities used the help chain together notebooks
- Widget utilities 
    - Utilities that can be used to help parameterizing notebooks
- Secrets utilities 
    - Used to store sensitive credentials 
- Library utilities
    - Please note these can only be used to install Python libraries, not R libraries. 
Read more about utility syntax and specifics [here](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#dbutils)

## A quick note on `display` vs `ggplot`
An additional means towards quickly visualizing data withing Databricks notebooks, using a Databricks
utility function, is using the `display` command. `Display` can help visualize data in both a table formact 
and in a variety of different visualizations. Read more about the `display` utility [here](https://docs.databricks.com/user-guide/visualizations/index.html#display-function)


Beyond using `display` to visualize data, Databricks also supports the use of the common visualization package `gglot2`
See an example of using `ggplot2` in a Databricks notebooks [here](https://docs.databricks.com/user-guide/visualizations/index.html#visualizations-in-r)


# Magic Commands 
While default languages can be set in a notebook, but magic commands allow for the ability to jump from language
to language within a single notebook. Learn more about magic command syntax and capabilities [here](https://docs.databricks.com/user-guide/notebooks/notebook-use.html#mix-languages)


