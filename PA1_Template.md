Setting global options
----------------------

    knitr::opts_chunk$set(echo = TRUE)

Calling required libraries
--------------------------

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.3.2

    library(lubridate)

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

    library(lattice)
    library(knitr)

Reading data to R
-----------------

    data <- read.csv("./activity.csv")
    df <- tbl_df(data)
    df$date <- as.Date(df$date)

Calculating the summary and making a histogram
----------------------------------------------

    s <- df %>% group_by(date) %>% summarize(daily_steps = sum(steps, na.rm = TRUE))
    mean <- mean(s$daily_steps)
    median <- median(s$daily_steps)
    hist(s$daily_steps, breaks = 20, main = "Histogram of Steps", xlab = "Daily Steps")

![](PA1_Template_files/figure-markdown_strict/summary-1.png)

The total number of steps taken each day

-   Mean 9354
-   Median 10395

Daily walking pattern
---------------------

    p <- df %>% group_by(interval) %>% summarize(steps = mean(steps, na.rm = TRUE))
    p$minute <- p$interval %% 100 
    p$hour <- p$interval %/% 100
    p$time <- hm(paste(p$hour,p$minute, sep = ":"))
    p$time <- as_date(p$time)
    plot(p$time, p$steps, type = "l" )

![](PA1_Template_files/figure-markdown_strict/pattern-1.png)

    max <- p[which(p$steps == max(p$steps)),]$time
    hour <- hour(max)
    minute <- minute(max)
    time <- sprintf("%s:%s",hour, minute)

The interval of the day when the person is most active in walking is
morning time 8:35

Imputing missing values
-----------------------

We input missing values using the mean values of each interval

    miss <- sum(is.na(df$steps))
    impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))
    dk <- df  %>% mutate(steps = impute.mean(steps))

Now when we calculated the missing values it is time to see how it
changed the median and mean values of the steps taken each day.

    sk <- dk %>% group_by(date) %>% 
            summarize(daily_steps = sum(steps, na.rm = TRUE))
    mean_f <- mean(sk$daily_steps)
    median_f <- median(sk$daily_steps)
    hist(s$daily_steps, breaks = 20, main = "Histogram of Steps", xlab = "Daily Steps")

![](PA1_Template_files/figure-markdown_strict/summary_full-1.png)

The total number of steps taken each day when missing values are filled

-   Mean 10766
-   Median 10766

Looking at weekend effect
-------------------------

At weekend people wake up later and their walking activity is much
dispersed throughout a day

    df <- df %>% 
            mutate(weekday = ifelse(weekdays(date) 
                    %in% c("Saturday", "Sunday"), "Weekend", "Weekday"))
    p <- df %>% group_by(interval, weekday) %>% 
            summarize(steps = mean(steps, na.rm = TRUE))
    xyplot(steps ~ interval | weekday, p, type = 'l', layout = c(1,2))

![](PA1_Template_files/figure-markdown_strict/weekend-1.png)
