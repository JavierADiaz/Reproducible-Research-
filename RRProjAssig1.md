Reproducible Research Project1
================
Javier A Diaz
26 de junio de 2018

Introduction
------------

This is a project of Reproducible Research with the objective of data analysis of information of steps done by a volunteer during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day. The data was provided from Activity monitoring data set.

Data adquisition and processiong
--------------------------------

Read data

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(ggplot2)
act<-read.csv("activity.csv")
# Number of days
numd<-unique(act$date)
```

Mean total number of steps taken per day
----------------------------------------

-   The total number of steps taken per day in variable actd is calculated.
-   A histogram of the total number of steps taken each day is made.
    -The mean and median of the total number of steps taken per day is calculated.

``` r
actd<-as.data.frame(matrix(numd,length(numd),2))
colnames(actd)<- c("date","steps")
# na.rm is TRUE to ignor NA's, otherwise NA is returned. 
actd[,2]<-as.integer(tapply(act$steps,act$date,sum,na.rm=TRUE))
qplot(steps,data=actd,binwidth=1000,na.rm=TRUE  )
```

![](RRProjAssig1_files/figure-markdown_github/Histogram%20steps%20per%20day-1.png)

``` r
#Total number of steps per day is the mean per day of actd. Again NA'S are ignored
meand<-mean(actd$steps,na.rm=TRUE)
meand
```

    ## [1] 9354.23

``` r
mediand<-median(actd$steps,na.rm=TRUE)
mediand
```

    ## [1] 10395

-   The mean total number of steps per day is 9354.23 and the median of the total of steps per day is 10395

Average daily activity pattern
------------------------------

-   A time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis) is made.
-   The 5-minute interval, on average across all the days in the data set, which contains the maximum number of steps is identified

``` r
numIn<-(unique(act$interval))
numIn<-numIn-as.integer(numIn/100)*40
#numIn<-ymd("2012-10-02")+hours(as.integer(numIn/100))+
#             minutes(numIn-as.integer(numIn/100)*100)


actIn<-as.data.frame(matrix(numIn,length(numIn),2))
colnames(actIn)<- c("Interval","steps")
actIn[,2]<-tapply(act$steps,act$interval ,mean,na.rm=TRUE)

qplot(Interval,steps,data=actIn,geom=c("line"))
```

![](RRProjAssig1_files/figure-markdown_github/Steps%20during%20the%20day-1.png)

-   The mean steps per day and
-   The 5-minute interval, on average across all the days in the data set, contains the maximum number of steps are calculated.

``` r
meanIn<-mean(actIn$steps)
Max<-actIn[which(actIn$steps==max(actIn$steps),arr.ind=TRUE),]
meanIn
```

    ## [1] 37.3826

``` r
actIn[as.numeric(row.names (Max)),]
```

    ##     Interval    steps
    ## 104      515 206.1698

The maximum is in the interval 104 with a value of 206.17 steps

Imputing missing values
-----------------------

The presence of missing days may introduce bias into some calculations or summaries of the data.

Calculate and report the total number of missing values in the data set (i.e. the total number of rows with N As)

``` r
missteps<-which(is.na(act$steps),arr.ind=TRUE)
length(missteps)
```

    ## [1] 2304

The total number of rows with N As is 2304

The mean for that 5-minute interval is the strategy for filling in all of the missing values in the data set. A new data set that is equal to the original data set but with the missing data filled in is created.

``` r
# The na's are filled with the mean of all days for each interval 
act2<-act
for(i in missteps){
  nuin<-act2[i,3]
  act2[i,1]<-
    actIn[1+(nuin-as.integer(nuin/100)*40)/5,2]}
# Add one to consider  first intervale starts in 0 min.
# 40 is to consider 60 min per hour and 
# 5 is intervale duration.
anyNA(act2) # To check if there is any NA
```

    ## [1] FALSE

A histogram of the total number of steps taken each day is made and the mean and median total number of steps taken per day are calculated and reported.

``` r
actd2<-as.data.frame(matrix(numd,length(numd),2))
colnames(actd2)<- c("date","steps")
actd2[,2]<-as.integer(tapply(act2$steps,act2$date,sum))

qplot(steps,data=actd2,binwidth=1000  )
```

![](RRProjAssig1_files/figure-markdown_github/Histogram%20NAs%20filled-1.png)

``` r
meand2<-round(mean(actd2$steps),2)
meand2
```

    ## [1] 10766.16

``` r
mediand2<-median(actd2$steps)
mediand2
```

    ## [1] 10766

Mean and median,1.07661610^{4}, 10766, do differ from the estimates from the first part of the assignment 9354.23, 10395
The impact of imputing missing data on the estimates of the total daily number of steps is an increment of values. Since we are assuming a positive value instead of zero .

Activity patterns between weekdays and weekends
-----------------------------------------------

The data set with the filled-in missing values is used for this part. A new factor variable in the data set is created with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

``` r
library (dplyr)
## start with all days aas weekdays and then change only the weekends
act2<-mutate(act2, tyday = "weekday")

weeke<-which(weekdays(as.Date(act2$date))=="sábado"|
               weekdays(as.Date(act2$date))=="domingo")
act2[weeke,4]<-"weekend"
act2$tyday<-as.factor(act2$tyday)
```

Panel plot is made containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

``` r
act2w<-data.frame(numIn,length(numIn),3)
names(act2w)<-c("Interval", "steps", "tyday")
act2wd<-data.frame(numIn,length(numIn),3)
names(act2wd)<-c("Interval", "steps", "tyday")

act3<-split(act2,act2$tyday)
act2w$steps<-tapply(act3$weekday$steps,act3$weekday$interval,mean)
act2wd$steps<-tapply(act3$weekend$steps,act3$weekend$interval,mean)

act2w$tyday<-"weekday"
act2wd$tyday<-"weekend"
act2w<-rbind(act2w,act2wd)

qplot(Interval,steps,data=act2w,facets=tyday~.,geom=c("line"))
```

![](RRProjAssig1_files/figure-markdown_github/unnamed-chunk-5-1.png)

Conclusions
-----------

-   N As could bias results if they are ignored.
-   Considering and imputing of missing values with the average for each interval produces an increment on mean and median with respect to ignore ring the N As.
-   there is a different activity pattern for weekdays and weekends. although the pattern is similar but intensity of activity is higher in weekdays.
