# Reproducible Research: Peer Assessment 1
## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a [Fitbit][1], [Nike Fuelband][2], or [Jawbone Up][3]. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

[1]: http://www.fitbit.com/ "Fitbit"
[2]: http://www.nike.com/us/en_us/c/nikeplus-fuelband "Nike Fuelband" 
[3]: https://jawbone.com/up "Jawbone Up"

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.



## Data
The data for this assignment can be downloaded from the following link:

Dataset: [Activity monitoring data (52K)][4]
    
[4]: https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip "Activity monitoring data (52K)"

The variables included in this dataset are:

    - steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

    - date: The date on which the measurement was taken in YYYY-MM-DD format

    - interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.



## Loading and preprocessing the data
It is assumed that the data for this assignment will be stored in a file named *activity.csv* contained within the same directory as the *.R* file performing the processing. The data is loaded into a variable *steps*. Structure of *steps* is examined as well as the first 10 rows:


```r
steps <- read.csv("activity.csv")
str(steps)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
head(steps)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

It is observed that the date column is a factor. This column is converted to the Date class with the following code:


```r
steps$date <- as.Date(as.character(steps$date), format="%Y-%m-%d")
str(steps)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```


## What is mean total number of steps taken per day?
For this portion of the assignment missing values can be ignored. A new dataset is created, which retains only complete observations of the *steps* data:


```r
steps_complete <- steps[complete.cases(steps), ]
str(steps_complete)
```

```
## 'data.frame':	15264 obs. of  3 variables:
##  $ steps   : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ date    : Date, format: "2012-10-02" "2012-10-02" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

Next we calculate the total number of steps per day. To accomplish this we will use the *plyr* library. If you do not have the *plyr* library installed in R you can install it using the following command *install.packages("plyr")*:

Then load the library and perform the calculations saving the result in a new variable *steps_perday*


```r
library(plyr)
steps_perday <- ddply(steps_complete, .(date), summarize, total_steps=sum(steps))
head(steps_perday)
```

```
##         date total_steps
## 1 2012-10-02         126
## 2 2012-10-03       11352
## 3 2012-10-04       12116
## 4 2012-10-05       13294
## 5 2012-10-06       15420
## 6 2012-10-07       11015
```

Next make a histogram of the total number of steps per day:


```r
hist(steps_perday$total_steps, col="red", xlab="Total number of steps per day", ylab="Frequency", main="Steps Per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

The mean and median of the total number of steps per day are calculated, rounded to 2 decimal places:


```r
round(mean(steps_perday$total_steps), 2)
```

```
## [1] 10766.19
```

```r
round(median(steps_perday$total_steps), 2)
```

```
## [1] 10765
```

```r
#remove steps_perday from environment since we have no more use for it
rm(steps_perday)
```

**The mean number of steps per day is 10766.19 and the median is 10765.**



## What is the average daily activity pattern?
Using the plyr package and ddply function as in the previous step, we will arrange our dataset to get the average number of steps per interval, across all days. Again we will use our only complete observations in this analysis.


```r
steps_perinterval <- ddply(steps_complete, .(interval), summarize, average_steps=round(mean(steps), 2))
head(steps_perinterval)
```

```
##   interval average_steps
## 1        0          1.72
## 2        5          0.34
## 3       10          0.13
## 4       15          0.15
## 5       20          0.08
## 6       25          2.09
```

We now make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
plot(steps_perinterval$interval, steps_perinterval$average_steps, type="l", col="red", xlab="Interval", ylab="Average Steps", main="Average Steps Per Interval Across all Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png) 

Determining which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps:


```r
steps_perinterval[steps_perinterval$average_steps == max(steps_perinterval$average_steps), ]
```

```
##     interval average_steps
## 104      835        206.17
```

**Interval 835 contains the maximum number of steps averaged across all days.**

## Imputing missing values

There are a number of days/intervals in the original data where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

The following code determines the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(is.na(steps$steps))
```

```
## [1] 2304
```

**There are 2304 rows with missing values in the dataset.**


Missing values in the dataset will be filled in using the mean for that 5-minute interval as calculated in the previous section on average daily activity pattern.

We will now create a new dataset that is equal to the original dataset but with the missing data filled in. To accomplish this, we will extract all rows with NA values into a separate dataset. This dataset is merged with the steps_perinterval dataset on the interval variable. In the event that sorting occurs on the merged dataset re-sort dataset by date and then interval.


```r
steps_na <- steps[is.na(steps$steps), ]
steps_merged <- arrange((merge(steps_na, steps_perinterval, by="interval", sort=FALSE)), date, interval)
head(steps_merged)
```

```
##   interval steps       date average_steps
## 1        0    NA 2012-10-01          1.72
## 2        5    NA 2012-10-01          0.34
## 3       10    NA 2012-10-01          0.13
## 4       15    NA 2012-10-01          0.15
## 5       20    NA 2012-10-01          0.08
## 6       25    NA 2012-10-01          2.09
```

```r
#remove steps_na and steps_perinterval from environment since we have no more use for them
rm(steps_na)
rm(steps_perinterval)
```

Perform an rbind on the *steps_complete* and the relevant subset of the *steps_merged* dataset, then arrange by date then interval.


```r
#rename average_steps columns to steps for rbind operation
names(steps_merged)[c(2, 4)] <- c("steps_old", "steps")
steps <- arrange(rbind(steps_complete, steps_merged[,c(4,3, 1)]), date, interval)
head(steps)
```

```
##   steps       date interval
## 1  1.72 2012-10-01        0
## 2  0.34 2012-10-01        5
## 3  0.13 2012-10-01       10
## 4  0.15 2012-10-01       15
## 5  0.08 2012-10-01       20
## 6  2.09 2012-10-01       25
```

```r
#recount the number of missing values - this should now be 0
sum(is.na(steps$steps))
```

```
## [1] 0
```

```r
#remove steps_complete and steps_merged from environment since we have no more use for them
rm(steps_complete)
rm(steps_merged)
```

Reprocess the number of steps taken each day along with mean and median


```r
steps_perday <- ddply(steps, .(date), summarize, total_steps=sum(steps))
head(steps_perday)
```

```
##         date total_steps
## 1 2012-10-01    10766.13
## 2 2012-10-02      126.00
## 3 2012-10-03    11352.00
## 4 2012-10-04    12116.00
## 5 2012-10-05    13294.00
## 6 2012-10-06    15420.00
```

The following plot shows a histogram of the total number of steps taken each day


```r
hist(steps_perday$total_steps, col="red", xlab="Total number of steps per day (NAs Estimated)", ylab="Frequency", main="Steps Per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 

The mean and median of the total number of steps per day with NA values estimated are calculated as follows:


```r
round(mean(steps_perday$total_steps),2)
```

```
## [1] 10766.18
```

```r
round(median(steps_perday$total_steps), 2)
```

```
## [1] 10766.13
```

```r
#remove steps_perday from environment since we have no more use for it
rm(steps_perday)
```

**The mean number of steps per day, having estimated and filled in NAs is 10766.18 and the median is 10766.13.** These values do not differ much from the estimates from the first part of the assignment. The mean was relatively consistent with the previously calculated value while the median was slightly higher than the previously calculated values. Imputing missing data on the estimates of the total daily number of steps resulted in the values of both mean and median becoming closer, to near equality.


## Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

To do this we use the mutate function to add a new column to the steps dataset. If the day of *date* is "Saturday" or "Sunday" this column is given the values "Weekend", and "Weekday" otherwise. The column is then converted to a factor variable.


```r
steps <- mutate(steps, time_of_week = ifelse(weekdays(date)=="Saturday"|weekdays(date)=="Sunday", "Weekend", "Weekday"))
steps$time_of_week <- as.factor(as.character(steps$time_of_week))
```

The following creates a time series panel plot comparison of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days for weekday and weekends.

We will utilize the *lattice* plotting library for this.


```r
library(lattice)
weeksteps_perinterval <- ddply(steps, .(time_of_week, interval), summarize, average_steps=round(mean(steps), 2))
xyplot(average_steps ~ interval | time_of_week, data=weeksteps_perinterval, layout=c(1,2), type='l', xlab="Interval", ylab="Average Steps", main="Average Steps Per Interval Across all Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-17-1.png) 

