---
title: "Project"
author: "Conghuan Xu"
date: "April 26, 2015"
output: html_document
---
Before we start we shoud include come libraries.

```r
library(Hmisc)
library(caret)
library(doMC)
library(kernlab)
registerDoMC(cores = 5)
set.seed(1024)
```
# Read and process the data
First we need to do is to download the data to our current workspace and read the data. In particular, we change '#DIV/0' to NA.

```r
training_data <- read.csv("pml-training.csv", na.strings=c("#DIV/0!") )
testing_data <- read.csv("pml-testing.csv", na.strings=c("#DIV/0!") )
```
After we read the data, we need do some preprocess to the data before we train the data. 
Here, we delete user name, timestamps and windows first.

```r
training_data <- training_data[,-1:7]
testing_data <- testing_data[,-1:7]
```
Next we need to omit all the column which have the NA

```r
prodata_training <- training_data[colnames(training_data[colSums(is.na(training_data)) 
                                                         == 0])]
prodata_testing <- testing_data[colnames(training_data[colSums(is.na(training_data)) 
                                                         == 0])[1:52]]
```
Now we can create our training data and our corss validation data.

```r
temp <- createDataPartition(y=prodata_training$classe, p=0.75, list=FALSE )
training <- prodata_training[temp,]
cross <- prodata_training[-temp,]
```

# Training our data

```r
modelFit <- train(training$classe ~ .,method="glm",preProcess="pca",data=training[,-53])
```

# Error report

```r
confusionMatrix(training$classe,predict(modelFit,training[,-53]))
confusionMatrix(cross$classe,predict(modelFit,cross[,-53]))
```

# Forcast on test data
Here we do two things, one is forcast our testing data 'classe', second is write the results down.

```r
pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
  }
}
results <- predict(modelFit, prodata_testing)
results
pml_write_files(results)
```

# More things need mention
Using random forest will give us very good accuracy, however, it takes very long time to do it.  
I also try to use PCA method to compress the data, but I get some pretty big error, and ugly confusion matrix. Maybe many of the components are important, thus we cannot compress the data.
