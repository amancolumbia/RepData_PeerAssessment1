# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
data <- read.csv("data/activity.csv")
```

## What is mean total number of steps taken per day?
#### 1) Calculate the total number of steps taken per day

```r
# I have chosen not to remove the NA's at the following aggregation step. I will remove the NA while taking Mean/Median, in few steps later. Reason for this choice is to avoid filling the missing data with zeros, which will skew the mean and median calculations. 
steps_per_day <- with(data, aggregate(steps, by = list(date = date), FUN = sum))
names(steps_per_day)[2] <- "steps"
```
#### 2) Make a histogram of the total number of steps taken each day

```r
library(ggplot2)
ggplot(steps_per_day, aes(steps))+ geom_histogram(boundary=0, binwidth=2500, col="blue", fill="lightblue") + ggtitle("Histogram of the total number of steps taken each day") + xlab("Steps") + ylab("Frequency")
```

```
## Warning: Removed 8 rows containing non-finite values (stat_bin).
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

#### 3) Calculate and report the mean and median of the total number of steps taken per day

```r
mean(steps_per_day$steps,na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(steps_per_day$steps,na.rm=TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

#### 1) Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
steps_per_interval <- with(data, aggregate(steps, by=list(interval = interval), FUN=sum,na.rm=TRUE))
names(steps_per_interval)[2] <- "steps"
#plot
ggplot(steps_per_interval, aes(interval, steps)) + geom_line(col="blue")+ggtitle("Time Series Plot of Average Number of Steps Taken ")+xlab("Time")+ylab("Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

#### 2) Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
steps_per_interval[which.max(steps_per_interval$steps),1]
```

```
## [1] 835
```

## Imputing missing values

#### 1) Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```
#### 2) Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
# Using 'mean for that 5-minute interval' for filling the missing data. Step 1 is to calculate the mean value per interval
steps_per_interval <- with(data, aggregate(steps, by=list(interval = interval), FUN=mean,na.rm=TRUE))
names(steps_per_interval)[2] <- "steps"

#Step2: make a new column in dataframe which will store all the imputed values. Initialize this column with the original steps column 
data$imputed_steps <- data$steps 

#Step3: Now find the rows from imputed_steps column, which are NA, and fill them with the mean from step_per_interval dataframe by matching the interval.
data$imputed_steps[is.na(data$imputed_steps)] <- steps_per_interval$steps[match(data$interval[is.na(data$imputed_steps)],steps_per_interval$interval)]
```
#### 3) Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
#Created new dataframe with imputed_steps as 'steps'
new_data <- data.frame(steps=data$imputed_steps, interval=data$interval, date=data$date)
```

#### 4) Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
steps_per_day_newData <- with(new_data, aggregate(steps, by = list(date = date), FUN = sum))
names(steps_per_day_newData)[2] <- "steps"


ggplot(steps_per_day_newData, aes(steps))+ geom_histogram(boundary=0, binwidth=2500, col="blue", fill="lightblue") + ggtitle("Histogram of the total number of steps taken each day") + xlab("Steps") + ylab("Frequency") 
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->


```r
#Calculate the Mean and Median of imputed data
mean(steps_per_day_newData$steps)
```

```
## [1] 10766.19
```

```r
median(steps_per_day_newData$steps)
```

```
## [1] 10766.19
```
##### I noted that the imputed mean and median are almost same as the original mean and median calculated above. This shows that in the case of given data, the imputation has minimal impact.

##### Impact of imputing missing data on the estimates of the total daily number of steps is that we can see from histogram that the frequency of of some steps has shifted perticularly between 10000 and 15000.

## Are there differences in activity patterns between weekdays and weekends?
#### 1) Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
new_data$day_type <- ifelse(weekdays(as.Date(new_data$date))=='Saturday' | weekdays(as.Date(new_data$date))=='Sunday', 'weekend','weekday')
```

#### 2) Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
# create table with steps per time across weekdaydays or weekend days
steps_per_interval_newData <- with(new_data, aggregate(steps, by=list(interval = interval, day_type = day_type), FUN=mean,na.rm=TRUE))
names(steps_per_interval_newData)[3] <- "steps"

# draw the line plot
ggplot(steps_per_interval_newData, aes(interval, steps)) + geom_line(col="darkblue") + ggtitle("Time Series Plot of Average Number of Steps Taken") + xlab("Time") + ylab("Steps") + facet_grid(day_type ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)<!-- -->
