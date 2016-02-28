# Getting and Cleaning Data Course Project
library(data.table)

## Download file
if(!file.exists("./data")){dir.create("./data")}
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl,destfile="./data/Dataset.zip",method="curl")

## Unzip file
unzip("./data/Dataset.zip",exdir="./data")

## Get file list
path_file <- file.path("./data" , "UCI HAR Dataset")
files<-list.files(path_file, recursive=TRUE)

## Load data
Xtest  <- data.table(read.table(file.path(path_file, "test" , "X_test.txt" ),header = FALSE))
Ytest  <- data.table(read.table(file.path(path_file, "test" , "y_test.txt" ),header = FALSE))
SubjectTest  <- data.table(read.table(file.path(path_file, "test" , "subject_test.txt" ),header = FALSE))

Xtrain  <- data.table(read.table(file.path(path_file, "train" , "X_train.txt" ),header = FALSE))
Ytrain  <- data.table(read.table(file.path(path_file, "train" , "y_train.txt" ),header = FALSE))
SubjectTrain  <- data.table(read.table(file.path(path_file, "train" , "subject_train.txt" ),header = FALSE))

features <- data.table(read.table(file.path(path_file , "features.txt" ),header = FALSE)[,2])
activity_labels <- data.table(read.table(file.path(path_file , "activity_labels.txt" ),header = FALSE)[,2])

Ytest <- cbind(Ytest,activity_labels[Ytest$V1])
Ytrain <- cbind(Ytrain,activity_labels[Ytrain$V1])

## Uses descriptive activity names to name the activities in the data set
## Appropriately labels the data set with descriptive variable names.

names(Xtest) <- as.character(features$V1)
names(Ytest) <- c("activity_ID", "activity_labels")
names(Xtrain) <- as.character(features$V1)
names(Ytrain) <- c("activity_ID", "activity_labels")
names(SubjectTest) <- "Subject"
names(SubjectTrain) <- "Subject"

## Extract features only mean and std
extract_features <- grepl("mean|std", features$V1)
Xtest <- Xtest[,extract_features,with=FALSE]
Xtrain <- Xtrain[,extract_features,with=FALSE]

## Merges the training and the test sets to create one data set.
testDT <- cbind(SubjectTest,Ytest,Xtest)
trainDT <- cbind(SubjectTrain,Ytrain,Xtrain)
dataT <- rbind(testDT,trainDT)

ilabels   = c("Subject", "activity_ID", "activity_labels")
dlabels = setdiff(colnames(dataT), ilabels)
melted_data = melt(dataT, id = ilabels, measure.vars = dlabels)

# Apply mean function to dataset using dcast function
tidyDT   = dcast(melted_data, Subject + activity_labels ~ variable, mean)

write.table(tidyDT, file = "./tidy_data.txt")

