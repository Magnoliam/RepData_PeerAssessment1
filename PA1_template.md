Loading the necessary packages
------------------------------

    library(knitr)
    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(lubridate)

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

    library(ggplot2)

Loading and preprocessing the data
----------------------------------

### Reading the data

    data <- read.csv("activity.csv", colClass=c('integer', 'Date', 'integer'))

### Tyding the data

Changing the date into dateformat using lubridate:

    data$date <- ymd(data$date)

What is mean total number of steps taken per day?
-------------------------------------------------

For this part of the assignment the missing values can be ignored. 1.
Calculate the total number of steps taken per day. 2. Make a histogram
of the total number of steps taken each day. 3. Calculate and report the
mean and median of the total number of steps taken per day.

### Methodology and Result

1.Calculate the total number of steps per day using dplyr and group by
date:

    steps <- data %>%
      filter(!is.na(steps)) %>%
      group_by(date) %>%
      summarize(steps = sum(steps)) %>%
      print

    ## # A tibble: 53 × 2
    ##          date steps
    ##        <date> <int>
    ## 1  2012-10-02   126
    ## 2  2012-10-03 11352
    ## 3  2012-10-04 12116
    ## 4  2012-10-05 13294
    ## 5  2012-10-06 15420
    ## 6  2012-10-07 11015
    ## 7  2012-10-09 12811
    ## 8  2012-10-10  9900
    ## 9  2012-10-11 10304
    ## 10 2012-10-12 17382
    ## # ... with 43 more rows

2.Use ggplot for making the histogram:

    ggplot(steps, aes(x = steps)) +
      geom_histogram(fill = "purple", binwidth = 1000) +
      labs(title = "Histogram of Steps per day", x = "Steps per day", y = "Frequency")

![](PA_files/figure-markdown_strict/unnamed-chunk-6-1.png)

3.Calculate the mean and median of the total number of steps taken per
day:

    mean_steps <- mean(steps$steps, na.rm = TRUE)
    median_steps <- median(steps$steps, na.rm = TRUE)

    mean_steps

    ## [1] 10766.19

    median_steps

    ## [1] 10765

Mean steps are 10766 and median steps are 10765.

What is the average daily activity pattern?
-------------------------------------------

1.  Make a time series plot (i.e. type = "l") of the 5-minute
    interval (x-axis) and the average number of steps taken, averaged
    across all days (y-axis).
2.  Which 5-minute interval, on average across all the days in the
    dataset, contains the maximum number of steps?.

### Methodology and Result

1.  Calculate the average number of steps taken in each 5-minute
    interval per day using dplyr and group by interval:

<!-- -->

    interval <- data %>%
      filter(!is.na(steps)) %>%
      group_by(interval) %>%
      summarize(steps = mean(steps))

Use ggplot for making the time series of the 5-minute interval and
average steps taken:

    ggplot(interval, aes(x=interval, y=steps)) +
      geom_line(color = "purple")

![](PA_files/figure-markdown_strict/unnamed-chunk-11-1.png)

1.  Use which.max() to find out the maximum steps, on average, across
    all the days:

<!-- -->

    interval[which.max(interval$steps),]

    ## # A tibble: 1 × 2
    ##   interval    steps
    ##      <int>    <dbl>
    ## 1      835 206.1698

The interval 835 has, on average, the highest count of steps, with 206
steps.

Imputing missing values
-----------------------

1.  Calculate and report the total number of missing values in the
    dataset (i.e. the total number of rows with NAs).
2.  Devise a strategy for filling in all of the missing values in
    the dataset. The strategy does not need to be sophisticated. For
    example, you could use the mean/median for that day, or the mean for
    that 5-minute interval, etc.
3.  Create a new dataset that is equal to the original dataset but with
    the missing data filled in.
4.  Make a histogram of the total number of steps taken each day and
    calculate and report the mean and median total number of steps taken
    per day. Do these values differ from the estimates from the first
    part of the assignment? What is the impact of imputing missing data
    on the estimates of the total daily number of steps?

### Methodology and Result

Summarize all the missing values:

    sum(is.na(data$steps))

    ## [1] 2304

Missing values are 2304.

1.  Taking the approach to fill in a missing NA with the average number
    of steps in the same 5-min interval.
2.  Creating a new dataset as the original and fill in the missing
    values with the average number of steps per 5-minute interval:

<!-- -->

    data_full <- data
    nas <- is.na(data_full$steps)
    avg_interval <- tapply(data_full$steps, data_full$interval, mean, na.rm=TRUE, simplify=TRUE)
    data_full$steps[nas] <- avg_interval[as.character(data_full$interval[nas])]

Check that there are no missing values:

    sum(is.na(data_full$steps))

    ## [1] 0

No more missing values.

1.  Calculating the number of steps taken in each 5-minute interval per
    day using dplyr and group by interval. Using ggplot for making the
    histogram:

<!-- -->

    steps_full <- data_full %>%
      filter(!is.na(steps)) %>%
      group_by(date) %>%
      summarize(steps = sum(steps)) %>%
      print

    ## # A tibble: 61 × 2
    ##          date    steps
    ##        <date>    <dbl>
    ## 1  2012-10-01 10766.19
    ## 2  2012-10-02   126.00
    ## 3  2012-10-03 11352.00
    ## 4  2012-10-04 12116.00
    ## 5  2012-10-05 13294.00
    ## 6  2012-10-06 15420.00
    ## 7  2012-10-07 11015.00
    ## 8  2012-10-08 10766.19
    ## 9  2012-10-09 12811.00
    ## 10 2012-10-10  9900.00
    ## # ... with 51 more rows

    ggplot(steps_full, aes(x = steps)) +
      geom_histogram(fill = "purple", binwidth = 1000) +
      labs(title = "Histogram of Steps per day, including missing values", x = "Steps per day", y = "Frequency")

![](PA_files/figure-markdown_strict/unnamed-chunk-17-1.png)

Calculating the mean and median steps with the filled in values:

    mean_steps_full <- mean(steps_full$steps, na.rm = TRUE)
    median_steps_full <- median(steps_full$steps, na.rm = TRUE)

    mean_steps_full

    ## [1] 10766.19

    median_steps_full

    ## [1] 10766.19

The impact of imputing missing data with the average number of steps in
the same 5-min interval is that both the mean and the median are equal
to the same value: 10766.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

1.  Creating a new factor variable in the dataset with two levels -
    "weekday" and "weekend" indicating whether a given date is a weekday
    or weekend day.
2.  Making a panel plot containing a time series plot (i.e. type = "l")
    of the 5-minute interval (x-axis) and the average number of steps
    taken, averaged across all weekday days or weekend days (y-axis).

### Methodology and Result

Using dplyr and mutate to create a new column, weektype, and apply
whether the day is weekend or weekday:

    data_full <- mutate(data_full, weektype = ifelse(weekdays(data_full$date) == "Saturday" | weekdays(data_full$date) == "Sunday", "weekend", "weekday"))
    data_full$weektype <- as.factor(data_full$weektype)
    head(data_full)

    ##       steps       date interval weektype
    ## 1 1.7169811 2012-10-01        0  weekday
    ## 2 0.3396226 2012-10-01        5  weekday
    ## 3 0.1320755 2012-10-01       10  weekday
    ## 4 0.1509434 2012-10-01       15  weekday
    ## 5 0.0754717 2012-10-01       20  weekday
    ## 6 2.0943396 2012-10-01       25  weekday

1.  Calculate the average steps in the 5-minute interval and use ggplot
    for making the time series of the 5-minute interval for weekday and
    weekend, and compare the average steps:

<!-- -->

    interval_full <- data_full %>%
      group_by(interval, weektype) %>%
      summarise(steps = mean(steps))
    s <- ggplot(interval_full, aes(x=interval, y=steps, color = weektype)) +
      geom_line() +
      facet_wrap(~weektype, ncol = 1, nrow=2)
    print(s)

![](PA_files/figure-markdown_strict/unnamed-chunk-22-1.png)

From the two plots it seems that the test object is more active earlier
in the day during weekdays compared to weekends, but more active
throughout the weekends compared with weekdays (probably because the
oject is working during the weekdays, hence moving less during the day).
