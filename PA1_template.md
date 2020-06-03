---
title: "Reproducible Research: Peer Assessment 1"
author: "Bart Slagt"
date: "6/3/2020"
output: 
  html_document:
    keep_md: true
---
## Introduction

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a
[Fitbit](http://www.fitbit.com), [Nike
Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or
[Jawbone Up](https://jawbone.com/up). These type of devices are part of
the "quantified self" movement -- a group of enthusiasts who take
measurements about themselves regularly to improve their health, to
find patterns in their behavior, or because they are tech geeks. But
these data remain under-utilized both because the raw data are hard to
obtain and there is a lack of statistical methods and software for
processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012
and include the number of steps taken in 5 minute intervals each day.

## Data

The data for this assignment can be downloaded from the course web
site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken




The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this
dataset.


## Assignment

This assignment will be described in multiple parts. You will need to
write a report that answers the questions detailed below. Ultimately,
you will need to complete the entire assignment in a **single R
markdown** document that can be processed by **knitr** and be
transformed into an HTML file.

Throughout your report make sure you always include the code that you
used to generate the output you present. When writing code chunks in
the R markdown document, always use `echo = TRUE` so that someone else
will be able to read the code. **This assignment will be evaluated via
peer assessment so it is essential that your peer evaluators be able
to review the code for your analysis**.

For the plotting aspects of this assignment, feel free to use any
plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the [GitHub repository created for this
assignment](http://github.com/rdpeng/RepData_PeerAssessment1). You
will submit this assignment by pushing your completed files into your
forked repository on GitHub. The assignment submission will consist of
the URL to your GitHub repository and the SHA-1 commit ID for your
repository state.

NOTE: The GitHub repository also contains the dataset for the
assignment so you do not have to download the data separately.



### Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. `read.csv()`)

2. Process/transform the data (if necessary) into a format suitable for your analysis
   Date to a date class



```r
library(ggplot2)
unzip("activity.zip")
activity <- activity<-read.csv("activity.csv")
activity$date<-as.Date(activity$date,"%Y-%m-%d")
```


### What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in
the dataset.

1. Make a histogram of the total number of steps taken each day


```r
activity_only_na <- activity[is.na(activity$steps),]
activity_no_na <- activity[!is.na(activity$steps),]
total_steps_day <- with(activity_no_na, tapply(steps, as.factor(activity_no_na$date), sum, na.rm = TRUE))
hist(total_steps_day, main = "Histogram of steps taken per day", xlab = "Total number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

2. Calculate and report the **mean** and **median** total number of steps taken per day

```r
summary(total_steps_day)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10765   10766   13294   21194
```

### What is the average daily activity pattern?

1. Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
mean_steps <- with(activity_no_na, tapply(steps, activity_no_na$interval, mean))
interval <- levels(as.factor(activity_no_na$interval))
plot(interval, mean_steps, type = "l", main = "Time series plot of the average number of steps taken", xlab = "interval", ylab = "Mean steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
interval_table <- data.frame(mean_steps, interval)
interval_table[interval_table$mean_steps==max(interval_table$mean_steps),][2]
```

```
##     interval
## 835      835
```

### Imputing missing values

Note that there are a number of days/intervals where there are missing
values (coded as `NA`). The presence of missing days may introduce
bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s)

```r
nrow(activity)-nrow(activity_no_na)
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
#as some of the dates have only NA's I chose the mean over the entire dataset
total_mean<-floor(mean(activity_no_na$steps))
activity_only_na$steps<-total_mean
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
#combine and sort the two tables and write it as an tidy CSV file
tidy_activity <- rbind(activity_no_na, activity_only_na)
tidy_activity <- tidy_activity[order(tidy_activity$date), ]
write.csv(tidy_activity,"tidy_activity.csv",row.names=FALSE)
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the **mean** and **median** total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
tidy_total_steps_day <- with(tidy_activity, tapply(steps, as.factor(tidy_activity$date), sum, na.rm = TRUE))
hist(tidy_total_steps_day, main = "Histogram of steps taken per day (Tidy CSV)", xlab = "Total number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
#new summary
summary(tidy_total_steps_day)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9819   10656   10752   12811   21194
```

```r
#old summary
summary(total_steps_day)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10765   10766   13294   21194
```

### Are there differences in activity patterns between weekdays and weekends?

For this part the `weekdays()` function may be of some help here. Use
the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
tidy_activity$weekday<-weekdays(tidy_activity$date)
index_of_weekends<- grep("Saturday|Sunday", tidy_activity$weekday, ignore.case = TRUE)
index_of_workdays<- grep("Monday|Tuesday|Wednesday|Thursday|Friday", tidy_activity$weekday, ignore.case = TRUE)
weekend_activity<-  tidy_activity[index_of_weekends, ]
weekend_activity$sortofday <- "weekend"
workday_activity<-  tidy_activity[index_of_workdays, ]
workday_activity$sortofday <- "weekday"
tidy_activity <- rbind(weekend_activity, workday_activity)
tidy_activity <- tidy_activity[order(tidy_activity$date), ]
```

1. Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was created using **simulated data**:

![Sample panel plot](instructions_fig/sample_panelplot.png) 


**Your plot will look different from the one above** because you will
be using the activity monitor data. Note that the above plot was made
using the lattice system but you can make the same version of the plot
using any plotting system you choose.


```r
mean_number_steps <- aggregate(steps~ interval+sortofday, tidy_activity, mean)
g <- qplot(interval, steps, data = mean_number_steps, facets = sortofday~.)
g + geom_line(size = 1) + ylab("Mean steps") + ggtitle("Average number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->
