---
output: html_document
---
# Reproducible Research: Peer Assessment 1
**Author:** Florian Knaus
**Last changed:** July 20, 2014
## Introduction
This document is for the Peer Assessment 1 of the Coursera Reproducible Research course by Roger Peng.


## Loading and preprocessing the data
First we set the working directory:

```r
setwd("C:/Users/Florian/Documents/Development/git/RepData_PeerAssessment1");
```
Then we read the file "activity.csv" and format the date column

```r
activity <- read.table(file="activity.csv", 
                       header=TRUE,
                       sep=",",
                       colClasses=c("numeric","character","numeric"));
activity$date <- as.Date(activity$date, "%Y-%m-%d");
clean_activity <- activity[which(activity$steps != "NA"),];
```


## What is mean total number of steps taken per day?
We create a histogram and use functions mean and median on column 'steps' of data-frame 'activity' and remove any possible 'NA' values.

```r
library(plyr)
daily_activity <- ddply(clean_activity, 
                        .(date), 
                        summarise, 
                        steps=sum(steps));
hist(daily_activity$steps,
     main="Histogram of Steps", 
     xlab="Steps");
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

```r
mean(daily_activity$steps);
```

```
## [1] 10766
```

```r
median(daily_activity$steps);
```

```
## [1] 10765
```


## What is the average daily activity pattern?

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
# sum up the mean of intervals
average_by_interval <- ddply(clean_activity, 
                             .(interval), 
                             summarise, 
                             steps=mean(steps));

plot(average_by_interval$interval, 
     average_by_interval$steps, 
     type="l",
     xlab="Interval",
     ylab="Steps",
     main="Average Daily Activity Pattern");
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
average_by_interval[average_by_interval$steps==max(average_by_interval$steps),]
```

```
##     interval steps
## 104      835 206.2
```


## Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(activity))
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

My strategy is to take each activity with steps = NA and fill it's steps with mean of interval from clean activity set.


```r
na_activity <- activity[is.na(activity),];
#mactivity_na_rm <- ddply(activity, .(interval), function(x) apply(x[1], 2, mean))

for (i in 1:nrow(na_activity)) {
  #print(i)
  na_activity[i,]$steps <- mean(clean_activity[which(clean_activity$interval == na_activity[i,]$interval),]$steps);
}
```

Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activity_na_rm <- rbind(clean_activity, na_activity);
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
daily_activity_na_rm <- ddply(activity_na_rm, 
                        .(date), 
                        summarise, 
                        steps=sum(steps));
hist(daily_activity_na_rm$steps,
     main="Histogram of Steps", 
     xlab="Steps");
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

```r
mean(daily_activity_na_rm$steps);
```

```
## [1] 10766
```

```r
median(daily_activity_na_rm$steps);
```

```
## [1] 10766
```

Due to the strategy taken the mean is the same but the median is different as decimal places have been introduced into column 'steps'.

## Are there differences in activity patterns between weekdays and weekends?

```r
# copy dataset
activity_weekdays <- activity_na_rm
# add column that identifies day of week
activity_weekdays$dayOfWeek <- weekdays(activity_weekdays$date)
# add column that defines if weekend or not
activity_weekdays$weekend <- activity_weekdays$dayOfWeek %in% c("Saturday", "Sunday")

data_weekend <- ddply(activity_weekdays[activity_weekdays$weekend,], 
                             .(interval), 
                             summarise, 
                             steps=mean(steps));

data_weekdays <- ddply(activity_weekdays[!activity_weekdays$weekend,], 
                             .(interval), 
                             summarise, 
                             steps=mean(steps));

plot(data_weekdays$interval, 
     data_weekdays$steps, 
     type="l",
     xlab="Interval",
     ylab="Steps",
     main="Average Daily Activity Pattern (weekdays)");
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-101.png) 

```r
plot(data_weekend$interval, 
     data_weekend$steps, 
     type="l",
     xlab="Interval",
     ylab="Steps",
     main="Average Daily Activity Pattern (weekend)");
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-102.png) 

```r
# comparison in one chart
# summarize mean by interval and weekend-identifier
activity_all <- ddply(activity_weekdays, .(interval,weekend), summarise, steps=mean(steps))
# plot with ggplot2
require(ggplot2)
ggplot(data=activity_all, aes(x=interval, y=steps, colour=weekend)) + geom_line()
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-103.png) 

There plots above show the difference between weekend and weekday activity.
