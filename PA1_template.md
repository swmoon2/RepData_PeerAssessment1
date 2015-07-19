# Reproducible Research: Peer Assessment 1

```r
library(knitr)
library(dplyr)
library(lattice)
opts_chunk$set(echo=TRUE)
```

## Loading and preprocessing the data

```r
activity <- read.csv("./activity.csv")
activity$date <- as.Date(activity$date)
```

## What is mean total number of steps taken per day?

```r
steps_grouped <- group_by(activity, date)
steps_per_day <- summarize(steps_grouped, totalsteps=sum(steps, na.rm=TRUE))
hist(steps_per_day$totalsteps, 
     main="Total number of steps taken each day",
     xlab="total steps per day")
```

![](PA1_template_files/figure-html/calculate_total_steps-1.png) 

The mean of the total number of steps taken per day

```r
mean(steps_per_day$totalsteps)
```

```
## [1] 9354.23
```

The median of the total number of steps taken per day

```r
median(steps_per_day$totalsteps)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

```r
steps_grouped_by_interval <- group_by(activity, interval)
average_steps <- summarize(steps_grouped_by_interval, avgsteps=mean(steps, na.rm=TRUE))
with(average_steps, plot(interval, avgsteps, type="l",
                         main="Average number of steps by interval", 
                         ylab="number of steps"))
```

![](PA1_template_files/figure-html/analyse_pattern-1.png) 

5-minute interval which contains the maximum number of steps

```r
maxavgsteps = max(average_steps$avgsteps)
filter(average_steps, avgsteps == maxavgsteps)$interval
```

```
## [1] 835
```

## Imputing missing values
The total number of missing values in the dataset

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```
The strategy for filling NAs --> use the mean for that 5-minute interval

```r
act2 <- activity
for(i in 1:nrow(steps_per_day)) {
    for(j in 1:nrow(average_steps)) {
        if(is.na(act2[nrow(average_steps)*(i-1)+j,"steps"])) {
            act2[nrow(average_steps)*(i-1)+j,"steps"] <- average_steps[j, "avgsteps"]
        }
    }
}
```

Make a histogam again using imputed dataset

```r
steps_grouped2 <- group_by(act2, date)
steps_per_day2 <- summarize(steps_grouped2, totalsteps=sum(steps))
hist(steps_per_day2$totalsteps, 
     main="Total number of steps taken each day",
     xlab="total steps per day")
```

![](PA1_template_files/figure-html/calculate_total_steps2-1.png) 

The mean of the total number of steps taken per day using imputed dataset

```r
mean(steps_per_day2$totalsteps)
```

```
## [1] 10766.19
```

The median of the total number of steps taken per day using imputed dataset

```r
median(steps_per_day2$totalsteps)
```

```
## [1] 10766.19
```

These results are different from them which are calculated from the original dataset which contains missing data.

By imputing data, both mean and median of the total number of steps taken per interval increase.


## Are there differences in activity patterns between weekdays and weekends?

```r
act2 <- mutate(act2, wd = factor(weekdays(act2$date) %in% c("토요일", "일요일"), labels=c("weekday", "weekend")))

steps_grouped_by_interval_wd <- group_by(act2, wd, interval)
average_steps_wd <- summarize(steps_grouped_by_interval_wd, avgsteps=mean(steps, na.rm=TRUE))
xyplot(avgsteps ~ interval | wd, 
       data=average_steps_wd, type="l", layout=c(1,2),
       main="Average number of steps by interval and weekdays",
       ylab="number of steps")
```

![](PA1_template_files/figure-html/actpatterns-1.png) 
