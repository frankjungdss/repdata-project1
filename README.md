---
title: "README"
author: "Frank Jung"
date: "6 June 2015"
output: html_document
---

repdata-015 peer assessment 1
=============================

This project contains my work for peer assessment 1.

Manifest
--------

* [PA1_template.md](PA1_template.md) - generated markdown
* [PA1_template.Rmd](PA1_template.Rmd) - source markdown
* [PA1_template.html](PA1_template.html) - generated HTML
* [figure/](figure/) - directory containing generated figures

Build
-----

To build HTML output from `Rmd` use:

```r
library(knitr)
knit2html("PA1_template.Rmd")
```

Print with table of contents:

```r
library(knitr)
library(markdown)
knit2html("PA1_template.Rmd",  options = c("toc", markdown::markdownHTMLOptions(TRUE)))
```

Calculating the Average Steps per Day
-------------------------------------

To calcuate the average steps per day I used:

```r
dailyTotals <- aggregate(steps ~ date, data, FUN = sum)
mean(dailyTotals$steps)
# [1] 10766.19
```

This is the same as:

```r
dailyTotals <- data %>% filter(!is.na(steps)) %>% group_by(date) %>% summarise(steps = sum(steps))
mean(dailyTotals$steps)
# [1] 10766.19
```

Both of these produce a `dailyTotals` like:

```r
> head(data.table(dailyTotals), 10)
          date steps
 1: 2012-10-02   126
 2: 2012-10-03 11352
 3: 2012-10-04 12116
 4: 2012-10-05 13294
 5: 2012-10-06 15420
 6: 2012-10-07 11015
 7: 2012-10-09 12811
 8: 2012-10-10  9900
 9: 2012-10-11 10304
10: 2012-10-12 17382
```

Which is different than calculating the average using:

```r
dailyTotals <- data %>% group_by(date) %>% summarise(steps = sum(steps, na.rm = TRUE))
mean(dailyTotals$steps)
# [1] 9354.23
```

Which fills in missing days with zero's. I don't think that is the correct
approach.

```r
> head(data.table(dailyTotals), 10)
          date steps
 1: 2012-10-01     0
 2: 2012-10-02   126
 3: 2012-10-03 11352
 4: 2012-10-04 12116
 5: 2012-10-05 13294
 6: 2012-10-06 15420
 7: 2012-10-07 11015
 8: 2012-10-08     0
 9: 2012-10-09 12811
10: 2012-10-10  9900
```

Environment
-----------

* RStudio Version 0.99.441 
* R Version 3.2.0 (2015-04-16)
* Platform: x86_64-pc-linux-gnu (64-bit), Linux Mint LMDE
