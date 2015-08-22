##Getting and Cleaning Data Course Project

### 1. Description
This project will download and clean the UCI Human Activity Recognition Using Smartphones Data Set. Major steps include:
* Step 1: Merges the training and the test sets to create one data set.
* Step 2: Extracts only the measurements on the mean and standard deviation for each measurement. 
* Step 3: Uses descriptive activity names to name the activities in the data set
* Step 4: Appropriately labels the data set with descriptive variable names. 
* Step 5: Creates a second, independent tidy data set with the average of each variable for each activity and each subject.

The input data file is https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip and the output tidy data file is tidydata.txt

### 2. Download data 

Set working directory
```
setwd("./course_project")
```

Download the UCI Human Activity Recognition Using Smartphones Data Set and put it to the working directory
```
filename<-"UCI_HAR_Dataset.zip"
if (!file.exists(filename)) {
  fileUrl<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(fileUrl, filename)
}
```
Unzip file

folder_name<-"UCI HAR Dataset"
if(!file.exists(folder_name)) {
  unzip(filename)
}
  
### 3. Processing data based on requirements

**Step 1: Read all the data and merges the training and the test sets to create one data set.**

Below is a description of all the variables used in this script
* Variable features consists of feature names from "features.txt" 
* Variable activity_labels consists of activity IDs and names from "activity_labels.txt"
* Variable train consists of measurements and derived data from "X_train.txt"
* Variable trainActivities consists of activity data from "Y_train.txt"
* Variable trainSubjects consists of subject IDs from "subject_train.txt"
* Variable trainData consists of the combined training data
* Variable test consists of measurements and derived data from "X_test.txt"
* Variable testActivities consists of activity data from "Y_test.txt"
* Variable testSubjects consists of subject IDs from "subject_test.txt"
* Variable testData consists of the combined testing data
* Variable allData consists of the combined training and testing data

Loading the features and activity label files and rename the variables
Load feature.txt file
features<-read.table("./UCI HAR Dataset/features.txt",header=FALSE)
colnames(features)<-c("index", "feature_name")

Load activity_labels.txt and rename the variables
activity_labels<-read.table("./UCI HAR Dataset/activity_labels.txt",header=FALSE)
colnames(activity_labels)<-c("activityID", "activity_name")

Load training data
train<-read.table("./UCI HAR Dataset/train/X_train.txt",header=FALSE)
trainActivities<-read.table("./UCI HAR Dataset/train/Y_train.txt",header=FALSE)  
trainSubjects<-read.table("./UCI HAR Dataset/train/subject_train.txt",header=FALSE) 

Assign column names
colnames(train)<-features[,2]
colnames(trainActivities)<-"activityID"
colnames(trainSubjects)<-"subjectID"

Combine training data
trainData<-cbind(trainSubjects, trainActivities, train)

Load test data
test<-read.table("./UCI HAR Dataset/test/X_test.txt",header=FALSE)
testActivities<-read.table("./UCI HAR Dataset/test/Y_test.txt",header=FALSE)  
testSubjects<-read.table("./UCI HAR Dataset/test/subject_test.txt",header=FALSE) 

Assign column names
colnames(test)<-features[,2]
colnames(testActivities)<-"activityID"
colnames(testSubjects)<-"subjectID"

Combine training data
testData<-cbind(testSubjects, testActivities, test)

Combine training and test datasets
allData<-rbind(trainData, testData)

**Step 2: Extracts only the measurements on the mean and standard deviation for each measurement, that is, features with**
* mean(): Mean value
* std(): Standard deviation

View the structure of data
str(allData)

Select the mean and standard deviation and extract them into the variable subsetData
colNames  <- colnames(allData) 
featureSelected<- (grepl("subjectID",colNames) | grepl("activityID",colNames) | (!grepl("-meanFreq",colNames) & (grepl("-mean()..-",colNames) | grepl("-std()..-",colNames) | grepl("-mean()",colNames) | grepl("-std()",colNames)))) 
subsetData<-allData[,featureSelected==TRUE]


**Step 3: Uses descriptive activity names in activity_labels variable to name the activities in the subsetData**
subsetData=merge(subsetData, activity_labels, by.x="activityID", by.y="activityID", all.x =TRUE) 

Check activities names
head(subsetData$activity_name)


**Step 4: Appropriately labels the data set with descriptive variable names.** 
Change varaible names:
* Change mean() to mean
* Change std() to stdev
* Change prefix t to time
* Change prefix f to frequency
* Change Acc to Acceleration
* Change Gyro to Gyroscope
* Change Mag to Magnitude
* Change BodyBody to Body

colNames<-colnames(subsetData)
colNames<-gsub("mean\\()", "mean", colNames)
colNames<-gsub("std\\()", "stdev", colNames)
colNames<-gsub("^t", "time", colNames)
colNames<-gsub("^f", "frequency", colNames)
colNames<-gsub("Acc", "Acceleration", colNames)
colNames<-gsub("Gyro", "Gyroscope", colNames)
colNames<-gsub("Mag", "Magnitude", colNames)
colNames<-gsub("BodyBody", "Body", colNames)

Check colNames
colNames

Reassign the modified variable names to the dataset
colnames(subsetData) <- colNames

**Step 5: Creates a second, independent tidy data set with the average of each variable for each activity and each subject.**

Remove the activity type number as this data already has the activity names and activity ID information is redundant
subsetData<-subsetData[, names(subsetData)!="activityID" ]

Create tidy data set with the average of each variable for each activity and each subject
tidyData<-aggregate(.~ activity_name + subjectID, subsetData, mean)

Sort data by subject ID and activity name
tidyDataFinal<-tidyData[order(tidyData$activity_name, tidyData$subjectID),]

### 4. Output data to tidydata.txt
write.table(tidyDataFinal, file="tidydata.txt", row.name=FALSE,quote = FALSE)
