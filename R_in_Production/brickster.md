Automating R Jobs on Databricks with bricksteR
================
Rafi Kurlansik, Solutions Architect, Databricks
2019-12-18

While Databricks supports R users through interactive notebooks and a hosted instance of RStudio Server, it can be cumbersome to convert R files into production jobs. `bricksteR` makes it easy to quickly turn .R and .Rmd files into automated jobs that run on Databricks by using the [Databricks REST API](https://docs.databricks.com/dev-tools/api/latest/).

Here are some highlights of what you can do with `bricksteR`:

-   Import any .R or .Rmd file into the Databricks Workspace, from anywhere
-   Create and run a new job in a single function
-   Check on the status of a job run by *name* or id
-   Use JSON files or strings for job configurations

Table of Contents
-----------------

*   [Installation & Authentication](#installation)
*   [Importing R Scripts to Databricks](#importing-r-scripts-to-databricks)
*   [Jobs](#jobs)
  *   [Create and Run a Job](#create-and-run-a-job)
  *   [View Jobs and Runs](#view-jobs-and-runs)
  *   [Reset a Job](#reset-a-job)
  *   [Delete a Job](#delete-a-job)
-   [Export from Databricks](#export-from-databricks)

Installation
------------

The first thing you'll need to get started is to install the package from GitHub.

``` r
devtools::install_github("RafiKurlansik/bricksteR")
library(bricksteR)
```

#### Authentication

You will need an [Access Token](https://docs.databricks.com/dev-tools/api/latest/authentication.html#authentication) in order to authenticate with the Databricks REST API.

If you are working locally or using [DB Connect](https://docs.databricks.com/dev-tools/databricks-connect.html#databricks-connect), a good way to authenticate is to use a [.netrc](https://docs.databricks.com/dev-tools/api/latest/authentication.html#store-token-in-netrc-file-and-use-in-curl) file. By default, `bricksteR` will look for the `.netrc` file before checking for a token in the function call. If you don't already have one, create a .netrc file in your home directory that looks like this:

    machine <databricks-instance>
    login token
    password <personal-access-token-value>

You can also authenticate by [passing a token to Bearer authentication](https://docs.databricks.com/dev-tools/api/latest/authentication.html#pass-token-to-bearer-authentication). To be clear, **it is not recommended to store your credentials directly in your code**. If you are working on Databricks in a notebook, you can use `dbutils.secrets.get()` to avoid printing your token.

Finally, you will need to identify the workspace URL of your Databricks instance. On AWS this typically has the form `https://dbc-a64b1209f-d811.cloud.databricks.com`, or if you have a vanity URL it may look more like `https://mycompany.cloud.databricks.com`. On Azure it will have the form `https://eastus2.azuredatabricks.net`, or whichever region your instance is located in.

``` r
workspace <- "https://demo.cloud.databricks.com"

# If running in a Databricks Notebook
# token <- dbutils.secrets.get(scope = "bricksteR", key = "rest_api")
```

Importing R Scripts to Databricks
---------------------------------

A common use case is moving R scripts or R Markdown files to Databricks to run as a notebook. We can automate this process using `import_to_workspace()`.

``` r
import_to_workspace(file = "/Users/rafi.kurlansik/RProjects/AnRMarkdownDoc.Rmd",
                    notebook_path = "/Users/rafi.kurlansik@databricks.com/AnRMarkdownDoc",
                    workspace = workspace,
                    overwrite = 'true')
#> Status: 200
#> 
#> Object: /Users/rafi.kurlansik/RProjects/AnRMarkdownDoc.Rmd was added to the workspace at /Users/rafi.kurlansik@databricks.com/AnRMarkdownDoc
#> Response [https://demo.cloud.databricks.com/api/2.0/workspace/import]
#>   Date: 2019-12-18 05:26
#>   Status: 200
#>   Content-Type: application/json;charset=utf-8
#>   Size: 3 B
#> {}
```

Jobs
----

### Create and Run a Job

Now that the file is in our workspace, we can create a job that will run it as a [Notebook Task](https://docs.databricks.com/dev-tools/api/latest/jobs.html#jobsnotebooktask).

``` r
new_job <- create_job(notebook_path = "/Users/rafi.kurlansik@databricks.com/AnRMarkDownDoc", 
                       name = "Beer Sales Forecasting Job",
                       job_config = "default",
                       workspace = workspace)
#> Status: 200
#> Job "Beer Sales Forecasting Job" created.
#> Job ID: 31123

job_id <- new_job$job_id
```

Each job requires a `job_config`, which is a JSON file or JSON formatted string specifying (at a minimum) the task and type of infrastructure required. If no config is supplied, a 'default' cluster is created with 1 driver and 2 workers. Here's what that default configuration looks like:

``` r
# Default JSON configs
 job_config <- paste0('{
  "name": ', name,'
  "new_cluster": {
    "spark_version": "5.3.x-scala2.11",
    "node_type_id": "r3.xlarge",
    "aws_attributes": {
      "availability": "ON_DEMAND"
    },
    "num_workers": 2
  },
  "email_notifications": {
    "on_start": [],
    "on_success": [],
    "on_failure": []
  },
  "notebook_task": {
    "notebook_path": ', notebook_path, '
  }
}')
```

Where `name` is the name of your job and `notebook_path` is the path to the notebook in your workspace that will be run.

Creating a job by itself will not actually execute any code. If we want our beer forecast - and we do - then we'll need to actually run the job. We can run the job by id, or by name.

``` r
# Specifying the job_id
run_job(job_id = job_id, workspace = workspace, token = NULL)
#> Status: 200
#> Run ID: 2393205
#> Number in Job: 1

# Specifying the job name
run_job(name = "Beer Sales Forecasting Job", workspace = workspace, token = NULL)
#> Job "Beer Sales Forecasting Job" found with 31123.
#> Status: 200
#> Run ID: 2393206
#> Number in Job: 2
```

Running a job by name is convenient, but there's nothing stopping you from giving two jobs the same name on Databricks. The way `bricksteR` handles those conflicts today is by forcing you to use the unique Job ID or rename the job you want to run with a distinct moniker.

If you want to run a job immediately, one option is to use the `create_and_run_job()` function:

``` r
# By specifying a local file we implicitly import it
running_new_job <- create_and_run_job(file = "/Users/rafi.kurlansik/agg_and_widen.R",
                                      name = "Lager Sales Forecasting Job",
                                      notebook_path = "/Users/rafi.kurlansik@databricks.com/agg_and_widen", 
                                      job_config = "default",
                                      workspace = workspace)
#> Job "Lager Sales Forecasting Job" successfully created...  
#> The Job ID is: 31124
#> 
#> Run successfully launched...  
#> The run_id is: 2393207
#> 
#> You can check the status of this run at: 
#>  https://demo.cloud.databricks.com#job/31124/run/1
```

### View Jobs and Runs

There are two handy functions that will return a list of Jobs and their runs - `jobs_list()` and `runs_list()`. One of the objects returned is a concise R dataframe that allows for quick inspection.

``` r
jobs <- jobs_list(workspace = workspace)
#> Status: 200
#> Number of jobs: 629
beer_runs <- runs_list(name = "Lager Sales Forecasting Job", workspace = workspace)
#> Job "Lager Sales Forecasting Job" found with 31124.
#> Status: 200
#> Job ID: 31124
#> Number of runs: 1
#> 
#> Are there more runs to list? 
#>  FALSE

head(jobs$response_tidy)
#>   job_id          settings.name        created_time
#> 1  12017  test-job-dollar-signs 2018-12-04 15:55:01
#> 2  18676 Arun_Spark_Submit_Test 2019-08-28 18:32:49
#> 3  21042              test-seif 2019-10-30 05:06:32
#> 4   4441        Ad hoc Analysis 2017-08-29 20:21:20
#> 5  29355       Start ETL Stream 2019-12-04 18:26:40
#> 6  30837          Turo-Job-Test 2019-12-10 20:18:40
#>                 creator_user_name
#> 1     sid.murching@databricks.com
#> 2  arun.pamulapati@databricks.com
#> 3 seifeddine.saafi@databricks.com
#> 4            parag@databricks.com
#> 5              mwc@databricks.com
#> 6          lei.pan@databricks.com
head(beer_runs$response_tidy)
#>   runs.job_id runs.run_id        runs.creator_user_name
#> 1       31124     2393207 rafi.kurlansik@databricks.com
#>                                   runs.run_page_url
#> 1 https://demo.cloud.databricks.com#job/31124/run/1
```

### Reset a Job

If you need to update the config for a job, use `reset_job()`.

``` r
# JSON string or file path can be passed
new_config <- "/configs/nightly_model_training.json"

reset_job(job_id = 31114, new_config, workspace = workspace)
```

### Delete a Job

If you need to clean up a job from the workspace, use `delete_job()`.

``` r
delete_job(job_id = job_id, workspace = workspace)
#> Status: 200
#> { "job_id": 31123 } has been deleted.
```

Export from Databricks
----------------------

To export notebooks from Databricks, use `export_from_workspace()`.

``` r
export_from_workspace(workspace_path = "/Users/rafi.kurlansik@databricks.com/AnRMarkdownDoc",
                      format = "SOURCE",
                      filename = "/Users/rafi.kurlansik/RProjects/anotherdoc.Rmd",
                      workspace = workspace)
```

Passing a `filename` will save the exported data to your local file system. Otherwise it will simply be returned as part of the API response.

Future Plans
------------

That's about all there is to it to get started with automating R on Databricks! Whether you are using `blastula` to email a report or are spinning up a massive cluster to perform ETL or model training, these APIs will make it easy for you to define, schedule, and monitor those jobs from R.

In 2020 I'd like to add support for DBFS operations, and provide additional tools for managing libraries.

Questions or feedback? Feel free to contact me at <rafi.kurlansik@databricks.com>.

