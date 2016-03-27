# Meachine-learning-project
It is my project in My Meachine Learning class
---
title: "Meachine learning project"
author: "zouxia"
date: "2016年3月26日"
output: html_document
---
##preprocessing of the data
```{r,results='hide',cache=TRUE,warning=FALSE}
setwd("D:\\R programming\\meachine learning")
trainingp<-read.csv("pml-training.csv",h=T)
trainingt<-trainingp[1:10,]
testing<-read.csv("pml-testing.csv",h=T)
#dealing with Missing data. After observe the missing data, i found the all missing data are max min ...std. It means these data is calculate during a period of time, so there must be Missing data. So i come up with a method that substitute the missing data with it's lastest below data. The mothod is:
na.lomf <- function(x) {
  
  na.lomf.0 <- function(x) {
    non.na.idx <- which(!is.na(x))
    non.na.idx<-c(0,non.na.idx)
    as.numeric(rep.int(x[non.na.idx], diff(c(non.na.idx))))
  }
  
  dim.len <- length(dim(x))
  
  if (dim.len == 0L) {
    na.lomf.0(x)
  } else {
    apply(x, dim.len, na.lomf.0)
  }
}
#then find out cols that have missing data
numcol<-grep("kurtosis|skewness|max|min|ampli|var|std|avg", colnames(trainingp))
trainingp[,numcol]<-apply(trainingp[,numcol],2,na.lomf)
testing[,numcol]<-apply(testing[,numcol],2,na.lomf)
a<-which(!is.na(trainingt[1,numcol]))
trainingp<-trainingp[,-numcol[a]]
testing<-testing[,-numcol[a]]
#Split the trainging data into validation data and training data
library(caret)
intrain<-createDataPartition(trainingp$classe,p=0.7,list=F)
training<-trainingp[intrain,]
valida<-trainingp[-intrain,]
```
After a clear examination of the variables,i found there are six aspects,(roll,pitch,yaw,raw accelerometer ,gyroscope and magnetometer readings. And for each aspects there are eight features,mean, variance,standard deviation, max, min, amplitude, kurtosis and skewness.

##exploratory data analysis and variables selection
```{r,results="hide",cache=TRUE,warning=F}
summary(training)
#Remove redudant variables. Remove variables that are highly correlated
correlationmatrix<-cor(training[,-c(1:7,127)])
highlycorrelated<-findCorrelation(correlationmatrix,cutoff = 0.75)
trainingr<-training[,-(highlycorrelated+7)]
validar<-valida[,-(highlycorrelated+7)]
#it still have 71 variables. So i rank the variables by importance.
#rankmodel<-train(classe~.,data=trainingr[(1:1000),-(1:7)],method="lvq",preProcess="scale",trControl=control)
#importance<-varImp(rankmodel,scale=F)
#Using automatic feature selection to select variables.
control <- rfeControl(functions=rfFuncs, method="cv", number=10)
samplenum<-sample(length(trainingr$classe),1000,replace = F)
results <- rfe(trainingr[samplenum,-c(1:7,71)], trainingr[samplenum,71], rfeControl=control)
#select 16 variables base on it
trainings<-training[,results$optVariables]
```
After the process of removing redudant variables and select variables via rfe . i choose 16 variables `r results$optVariables`

##Build Model on the training data
```{r,warning=F,cache=TRUE}
#First try random forest method
model1<-train(classe~.,data=training[,c(results$optVariables,"classe")],method="rf",preProcess="scale")
library(adabag)
model2<-train(classe~.,data=training[,c(results$optVariables,"classe")],method="AdaBag",preProcess="scale")
#test accuracy on the validation dataset
accuracy1<-confusionMatrix(predict(model1,valida),valida$classe)
accuracy2<-confusionMatrix(predict(model2,valida),valida$classe)

```


##predict with testing dataset
```{r}
predicttest<-predict(model1,testing)
predicttest
```