---
title: "Reproducible Research: Peer Assessment 1"
output:
  html_document:
    keep_md: yes
---

# Reproducible Research: Peer Assessment 1

## Introduction

It is now possible to collect a large amount of data about personal movement
using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone
Up. These type of devices are part of the “quantified self” movement – a group
of enthusiasts who take measurements about themselves regularly to improve
their health, to find patterns in their behavior, or because they are tech geeks.
But these data remain under-utilized both because the raw data are hard to
obtain and there is a lack of statistical methods and software for processing and
interpreting the data.

This assessment makes use of data from a personal activity monitoring device.
This device collects data at 5 minute intervals through out the day. The data
consists of two months of data from an anonymous individual collected during
the months of October and November, 2012 and include the number of steps
taken in 5 minute intervals each day.

## Data

The data can be downloaded from the course web site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a 
total of 17,568 observations in this dataset.

For convience the raw dataset is included in this repository.

## Loading and preprocessing the data

Ensure that we show all our working:

```r
require(knitr, quietly = TRUE)
opts_chunk$set(echo = TRUE, cache = TRUE, fig.width = 10)
```

Load data into a data frame:

```r
require(utils, quietly = TRUE)

# unzip overwriting existing directory to ensure clean setup
if (!file.exists("activity.csv")) {
    # download archive into local directory
    zipFileName <- file.path("activity.zip")
    if (!file.exists(zipFileName)) {
        zipUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
        download.file(zipUrl, destfile = zipFileName, method = "curl", mode = "wb")
        print(paste(Sys.time(), "archive downloaded"))
    }
    # unpack archive
    unzip(zipFileName, overwrite = FALSE)
    print(paste(Sys.time(), "archive unpacked"))
    rm(zipFileName)
}

# load into data frame and convert date column to date
data <- read.csv("activity.csv", stringsAsFactors = FALSE)
# make new column datetime using date and 5 minute intervals
data <- transform(data, 
                  datetime = strptime(
                                paste(date, formatC(interval, flag = "0", width = 4)), 
                                format = "%Y-%m-%d %H%M"), 
                  date = as.Date(data$date, "%Y-%m-%d")
                  )
```

## What is mean total number of steps taken per day?

The following histogram shows the total number of steps taken each day during 
the two month period from **Mon Oct 01, 2012**
to **Fri Nov 30, 2012**. This ignores days for which 
no data was recorded.

Aggregate the total number of steps per day:

```r
dailyTotals <- aggregate(steps ~ date, data, FUN = sum)
```

The mean number of steps was 
**10,766** per day. 
_(Rounded up from **10,766.19**
as fractional steps do not make sense.)_  
The median number of steps was 
**10,765** per day. 

Below is a plot showing the total number of steps per day as a histogram:

```r
require(magrittr, quietly = TRUE)
require(ggplot2, quietly = TRUE)
require(scales, quietly = TRUE)
dailyTotals %>%
    ggplot(aes(steps)) + 
    geom_histogram(binwidth = 1000, fill = "purple", colour = "black", alpha = 0.7) +
    theme_light(base_family = "Avenir", base_size = 11) +
    scale_y_continuous(breaks = pretty_breaks(10)) +
    labs(x = "Steps") +
    labs(y = "Frequency") +
    ggtitle("Histogram: total steps per day")
```

![plot of chunk histogram](figure/histogram-1.png) 

## What is the average daily activity pattern?

Average the number of steps taken across all days:


```r
intervalTotals <- aggregate(steps ~ interval, data, FUN = mean)
```

The 5-minute interval which on average across all the days in the dataset
contains the maximum number of steps is
**835**.
This peak is shown in the time series (line) plot of the 5-minute intervals and
the number of steps taken averaged across all days:


```r
require(magrittr, quietly = TRUE)
require(ggplot2, quietly = TRUE)
require(scales, quietly = TRUE)
intervalTotals %>%
    ggplot(aes(interval, steps)) + 
    geom_line(color = "purple") +
    theme_light(base_family = "Avenir", base_size = 11) +
    scale_x_discrete(breaks = pretty_breaks(15)) +
    scale_y_continuous(breaks = pretty_breaks(10)) +
    labs(x = "Interval") +
    labs(y = "Steps") +
    theme(legend.position = "none") +
    ggtitle("Time Series: steps per 5-minute interval")
```

![plot of chunk timeseries](figure/timeseries-1.png) 

## Imputing missing values

Note that there are a number of days/intervals where there are missing
values (coded as `NA`). The presence of missing days may introduce
bias into some calculations or summaries of the data.

There is missing steps data represented by rows with steps value of `NA`.
There are **``2304`` ** of **``17568`` **
rows without step values. That is, around
**``13``% ** of 
step data is missing.

We will impute this missing steps data using the _median_ for that _weekdays_ 
5-minute interval. That is modelling using similar activity by day of week.

```r
require(dplyr, quietly = TRUE)
intervalsByDay <- data %>%
    mutate(dayint = paste0(format(as.Date(date), "%a"), formatC(interval, flag = "0", width = 4))) %>%
    select(dayint, steps) %>%
    group_by(dayint) %>%
    summarise(median = as.integer(median(steps, na.rm = TRUE)))
```

Using these day / interval medians we can now impute the missing steps data.

```r
require(dplyr, quietly = TRUE)
imputedData <- data %>%
    mutate(dayint = paste0(format(as.Date(date), "%a"), formatC(interval, flag = "0", width = 4))) %>%
    select(dayint, date, interval, steps)
## add median for days interval into data frame
imputedData <- merge(imputedData, intervalsByDay)
## use day interval median into steps where steps is missing
imputedData <- imputedData %>%
    mutate(steps = ifelse(is.na(steps), median, steps)) %>%
    select(date, interval, steps)
```
Further analysis will be down on the results of this imputed data set. Firstly,
summarise the total number of steps per day, which we will show in a histogram.


```r
dailyImputedTotals <- aggregate(steps ~ date, imputedData, FUN = sum)
```

The mean number of steps was 
**9,705**
per day. _(Rounded up from 
**9,704.656** as fractional 
steps do not make sense.)_  
The median number of steps was 
**10,395** per day. 

Notice that imputed data shows an increase in frequency of lesser steps. This 
has the side-effect of reducing the mean step count per day, while essentially 
leaving the median step count unchanged.


```r
require(magrittr, quietly = TRUE)
require(ggplot2, quietly = TRUE)
require(scales, quietly = TRUE)
dailyImputedTotals %>%
    ggplot(aes(steps)) + 
    geom_histogram(binwidth = 1000, fill = "purple", colour = "black", alpha = 0.7) +
    theme_light(base_family = "Avenir", base_size = 11) +
    scale_y_continuous(breaks = pretty_breaks(10)) +
    labs(x = "Steps") +
    labs(y = "Frequency") +
    ggtitle("Histogram: total imputed steps per day")
```

![plot of chunk histogramimputed](figure/histogramimputed-1.png) 

## Are there differences in activity patterns between weekdays and weekends?

Using imputed data, we will now compare weekday activity to weekends.


```r
require(dplyr, quietly = TRUE)
## create a factor for weekday / weekend
## format %u gives weekday as a decimal number (1–7, Monday is 1)
## so weekdays are when %u < 6, weekends when %u = 6 or 7
imputedWeekDayData <- imputedData %>%
    mutate(weekday = factor(ifelse(format(date, "%u") < 6, "weekday", "weekend"))) %>%
    select(weekday, interval, steps) %>%
    # arrange(weekday, interval) %>%
    group_by(weekday, interval) %>%
    summarise(average = mean(steps))
```

The following time series plot shows the 5-minute interval by weekday/weekend:


```r
require(magrittr, quietly = TRUE)
require(ggplot2, quietly = TRUE)
require(scales, quietly = TRUE)
imputedWeekDayData %>%
    ggplot(aes(x = interval, y = average)) + 
    geom_line(color = "purple") +
    theme_light(base_family = "Avenir", base_size = 11) +
    scale_x_discrete(breaks = pretty_breaks(15)) +
    scale_y_continuous(breaks = pretty_breaks(10)) +
    labs(x = "Interval") +
    labs(y = "Steps (averaged)") +
    facet_grid(weekday ~ .) +
    ggtitle("Time Series: averaged steps in 5 minute intervals by weekday/weekend")
```

![plot of chunk timeseriesimputed](figure/timeseriesimputed-1.png) 

The weekday step average is higher in the morning, possibly indicating that the
individual was active earlier during a weekday. Did they enjoy a sleep-in on
weekends?