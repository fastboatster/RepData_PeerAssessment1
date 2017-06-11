# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
require(stats)
require(readr)
```

```
## Loading required package: readr
```

```r
require(ggplot2)
```

```
## Loading required package: ggplot2
```

```r
require(scales)
```

```
## Loading required package: scales
```

```
## 
## Attaching package: 'scales'
```

```
## The following object is masked from 'package:readr':
## 
##     col_factor
```

```r
activity <- read_csv("activity.csv")
```

```
## Parsed with column specification:
## cols(
##   steps = col_integer(),
##   date = col_date(format = ""),
##   interval = col_integer()
## )
```


## What is mean total number of steps taken per day?
* Make a histogram of the total number of steps taken each day:


```r
steps_per_day <- aggregate(activity$steps, list(activity$date), sum)
qplot(steps_per_day$x, geom = "histogram") 
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 8 rows containing non-finite values (stat_bin).
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

* Calculate and report the mean and median total number of steps taken per day:


```r
mean_steps <- mean(steps_per_day$x, na.rm = TRUE)
median_steps <- median(steps_per_day$x, na.rm = TRUE)
print(mean_steps)
```

```
## [1] 10766.19
```

```r
print(median_steps)
```

```
## [1] 10765
```

_Mean number of steps per day is 10766.1886792453, while median number of steps per day is 10765._

## What is the average daily activity pattern?

* Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
steps_per_interval <- aggregate(activity$steps, list(activity$interval), function(x) {mean(x, na.rm = TRUE)})
ggplot(steps_per_interval, aes(Group.1, x)) + geom_line()+ xlab("Interval") + ylab("Number of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

* Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
max_steps <- max(steps_per_interval$x)
max_interval <- steps_per_interval[steps_per_interval$x == max_steps,]$Group.1
print(max_interval)
```

```
## [1] 835
```

```r
print(max_steps)
```

```
## [1] 206.1698
```
_We can see that interval 835 contains the maximum number of steps (206.1698113)._

## Imputing missing values

* Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
num_na <- nrow(activity[is.na(activity$steps) == TRUE,])
print(num_na)
```

```
## [1] 2304
```
_Total number of missing values in the dataset is 2304._

* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, **or the mean for that 5-minute interval**, etc.

_We will use strategy highlighted in bold (see above). I.e., for every missing value, we will use mean for that 5-minute interval._

* Create a new dataset that is equal to the original dataset but with the missing data filled in:


```r
impute_values <- function(row_vector, df_value_source){
        if (is.na(row_vector["steps"])) {
                row_vector["steps"] <- df_value_source[df_value_source$Group.1 == row_vector["interval"],]$x
        }
        else {
                row_vector["steps"]     
        }

}
activity_imputed <- activity
# iterate over the dataframe row by row and impute missing values with values from
# dataframe containing mean steps values for each interval
activity_imputed$steps <- apply(data.matrix(activity_imputed), MARGIN = 1, FUN = impute_values, steps_per_interval)
```

* Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
steps_per_day_imputed <- aggregate(activity_imputed$steps, list(activity_imputed$date), sum)
mean_steps_imp <- mean(steps_per_day_imputed$x)
median_steps_imp <- median(steps_per_day_imputed$x)
print(median_steps_imp)
```

```
## [1] 10766.19
```

```r
print(mean_steps_imp)
```

```
## [1] 10766.19
```

```r
qplot(steps_per_day_imputed$x, geom = "histogram") 
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

_After imputing missing values, mean number of steps per day is 10766.1886792453 and median number of steps per day is 10766.1886792453. It can be seen that histogram's shape hasn't
changed much and mean number of steps also has stayed the same (which makes sense
because we used mean values to impute our dataset). Median number of steps per
day, however, has changed slightly and has become the same as mean number._

## Are there differences in activity patterns between weekdays and weekends?

* Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
get_day_type <- function(date_val) {
        day_type <- "Weekday"
        weekends <- c("Saturday", "Sunday")
        if (weekdays(date_val) %in% weekends) {
                day_type <- "Weekend"
        }
        day_type
}
activity_imputed$day <- as.factor(sapply(activity_imputed$date, get_day_type))
```

* Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis):


```r
# calculates average number of steps per interval per weekday type
activity_per_day <- aggregate(activity_imputed$steps, list(activity_imputed$interval, activity_imputed$day), mean)
colnames(activity_per_day) <- c("interval", "day", "steps")
# plots timeline charts for weekdays and weekends, respectively
ggplot(activity_per_day, aes(interval, steps)) + geom_line() + xlab("Interval") + ylab("Number of Steps") + facet_wrap( ~day, ncol = 1)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
