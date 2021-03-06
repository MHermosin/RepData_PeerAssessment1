# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
data <- read.csv(unz("activity.zip", "activity.csv"), header = TRUE, na.strings = "NA", 
    colClasses = c("numeric", "character", "numeric"))
data$DateTime <- strptime(paste(data$date, sprintf("%04d", data$interval), sep = " "), 
    format = "%Y-%m-%d %H%M")
```



## What is mean total number of steps taken per day?

```r
stepsPerDay <- aggregate(steps ~ date, data, FUN = sum)
hist(stepsPerDay$steps, xlab = "Steps Per Day", breaks = 15, main = "Histogram of Steps Per Day")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

```r
mean(stepsPerDay$steps)
```

```
## [1] 10766
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10765
```



## What is the average daily activity pattern?

```r
stepsPerFive <- aggregate(steps ~ interval, data, FUN = mean)
plot(stepsPerFive$interval, stepsPerFive$steps, type = "l", xlab = "Interval", 
    ylab = "Mean number of steps")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

Find the interval with the maximum number of average steps

```r
stepsPerFive[stepsPerFive$steps == max(stepsPerFive$steps), ]$interval
```

```
## [1] 835
```



## Imputing missing values
Calculate total number of missing values

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

We want to replace the NA values with the mean for that interval, which we calculated above

```r
noData <- data[is.na(data$steps), ]
replacementData <- stepsPerFive[stepsPerFive$interval == noData$interval, ]
data$steps <- replace(data$steps, is.na(data$steps), replacementData$steps)
```

Make a new histogram with the imputed values as well as calculating the new mean and median

```r
newStepsPerDay <- aggregate(steps ~ date, data, FUN = sum)
hist(newStepsPerDay$steps, xlab = "Steps Per Day", breaks = 15, main = "Histogram of Steps Per Day with Imputed Values")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 

```r
mean(newStepsPerDay$steps)
```

```
## [1] 10766
```

```r
median(newStepsPerDay$steps)
```

```
## [1] 10766
```

Clearly, there is very little difference whether we impute the missing values or simply ignore them.
## Are there differences in activity patterns between weekdays and weekends?

```r
library(lattice)
data$DayOfWeek = factor(ifelse(weekdays(data$DateTime) == "Saturday" | weekdays(data$DateTime) == 
    "Sunday", "weekend", "weekday"))
weekendData <- data[data$DayOfWeek == "weekend", ]
weekdayData <- data[data$DayOfWeek == "weekday", ]
stepsPerFiveWeekend <- aggregate(steps ~ interval, weekendData, FUN = mean)
stepsPerFiveWeekend$DayOfWeek <- "weekend"
stepsPerFiveWeekday <- aggregate(steps ~ interval, weekdayData, FUN = mean)
stepsPerFiveWeekday$DayOfWeek <- "weekday"
stepsPerFiveTotal <- rbind(stepsPerFiveWeekend, stepsPerFiveWeekday)
xyplot(steps ~ interval | DayOfWeek, data = stepsPerFiveTotal, ylab = "Number of steps", 
    xlab = "Interval", layout = c(1, 2), type = "l")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

