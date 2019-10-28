
### The Problem with ML Engineering Today

Staying organized with ML work can be hard.

Imagine you've created a nice workflow for model training.  Maybe you wrote some custom ETL and feature engineering, trained models of various flavors on that data, extracted performance metrics, and even saved the model object to disk.  You share the results of a good model by writing scores to a flat file, database, or perhaps standing up a web service to make the model accessible via REST with `plumber`, Sagemaker, or Azure ML.

However, what if we train dozens or hundreds of models at once?  What about hyperparameter tuning, and all of the metrics and data that will generate?  Furthermore, the model will inevitably need to be retrained in the future.  While we saved the model object, we have no robust framework for reproducing any particular training run nor for tracking all of the data we generate during experimentation.  How are we going to keep track of models, features, hyperparameters, code, and training data - *without* resorting to Excel?  

### MLflow

MLflow is an [open source](https://www.mlflow.org) project developed at Databricks to solve the problem of managing the machine learning lifecycle.  It does so through a combination of APIs and user interfaces for tracking, deploying, and managing machine learning models.  In this section we will go over the MLflow APIs for R with examples for tracking, model deployment, and projects.

  * [Tracking](https://github.com/marygracemoesta/R-User-Guide/blob/master/MLflow/tracking.md)
  * [Model Deployment](https://github.com/marygracemoesta/R-User-Guide/blob/master/MLflow/model_deployment.md)
  * [Projects](https://github.com/marygracemoesta/R-User-Guide/blob/master/MLflow/projects.md)

