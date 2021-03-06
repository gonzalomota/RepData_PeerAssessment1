# Reproducible Research: Peer Assessment 1

#Introduction
The object of this report is analyzing the data collected from a personal activity monitoring device. This devices records the number of steps taken in 5 minute intervals each day, during a number of days.

The data is that will be used is in a file structured with three columns:

- "steps", indicating the number of steps taken.
- "date", indicating the date of the measurement.
- "interval", indicating the 5 minute interval that the measurement was taken in.

#Analysis

## Loading and preprocessing the data

Before anything else is done, some R libraries will be loaded to assist the data analysis.

```r
library(plyr)
library(lubridate)
library(dplyr)
library(lattice)
```
Plyr and dplyr will allow for an easier manipulation of the dataframes, permitting the use of the rename and mutate functions. Lubridate allows a very intuitive approach to working with date and time. Finally lattice will be used to plot a specific graph at the end of the report.  

###1. Load the data (i.e. read.csv())

Since the data to be analyzed is in a zip file, first it must be unzipped, and then read to a dataframe.


```r
unzip ("activity.zip", exdir = "./")
activity <- read.csv("activity.csv", sep=",", stringsAsFactors = FALSE, quote="", dec=".", numerals = "no.loss", header=FALSE, col.names=c("steps","date", "interval"),blank.lines.skip=TRUE,skip=1)
```

###2. Process/transform the data (if necessary) into a format suitable for your analysis

The next steps are used to load the data into a dplyr dataframe, where it will be easier to manipulate. Both the "steps" and "interval" columns are converted to numeric type.


```r
activityDf <- tbl_df(activity)
rm(activity)
activityDf$steps <- as.numeric(as.character(activityDf$steps))
activityDf$interval <- as.numeric(as.character(activityDf$interval))
```

## What is mean total number of steps taken per day?


###1. Calculate the total number of steps taken per day


```r
stepsPerDay <- activityDf %>% group_by(date) %>% summarise(mean(steps, na.rm=TRUE))
stepsPerDay <- plyr::rename(stepsPerDay, c("mean(steps, na.rm = TRUE)"= "meanSteps"))
```

###2. Make a histogram of the total number of steps taken each day


```r
hist(stepsPerDay$meanSteps, xlab = "Number of steps", main = "Histogram of the total number of steps taken each day")
```

![](PA1_template_files/figure-html/histStepsPerDay-1.png) 

###3. Calculate and report the mean and median of the total number of steps taken per day


```r
mean(activityDf$steps, na.rm=TRUE)
```

```
## [1] 37.3826
```

```r
median(activityDf$steps, na.rm=TRUE)
```

```
## [1] 0
```

## What is the average daily activity pattern?

###1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
dailyAct <- activityDf %>% group_by(interval) %>% summarise(mean(steps, na.rm=TRUE))
dailyAct <- plyr::rename(dailyAct, c("mean(steps, na.rm = TRUE)"= "meanSteps"))
plot(dailyAct$interval, dailyAct$meanSteps, ylab = "", xlab="",type="l")
```

![](PA1_template_files/figure-html/plotDailyPattern-1.png) 

###2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
dailyActOrdered <- arrange(dailyAct,desc(meanSteps))
dailyActOrdered[1,1]
```

```
## Source: local data frame [1 x 1]
## 
##   interval
## 1      835
```

## Imputing missing values

###1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(is.na(activityDf$steps))
```

```
## [1] 2304
```

###2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

We will use the median for the day.

###3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
medianDf <-activityDf %>% group_by(interval) %>% summarise(median(steps, na.rm=TRUE))
medianDf <- plyr::rename(medianDf, c("median(steps, na.rm = TRUE)"= "medianSteps"))
medianDf <- inner_join(medianDf,activityDf, by='interval')
medianDf$steps[is.na(medianDf$steps)] <- medianDf$medianSteps[is.na(medianDf$steps)]
completeActivityDf <- select(medianDf, -medianSteps)
```

###4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
stepsPerDay <- completeActivityDf %>% group_by(date) %>% summarise(mean(steps, na.rm=TRUE))
stepsPerDay <- plyr::rename(stepsPerDay, c("mean(steps, na.rm = TRUE)"= "meanSteps"))
hist(stepsPerDay$meanSteps, xlab = "Number of steps", main = "Histogram of the total number of steps taken each day")
```

![](PA1_template_files/figure-html/imputtingValues-1.png) 

```r
mean(completeActivityDf$steps, na.rm=TRUE)
```

```
## [1] 32.99954
```

```r
median(completeActivityDf$steps, na.rm=TRUE)
```

```
## [1] 0
```

The histogram is clearly imbalanced towards 0 now. The impact in the values is not great, especially in the median. The mean varies slightly due to the high difference between the mean and the median (the value we chose to substitute the missing value).

## Are there differences in activity patterns between weekdays and weekends?

###1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
completeActivityDf <- mutate (completeActivityDf, day = as.factor(ifelse(wday(ymd(date), label = TRUE)%in% c("Sat","Sun"), "weekend", "weekday")))
```

###2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
weekdayDf <- completeActivityDf %>% group_by(interval,day) %>% summarise(mean(steps, na.rm=TRUE))
weekdayDf <- plyr::rename(weekdayDf, c("mean(steps, na.rm = TRUE)"= "steps"))
weekdayDf <- transform(weekdayDf, day = factor(day))
xyplot(steps ~ interval | day, data = weekdayDf, layout = c(1, 2), type ='l', xlab="Interval", ylab="Number of steps")
```

![](PA1_template_files/figure-html/activityPattern-1.png) 


