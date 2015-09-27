first we install the packages dplyr using 
install.packages("dplyr")
and loading it using
library("dplyr")
Then we download the file in R and unzip it by unzip function using
filename <- "getdata-projectfiles-UCI-HAR Dataset.zip"
if (!file.exists(filename)){
  fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(fileURL, filename, method="curl")
}
if (!file.exists("getdata-projectfiles-UCI-HAR Dataset")) { 
  unzip(filename) }
  
 then we  Load activity labels + features
  activityLabels <- read.table("getdata-projectfiles-UCI-HAR Dataset/activity_labels.txt")
activityLabels[,2] <- as.character(activityLabels[,2])
features <- read.table("getdata-projectfiles-UCI-HAR Dataset/features.txt")
features[,2] <- as.character(features[,2])

then  Extract only the data on mean and standard deviation

featuresWanted <- grep(".*mean.*|.*std.*", features[,2])
featuresWanted.names <- features[featuresWanted,2]
featuresWanted.names = gsub('-mean', 'Mean', featuresWanted.names)
featuresWanted.names = gsub('-std', 'Std', featuresWanted.names)
featuresWanted.names <- gsub('[-()]', '', featuresWanted.names)


# Load the datasets
train <- read.table("getdata-projectfiles-UCI-HAR Dataset/train/X_train.txt")[featuresWanted]
trainActivities <- read.table("getdata-projectfiles-UCI-HAR Dataset/train/Y_train.txt")
trainSubjects <- read.table("getdata-projectfiles-UCI-HAR Dataset/train/subject_train.txt")
train <- cbind(trainSubjects, trainActivities, train)

test <- read.table("getdata-projectfiles-UCI-HAR Dataset/test/X_test.txt")[featuresWanted]
testActivities <- read.table("getdata-projectfiles-UCI-HAR Dataset/test/Y_test.txt")
testSubjects <- read.table("getdata-projectfiles-UCI-HAR Dataset/test/subject_test.txt")
test <- cbind(testSubjects, testActivities, test)

# merge datasets and add labels
allData <- rbind(train, test)
colnames(allData) <- c("subject", "activity", featuresWanted.names)

# turn activities & subjects into factors
allData$activity <- factor(allData$activity, levels = activityLabels[,1], labels = activityLabels[,2])
allData$subject <- as.factor(allData$subject)

allData.melted <- melt(allData, id = c("subject", "activity"))
allData.mean <- dcast(allData.melted, subject + activity ~ variable, mean)

write.table(allData.mean, "tidy.txt", row.names = FALSE, quote = FALSE)
