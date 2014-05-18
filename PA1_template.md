# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
I load the data with the command, and the result is stored in the *activities* variables

```r
activities <- read.csv("activity.csv")
```



## What is mean total number of steps taken per day?

* The histogram:

The number of steps taken per day is stored in the *histogram* vector


```r
histogram <- NULL
for (day in levels(activities$date)) {
    histogram <- c(histogram, sum(activities$steps[activities$date == day], 
        na.rm = T))
}
```

And the results are visualized by the **hist** function :


```r
hist(histogram, main = "Histogram of the total of steps per day", xlab = "total of steps per day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 


* The mean and the median:

Thank to the function **mean**, the mean of the total number of steps per day is computed:

```r
mean(histogram)
```

```
## [1] 9354
```

The same is done with the function **median**:

```r
median(histogram)
```

```
## [1] 10395
```

## What is the average daily activity pattern?
* The average daily activity pattern:

This time the function **tapply** is used to compute the mean of steps for each time interval.

```r
act_pattern <- tapply(activities$steps, activities$interval, mean, na.rm = T)
plot(seq(1/12, 24, 1/12), act_pattern, type = "l", main = "Average number of steps in a day", 
    xlab = "Time (hour)", ylab = "Number of steps")  # the scale of x-axis is changed to have the time in hours
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 


* Interval which contains the maximum of steps in average:


```r
which(act_pattern == max(act_pattern))
```

```
## 835 
## 104
```

Which means that the maximum valued is reached for the 104th element of the vector *act_pattern* and corresponds to the 5-minute interval 835.
## Imputing missing values
* Number of missing values:

The total number of missing values is determined with **is.na**.

```r
sum(is.na(activities$steps))
```

```
## [1] 2304
```

There **2304** missing values.
* Strategy for filling the missing values in the dataset:

The strategy used is to replace the NA values by the previously computed means stored in *act_pattern*.

* Generating the new dataset:

```r
# Initializing the new dataset
activities_new <- activities
for (i in 1:length(activities$steps)) {
    if (is.na(activities$steps[i])) {
        activities_new$steps[i] = act_pattern[(i - 1)%%289 + 1]
    }
}
```

To check that the two data sets are equals everywhere except for NA values, the following number is computed:

```r
sum(abs(activities$steps - activities_new$steps), na.rm = T)
```

```
## [1] 0
```

Which proves that *activities* and *activities_new* are the same data sets (except for NA values).

* Histogram, mean and median:
We can reuse the code written for the first part:
  * Histogram:

```r
histogram_new <- NULL
for (day in levels(activities_new$date)) {
    histogram_new <- c(histogram_new, sum(activities_new$steps[activities_new$date == 
        day]))
}
hist(histogram_new, main = "Histogram of the total of steps per day with the new data", 
    xlab = "Total of steps per day")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 

  * Mean:

```r
mean(histogram_new, na.rm = T)
```

```
## [1] 10766
```

  * Median:

```r
median(histogram_new, na.rm = T)
```

```
## [1] 10766
```

Adding values with this method seems implies a decrease of the extreme value and a concentration around the mean.
## Are there differences in activity patterns between weekdays and weekends?
First I add the new column to the data set as defined in the Peer assignment:

```r
options(warn = -1)
# Initialisation at 'weekday'
activities_new$week <- "weekday"
# With the weekdays function, we can find Saturdays and Sundays (my RStudio
# is in French, so here 'Samedi' and 'Dimanche')
activities_new$week[weekdays(as.Date(activities_new$date)) == "Samedi"] <- "weekend"
activities_new$week[weekdays(as.Date(activities_new$date)) == "Dimanche"] <- "weekend"
# And the levels are set:
levels(activities_new$week) <- factor(activities_new$week)
options(warn = 0)
```

Using the plotting system, we can now compare the average activities in the weekend and during the week:

```r
act_pattern_WD <- tapply(activities_new$steps[activities_new$week == "weekday"], 
    activities_new$interval[activities_new$week == "weekday"], mean, na.rm = T)

act_pattern_WE <- tapply(activities_new$steps[activities_new$week == "weekend"], 
    activities_new$interval[activities_new$week == "weekend"], mean, na.rm = T)

par(mfrow = c(2, 1))
plot(seq(1/12, 24, 1/12), act_pattern_WD, type = "l", main = "Average number of steps in a day (weekdays)", 
    xlab = "Time (hour)", ylab = "Number of steps")
plot(seq(1/12, 24, 1/12), act_pattern_WE, type = "l", main = "Average number of steps in a day (weekends)", 
    xlab = "Time (hour)", ylab = "Number of steps")
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15.png) 

Those two patterns are quite different. During the weekdays, the activity increases rapidly in the morning, which corresponds to the period where the person starts his/her day and stays approximately constant instantly with some pics during the day. Considering the weekend pattern, we can see that during the day, the activity is more or less random. This is mostly because during the weekdays, there is a predefined routine and during the weekends the activities are usually not the same.
