---
title:    "<center> Reproducible Research </center> "
subtitle: "<center> JHU | Peer Assessment-1 </center> "
output:
  html_document:
    keep_md: true
  pdf_document: default
---
#### *Loading required libraries* 


```r
library(tidyverse)
library(ggplot2)
library(lubridate)
```

#### *Load and Process Assignment Data* 


```r
# Check if data is already unzipped and ready for analysis.
if (file.exists("activity.csv")) {
  
                  print("Activity Data Loaded")
                } else {
                  unzip("activity.zip")
                  print("Unzipped & Loaded Activity Data")
                }
```

```
## [1] "Activity Data Loaded"
```

```r
# Store the data & change the date's data type to date

devices_data  <-  read.csv("activity.csv", header = TRUE) 

devices_data$date <- as.Date(devices_data$date)
```

> ## Section I : What is mean total number of steps taken per day?

#### 1- Calculate the total number of steps taken per day


```r
# Group data by date,and summarize by the total daily steps.
                
DailySteps <- devices_data %>% 
               group_by(date) %>%
                summarize(daily_steps = sum(steps)) %>% na.omit()
```

#### 2-Histogram of the total number of steps taken each day


```r
# Histogram of total daily steps 

ggplot(data = DailySteps,
      aes(x = daily_steps)) +
      geom_histogram(binwidth = 2000, 
                      fill = "#E69F00",
                      color = "black"
                     ) +
                     xlab("Daily Steps")+
                     ylab("Frequency")+
                     labs(title = "Frequency of total daily steps")+
                     theme(plot.title = element_text(hjust = 0.5))  
```

<img src="PA1_template_files/figure-html/hist-1.png" style="display: block; margin: auto;" />

#### 3-Calculate and report the mean and median of the total number of steps taken per day

```r
# Mean steps per day
x1 <- mean(DailySteps$daily_steps)
paste0("mean total number of steps taken per day =  ",x1 )
```

```
## [1] "mean total number of steps taken per day =  10766.1886792453"
```

```r
# Median steps per day 
z1 <- median(DailySteps$daily_steps)
paste0("average number of steps taken per day =  ", z1)
```

```
## [1] "average number of steps taken per day =  10765"
```
> ### Section II : What is the average daily activity pattern?

#### 1-Plot shows the time of the day on X and the average daily on Y


```r
# Calculate average daily steps.
##Group data by interval, remove NAs, and summarize by the mean of daily steps.
                
StepsByInt <- devices_data %>% 
          na.omit() %>%
          group_by(interval) %>%
          summarize(daily_avge = mean(steps))
                                       
# Format the interval to 4 char to convert to time   
                                 
StepsByInt$interval <- sprintf("%04d", StepsByInt$interval)
                
# convert interval to time 
                
StepsByInt$interval <- as.character(format(
                                   strptime(StepsByInt$interval, format="%H%M")
                                   ,format = "%H:%M"
                                   )) %>%
                                    parse_time(StepsByInt$interval, format = "%H:%M")
                              

# Plot the data to show average daily activity pattern across 5 minutes intervals
# highlight the highest average 5 minutes.
                
# Define the highest average interval
max_row <- filter(StepsByInt, daily_avge == max(daily_avge))
max_avge_time <- max_row[1] ; max_avge <-  max_row[2]
                
# line chart 
               ggplot(StepsByInt,
                      aes(x=`interval`)) +
                      geom_line(aes(y=daily_avge))+
                      xlab("Time of the day(Interval)")+ylab("Daily Average")+
                      labs(title = "Average daily steps by 5 Minutes interval")+
                      theme(plot.title = element_text(hjust = 0.5))+
                      geom_point(data=filter(StepsByInt, daily_avge == max(daily_avge)),
                                 aes(x = `interval`, y = daily_avge), color = "#E69F00", size = 5) 
```

<img src="PA1_template_files/figure-html/avg_pattern-1.png" style="display: block; margin: auto;" />

#### 2-Interval Contains the maximum average steps.

```r
print(as.data.frame(max_row))
```

```
##   interval daily_avge
## 1 08:35:00   206.1698
```

> ### Section III: Imputing missing values

#### 1- Calculate and report the total number of missing values in the dataset

```r
NAs <- filter(devices_data, is.na(devices_data$steps))
paste0(nrow(NAs), " Missing Value")
```

```
## [1] "2304 Missing Value"
```

#### 2- Strategy to fill missing Value**


```r
#Replace NA with the mean
#Average steps by interval

avgByInt <- devices_data %>%
                na.omit() %>%
                group_by(interval) %>%
                summarise(avgsteps = mean(steps))


Rep_NA <- function(steps, interval) {
  IntSteps <- NA
  if (!is.na(steps)) {
    IntSteps = steps
  } else 
    IntSteps = round(subset(avgByInt$avgsteps, avgByInt$interval == interval),2)
  return(IntSteps)
}
```

#### 3- New dataset that equal to the original dataset  with the missing data filled

```r
filled_device_data <- devices_data

filled_device_data$steps <- mapply(Rep_NA, filled_device_data$steps, filled_device_data$interval)
```
#### 4- Histogram of total steps taken per day - calculate mean and median and compare to P1 of assignment


```r
# Total number of steps taken each day 

AvgByDay <- filled_device_data %>% group_by(date) %>% summarise(daily_steps = sum(steps))

# Mean steps per day

x2 <-  mean(AvgByDay$daily_steps)
paste0("mean total number of steps taken per day =  ", x2)
```

```
## [1] "mean total number of steps taken per day =  10766.1809836066"
```

```r
# Median steps per day 
z2 <- median(AvgByDay$daily_steps)
paste0("average number of steps taken per day =  ", z2)
```

```
## [1] "average number of steps taken per day =  10766.13"
```

```r
# Do these values differ from the estimates in P1
# Mean & Median changes | The dataset contains 61 days, 8 days with NA values across all intervals.
# First mean was based on 53 days only after omitting NAs.
# Replacing the NAs with the mean of the same interval resulted in a slight decrease in the mean and an increase in the median.

#mean difference 
x1 == x2 ; x2 - x1
```

```
## [1] FALSE
```

```
## [1] -0.007695639
```

```r
#median difference 
z1 == z2 ; z2 - z1
```

```
## [1] FALSE
```

```
## [1] 1.13
```

```r
# Histogram of total number steps per day

                ggplot(data = AvgByDay,
                      aes(x = daily_steps)) +
                      geom_histogram(binwidth = 2000,
                                     fill = "green",
                                     color = "black") +
                                     xlab("Daily Steps")+
                                     ylab("Frequency")+
                                     labs(title = "Frequency of total daily steps")+
                                     theme(plot.title = element_text(hjust = 0.5))
```

<img src="PA1_template_files/figure-html/hist2-1.png" style="display: block; margin: auto;" />

> ### Section IV : Differences in activity patterns between weekdays and weekends

#### 1- Create a new factor variable in the dataset with two levels (weekday , weekend)

```r
new_ds <- filled_device_data
new_ds$day <- as.factor(if_else(weekdays(new_ds$date) %in% c("Saturday", "Sunday"), "Weekend", "Weekday" ))

head(new_ds)
```

```
##   steps       date interval     day
## 1  1.72 2012-10-01        0 Weekday
## 2  0.34 2012-10-01        5 Weekday
## 3  0.13 2012-10-01       10 Weekday
## 4  0.15 2012-10-01       15 Weekday
## 5  0.08 2012-10-01       20 Weekday
## 6  2.09 2012-10-01       25 Weekday
```
#### 2- Panel plot 


```r
avgByIntByDay <- aggregate(steps ~ interval + day, mean, data = new_ds)

               ggplot(avgByIntByDay,
                      aes(x = interval, y = steps)) +
                      geom_line()+
                      facet_wrap(~ day
                                  ,ncol = 1)+
                                  theme_bw() +
                                  theme(strip.background = element_rect(fill = "#E69F00"))+
                                  theme(strip.text = element_text(color = "blue"))+
                      xlab("interval (5 Minutes)")+
                      ylab("Avergae steps")+
                      theme(panel.background = element_rect("lightblue")
                            ,panel.grid.major =  element_blank()
                            ,panel.grid.minor = element_blank())
```

<img src="PA1_template_files/figure-html/panelplot-1.png" style="display: block; margin: auto;" />
