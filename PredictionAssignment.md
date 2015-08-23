---
title: "Prediction Assignment Writeup"
author: "Billy Chen"
date: "Aug 23, 2015"
output: html
---

The goal of this project is to predict the manner in which those did the exercise.
The data for this project come from this source: http://groupware.les.inf.puc-rio.br/har.

## Preprocessing Data
1. Delete columns with empty values
2. Delete the first 7 columns which not related to the prediction: X, user_name, raw_timestamp_part_1, raw_timestamp_part_2, cvtd_timestamp, new_window, num_window.


```r
library(caret)
set.seed(12463)

training <- read.csv("pml-training.csv", stringsAsFactors=FALSE)
training$classe <- as.factor(training$classe)
training <- training[,-nearZeroVar(training)]
training <- training[,-c(1,2,3,4,5,6,7)]
```


ThereGet rid of NA values using KnnImpute method to impute those values.


```r
inTrain <- createDataPartition(y=training$classe, p=0.75, list=FALSE)
training <- training[inTrain,]
testing <- training[-inTrain,]

preObj <- preProcess(training[,-length(training)],method=c("center", "scale", "knnImpute", "pca"), thresh=0.9)
clean_data <- predict(preObj,training[,-length(training)])
```

## Prediction

Knn method will be used to build the model.  The accuracy is 0.9748. 


```r
modelFit <- train(training$classe ~.,data=clean_data, method="knn")
test <- predict(preObj, testing[,-length(testing)])
confusionMatrix(testing$classe, predict(modelFit,test))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1018    3    9    2    0
##          B   15  709   12    1    0
##          C    6   10  607    7    2
##          D    2    1   15  582    1
##          E    0    4    2    0  678
## 
## Overall Statistics
##                                           
##                Accuracy : 0.975           
##                  95% CI : (0.9695, 0.9798)
##     No Information Rate : 0.2824          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9684          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9779   0.9752   0.9411   0.9831   0.9956
## Specificity            0.9947   0.9905   0.9918   0.9939   0.9980
## Pos Pred Value         0.9864   0.9620   0.9604   0.9684   0.9912
## Neg Pred Value         0.9913   0.9939   0.9876   0.9968   0.9990
## Prevalence             0.2824   0.1972   0.1750   0.1606   0.1848
## Detection Rate         0.2762   0.1923   0.1647   0.1579   0.1839
## Detection Prevalence   0.2800   0.1999   0.1715   0.1630   0.1856
## Balanced Accuracy      0.9863   0.9829   0.9664   0.9885   0.9968
```


Finally, predict the result with the testing data:


```r
testing <- read.csv("pml-testing.csv", stringsAsFactors=FALSE)
testing <- testing[,names(testing) %in% names(training)]

test <- predict(preObj, testing)
result <- predict(modelFit, test)
result
```

```
##  [1] B A A A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
