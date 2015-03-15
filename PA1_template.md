---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

```
## Warning: package 'ggplot2' was built under R version 3.1.2
```

```
## Warning: package 'dplyr' was built under R version 3.1.2
```

```
## 
## Attaching package: 'dplyr'
## 
## The following object is masked from 'package:stats':
## 
##     filter
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

## Loading and preprocessing the data

```r
data = read.csv(file="activity.csv", header=TRUE,)
```

## What is mean total number of steps taken per day?

```r
by_day <- group_by(data, date)
by_day_summary <- summarize(group_by(data, date), total = sum(steps,na.rm=TRUE))
by_day_summary
```

```
## Source: local data frame [61 x 2]
## 
##          date total
## 1  2012-10-01     0
## 2  2012-10-02   126
## 3  2012-10-03 11352
## 4  2012-10-04 12116
## 5  2012-10-05 13294
## 6  2012-10-06 15420
## 7  2012-10-07 11015
## 8  2012-10-08     0
## 9  2012-10-09 12811
## 10 2012-10-10  9900
## ..        ...   ...
```

```r
qplot(by_day_summary$total, binwidth=3000)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

Steps per day: Mean

```r
mean(by_day_summary$total)
```

```
## [1] 9354.23
```

Steps per day: Median

```r
median(by_day_summary$total)
```

```
## [1] 10395
```


## What is the average daily activity pattern?

```r
by_interval <- aggregate(steps ~ interval, data, mean)
qplot(y=by_interval$steps, x=by_interval$interval, geom="line")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

5 minute interval with most steps averaged across all days is

```r
by_interval[which.max(x=by_interval$steps),1]
```

```
## [1] 835
```


## Imputing missing values

Number of rows with missing values

```r
dim(data[data$steps == "NA",])[1]
```

```
## [1] 2304
```

Alternate dataset

```r
alternate_data <- data
for (i in 1:nrow(alternate_data)) {
  if (is.na(alternate_data[i,"steps"])) {
    alternate_data[i,"steps"] <- mean(data[data$date==alternate_data[i,"date"],"steps"])
  }
}

qplot(by_day_summary$total, binwidth=3000)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

## Are there differences in activity patterns between weekdays and weekends?

```r
alternate_data <- mutate(alternate_data, day = weekdays(as.Date(data$date,format="%Y-%m-%d")))
alternate_data[alternate_data$day == "Saturday" | alternate_data$day == "Sunday", "day"] <- "weekend" 
alternate_data[alternate_data$day != "weekend", "day"] <- "weekday" 
alternate_data$day <- as.factor(alternate_data$day)

by_interval_alternate <- aggregate(steps ~ interval + day, alternate_data, mean)
qplot(y=steps, x=interval, data= by_interval_alternate, geom="line", facets=day ~ .)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 
