# Reproducible Research: Peer Assessment 1
**Author:** Florian Knaus
**Last changed:** May 18, 2014
## Introduction
This document is for the Peer Assessment 1 of the Coursera Reproducible Research course by Roger Peng.


## Loading and preprocessing the data
First we set the working directory:

```r
setwd("C:/Users/Florian/Documents/Development/git/RepData_PeerAssessment1")
```

Then we read the file "activity.csv" and format the date column

```r
activity <- read.table(file = "activity.csv", header = TRUE, sep = ",", colClasses = c("numeric", 
    "character", "numeric"))
activity$date <- as.Date(activity$date, "%Y-%m-%d")
clean_activity <- activity[which(activity$steps != "NA"), ]
```



## What is mean total number of steps taken per day?
We create a histogram and use functions mean and median on column 'steps' of data-frame 'activity' and remove any possible 'NA' values.

```r
library(plyr)
daily_activity <- ddply(clean_activity, .(date), summarise, steps = sum(steps))
hist(daily_activity$steps, main = "Histogram of Steps", xlab = "Steps")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

```r
mean(daily_activity$steps)
```

```
## [1] 10766
```

```r
median(daily_activity$steps)
```

```
## [1] 10765
```



## What is the average daily activity pattern?


```r
average_by_interval <- ddply(clean_activity, .(interval), summarise, steps = mean(steps))

plot(average_by_interval$interval, average_by_interval$steps, type = "l", xlab = "Interval", 
    ylab = "Steps", main = "Average Daily Activity Pattern")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 




## Imputing missing values





## Are there differences in activity patterns between weekdays and weekends?



