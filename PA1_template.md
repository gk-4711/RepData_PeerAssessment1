========================================

Loading and preprocessing the data
----------------------------------

    unzip(zipfile="activity.zip")

    ## Warning in unzip(zipfile = "activity.zip"): error 1 in extracting from zip file

    data<- read.csv("activity.csv")

What is mean total number of steps taken per day?
-------------------------------------------------

    library(ggplot2)
    total.steps<-data.frame(tapply(data$steps,data$date,sum,na.rm=T))
    total.steps$date<- rownames(total.steps)
    colnames(total.steps)<- c("Steps","Date")
    ggplot(total.steps,aes(x=Steps)) + geom_histogram(fill="steelblue",binwidth = 1000) + labs(title="Histogram of the total number of steps taken each day",x="Steps",y="Frequency")

![](PA1_template_files/figure-markdown_strict/histogram-1.png)

    mea<- mean(total.steps$Steps,na.rm=T)
    med<- median(total.steps$Steps,na.rm=T)

Mean : 9354.2295082 Median : 10395

What is the average daily activity pattern?
-------------------------------------------

    library(ggplot2)
    steps.interval<- data.frame(tapply(data$steps,data$interval,mean,na.rm=T))
    steps.interval$interval<- rownames(steps.interval)
    colnames(steps.interval)<- c("Steps","Interval")
    ggplot(steps.interval,aes(x=as.numeric(Interval),y=Steps)) + geom_line(color="red") + labs(title="Time series plot of the average number of steps taken",x="5 Minute Intervals",y="Steps")

![](PA1_template_files/figure-markdown_strict/time_plot-1.png)

    maxInterval<- steps.interval[which.max(steps.interval$Steps),]

The 5-minute interval that, on average, contains the maximum number of
steps : 206.1698113, 835

Imputing missing values
-----------------------

    missing<- is.na(data$steps)
    tab<-table(missing)

15264, 2304

All of the missing values are filled in with mean value for that
5-minute interval.

    fill.data<- function(steps,interval){
        filled<-NA
        if(!is.na(steps))
            filled<-c(steps)
        else
            filled<- (steps.interval[steps.interval$Interval==interval,"Steps"])
        return(filled)
    }
    filled.data<- data
    filled.data$steps<- mapply(fill.data,filled.data$steps,filled.data$interval)

Histogram of the total number of steps taken each day after missing
values are imputed

    new.total.steps<- data.frame(tapply(filled.data$steps,filled.data$date,sum,na.rm=T))
    new.total.steps$date<-rownames(new.total.steps)
    colnames(new.total.steps)<- c("Steps","Date")
    ggplot(new.total.steps,aes(x=Steps)) + geom_histogram(fill="green",binwidth = 1000) + labs(x="Steps",y="Frequency",title="Histogram")

![](PA1_template_files/figure-markdown_strict/new_data_hist-1.png)

Calculate mean and median of the new modified data.

    mea1<- mean(new.total.steps$Steps)
    med1<- median(new.total.steps$Steps)

Old Mean : 9354.2295082 Old Median : 10395

New Mean : 1.076618910^{4} New Median : 1.076618910^{4}

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

    filled.data$datetype<- ifelse(as.POSIXlt(filled.data$date)$wday %in% c(0,6),"weekend","weekday")

Now, let’s make a panel plot containing plots of average number of steps
taken on weekdays and weekends.

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    day.data<- filled.data %>% group_by(datetype,interval) %>% summarize(totalSteps=mean(steps))
    ggplot(day.data,aes(x=interval,y=totalSteps,color=datetype)) + geom_line() + facet_grid(datetype~.) + labs(x="5 Minute Interval",y="Steps",title="Average steps taken in Weekdays/Weekend")

![](PA1_template_files/figure-markdown_strict/panel_plot-1.png)
