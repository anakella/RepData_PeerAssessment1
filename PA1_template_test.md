
Course Project :Reproducible Research: Peer Assessment 1
title: “PA1_template.rmd”
author: “Anand Akella”
date: “July 19, 2015”

Reproducible Research: Peer Assessment 1

Loading and Processing Activity Data

library(ggplot2)

## Warning: package 'ggplot2' was built under R version 3.2.1

library(Hmisc)

## Warning: package 'Hmisc' was built under R version 3.2.1

## Loading required package: grid
## Loading required package: lattice
## Loading required package: survival
## Loading required package: Formula

## Warning: package 'Formula' was built under R version 3.2.1

## 
## Attaching package: 'Hmisc'
## 
## The following objects are masked from 'package:base':
## 
##     format.pval, round.POSIXt, trunc.POSIXt, units

filePath <- "./Data/activity.csv"
activityData <- read.csv(filePath)

Install ggplot package in R before running all the chunks of code that follow

What is the mean total number of steps taken per day?

totalStepsPerDay <- tapply(activityData$steps, activityData$date, sum, na.rm=TRUE)
MeanStepsPerDay <- mean(totalStepsPerDay)
MeanStepsPerDay

## [1] 9354.23

MedianStepsPerDay <- median(totalStepsPerDay)
MedianStepsPerDay

## [1] 10395

qplot(totalStepsPerDay, binwidth=1000, xlab="total number of steps taken each day")

What is the average daily activity Pattern?

averageStepsPerTimeBlock <- aggregate(x=list(meanSteps=activityData$steps), by=list(interval=activityData$interval), FUN=mean, na.rm=TRUE)

    Make a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

ggplot(data=averageStepsPerTimeBlock, aes(x=interval, y=meanSteps)) +
    geom_line() +
    xlab("5-minute interval") +
    ylab("average number of steps taken") 

2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

mostSteps <- which.max(averageStepsPerTimeBlock$meanSteps)
timeMostSteps <-  gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", averageStepsPerTimeBlock[mostSteps,'interval'])

Imputing missing values

numMissingValues <- length(which(is.na(activityData$steps)))
numMissingValues

## [1] 2304

Replace Missing Values with mean values of 5 minute interval

fillData <- activityData
fillData$steps <- impute(activityData$steps, fun=mean)

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

stepsByDayImputed <- tapply(fillData$steps, fillData$date, sum)
qplot(stepsByDayImputed, xlab='Total steps per day (Imputed)', ylab='Frequency using binwidth 500', binwidth=500)

Mean and Total Steps per day with imputed Data

mean(stepsByDayImputed)

## [1] 10766.19

median(stepsByDayImputed)

## [1] 10766.19

Yes, there are difference between mean and medians of non imputed data vs imputed data since the non imputed data set we removed all those time steps which had NA values and calculated mean and median and for the imputed data we filled those NA values with the mean of the 5 minute interval

Are there differences in activity patterns between weekdays and weekends?

fillData$dateType <-  ifelse(as.POSIXlt(fillData$date)$wday %in% c(0,6), 'weekend', 'weekday')

Make a panel plot containing a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

avgFillData <- aggregate(steps ~ interval + dateType, data=fillData, mean)
ggplot(avgFillData, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dateType ~ .) +
    xlab("5-minute interval") + 
    ylab("avarage number of steps")

