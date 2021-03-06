---
title: "PA1_template.Rmd"
author: "ME@"
date: "July 23, 2020"
output: 
  html_document:
      keep_md: true
---

###Load and Pre-Process Data


```r
data <- read.csv("activity.csv")

#Change the column 'interval' from integer to factor
data$interval <- as.factor(data$interval)
```

###Mean Total Number of Steps per Day


```r
  #Per assignment instruction, subset out NA values for this part
  data_na_rm <- data[!is.na(data$steps),]
  
  #Calculae daily steps (8 days of data are no longer represented due to NA removal in previous step)
  DailySteps <- aggregate(steps ~ date, data = data_na_rm, FUN = sum)
  
  hist(DailySteps$steps, breaks = 10, xlab = "Daily Steps", main = "Histogram of Daily Steps")
```

![](Project_1_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
  mean(DailySteps$steps)
```

```
## [1] 10766.19
```

```r
  median(DailySteps$steps)
```

```
## [1] 10765
```

###Average Daily Activity Pattern

```r
### Average Daily Activity by Interval  
  IntervalSteps <- aggregate(steps ~ interval, data = data_na_rm, FUN = mean)
  plot(x = levels(IntervalSteps$interval), y = IntervalSteps$steps, type = "l", xlab = "Interval", ylab = "Mean Steps", main = "Plot of mean steps per day by interval")
```

![](Project_1_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
  MaxInterval <- IntervalSteps[IntervalSteps$steps==max(IntervalSteps$steps),]
```
The interval with the most steps is:

```r
MaxInterval[1,1]
```

```
## [1] 835
## 288 Levels: 0 5 10 15 20 25 30 35 40 45 50 55 100 105 110 115 120 ... 2355
```
And the average number of steps taken in that interval is:

```r
MaxInterval[1,2]
```

```
## [1] 206.1698
```
###Imputing Missing Values
The number of missing values in the data set is:

```r
NAcount <- length(data$steps[is.na(data$steps)])
NAcount
```

```
## [1] 2304
```
I use the average number of steps per day by interval (from the previous step) to impute missing values and create a new data set as shown in the following:

```r
#Subset data on only NA values
NAdata <- data[is.na(data$steps),]
#Set imputed step values using IntervalSteps previously calculated
NAdata$steps <- rep(IntervalSteps$steps, length(unique(NAdata$date)))
#Combine data subsets for new data set
Newdata <- rbind(data_na_rm,NAdata)
```
Recalculate daily steps and output histogram, mean, and median

```r
#Hist, mean, and median for number of steps taken each day
NewDailySteps <- aggregate(steps ~ date, data = Newdata, FUN = sum)
hist(NewDailySteps$steps, breaks = 10, xlab = "Daily Steps", main = "Histogram of Daily Steps")
```

![](Project_1_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
mean(NewDailySteps$steps)
```

```
## [1] 10766.19
```

```r
median(NewDailySteps$steps)
```

```
## [1] 10766.19
```
Note that the median value calculated here differs slightly from the previous median value, but not significantly.  The mean values are identical, but this is not surprising given that the imputed values were themselves an average taken from the available data.  The impact of including the imputed values is that eight additional days of data are accounted for resulting in an increase in the number of steps.  Additionally, since the the total number of daily steps for the imputed days are identical (due to the method of imputation) they are all added to the same bin as shown in the histogram.

###Activity Differences for Weekday vs. Weekend

Create a new variable to capture weekend vs weekday.  Recalculate average steps taken by interval by weekday type.

```r
#Format date column as a date and create new column 'DayType' to label as weekend or weekday  
Newdata$date <- as.Date(Newdata$date)
Newdata$DayType <- weekdays(Newdata$date)
  
# Revalue weekday names as Weekday or Weekend
for(i in 1:nrow(Newdata)){
  
  if(Newdata$DayType[i] == "Saturday" | Newdata$DayType[i] == "Sunday"){Newdata$DayType[i] = "Weekend"
  } else{Newdata$DayType[i] = "Weekday"}
}  
#Calculate average steps taken by DayType (weekday vs weekend)
DayTypeSteps <- aggregate(steps ~ DayType + interval, data = Newdata, mean)
```
Plot the new DayTypeSteps calculations

```r
library(ggplot2)
  plt <- ggplot(DayTypeSteps, aes(interval, steps))  
  plt + facet_grid(rows = vars(DayType)) + geom_line(aes(group = DayType)) + scale_x_discrete(breaks = DayTypeSteps$interval[seq(1, length(DayTypeSteps$interval), by = 10)]) + theme(axis.text.x = element_text(angle = 90)) + ggtitle("Weekday vs. Weekend Activity Levels")
```

![](Project_1_files/figure-html/unnamed-chunk-10-1.png)<!-- -->




















