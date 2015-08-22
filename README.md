###########################################################################################################################

# Getting and Cleaning Data Course Project

# Description
## This project will download and clean the UCI Human Activity Recognition Using Smartphones Data Set. Major steps include:
* Step 1: Merges the training and the test sets to create one data set.
* Step 2: Extracts only the measurements on the mean and standard deviation for each measurement. 
* Step 3: Uses descriptive activity names to name the activities in the data set
* Step 4: Appropriately labels the data set with descriptive variable names. 
* Step 5: Creates a second, independent tidy data set with the average of each variable for each activity and each subject.

###########################################################################################################################

# Set working directory
setwd("./course_project")

# Download the UCI Human Activity Recognition Using Smartphones Data Set and put it to the working directory
filename<-"UCI_HAR_Dataset.zip"
if (!file.exists(filename)) {
  fileUrl<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(fileUrl, filename)
}

# Unzip file
folder_name<-"UCI HAR Dataset"
if(!file.exists(folder_name)) {
  unzip(filename)
}
  
Step 1: Merges the training and the test sets to create one data set.

# Loading the data
# Load feature.txt file
features<-read.table("./UCI HAR Dataset/features.txt",header=FALSE)
colnames(features)<-c("index", "feature_name")

# Load activity_labels.txt
activity_labels<-read.table("./UCI HAR Dataset/activity_labels.txt",header=FALSE)
colnames(activity_labels)<-c("activityID", "activity_name")

# Load training data
train<-read.table("./UCI HAR Dataset/train/X_train.txt",header=FALSE)
trainActivities<-read.table("./UCI HAR Dataset/train/Y_train.txt",header=FALSE)  
trainSubjects<-read.table("./UCI HAR Dataset/train/subject_train.txt",header=FALSE) 

# Assign column names
colnames(train)<-features[,2]
colnames(trainActivities)<-"activityID"
colnames(trainSubjects)<-"subjectID"

# Combine training data
trainData<-cbind(trainSubjects, trainActivities, train)

# Load test data
test<-read.table("./UCI HAR Dataset/test/X_test.txt",header=FALSE)
testActivities<-read.table("./UCI HAR Dataset/test/Y_test.txt",header=FALSE)  
testSubjects<-read.table("./UCI HAR Dataset/test/subject_test.txt",header=FALSE) 

# Assign column names
colnames(test)<-features[,2]
colnames(testActivities)<-"activityID"
colnames(testSubjects)<-"subjectID"

# Combine training data
testData<-cbind(testSubjects, testActivities, test)

# Combine training and test datasets
allData<-rbind(trainData, testData)


# Step 2: Extracts only the measurements on the mean and standard deviation for each measurement.
# View the structure of data
str(allData)

# Select the mean and standard deviation
colNames  <- colnames(allData) 
featureSelected<- (grepl("subjectID",colNames) | grepl("activityID",colNames) | (!grepl("-meanFreq",colNames) & (grepl("-mean()..-",colNames) | grepl("-std()..-",colNames) | grepl("-mean()",colNames) | grepl("-std()",colNames)))) 
subsetData<-allData[,featureSelected==TRUE]


# Step 3: Uses descriptive activity names to name the activities in the data set
subsetData=merge(subsetData, activity_labels, by.x="activityID", by.y="activityID", all.x =TRUE) 
# Check activities names
head(subsetData$activity_name)


# Step 4: Appropriately labels the data set with descriptive variable names. 
# Change varaible names
colNames<-colnames(subsetData)
colNames<-gsub("mean\\()", "mean", colNames)
colNames<-gsub("std\\()", "stdev", colNames)
colNames<-gsub("^t", "time", colNames)
colNames<-gsub("^f", "frequency", colNames)
colNames<-gsub("Acc", "Acceleration", colNames)
colNames<-gsub("Gyro", "Gyroscope", colNames)
colNames<-gsub("Mag", "Magnitude", colNames)
colNames<-gsub("BodyBody", "Body", colNames)

# Check colNames
colNames

# Reassign the modified variable names to the dataset
colnames(subsetData) <- colNames

# Step 5: Creates a second, independent tidy data set with the average of each variable for each activity and each subject and output tidy data.
# Remove the activity type number
subsetData<-subsetData[, names(subsetData)!="activityID" ]

# Create tidy data set with the average of each variable for each activity and each subject
tidyData<-aggregate(.~ activity_name + subjectID, subsetData, mean)

# Sort data by subject and activity
tidyDataFinal<-tidyData[order(tidyData$activity_name, tidyData$subjectID),]

# Output data
write.table(tidyDataFinal, file="tidydata.txt", row.name=FALSE,quote = FALSE)
