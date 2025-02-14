---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

Author: Joshua Parsell

Date: 9/8/2021

## Loading and preprocessing the data


```r
library("dplyr")
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
unzip("activity.zip")
activity <- read.csv("activity.csv")

#Pad leading zeroes on the interval
activity <- mutate(activity, interval = formatC(interval, width=4, flag="0"))

#Make an alternate interval label column with a colon
activity <- mutate(activity, int_lbl = paste0(substr(interval,1,2),":",substr(interval,3,4)))
```


## What is mean total number of steps taken per day?


```r
by_day <- group_by(activity, date)
steps_per_day <- summarise(by_day, steps = sum (steps))
hist(steps_per_day$steps, main = "Histogram of Daily Step Counts, Oct-Nov 2012", labels = TRUE)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
mean_steps <- mean (steps_per_day$steps, na.rm = TRUE)
median_steps <- median (steps_per_day$steps, na.rm = TRUE)
cat ("Mean Steps Per Day = ", mean_steps, "\n")
```

```
## Mean Steps Per Day =  10766.19
```

```r
cat ("Median Steps Per Day = ", median_steps, "\n")
```

```
## Median Steps Per Day =  10765
```

## What is the average daily activity pattern?


```r
by_interval <- group_by(activity, interval, int_lbl)
steps_per_interval <- summarise(by_interval, avg_steps_per_int = mean (steps, na.rm = TRUE))

plot(steps_per_interval$interval, steps_per_interval$avg_steps_per_int, 
     type = "l", 
     main = "Average Daily Steps Per Five Minute Interval",
     sub = "(Data collected Oct - Nov 2012)",
     ylab = "Average Steps",
     xlab = "Five Minute Interval throughout the day (0000 - 2355)")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
cat("Five Minute Interval with Maximum Average Steps = ", steps_per_interval$int_lbl[which.max(steps_per_interval$avg_steps_per_int)])
```

```
## Five Minute Interval with Maximum Average Steps =  08:35
```

## Imputing missing values


```r
#Find number of observations with missing values
missing_steps <- nrow( filter(activity, is.na(steps)) )

cat( missing_steps , "rows in 'activity.csv' are missing values for steps.\n")
```

```
## 2304 rows in 'activity.csv' are missing values for steps.
```

```r
#Impute the average number of steps for that interval into each row where the value is missing

activity2 <- left_join(activity, steps_per_interval, by=c("interval", "int_lbl"))

activity2 <- mutate(activity2, steps = coalesce(as.double(steps),avg_steps_per_int))

activity2 <- select(activity2, -avg_steps_per_int)

cat("activity2 replaces all missing values with the averages for the same interval.\n\n")
```

```
## activity2 replaces all missing values with the averages for the same interval.
```

```r
by_day2 <- group_by(activity2, date)
steps_per_day2 <- summarise(by_day2, steps = sum (steps))
hist(steps_per_day2$steps, main = "Histogram of Daily Step Counts, Oct-Nov 2012", labels = TRUE)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
mean_steps2 <- mean (steps_per_day2$steps, na.rm = TRUE)
median_steps2 <- median (steps_per_day2$steps, na.rm = TRUE)
cat ("Mean Steps Per Day = ", mean_steps2, "\n")
```

```
## Mean Steps Per Day =  10766.19
```

```r
cat ("Median Steps Per Day = ", median_steps2, "\n")
```

```
## Median Steps Per Day =  10766.19
```

## Are there differences in activity patterns between weekdays and weekends?


```r
#Create a factor variable with weekday/weekend levels
activity2 <- mutate(activity2, daytype = as.factor(case_when( weekdays(as.Date(date)) == "Saturday" ~ "weekend", 
                                                    weekdays(as.Date(date)) == "Sunday" ~ "weekend", 
                                                    TRUE ~ "weekday" ) ) )

by_interval2 <- group_by(activity2, interval, int_lbl, daytype)
steps_per_interval2 <- summarise(by_interval2, avg_steps_per_int = mean (steps, na.rm = TRUE))


par(mfrow=c(2,1))
plot(filter(steps_per_interval2,daytype=="weekday")$interval, filter(steps_per_interval2,daytype=="weekday")$avg_steps_per_int, 
     type = "l", 
     main = "Average Daily Steps Per Five Minute Interval - Weekdays",
     ylab = "Average Steps",
     xlab = "")
plot(filter(steps_per_interval2,daytype=="weekend")$interval, filter(steps_per_interval2,daytype=="weekend")$avg_steps_per_int, 
     type = "l", 
     main = "Average Daily Steps Per Five Minute Interval - Weekends",
     col = "blue",
     sub = "(Data collected Oct - Nov 2012)",
     ylab = "Average Steps",
     xlab = "Five Minute Interval throughout the day (0000 - 2355)")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->
