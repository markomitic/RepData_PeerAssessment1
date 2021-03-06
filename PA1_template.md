# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Data file -  "activity.csv" is in working directory. We can get the csv by unzipping the "activity.zip". 
We can load the data from that CSV file.


```r
activityData = read.csv("activity.csv", header=TRUE)
```

There's no need for additional processing the data.

## What is mean total number of steps taken per day?

We need to aggregate the data by making a sum of number of steps per each day. We are dropping missing values from the dataset.


```r
stepsPerDay <- aggregate(activityData[, 'steps'], by = list(activityData$date), FUN = sum, na.rm = TRUE)
```

And we can give more descriptive names to our columns.


```r
names(stepsPerDay) <- c("date", "steps")
```

So now we can use stepsPerDay variable as our dataset to make histogram of the total number of steps taken each day. We are using ggplot2 as our plotting system.


```r
library(ggplot2)
qplot(x = stepsPerDay$steps, 
      data = stepsPerDay, 
      geom = "histogram", 
      binwidth=1000,
      main =  "Histogram of the total number of steps taken each day",
      xlab = "Number of steps per day",
      ylab = "Days with same steps count")
```

![](figures/histogram-1.png) 

These are the mean and median total number of steps taken per day.


```r
mean(stepsPerDay$steps)
```

```
## [1] 9354.23
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10395
```

## What is the average daily activity pattern?
Now we're aggreagating the data by 5-minute intervals, and we're also dropping NA values.


```r
stepsPerInterval = aggregate(activityData[, 'steps'], by = list(activityData$interval), FUN = mean, na.rm = TRUE)
```

Again, giving columns proper names.


```r
names(stepsPerInterval) <- c("interval", "steps")
```

And making a time series plot with ggplot2 plotting system.


```r
ggplot(stepsPerInterval) + 
    aes(x = interval, y = steps) + 
    geom_line() + 
    labs(title = "Time series plot of average number of steps per 5-minute interval")
```

![](figures/stepsPerInterval-1.png) 

To answer the question which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps we get that this is the interval:


```r
maxStepsInterval <- stepsPerInterval[which.max(stepsPerInterval$steps), ]$interval
```

The interval with the maximum number of steps is ``835``.

## Imputing missing values
Total number of missing values in the initial dataset that we loaded from CSV (the total number of rows with NAs) we can get by doing:


```r
missingValues <- nrow(activityData[is.na(activityData$steps),])
```

So the total number of NA values in the dataset is ``2304``

Now we're going to take initial dataset and fill all the missing values with mean values for that 5-minute interval.


```r
newActivityData <- activityData

for (i in 1:nrow(newActivityData)) {
# check if the 'steps' value is missing
    if (is.na(newActivityData$steps[i])) {
        interval <- as.numeric(newActivityData$interval[i])
        avgsteps <- stepsPerInterval[which(stepsPerInterval$interval == interval),]$steps
        newActivityData$steps[i] <- avgsteps
    }
}
```

Now, we're making a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day but using the new dataset

First, we need to aggregate the data by making a sum of number of steps per each day.


```r
newStepsPerDay <- aggregate(newActivityData[, 'steps'], by = list(newActivityData$date), FUN = sum)
names(newStepsPerDay) <- c("date", "steps")
```

And now plotting the actual histogram.


```r
library(ggplot2)
qplot(x = newStepsPerDay$steps, 
      data = newStepsPerDay, 
      geom = "histogram", 
      binwidth=1000,
      main =  "Histogram of the total number of steps taken each day",
      xlab = "Number of steps per day",
      ylab = "Days with the same steps count")
```

![](figures/histogram2-1.png) 

The mean and median total number of steps taken per day but using the new dataset


```r
mean(newStepsPerDay$steps)
```

```
## [1] 10766.19
```

```r
median(newStepsPerDay$steps)
```

```
## [1] 10766.19
```

Mean and median values have increased comparing to the mean and median of the initial dataset. This is because we replaced all NA values wich were treated as zeros with mean values (larger than 0) and that made an impact on the estimates of the total daily number of steps.

## Are there differences in activity patterns between weekdays and weekends?

Now we're going to check if there are any differences in activity patterns between weekdays and weekends.
We need to create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day. The easiest way to determine if a date is weekday or not is using 'lubridate' package.
We're using the new dataset with no NA values.


```r
library(lubridate)
newActivityData$type <- as.factor(ifelse(wday(newActivityData$date, label = TRUE) %in% 
    c("Sat", "Sun"), "weekend", "weekday"))

# Split dataset by date type
splittedData <- split(newActivityData, newActivityData$type)

# Data frame with weekend averaged steps per interval
weekendStepsPerInterval =  aggregate(splittedData$weekend[, 'steps'], 
                                     by = list(splittedData$weekend$interval), 
                                     FUN = mean)
names(weekendStepsPerInterval) <- c("interval", "steps")

# Data frame with weekday averaged steps per interval
weekdayStepsPerInterval =  aggregate(splittedData$weekday[, 'steps'], 
                                     by = list(splittedData$weekday$interval), 
                                     FUN = mean)
names(weekdayStepsPerInterval) <- c("interval", "steps")

# Adding 'type' variable in each datasets needed for combined plot
weekendStepsPerInterval$type <- factor(rep("weekend", nrow(weekendStepsPerInterval)), 
                                       levels = c("weekday", "weekend"))
weekdayStepsPerInterval$type <- factor(rep("weekday", nrow(weekdayStepsPerInterval)), 
                                       levels = c("weekday", "weekend"))
```

With the two new datasets for weekends and weekdays we can make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
ggplot(rbind(weekdayStepsPerInterval, weekendStepsPerInterval)) + 
    aes(x = interval, y = steps) + 
    facet_grid(type ~ .) + 
    geom_line() + 
    labs(title = "Time series plot of average number of steps per 5-minute interval")
```

![](figures/weekends-weekdays_steps-1.png) 

From the comparing plots we can see that activity patterns between weekdays and weekends are different.
