


## PA1_template.Rmd

###This is an R Markdown document.
####This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below.

## Loading and preprocessing the data

 Show any code that is needed to
 1. Load the data (i.e. read.csv())
 2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
if(file.exists("activity.csv")){
        dat <- read.csv("activity.csv")
}
```

## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day
2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
3. Calculate and report the mean and median of the total number of steps taken per day


```r
# Chargue data only with data different to NA

# to convert kind of type for date column
dat$date <- as.Date(dat$date)
dat$date <- as.factor(dat$date)

# to obtein sum from each day in data
s1 <- with(dat, tapply(steps,date,na.rm=TRUE,sum))

# Create a histogram with ggplot2
library(ggplot2)
qplot(s1, geom="histogram", xlab = "Total steps per day", ylab = "Frequency", binwidth=400)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png)

* mean
 

```r
# Calculate mean
 mean(s1)
```

```
## [1] 9354.23
```

* median


```r
# Calculate median
median(s1)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)




```r
        s2 <- dat %>% filter(!is.na(steps)) %>% group_by(interval) %>% summarise(steps = mean(steps))
        ggplot(data=s2,aes(interval,steps)) + geom_line() + xlab("5 minute interval") + ylab("Average of number steps")
```

![plot of chunk series](figure/series-1.png)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

The 5-minute interval that contains the maximum number of steps is:


```r
        s2[which.max(s2$steps),"interval"]
```

```
## # A tibble: 1 x 1
##   interval
##      <int>
## 1      835
```

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
        datMiss <- dat[is.na(dat$steps),]
        numMiss <- nrow(datMiss)
```
The total number are: 2304

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
        # s2 has the mean for steps group by interval, we created an merge
        # s3 will content interval correct
        s3 <- merge(datMiss,s2,by.x="interval", by.y = "interval", all = TRUE)
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
        # Missed data with the corresponding mean
        s4 <- dat
        s4[is.na(s4$steps),]$steps <- s3$steps.y
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
        # to convert kind of type for date column
        s4$date <- as.Date(s4$date)
        s4$date <- as.factor(s4$date)

        # to obtein sum from each day in data
        s5 <- with(s4, tapply(steps,date,na.rm=TRUE,sum))
        qplot(s5, geom="histogram", xlab = "Total steps per day", ylab = "Frequency", binwidth=400)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png)

* mean
 

```r
# Calculate mean
 mean(s5)
```

```
## [1] 10766.19
```

* median


```r
# Calculate median
median(s5)
```

```
## [1] 11015
```

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.



```r
        s4$typeDate <-  ifelse(as.POSIXlt(s4$date)$wday %in% c(0,6), 'weekend', 'weekday')
```

2. Make a panel plot containing a time series plot


```r
        s6 <- s4 %>% group_by(interval,typeDate) %>% summarise(steps = mean(steps))
        ggplot(data=s6,aes(interval,steps)) + geom_line() + facet_grid(typeDate ~ .) + xlab("5 minute interval") + ylab("Average of number steps")
```

![plot of chunk series2](figure/series2-1.png)

