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
    library(lattice)

Reading in the dataset, creating the "raw" file, then pulling out the "NA"
--------------------------------------------------------------------------

    raw_file <- read.csv("activity.csv")
    data <- na.omit(raw_file)

Daily descriptive statistics for step count
-------------------------------------------

    daily_sum <- aggregate(steps ~ date, data, sum)
    daily_mean <- mean(daily_sum$steps)
    daily_median <- median(daily_sum$steps)

The mean daily step count is 1.076618910^{4} and the daily median is
10765.

    hist(daily_sum$steps, main = "Total Steps Each Day", xlab = "Number of Steps", ylab = "Frequency", col = "orange")

![](PA1_template_files/figure-markdown_strict/daily%202-1.png)

    interval_average <- aggregate(steps ~ interval, data, mean)
    plot(x = interval_average$interval, y = interval_average$steps, xlab = "Interval"
         , ylab = "Average number of steps", main = "Average Daily Number of Steps by Interval", 
         type = "l")

![](PA1_template_files/figure-markdown_strict/daily%20pattern%201-1.png)

    fiveminmax <- max(interval_average$steps)
    maxinterval <- interval_average[interval_average$steps==fiveminmax,1]

The highest average step count for an interval was 206.1698113, which
was during the 835 interval.

How to handle missing values
----------------------------

    number_na <- sum(is.na(raw_file$steps))

Total number of missing values in the dataset is 2304.

    na_data <- raw_file[is.na(raw_file$steps),]
    non_na_data <- merge(na_data, interval_average, by = "interval")
    non_na_data <- non_na_data[,c(4,3,1)]
    colnames(non_na_data) <- c("steps", "date", "interval")
    complete_data <- rbind(data, non_na_data)

Missing values were replaced with the average for that interval across
the entire sampling period.

    missingno_sum <- aggregate(steps ~ date, complete_data, sum)
    missingno_mean <- mean(missingno_sum$steps)
    missingno_median <- median(missingno_sum$steps)
    hist(missingno_sum$steps, main = "Total Steps Each Day Including Missing Values",xlab = "Number of Steps", ylab = "Frequency", col = "orange")

![](PA1_template_files/figure-markdown_strict/missingno%204-1.png)

    complete_sum <- sum(missingno_sum$steps) 
    cleaned_sum <- sum(daily_sum$steps)

After imputing missing values, the new mean daily step count is
1.076618910^{4} and the daily median is 1.076618910^{4} The values do
not differ from the previous estimates - 1.076618910^{4} and 10765 as
the previous mean and median estimate, respectively, because I replaced
missing values with the mean for that interval, so I wouldn't expect any
differences in the basic descriptive statistics. Imputing missing data
greatly increased the total daily number of steps: the imputed total
(570608) is quite a bit larger than the pre-replacement total
(6.567375110^{5})

weekday vs weekend?
-------------------

    complete_data$week <- as.factor(weekdays(as.Date(complete_data$date)))
    final_dataset <- complete_data
    final_dataset$daytype <- as.factor(ifelse(final_dataset$week %in% c("Saturday", "Sunday"), "Weekend", "Weekday"))

    final_dataset_table <- aggregate(steps ~ interval + daytype, final_dataset, mean)
    with(final_dataset_table, xyplot(steps ~ interval|daytype, main = "Average Steps per Day by Interval for Weekend vs Weekdays", xlab = "Interval", ylab = "Steps", 
                                     layout = c(1, 2),type = "l"))

![](PA1_template_files/figure-markdown_strict/week%202-1.png)

    ""

    ## [1] ""

The activity pattern for weekdays compared to weekends are noticably
different. Weekdays tend to start and end earlier, with fewer steps
taken overall.
