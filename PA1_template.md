---
title: "Peer_Review"
author: "Rodney Waiters"
date: "September 22, 2016"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
## The Assignment
This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in a single R markdown document that can be processed by knitr and be transformed into an HTML file.

Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use echo = TRUE so that someone else will be able to read the code. This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.

For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the GitHub repository created for this assignment. You will submit this assignment by pushing your completed files into your forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.

NOTE: The GitHub repository also contains the dataset for the assignment so you do not have to download the data separately.
## Loading and preprocessing the data
```{r activity}
# Download the data
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", destfile = temp )
unzip(temp, "activity.csv")
data <- read.csv("activity.csv")
#Process/transform the data (if necessary) into a format suitable for your analysis
stepsbyday <- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)
```
## Make a histogram of steps taken each day
```{r stepsbyday}
hist(stepsbyday, breaks = 47,col = "blue",main = "Steps by Day")
```
```{r}
# Mean and median number of steps taken each day
mean(stepsbyday, na.rm = TRUE)
```
median(stepsbyday, na.rm = TRUE)
```
## Time Series
```{r cleandata}
# Time series plot of the average number of steps taken
cleandata <- na.omit(data)

cleandata$date <- as.Date(cleandata$date)

averageintervals <- aggregate(steps ~ interval, data=cleandata, FUN = mean)

plot(averageintervals$interval, averageintervals$steps, type = "l", xlab = "5-minute interval", ylab = "average number of steps taken")

# The 5-minute interval that, on average, contains the maximum number of steps
averageintervals[which.max(averageintervals$steps),]
```
## Imputing missing values
```{r baddata}
baddata <- is.na(data$steps)

table(baddata)

missValue <- function(steps, interval) {
    newvalue <- NA
    if (!is.na(steps))
        newvalue <- c(steps)
    else
        newvalue <- (averageintervals[averageintervals$interval==interval, "steps"])
    return(newvalue)
}
alldata <- data

alldata$steps <- mapply(missValue, alldata$steps, alldata$interval)

stepsbyday <- tapply(alldata$steps, alldata$date, FUN=sum)

hist(stepsbyday, breaks = 47,col = "red",main = "Steps by Day")
# Mean and median number of steps taken each day
mean(stepsbyday)

median(stepsbyday)
```
## Are there differences in activity patterns between weekdays and weekends?
```{r weekends}
library(ggplot2)

weeksplit <- function(date) {
    daytype <- weekdays(date)
            if (daytype %in% c("Saturday", "Sunday"))
                  return("Weekend")
            else 
                  return("Weekday")
   }
alldata$date <- as.Date(alldata$date)

alldata$daytype <- sapply(alldata$date, FUN=weeksplit)

head(alldata)

alldataavg <- aggregate(steps ~ interval + daytype, data=alldata, FUN =mean)

ggplot(alldataavg, aes(interval, steps)) + 
  geom_line() + facet_grid(daytype ~ .) +
  xlab("5 minute interval") +
  ylab("# of steps")

```

