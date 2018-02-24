# GettingAndCleaningData_Project
Getting and Cleaning Data Course Project

### The course project is the final assignment of the third course, "Getting and Cleaning Data", in the "Data science specialisazion". 

The purpose of this project is to demonstrate our ability to collect, work with, and clean a data set.
We are working on a dataset that represent data collected from the accelerometers from the Samsung Galaxy S smartphone.
See http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones for more info.

## The course project:
1. Merge the training and the test sets to create one data set.
2. Extract only the measurements on the mean and standard deviation for each measurement.
3. Use descriptive activity names to name the activities in the data set
4. Appropriately label the data set with descriptive variable names.
5. From the data set in step 4, create a second, independent tidy data set with the average of each variable for each activity and each subject.

## This repo includes three files:
1. run_data.R - The script that does the described project goals. Further explanations of the functions and workings of the script are found as notes in the script.
2. CookBook.md - Detailed info about the data, variables, measurements and functions used in the assignment.
3. README.md - You are.
 
## The Script:

######################################################
#                 run_data.R script                  #
######################################################

### Setting working directory 
Path <- "#your working_directory_here"
setwd(Path)

### Creating new directory and downloading file
if(!file.exists("./data")){dir.create("./data")}
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl,destfile="./data/Dataset.zip")

### Unzipping DataSet to the new directory
unzip(zipfile="./data/Dataset.zip",exdir="./data")

### Dplyr is used throughout the script
library(dplyr)

### Reading the files, setting class tbl_df and merging of the datasets
y_test	<- tbl_df(read.csv("./data/UCI HAR Dataset/test/y_test.txt", sep = "", header = FALSE))
subtest <- tbl_df(read.csv("./data/UCI HAR Dataset/test/subject_test.txt", sep = "", header = FALSE))
x_test	<- tbl_df(read.csv("./data/UCI HAR Dataset/test/X_test.txt", sep = "", header = FALSE))

test.data <- bind_cols(subtest,y_test,x_test)

y_train	 <- tbl_df(read.csv("./data/UCI HAR Dataset/train/y_train.txt" , sep = "", header = FALSE))
subtrain <- tbl_df(read.csv("./data/UCI HAR Dataset/train/subject_train.txt", sep = "", header = FALSE))
x_train	 <- tbl_df(read.csv("./data/UCI HAR Dataset/train/X_train.txt", sep = "", header = FALSE))

train.data <- bind_cols(subtrain,y_train,x_train)

### Merging train and test data into one
RunData <- bind_rows(test.data, train.data)

### Reading the features file
feat <- read.csv("./data/UCI HAR Dataset/features.txt", sep="", header=FALSE)

### Getting the new column names from features and renaming the columns in RunData
col.nm <- as.vector(feat[, 2])
colnames(RunData)[3:ncol(RunData)] <- col.nm
colnames(RunData)[c(1,2)] <- c("Subject_Id", "Activity")

### Finding the std and mean measures in RunData
keep <- grep("std|mean",colnames(RunData), value = F)
RunData <- RunData[,c(1,2,keep)]

### Reading the acivity_labels file and changeing the columnnames
act <- tbl_df(read.csv("./data/UCI HAR Dataset/activity_labels.txt", sep="", header=FALSE))
colnames(act) <- c("Activity", "Activity_Label")

### Joining RunData and activity_lables
RunData <- left_join(RunData,act)

### Changeing the order of the columns 
RunData <- RunData %>% select(Activity_Label, everything()) %>% select(-Activity)

### Doing some freshing-up to the colnames
RunData <- rename_all(RunData,funs(gsub("[[:punct:]]", "", names(RunData)))) 
RunData <- rename_all(RunData,funs(gsub("BodyBody", "Body", names(RunData)))) 
RunData <- rename_all(RunData,funs(gsub("^f", "frequency_", names(RunData))))
RunData <- rename_all(RunData,funs(gsub("^t", "time_", names(RunData))))
RunData <- rename_all(RunData,funs(gsub("mean", "_Mean ", names(RunData)))) 
RunData <- rename_all(RunData,funs(gsub("std", "_Std ", names(RunData))))

### Creating a new separate tidy dataset grouped by activity_label and subject
RunDataSum <- group_by(RunData, ActivityLabel, SubjectId) 

### Averageing all the columns
RunDataSum <- summarise_all(RunDataSum, funs(mean))

### Output the summery file to disk
write.table(RunDataSum, file="./data/RunDataSummery.txt", row.name=FALSE)

### Ommit the datasets and variables not in use 
rm(col.nm, keep, act, feat, subtest, subtrain, x_test, x_train, y_test, y_train, test.data, train.data)

### FIN
