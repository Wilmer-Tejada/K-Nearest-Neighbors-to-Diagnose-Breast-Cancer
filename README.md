
# Breast Cancer Diagnosis using K-Nearest Neighbors Algorithm (KNN)

## Using the KNN algorithm, we would like to predict whether a tumor is Benign (B) or Malignant (M).

# Step 1: Collecting Data
We will be using the wisc_bc_data.csv (Wisconsin Breast Cancer Diagnostic) which is a common dataset provided by UCI data reository. This data file contains measurements from digitized images of fine-needle aspirate of a breast mass. The breast cancer data includes 569 examples of cancer biopsies, each with 32 features. One feature is an identification number, another is the cancer diagnosis, and 30 are numeric-valued laboratory measurements such as radius, smoothness, symmetry, etc.

``` r 
library(class);library(gmodels);library(car)
datasetLocation="http://www.sci.csueastbay.edu/~esuess/classes/Statistics_6620/Presentations/ml4/wisc_bc_data.csv"
wbcd <- read.csv(url(datasetLocation))
```

# Step 2: Preparing and Exploring the Data
#### Removing ID Variable to avoid computation issues.
``` r 
wbcd = wbcd[-1]
```


#### Table showing B (Benign) and M (Malignant) Diagnosis counts and proportions.
``` r 
table(wbcd$diagnosis)
round(prop.table(table(wbcd$diagnosis)),digits = 2)
```
#### We have 357 Benign (B) and 212 Malignant (M) counts.

``` r 
scatterplotMatrix(~radius_mean+texture_mean+perimeter_mean+area_mean+smoothness_mean | diagnosis, data=wbcd)
```

#### Making a few adjustments to our data set in order to classify our diagnosis variable as a "factor" type variable rather than just a category. 
```{r}
wbcd$diagnosis <- factor(wbcd$diagnosis, levels = c("B", "M"),
                         labels = c("Benign", "Malignant"))
```

#### We create a function that will normalize all our numerical variables in order to properly run the KNN algorithm. Because KNN looks at all variables with equal wieghts, we must normalize our data or our results will be skewed. This will make it so that all our variables are in a (0,1) range. Using the lapply function we can instantly apply the normalize function across all our columns, or variables, instead of having to do it one by one.
```{r}
normalize <- function(x) {
  return ((x - min(x)) / (max(x) - min(x)))
}

wbcd_n <- as.data.frame(lapply(wbcd[2:31], normalize))
summary(wbcd_n$area_mean)
```

# Step 3: Training a Model on our data. 
#### We will use our first 469 out of 569 observations as a training set. We will use the 100 remaining obesrvations to test our model.  We inspect our dataset and notice that it is randomly organized, so there is no need for us to randomly sample our observations. Here we actually run our KNN algorithm looking at our 21 nearest neighbors.
```{r}
wbcd_train <- wbcd_n[1:469, ]
wbcd_test <- wbcd_n[470:569, ]

wbcd_train_labels <- wbcd[1:469, 1]
wbcd_test_labels <- wbcd[470:569, 1]

wbcd_prediction=knn(train = wbcd_train, 
              test = wbcd_test, 
              cl=wbcd_train_labels, 
              k=21 )
```

# Step 4: Evaluating Model Performance
#### The predicted diagnosis vs what the actual diagnosis is is given in the table. 
```{r}
CrossTable(x = wbcd_test_labels, y = wbcd_prediction,
           prop.chisq = FALSE)
```
#### The way to interpret this table is by looking at what our actual results were on the left, and what our model predicted on the top. As we can see when both our actual value and model values both match, we consider this a successful prediction and therefor our final accuracy result is 98/100 which is a fairly accurate model.

# Step 5: Improving Model Performance
#### The first thing we can do to improve model performance is test a different method of normalizing our data. In this case we use z-score standardization with the R scale() function. We then apply the KNN algorithm again. 
```{r}
wbcd_z <- as.data.frame(scale(wbcd[-1:-2]))

wbcd_train2=wbcd_z[1:469,]
wbcd_test2=wbcd_z[470:569,]
wbcd_train_labels2=wbcd$diagnosis[1:469]
wbcd_test_actual2=wbcd$diagnosis[470:569]
wbcd_prediction2=knn(train = wbcd_train2, 
              test = wbcd_test2, 
              cl=wbcd_train_labels2, 
              k=21 )
CrossTable(wbcd_prediction2,wbcd_test_actual2, prop.chisq = FALSE)
```
#### This table shows a decline in accuracy, and so it is probably best to stick with our original normalization of between (0,1)

#### We could also use a different number of Nearest Neighbors to check for possible higher accuracy, but the standard is to take the square root of total number of observations which is what we did in the model above.
