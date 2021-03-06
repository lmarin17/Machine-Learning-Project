# Machine Learning
Laura Marin  
Wednesday, February 24, 2016  
#  Machine Learning Project
***


## Initial Setup and Objectives

Data provided by the Human Activity Recognition project contains movement readings combined with a 5-level factor variable of the exercise class.  

The objective of this model is to predict the correct class given a set of variable data. 

### Initial load of libaries and data

The data come from http://groupware.les.inf.puc-rio.br/har, which is provided by the Human Activity Recognition site
Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H., Qualitative Activity Recognition of Weight Lifting Exercises.  Proceedings of 4th International Conference in Cooperation with SIGCHI (Augmented Human '13).  Stuttgart, Germany: ACM SIGCHI, 2013.  Read more: http://groupware.les.inf.puc-rio.br/har#ixzz3yfgbGpDg

The site contains two files, a training and a testing file.  I've decided to further split the training file, so I'll load the testing file as the validation set which are the 20 submittal test cases.  


```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
library(e1071)

setwd("~/a_Research and Reading/Data Scientist Toolbox/Machine Learning")
rawtraining <- read.csv("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv")
validation <- read.csv("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv")
```

## Cross Validation

Upon loading the training set I immediately split it into a 75% training set and a 25% testing set using random subsampling. I chose random subsampling since this is a classification problem and this ensures the testing and training sets to have a similar distribution of each class.  This will allow me to explore various models on my training set, test the best candidates on my test set, then do a final test of my chosen model on my validatation set.  In this way I attempt to minimize the out of sample error as much as possible.

Looking at the training set I found there were 160 variables, many of which were mostly 0 or null.  To reduce the overhead of these variables, which I assume won't add intelligence to the model, I dropped these columns from training and testing.




```r
inTrain <- createDataPartition(rawtraining$classe, p = 3/4)[[1]]
trainingComplete <- rawtraining[ inTrain,]
testingComplete <- rawtraining[-inTrain,]

deleteCols <- c(1, 2,3,4,5,6,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,
                29,30,31,32,33,34,35,36,50,51,52,53,54,55,56,57,58,59,69,70,
                71,72,73,74,75,76,77,78,79,80,81,82,83,87,88,89,90,91,92,
                93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,
                110,111,112,125,126,127,128,129,130,131,132,133,134,135,136,
                137,138,139,141,142,143,144,145,146,147,148,149,150)
 
training <- trainingComplete[, - deleteCols]
testing <- testingComplete[, -deleteCols]
```


There are 53 numeric variables left which very possibly would contain some correlation, at least between sets of variables.  I performed a PCA analysis to analyze these correlations, and determine how many PCA components I should include in my model to condense the number of variables involved but not lose significant predictive power.


```r
preProc <- prcomp(training[,-54])
summary(preProc)
```

```
## Importance of components:
##                             PC1      PC2      PC3      PC4       PC5
## Standard deviation     596.0634 533.5853 477.4959 378.9004 356.17640
## Proportion of Variance   0.2501   0.2004   0.1605   0.1011   0.08931
## Cumulative Proportion    0.2501   0.4506   0.6111   0.7122   0.80148
##                              PC6       PC7       PC8       PC9      PC10
## Standard deviation     255.97609 235.69259 198.83385 174.37293 157.57448
## Proportion of Variance   0.04613   0.03911   0.02783   0.02141   0.01748
## Cumulative Proportion    0.84761   0.88672   0.91455   0.93596   0.95344
##                             PC11     PC12     PC13     PC14    PC15
## Standard deviation     117.64645 95.83615 89.46650 76.20011 68.4787
## Proportion of Variance   0.00974  0.00647  0.00564  0.00409  0.0033
## Cumulative Proportion    0.96319  0.96965  0.97529  0.97938  0.9827
##                            PC16     PC17     PC18     PC19     PC20
## Standard deviation     62.89387 56.90054 53.47110 49.88990 48.89909
## Proportion of Variance  0.00278  0.00228  0.00201  0.00175  0.00168
## Cumulative Proportion   0.98546  0.98774  0.98975  0.99151  0.99319
##                            PC21    PC22     PC23     PC24     PC25
## Standard deviation     42.25251 37.7066 35.34512 33.13909 30.62566
## Proportion of Variance  0.00126  0.0010  0.00088  0.00077  0.00066
## Cumulative Proportion   0.99445  0.9954  0.99633  0.99710  0.99776
##                            PC26    PC27     PC28     PC29     PC30
## Standard deviation     25.50860 23.8619 21.53690 20.34510 17.20746
## Proportion of Variance  0.00046  0.0004  0.00033  0.00029  0.00021
## Cumulative Proportion   0.99822  0.9986  0.99895  0.99924  0.99945
##                            PC31     PC32    PC33    PC34    PC35    PC36
## Standard deviation     15.06308 14.22507 9.92268 7.66158 7.28812 6.66534
## Proportion of Variance  0.00016  0.00014 0.00007 0.00004 0.00004 0.00003
## Cumulative Proportion   0.99961  0.99975 0.99982 0.99986 0.99990 0.99993
##                           PC37    PC38    PC39    PC40    PC41  PC42  PC43
## Standard deviation     6.07030 4.63977 3.70983 3.50850 3.34912 1.954 1.516
## Proportion of Variance 0.00003 0.00002 0.00001 0.00001 0.00001 0.000 0.000
## Cumulative Proportion  0.99995 0.99997 0.99998 0.99999 0.99999 1.000 1.000
##                        PC44   PC45  PC46   PC47  PC48   PC49   PC50   PC51
## Standard deviation     1.08 0.4595 0.389 0.3597 0.316 0.2392 0.1999 0.1856
## Proportion of Variance 0.00 0.0000 0.000 0.0000 0.000 0.0000 0.0000 0.0000
## Cumulative Proportion  1.00 1.0000 1.000 1.0000 1.000 1.0000 1.0000 1.0000
##                          PC52    PC53
## Standard deviation     0.1026 0.03684
## Proportion of Variance 0.0000 0.00000
## Cumulative Proportion  1.0000 1.00000
```

The results show that about 99% of the variability is explained with 19 PCA components, and 99.9% with 29 components, far fewer than 53.  So by including the threshold = .999 in the preprocess options of the models explored below, I should be able to create models with a workable number of components, while still explaining much of the variability.


## Classification Modeling

This is a classification problem with a five-factor outcome.  Therefore, I looked at a few different classification and tree-based models, and compared their accuracies, then created an ensemble model to determine if a set of models would be a better option to move forward with.

###  A basic decision tree model  
Since this is a basic classification problem, I started with creating a simple decision tree model using pca preprocessing.  


```r
set.seed(62433)
modRpart <- train(training$classe ~ ., method = "rpart", preProcess="pca", data = training,
                  trControl = trainControl(preProcOptions = list(thresh = .999)))
predRpart <- predict(modRpart, training)
confusionMatrix(predRpart, training$classe)$overall[1]
```

```
##  Accuracy 
## 0.3717896
```


### Gradient Boosting
The next option was a gradient boosting model, which is an ensemble of decision trees.  This should increase the accuracy of the rpart model.  Overfitting was a risk here, but validation using the testing dataset would give an unbiased view of the overfitting risk.


```r
set.seed(62433)
modGBM <- train(training$classe ~ ., method = "gbm", data = training, preProcess = "pca", verbose = FALSE,
                trControl = trainControl(preProcOptions = list(thresh = .999)))

predGBM <- predict(modGBM, training)
confusionMatrix(predGBM, training$classe)$overall[1]
```

```
##  Accuracy 
## 0.9125561
```


### Linear Discriminant Analysis
This model tries to fit a number of lines through the data to determine the classifications.  


```r
set.seed(62433)
modLDA <- train(training$classe ~ ., method = "lda", data = training, preProcess = "pca", 
                trControl = trainControl(preProcOptions = list(thresh = .999)))

predLDA <- predict(modLDA, training)
confusionMatrix(predLDA, training$classe)$overall[1]
```

```
##  Accuracy 
## 0.6821579
```



### Quadratic Discriminant Analysis
This model increases the complexity of the LDA model by dropping the assumption of linearity, and fits a number of quadratic functions through the data. 


```r
set.seed(62433)
modQDA <- train(training$classe ~ ., method = "qda", data = training, preProcess = "pca",
                trControl = trainControl(preProcOptions = list(thresh = .999)))

predQDA <- predict(modQDA, training)
confusionMatrix(predQDA, training$classe)$overall[1]
```

```
## Accuracy 
## 0.887077
```


### Support Vector Machine
This model creates a number of hyperplanes through the data to classify the results. 


```r
set.seed(62433)
modSvm <- svm(training$classe ~ ., data = training)
predSvm <- predict(modSvm, training)
confusionMatrix(predSvm, training$classe)$overall[1]
```

```
##  Accuracy 
## 0.9559723
```



### Performance of the Individual Models on the Testing Set
Based on the model results on the training set, I chose not to go forward with the basic decision tree model using the rpart algorithm, nor the linear discriminant model.  I tested each of the remaining individual models on the testing subset to confirm their effectiveness on a new dataset. This confirmation will reduce the out of sample error I would expect on the validation set.  


```r
predGBMTest <- predict(modGBM, testing)
confusionMatrix(predGBMTest, testing$classe)$overall[1]
```

```
##  Accuracy 
## 0.8839723
```

```r
predQDATest <- predict(modQDA, testing)
confusionMatrix(predQDATest, testing$classe)$overall[1]
```

```
##  Accuracy 
## 0.8817292
```

```r
predSvmTest <- predict(modSvm, testing)
confusionMatrix(predSvmTest, testing$classe)$overall[1]
```

```
## Accuracy 
## 0.949429
```



### A random forest ensemble model 
Finally, I created an random forest ensemble model of the testing set predictions of the highest performing individual models, the GBM, QDA and SVM models.   


Testing Set Performance

```r
predDataTest <- data.frame(predGBMTest, predQDATest, predSvmTest, classe = testing$classe)
combModFitTest <- train(classe ~ ., method = "rf", data = predDataTest)
combPredTest <- predict(combModFitTest, predDataTest)
confusionMatrix(combPredTest, testing$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1386   54    2    3    0
##          B    1  869   28    1    2
##          C    5   25  818   76   21
##          D    1    0    4  722   12
##          E    2    1    3    2  866
## 
## Overall Statistics
##                                          
##                Accuracy : 0.9504         
##                  95% CI : (0.944, 0.9564)
##     No Information Rate : 0.2845         
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.9372         
##  Mcnemar's Test P-Value : < 2.2e-16      
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9935   0.9157   0.9567   0.8980   0.9612
## Specificity            0.9832   0.9919   0.9686   0.9959   0.9980
## Pos Pred Value         0.9592   0.9645   0.8656   0.9770   0.9908
## Neg Pred Value         0.9974   0.9800   0.9907   0.9803   0.9913
## Prevalence             0.2845   0.1935   0.1743   0.1639   0.1837
## Detection Rate         0.2826   0.1772   0.1668   0.1472   0.1766
## Detection Prevalence   0.2947   0.1837   0.1927   0.1507   0.1782
## Balanced Accuracy      0.9884   0.9538   0.9627   0.9469   0.9796
```


The final performance is comparable to the highest performing individual model, the Support Vector Machines model, as seen visually by comparison barplots of the testing values, Support Vector Machines predictions and Random Forest Ensemble predictions.  


```r
par(mfrow = c(1,3))
barplot(table(testing$classe),col="blue", main = "Testing Values")
barplot(table(predSvmTest), col="red", main = "SVM Predictions")
barplot(table(combPredTest), col="red", main = "Ensemble Predictions")
```

![](Machine_Learning_Project_files/figure-html/unnamed-chunk-11-1.png) 


## Conclusion
Doing a number of different models gave some assurance that the final Support Vector Machines model accuracy was sufficient for this exercise.  A final test on the validation set was successful with all 20 predictions correct.


