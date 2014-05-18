# Reproducible Research: Peer Assessment 1

## Initial configuration
Here we set a few parameters to help make the subsequent output look better.
Set options such that numbers with six or more digits before the decimal point
will be printed in scientific notation, and numbers will be rounded to two
decimal places.

We will also make use of the `lattice` plotting package, and load that here.

```r
options(scipen = 1, digits = 2)
library(lattice)
```


## Loading and preprocessing the data
The data used in this assignment are stored in the CSV file `activity.zip`,
compressed within the archive `activity.zip`. Here we extract the data and read
them into the R object `data`:

```r
data <- read.csv(unz("activity.zip", "activity.csv"))
data$hour <- floor(data$interval/100)
data$minute <- data$interval - 100 * data$hour
data$time <- sprintf("%02d:%02d", data$hour, data$minute)
data$interval <- data$interval
```



## What is mean total number of steps taken per day?
To look at the steps taken per day, instead of per five-minute interval, we
first aggregate the steps by date, summing all steps taken on a given date.
Once this has been done, the mean and median steps per day are straightforward
to calculate (with missing values ignored).

```r
stepsPerDay <- aggregate(steps ~ date, data, sum)
meanStepsPerDay <- mean(stepsPerDay$steps, na.rm = T)
medStepsPerDay <- median(stepsPerDay$steps, na.rm = T)
```


The distribution of number of steps per day over the two months can be easily
visualised with a histogram, on which the mean number of steps can also be
shown.

```r
hist(stepsPerDay$steps, main = "Frequency of steps taken per day", xlab = "Number of steps taken", 
    breaks = 10, col = "blue", sub = "(Mean shown by vertical red bar)")
abline(v = meanStepsPerDay, col = "red", lwd = 2)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

The mean number of steps per day is 10766.19, while the median is
almost the same, at 10765. The difference between these two cannot
be discerned on the graph, which is why only a single line has been added.


## What is the average daily activity pattern?

```r
meanStepsPerInterval <- aggregate(steps ~ interval, data, mean)
mostActiveInterval <- which.max(stepsPerInterval$steps)
```

```
## Error: object 'stepsPerInterval' not found
```


The most active interval, on average, is number 

```

Error in eval(expr, envir, enclos) : 
  object 'mostActiveInterval' not found

```

 in the
list. This is labelled as 

```

Error in eval(expr, envir, enclos) : 
  object 'mostActiveInterval' not found

```

, and
corresponds to the five-minute period beginning at


```

Error in eval(expr, envir, enclos) : 
  object 'mostActiveInterval' not found

```

.

The average daily activity is easily visualised with the following plot.


```r
plot(meanStepsPerInterval$interval, meanStepsPerInterval$steps, type = "l", 
    main = "Average distribution of steps throughout the day", xlab = "Five-minute interval", 
    ylab = "Average steps taken", col = "blue", sub = "(Most active interval in red)")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

```r
abline(v = meanStepsPerInterval$interval[mostActiveInterval], col = "red")
```

```
## Error: object 'mostActiveInterval' not found
```


The vertical red line indicates the location of the most active interval, on
average, which occurs at 

```

Error in eval(expr, envir, enclos) : 
  object 'mostActiveInterval' not found

```

.

## Inputing missing values

Several rows are missing the number of steps taken. We can count how many are
missing, and verify that no rows are missing the date or interval, by summing
the results of calling `is.na` on each variable:

```r
num.na.date <- sum(is.na(data$date))
num.na.interval <- sum(is.na(data$interval))
num.na.steps <- sum(is.na(data$steps))
```

The number of rows without step data is `num.na.steps=`2304, and
indeed there are no rows with missing intervals or dates:
`num.na.date=`0 and `num.na.interval=`0.

### Filling in missing values

We will replace all `NA` values with the mean number of steps in the same
interval, taken over the other non-`NA` intervals. These values are already
stored in the `meanStepsPerInterval` data frame. We begin by creating a new
dataset, `datafull`, that will have no missing values after we have processed
it.

```r
fulldata <- data
```


Now we can find which rows have missing values, look up the mean number of
steps for the interval in that row, and replace the `NA` with that mean.

```r
missing <- is.na(fulldata$steps)    # Isolate the NA values
missing.mean.idx <- match(          # Look up the mean for each interval
  fulldata$interval[missing],       # by matching the missing intervals
  meanStepsPerInterval$interval     # to the intervals in the list of means
)
fulldata$steps[missing] <- meanStepsPerInterval$steps[missing.mean.idx]
```


### Results of data filling
We can now replicate our earlier analysis, without having to remove any `NA`
values.

```r
fullStepsPerDay <- aggregate(steps ~ date, fulldata, sum)
fullMeanStepsPerDay <- mean(fullStepsPerDay$steps)
fullMedStepsPerDay <- median(fullStepsPerDay$steps)
```

The mean and median number of steps per day on the filled-in dataset are
10766.19 and 10766.19, respectively.
The mean number has not changed (`meanStepsPerDay=`10766.19),
however the median is now altered slightly. This is because non-integer numbers
of steps have been added to the dataset.


```r
hist(fullStepsPerDay$steps, main = "Frequency of steps taken per day", xlab = "Number of steps taken", 
    breaks = 10, col = "blue", sub = "(Mean shown by vertical red bar)")
abline(v = fullMeanStepsPerDay, col = "red", lwd = 2)
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 


## Are there differences in activity patterns between weekdays and weekends?
The `weekdays()` command returns the day of the week for a given date. To use 
it, the date strings in `data$date` must first be converted to `Date`-class
objects with the `as.Date()` function. The results `'Saturday'` and `'Sunday'`
can then be converted to `'weekend'`, and all other values to `'weekday'`.

```r
fulldata$day.type <- as.factor(ifelse(weekdays(as.Date(data$date)) %in% c("Saturday", 
    "Sunday"), "weekend", "weekday"))
```



```r
meanStepsPerIntervalDayTypes <- aggregate(steps ~ interval + day.type, fulldata, 
    FUN = mean)
```



```r
xyplot(
  steps ~ interval | day.type,
  data = meanStepsPerIntervalDayTypes,
  type = 'l', # Make a line plot
  layout = c(1, 2),
  ylab = 'Number of steps',
  xlab = 'Interval'
)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14.png) 
