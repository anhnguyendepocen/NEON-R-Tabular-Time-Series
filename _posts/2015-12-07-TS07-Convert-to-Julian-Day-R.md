---
layout: post
title: "Lesson 07: Converting to Julian Day"
date:   2015-10-18
authors: [Megan A. Jones, Marisa Guarinello, Courtney Soderberg]
contributors: 
dateCreated: 2015-10-18
lastModified: 2015-12-07
tags: [module-1]
packagesLibraries: lubridate
category: 
description: "This lesson explains why Julian days are useful and teaches how
to create a Julian day variable from a Date or DataTime class variable."
code1:
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Julian-Dates
comments: false
---

{% include _toc.html %}

##About
This lesson explains why Julian days are useful and teaches how to create a
Julian day variable from a Date or DataTime class variable.

**R Skill Level:** Intermediate - you've got the basics of `R` down.

<div id="objectives" markdown="1">

### Goals / Objectives
After completing this activity, you will:

 * Be able to convert a Date or Date/Time class variable to a Julian day variable.

###Things You'll Need To Complete This Lesson
Please be sure you have the most current version of `R` and, preferably
RStudio, to write your code.

####R Libraries to Install
<li><strong>lubridate:</strong> <code> install.packages("lubridate")</code></li>

####Data to Download
{% include _tabular-time-series-data.html %}

####Setting the Working Directory
The code in this lesson assumes that you have set your working directory to the
location of the unzipped file of data downloaded above.  If you would like a
refresher on setting the working directory, please view the [Setting A Working Directory In R](URL "R Working Directory Lesson") lesson prior to beginning
this lesson.

</div>

##Converting Between Time Formats - Julian Days
Julian days, as R interprets it, is a continuous count of the number of days 
beginning at Jan 1, each year. Thus each year will have up to 365 (non-leap year)
or 366 (leap year) days. 

Note: This format can also be called ordinal day or year day, and, ocassionally,
Julian day can refer to a numeric count since 1 January 4713 BCE instead of a
yearly count of 365 or 366 days.
{ : .notice1 }

Including a Julian day variable in your data set can be very useful when
comparing data across years, when plotting data, and when matching your data
with other types of data that include Julian day.  


    # Load packages required for entire script
    library(lubridate)  #work with dates
    
    #set working directory to ensure R can find the file we wish to import
    #setwd("working-dir-path-here")
    
    #Load csv file of daily meterological data from Harvard Forest
    #Factors=FALSE so strings, series of letters/ words/ numerals, remain characters
    harMet_DailyNoJD <- read.csv(file="AtmosData/HARV/hf001-06-daily-m-NoJD.csv",
                         stringsAsFactors = FALSE)
    
    #what data class is the date column? 
    str(harMet_DailyNoJD$date)

    ##  chr [1:5345] "2/11/01" "2/12/01" "2/13/01" "2/14/01" ...

    #convert "date" from chr to a Date class and specify current date format
    harMet_DailyNoJD$date<- as.Date(harMet_DailyNoJD$date, "%m/%d/%y")

To quickly convert from from Date to Julian days, can we use `yday`, a 
function from the `lubridate` package. 


    # to learn more about it type
    ?yday

We want to create a new column in the existing data frame, titled `julian`, that
contains the Julian day data.  


    harMet_DailyNoJD$julian <- yday(harMet_DailyNoJD$date)  
    #make sure it worked all the way through. 
    head(harMet_DailyNoJD$julian) 

    ## [1] 42 43 44 45 46 47

    tail(harMet_DailyNoJD$julian)

    ## [1] 268 269 270 271 272 273

In this lesson we converted from `Date` class to a julian day, however, you can
convert between any recocognized date/time class (POSIXct, POSIXlt, etc) and
Julian day using `yday`.  

