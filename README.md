# Spec_team_predict
Team of Junior Data Scientist at Kaggle
GA-customer-revenue-prediction Rcode

#This script just converts the JSON columns in the training and test sets to data frames,
#...then combines everything to make training and test sets that are much easier to work with.
#Every feature inside each JSON column will appear,
#...and the twice-nested adwordsClickInfo in trafficSource is also dealt with! by Erci Bruin https://www.kaggle.com/erikbruin

n <- Sys.time() #See how long this takes to run

library(tidyverse)
library(jsonlite)
library(caret)

train <- read_csv("../input/train.csv")
test <- read_csv("../input/test.csv")

#JSON columns are "device", "geoNetwork", "totals", "trafficSource"
tr_device <- paste("[", paste(train$device, collapse = ","), "]") %>% fromJSON(flatten = T)
tr_geoNetwork <- paste("[", paste(train$geoNetwork, collapse = ","), "]") %>% fromJSON(flatten = T)
tr_totals <- paste("[", paste(train$totals, collapse = ","), "]") %>% fromJSON(flatten = T)
tr_trafficSource <- paste("[", paste(train$trafficSource, collapse = ","), "]") %>% fromJSON(flatten = T)

te_device <- paste("[", paste(test$device, collapse = ","), "]") %>% fromJSON(flatten = T)
te_geoNetwork <- paste("[", paste(test$geoNetwork, collapse = ","), "]") %>% fromJSON(flatten = T)
te_totals <- paste("[", paste(test$totals, collapse = ","), "]") %>% fromJSON(flatten = T)
te_trafficSource <- paste("[", paste(test$trafficSource, collapse = ","), "]") %>% fromJSON(flatten = T)


#Check to see if the training and test sets have the same column names
setequal(names(tr_device), names(te_device))
setequal(names(tr_geoNetwork), names(te_geoNetwork))
setequal(names(tr_totals), names(te_totals))
setequal(names(tr_trafficSource), names(te_trafficSource))

#As expected, tr_totals and te_totals are different as the train set includes the target, transactionRevenue
names(tr_totals)
names(te_totals)
#Apparently tr_trafficSource contains an extra column as well - campaignCode
#It actually has only one non-NA value, so this column can safely be dropped later
table(tr_trafficSource$campaignCode, exclude = NULL)
names(tr_trafficSource)
names(te_trafficSource)


#Combine to make the full training and test sets
train <- train %>%
  cbind(tr_device, tr_geoNetwork, tr_totals, tr_trafficSource) %>%
  select(-device, -geoNetwork, -totals, -trafficSource)

test <- test %>%
  cbind(te_device, te_geoNetwork, te_totals, te_trafficSource) %>%
  select(-device, -geoNetwork, -totals, -trafficSource)

#Number of columns in the new training and test sets. 
ncol(train)
ncol(test)

#Remove temporary tr_ and te_ sets
rm(tr_device); rm(tr_geoNetwork); rm(tr_totals); rm(tr_trafficSource)
rm(te_device); rm(te_geoNetwork); rm(te_totals); rm(te_trafficSource)

#How long did this script take?
Sys.time() - n

write.csv(train, "train_flat.csv", row.names = F)
write.csv(test, "test_flat.csv", row.names = F)

#Removing columns
cols.dont.want <- c("socialEngagementType","browserVersion","browserSize","operatingSystemVersion",
                    "mobileDeviceBranding","mobileInputSelector","mobileDeviceModel", "mobileDeviceInfo", 
                    "mobileDeviceMarketingName","flashVersion","language","screenColors", "screenResolution","metro",
                    "region", "cityId", "latitude","longitude","networkLocation","campaignCode",
                    "adwordsClickInfo.criteriaParameters", "fullvisitorId" )
#some more columns to delete: socialEngagementType, city
data <- train[, ! names(train) %in% cols.dont.want, drop = F]

#Turn factor into matrix by dummy vars from caret (Onehotencoder bkb Labelencoder) https://amunategui.github.io/dummyVar-Walkthrough/

dmy <- dummyVars(" ~ browser", data = data)
trsf <- data.frame(predict(dmy, newdata = train))
print(trsf)
