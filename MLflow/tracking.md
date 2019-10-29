
## MLflow Tracking

The first thing to know about MLflow is that it will track whatever you tell it to.  That means model parameters, metrics, model objects, messages, bits of data... you decide!  

All of these *artifacts* are stored in the *MLflow Tracking Server*, where they are organized into distinct *experiments*.  Each iteration of experimentation is regarded as a *run*, and given a unique *run id*.  Represented as a heirarchy, 

```Tracking Server --> Experiment --> Run --> Artifact```

This will become clearer as we work through examples.  Let's start by taking a look at some basic model training code using the **wine quality** data set.

```r
# Read the wine-quality csv file
wine_quality <- read.csv("/dbfs/databricks-datasets-private/ML/mlflow_with_r/wine_quality.csv")

str(wine_quality)
```

```
# Output

'data.frame':	4898 obs. of  12 variables:
 $ fixed.acidity       : num  7 6.3 8.1 7.2 7.2 8.1 6.2 7 6.3 8.1 ...
 $ volatile.acidity    : num  0.27 0.3 0.28 0.23 0.23 0.28 0.32 0.27 0.3 0.22 ...
 $ citric.acid         : num  0.36 0.34 0.4 0.32 0.32 0.4 0.16 0.36 0.34 0.43 ...
 $ residual.sugar      : num  20.7 1.6 6.9 8.5 8.5 6.9 7 20.7 1.6 1.5 ...
 $ chlorides           : num  0.045 0.049 0.05 0.058 0.058 0.05 0.045 0.045 0.049 0.044 ...
 $ free.sulfur.dioxide : num  45 14 30 47 47 30 30 45 14 28 ...
 $ total.sulfur.dioxide: num  170 132 97 186 186 97 136 170 132 129 ...
 $ density             : num  1.001 0.994 0.995 0.996 0.996 ...
 $ pH                  : num  3 3.3 3.26 3.19 3.19 3.26 3.18 3 3.3 3.22 ...
 $ sulphates           : num  0.45 0.49 0.44 0.4 0.4 0.44 0.47 0.45 0.49 0.45 ...
 $ alcohol             : num  8.8 9.5 10.1 9.9 9.9 10.1 9.6 8.8 9.5 11 ...
 $ quality             : int  6 6 6 6 6 6 6 6 6 6 ...
```

We see 11 numeric variables with a `quality` label as an integer type.  We'll use there continuous features to estimate quality.
 
```r
# Split the data into training and test sets. (0.75, 0.25) split.
sampled <- base::sample(1:nrow(wine_quality), 0.75 * nrow(wine_quality))
train <- wine_quality[sampled, ]
test <- wine_quality[-sampled, ]

# The predicted column is "quality" which is a scalar from [3, 9]
train_x <- as.matrix(train[, !(names(train) == "quality")])
test_x <- as.matrix(test[, !(names(train) == "quality")])
train_y <- train[, "quality"]
test_y <- test[, "quality"]

## Train a model with alpha and lambda parameters
model <- glmnet(train_x, train_y, alpha = 0.5, lambda = 0.01, family= "gaussian", standardize = FALSE)

## Perform cross validation and extract the lambda
l1se <- cv.glmnet(train_x, train_y, alpha = alpha)$lambda.1se

## Predict on test data
predicted <- predict(model, test_x)

## Calculate accuracy metrics
rmse <- sqrt(mean((predicted - test_y) ^ 2))
mae <- mean(abs(predicted - test_y))
r2 <- as.numeric(cor(predicted, test_y) ^ 2)

## Save cross validation plot to disk
png(filename = "ElasticNet-CrossValidation.png")
plot(cv.glmnet(train_x, train_y, alpha = alpha), label = TRUE)
dev.off()

## Save model to disk
saveRDS(model, file = "glmnet_model.rds")
```


* Set of APIs/solution description
* Existing data science code
* Wrappers
* Logging plots and other arbitrary artifacts

* Interacting with Runs
* 

