class.coursera.org-predmachlearn-016
====================================

# Practical Machine Learning
## Prediction Assignment Writeup
This is my analisys and solution to the problem. The `pml-testing.csv´ dont have all the columns and then i toke the option of remove all the "unused" columns. The train is SLOW but very accurate.

```
#load data

library(caret)
library(doMC)
registerDoMC(cores = 2)
set.seed(2)



datos = read.csv("~/Descargas/pml-training.csv")
training <- subset(datos,select=-c(X,user_name,raw_timestamp_part_1,raw_timestamp_part_2,cvtd_timestamp))

test <- read.csv("~/Descargas/pml-testing.csv")
testing <- subset(test,select=-c(X,user_name,raw_timestamp_part_1,raw_timestamp_part_2,cvtd_timestamp,problem_id))


#visualization

dim(datos)
names(datos)
str(datos)
plot(datos$classe ~ . , data=datos)


# comparative of unused columns training vs test set
nzvTrain <- nearZeroVar(training)
nzvTest <- nearZeroVar(testing)
nzvTest %in% nzvTrain

filteredTraining <- training[, -nzvTest]# removing unused columns

dim(filteredTraining)

set.seed(2)

# creating a partition over the training set to achieve validation set
inTrain = createDataPartition(filteredTraining$classe, p = 0.6)[[1]]
trainFit = filteredTraining[ inTrain,]
testFit = filteredTraining[-inTrain,]

# fit to random forest with preprocessing
modFit <- train(trainFit$classe ~ . ,method="rf", allowParallel=TRUE ,data=trainFit , preProcess = c("center", "scale"))
print(modFit$finalModel)

____________________________________

Call:
 randomForest(x = x, y = y, mtry = param$mtry, allowParallel = TRUE) 
               Type of random forest: classification
                     Number of trees: 500
No. of variables tried at each split: 27

        OOB estimate of  error rate: 0.38%
Confusion matrix:
     A    B    C    D    E  class.error
A 3347    0    0    0    1 0.0002986858
B    9 2267    3    0    0 0.0052654673
C    0   11 2043    0    0 0.0053554041
D    0    0   13 1915    2 0.0077720207
E    0    0    0    6 2159 0.0027713626

____________________________________


filteredTest <- testing[, -nzvTest]# removing unused columns
filteredTest["classe"] <- filteredTraining[dim(filteredTraining)[2]]# adding classe column

# prediction over the test set
prediction <- predict(modFit, newdata=filteredTest)

____________________________________
prediction
 [1] B A B A A E D B A A B C B A E E A B B B
Levels: A B C D E
____________________________________



# prediction over the test set partition of training set,
# to calculate accuracy and confusion matrix
predict1 <- predict(modFit, testFit)
cm <- confusionMatrix(predict1,testFit$classe)



_____________________________________

Confusion Matrix and Statistics

          Reference
Prediction    A    B    C    D    E
         A 2232    8    0    0    0
         B    0 1508   11    0    0
         C    0    2 1357    9    0
         D    0    0    0 1277    7
         E    0    0    0    0 1435

Overall Statistics
                                          
               Accuracy : 0.9953          
                 95% CI : (0.9935, 0.9967)
    No Information Rate : 0.2845          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.994           
 Mcnemar's Test P-Value : NA              

Statistics by Class:

                     Class: A Class: B Class: C Class: D Class: E
Sensitivity            1.0000   0.9934   0.9920   0.9930   0.9951
Specificity            0.9986   0.9983   0.9983   0.9989   1.0000
Pos Pred Value         0.9964   0.9928   0.9920   0.9945   1.0000
Neg Pred Value         1.0000   0.9984   0.9983   0.9986   0.9989
Prevalence             0.2845   0.1935   0.1744   0.1639   0.1838
Detection Rate         0.2845   0.1922   0.1730   0.1628   0.1829
Detection Prevalence   0.2855   0.1936   0.1744   0.1637   0.1829
Balanced Accuracy      0.9993   0.9958   0.9951   0.9960   0.9976

_____________________________________

# importance of variables
vars <- varImp(modFit)
plot(vars)


# grafical model fiting
plot(modFit)
plot(modFit$finalModel, uniform=TRUE)

´´´

