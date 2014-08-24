Markdown file for Practical Machine Learning project.
=====================================================


Load data
---------

```r
library(caret)
```
* Load csv file.

```r
data<-read.csv("pml-training.csv")
```
Clean and partition data
-----------------------
* Remove rows which has new_window value yes.

```r
data<-data[data$new_window=="no",]
```
* Remove columns which has all NA values

```r
data<-data[,colSums(is.na(data)) != nrow(data)]
```
* Remove columns which has all empty values

```r
data<-data[,colSums(data=="") != nrow(data)]
```
* Remove first seven columns, as these values cannot be used for prediction.

```r
data<-data[,-c(1:7)]
```
* Partition data into training and cross validation set.

```r
inTrain <- createDataPartition(y=data$classe,p=0.75, list=FALSE)
training<-data[inTrain,]
cv<-data[-inTrain,]
```
Use machine learning
--------------------
* Create a trainControl for faster running to train function.

```r
trControl = trainControl(method = "cv", number = 4, allowParallel = TRUE,verboseIter = TRUE)
```
* Train using random forrest method.

```r
modFit <- train(classe ~., data=training, method="rf", prox=TRUE, trControl=trControl)
```

```
## + Fold1: mtry= 2 
## - Fold1: mtry= 2 
## + Fold1: mtry=27 
## - Fold1: mtry=27 
## + Fold1: mtry=52 
## - Fold1: mtry=52 
## + Fold2: mtry= 2 
## - Fold2: mtry= 2 
## + Fold2: mtry=27 
## - Fold2: mtry=27 
## + Fold2: mtry=52 
## - Fold2: mtry=52 
## + Fold3: mtry= 2 
## - Fold3: mtry= 2 
## + Fold3: mtry=27 
## - Fold3: mtry=27 
## + Fold3: mtry=52 
## - Fold3: mtry=52 
## + Fold4: mtry= 2 
## - Fold4: mtry= 2 
## + Fold4: mtry=27 
## - Fold4: mtry=27 
## + Fold4: mtry=52 
## - Fold4: mtry=52 
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 27 on full training set
```
Corss validation
----------------
* Find acuracy of the model on cross validation set.

```r
pred <- predict(modFit,cv)
cv$predRight <- pred==cv$classe
tab<-table(pred,cv$classe)
```
* Find out the accuracy in percent.

```r
correct=0; wrong=0;
for(i in 1:5) { for(j in 1:5) { if(i==j) {correct=correct+tab[i,j]; } else {wrong=wrong+tab[i,j];} } }
percent<-100*(correct)/(correct+wrong)
```
*Accuracy of the used model


```r
print(percent)	
```

```
## [1] 99.27
```
* If accuracy is good then use the model for predction on the actual test set.

```r
test_data<-read.csv("pml-testing.csv")
pred <- predict(modFit,test_data)
```
Prediction 
----------

```r
print(pred)
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
