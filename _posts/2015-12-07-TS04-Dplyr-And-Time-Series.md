---
layout: post
title: "Lesson 04: Manipulate and Subset Time Series Data with dplyr"
date:   2015-10-21
authors: [Megan A. Jones, Marisa Guarinello, Courtney Soderberg]
contributors: [Leah A. Wasser, Michael Patterson]
dateCreated: 2015-10-22
lastModified: 2015-12-07
tags: [module-1]
packagesLibraries: [lubridate, ggplot2, scales, gridExtra, dplyr]
category: 
description: "This lesson teaches how to conduct basic data manipulation using
`dplyr` functions including `group_by`, `summarize`and `mutate`. The use of pipes
is taught and used throughout the lesson. Finally the new data frames created in
the previous manipulations will be plotted using `ggplot()`. "
code1:
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Plotting-Time-Series
comments: false
---

{% include _toc.html %}


##About
This lesson teaches how to conduct basic data manipulation using `dplyr` 
functions including `groupby`, `summarize`and `mutate`. The use of pipes is 
taught and used throughout the lesson. 

**R Skill Level:** Intermediate - you've got the basics of `R` down.


<div id="objectives" markdown="1">

### Goals / Objectives
After completing this lesson, you will:

 * Know several ways to manipulate data using functions in the `dplyr` package
 in `R`.
 * Be able to use `group-by()`, `summarize()`, and `mutate()` functions. 
 * Write and understand `R` code with pipes for cleaner, efficient coding.
 * Use the `year()` function from the `lubridate` package to extract year from a
 date-time class variable. 

###Things You'll Need To Complete This Lesson
Please be sure you have the most current version of `R` and, preferably,
RStudio to write your code.

####R Libraries to Install
<li><strong>lubridate:</strong> <code> install.packages("lubridate")</code></li>
<li><strong>dplyr:</strong> <code> install.packages("dplyr")</code></li>

  <a href="http://neondataskills.org/R/Packages-In-R/" target="_blank">
More on Packages in R - Adapted from Software Carpentry.</a>

####Data to Download
{% include _tabular-time-series-data.html %}

####Setting the Working Directory
The code in this lesson assumes that you have set your working directory to the
location of the unzipped file of data downloaded above.  If you would like a
refresher on setting the working directory, please view the [Setting A Working Directory In R](URL "R Working Directory Lesson") lesson prior to beginning
this lesson.

</div>

#Introduction to dplyr
Working with micro-meteorology data from Harvard Forest, we are going to
learn skills that will enable us to manipulating our data to group and summarize
by year, daily average, and julian date.  To do this, we will use the `dplyr`
package.

The `dplyr` package is designed to simplify more complicated data manipulations
in data frames.  `dplyr` is a powerful package that has capabilities beyond the 
simple data manipulations we will cover in this lesson, including manipulating 
data stored in an external database. This is especially useful to know about if 
you will be working with very large datasets or relational databases in the 
future. 

If you are interested in learning more about `dplyr` consider following up with 
`dplyr` lessons from 

1. NEON Data Skills on <a href="http://neondataskills.org/R/GREPL-Filter-Piping-in-DPLYR-Using-R/" target="_blank"> spatial data and piping with dplyr</a> or 
2. Data Carpentry on <a href="http://www.datacarpentry.org/R-ecology/04-dplyr.html" target="_blank">Aggregating and Analyzing Data with dplyr</a> 

and reading the CRAN `dplyr` <a href="https://cran.r-project.org/web/packages/dplyr/dplyr.pdf" target="_blank"> package description</a>.

##Load the Data
To complete this lesson, we will need the 15-minute micro-meteorology data for
2009-2011 from the Harvard Forest. If you do not have this data set as R object
created in [Lesson 03: Refining Time Series Data](URL "2009-2011 HarMet Data Subset")
please load it from the .csv file in the downloaded data and convert the 
date-time column to a POSIXct class variable.  


    #Remember it is good coding technique to add additional packages to the top of
      #your script 
    library(lubridate) #for working with dates
    library(dplyr)    #for data manipulation (split, apply, combine)
    
    #set working directory to ensure R can find the file we wish to import
    #setwd("working-dir-path-here")
    
    #15-min Harvard Forest met data, 2009-2011
    harMet15.09.11<- read.csv(file="AtmosData/HARV/Met_HARV_15min_2009_2011.csv",
                              stringsAsFactors = FALSE)
    #convert datetime to POSIXct
    harMet15.09.11$datetime<-as.POSIXct(harMet15.09.11$datetime,
                        format = "%Y-%m-%d %H:%M",
                        tz = "America/New_York")

As we are interested in the annual patterns of air temperature, precipitation,
and PAR, we could simply create a figure of each of these variables at the
15-minute intervals that we have in our data.  
![ ]({{ site.baseurl }}/images/rfigs/TS04-Dplyr-And-Time-Series/15-min-plots-1.png) 

However, summarizing the data at
a coarser scale (e.g., daily, weekly, by season, or by year) may be easier to 
visually interpret or to compare with other data sets. 

As we learn how to use functions with in the `dplyr` package, our data goals are 
to create new variables based on aggregations of the existing data. These 
variable can then be used for other analyses or to create visually appealing and 
useful plots. 

#Manipulate Data using `dplyr`
For our data in the Harvard Forest, we want to be able to split our larger 
dataset into groups (e.g., by year), then manipulate each of the
smaller groups (e.g., take the average) before bringing them back together as a
whole (e.g., to plot the data). This is called using the "split-apply-combine"
technique, for which `dplyr` is particularly useful. 

While there are manipulations that are unique to `dplyr`, much of the benefit of
using the package is for more efficient and less verbose coding than using base
`R`.

`dplyr` works based on a series of *verb* functions that allow us to 
manipulate the data in different ways: 

 * `filter()` & `slice()`: filter rows based on values in specified columns
 * `group-by()`: group all data by a column
 * `arrange()`: sort data by values in specified columns 
 * `select()` & `rename()`: view and work with data from only specified columns
 * `distinct()`: view and work with only unique values from specified columns
 * `mutate()` & `transmute()`: add new data to a data frame
 * `summarise()`: calculate the specified summary statistics
 * `sample_n()` & `sample_frac()`: return a random sample of rows
 
(List modified from the CRAN `dplyr` <a href="https://cran.r-project.org/web/packages/dplyr/dplyr.pdf" target="_blank"> package description</a>. )

In this lesson we will use only some of these functions.  However, the syntax for
all `dplyr` functions is the same: 

 * the first argument is the target data frame, 
 * the subsequent arguments dictate what to do with that data frame and 
 * the output is a new data frame. 
 
Once the data frame is specified, column names can be directly referred to.  

##Use of Pipes
Pipes are an important tool in `dplyr` that allow 
us to skip creating and naming 
intermediate steps. They appear as `%>%` at the end of code lines.  Pipes,
like the name implies, allow us to take the output of one function and send it
(pipe it) directly to the next function. We don't have to save the intermediate
steps between functions. 


    harMet15.09.11 %>%      #Within the harMet15.09.11 data
      group_by(jd) %>%      #group the data by the Julian day
      tally()               #and tally how many observations per julian day

The above code essentially says: go into the harMet15.09.11 dataframe, find the 
Julian day column and group all data by each Julian day, then count (tally) how
many values for each of the grouped days.  

Bonus: Older `dplyr` versions coded pipes as %.% and you may still see that 
syntax in
some older StackOverflow answers and other tutorials. Pipes are ocassionally
referred to as chains. {: .notice2}

##Group by a Variable (or Two)
Getting back to our basic question of understanding annual phenology patterns,
we would like to look at the daily average temperature throughout the year. 
This means we need to calculate average temperature on a specific day for all
three years. To do this we can use the `group-by()` function. 

Before getting into our data, let's count up the number of observations per
Julian day (or year day) using the `group-by()` function as practice using the 
`dplyr` syntax and pipes. 


    harMet15.09.11 %>%      #Within the harMet15.09.11 data
      group_by(jd) %>%      #group the data by the Julian day
      tally()               #and tally how many observations per julian day

    ## Source: local data frame [366 x 2]
    ## 
    ##       jd     n
    ##    (int) (int)
    ## 1      1   288
    ## 2      2   288
    ## 3      3   288
    ## 4      4   288
    ## 5      5   288
    ## 6      6   288
    ## 7      7   288
    ## 8      8   288
    ## 9      9   288
    ## 10    10   288
    ## ..   ...   ...

As the output shows we have 288 values for each day.  Is that what we expect? 


    3*24*4  # 3 years * 24 hours/day * 4 15-min data points/hour

    ## [1] 288

Yep!  Looks good. 

Bonus: If Julian days weren't already in our dataset we could use the `yday()`
function <a href="http://www.inside-r.org/packages/cran/lubridate/docs/yday"/>
from the `lubridate` package to create a new column with this data. For 
details see [Lesson 07: Convert to Julian Day](URL "Julian Day lesson"). {: .notice2}

###Summarize within a Group
The `summarize()` function will collapse each group (e.g., Julian day) and
output the group value for whatever arguement (e.g., mean) we specify. 

Let's calculate the mean air temperature for each Julian day. Since we have a
few missing values we can add `na.rm=TRUE` to force R to ignore any NA values
when making the calculations. 


    harMet15.09.11 %>%
      group_by(jd) %>%
      summarize(mean_airt = mean(airt, na.rm = TRUE))  

    ## Source: local data frame [366 x 2]
    ## 
    ##       jd mean_airt
    ##    (int)     (dbl)
    ## 1      1 -3.672222
    ## 2      2 -3.145139
    ## 3      3 -6.812847
    ## 4      4 -5.780556
    ## 5      5 -4.217361
    ## 6      6 -6.406944
    ## 7      7 -4.398958
    ## 8      8 -5.293056
    ## 9      9 -8.268750
    ## 10    10 -9.787500
    ## ..   ...       ...

Julian days repeat 1-365 (or 366) each year, therefore what we have here is that
the mean temperature for January 1st across 2009, 2010, and 2011 was -3.7
Celsius. 

Sometimes we may want to summarize our data this way, other times we may want 
to summarize our air temperature data for each day in each of the three years.  
To do that we need to group our data by two different values at once, year and 
Julian day. 

###Extract Year from a Date-Time Column
Currently, our date and time information is in one column, `datetime`, but to 
group by year, we're first going to need to create a year-only column. For this 
we'll use the `lubridate` package. Since our date column is already a POSIXct 
date-time class, we can use the `year()` function. 


    harMet15.09.11$year <- year(harMet15.09.11$datetime)

Using `names()` we can see that we now have a `year` variable . 

    #check to make sure it worked
    names(harMet15.09.11)

    ##  [1] "X"        "datetime" "jd"       "airt"     "f.airt"   "rh"      
    ##  [7] "f.rh"     "dewp"     "f.dewp"   "prec"     "f.prec"   "slrr"    
    ## [13] "f.slrr"   "parr"     "f.parr"   "netr"     "f.netr"   "bar"     
    ## [19] "f.bar"    "wspd"     "f.wspd"   "wres"     "f.wres"   "wdir"    
    ## [25] "f.wdir"   "wdev"     "f.wdev"   "gspd"     "f.gspd"   "s10t"    
    ## [31] "f.s10t"   "year"

    str(harMet15.09.11$year)

    ##  num [1:105108] 2009 2009 2009 2009 2009 ...

Now we have our two variables: `jd` and `year`. To get the mean air temperature
for each day for each year we can use the `dplyr` pipes to group by year and
Julian day.


    harMet15.09.11 %>%
      group_by(year, jd) %>%
      summarize(mean_airt = mean(airt, na.rm = TRUE))

    ## Source: local data frame [1,096 x 3]
    ## Groups: year [?]
    ## 
    ##     year    jd  mean_airt
    ##    (dbl) (int)      (dbl)
    ## 1   2009     1 -15.128125
    ## 2   2009     2  -9.143750
    ## 3   2009     3  -5.539583
    ## 4   2009     4  -6.352083
    ## 5   2009     5  -2.414583
    ## 6   2009     6  -4.915625
    ## 7   2009     7  -2.590625
    ## 8   2009     8  -3.227083
    ## 9   2009     9  -9.915625
    ## 10  2009    10 -11.131250
    ## ..   ...   ...        ...

Given just the header in the output we can see the difference between the
3-year average temperature for 1 January (-3.7C, previous analysis) and the 
average temperature for 1 January 2009 (-15.1C, this analysis).

###Use Mutate & Combining Functions to Increase Efficiency
To create more efficient code, could we create the year variable within our
`dplyr` function call? 

Yes, we can use the `mutate()` function of `dplyr` and include our `lubridate`
`year()` function within the `mutate()` arguments. 

`mutate()` is used to add new data to a dataframe; often new data that
are created from a calculation or
manipulation of existing data. If you are familiar with the `transform()` base
`R` function the usage is similar, however,`mutate()` allows us to create and 
immediately use the variable (`year2`), something that `transform()` will not
do.


    harMet15.09.11 %>%
      mutate(year2 = year(datetime)) %>%
      group_by(year2, jd) %>%
      summarize(mean_airt = mean(airt, na.rm = TRUE))

    ## Source: local data frame [1,096 x 3]
    ## Groups: year2 [?]
    ## 
    ##    year2    jd  mean_airt
    ##    (dbl) (int)      (dbl)
    ## 1   2009     1 -15.128125
    ## 2   2009     2  -9.143750
    ## 3   2009     3  -5.539583
    ## 4   2009     4  -6.352083
    ## 5   2009     5  -2.414583
    ## 6   2009     6  -4.915625
    ## 7   2009     7  -2.590625
    ## 8   2009     8  -3.227083
    ## 9   2009     9  -9.915625
    ## 10  2009    10 -11.131250
    ## ..   ...   ...        ...

For illustration purposes, we named the new variable we just created using
`mutate()` `year2` so we could distinguish it from the `year` already created by
`year()`. Notice, below, that after using this code, we don't see `year2` as a 
column in our harMet15.09.11 dataframe yet we could see it the output. 


    names(harMet15.09.11)

    ##  [1] "X"        "datetime" "jd"       "airt"     "f.airt"   "rh"      
    ##  [7] "f.rh"     "dewp"     "f.dewp"   "prec"     "f.prec"   "slrr"    
    ## [13] "f.slrr"   "parr"     "f.parr"   "netr"     "f.netr"   "bar"     
    ## [19] "f.bar"    "wspd"     "f.wspd"   "wres"     "f.wres"   "wdir"    
    ## [25] "f.wdir"   "wdev"     "f.wdev"   "gspd"     "f.gspd"   "s10t"    
    ## [31] "f.s10t"   "year"

In order to save output from `mutate`, so we can later plot or use the data, we 
must create a new `R` object. 


    harTemp.daily.09.11<-harMet15.09.11 %>%
                        mutate(year2 = year(datetime)) %>%
                        group_by(year2, jd) %>%
                        summarize(mean_airt = mean(airt, na.rm = TRUE))
    
    head(harTemp.daily.09.11)

    ## Source: local data frame [6 x 3]
    ## Groups: year2 [1]
    ## 
    ##   year2    jd  mean_airt
    ##   (dbl) (int)      (dbl)
    ## 1  2009     1 -15.128125
    ## 2  2009     2  -9.143750
    ## 3  2009     3  -5.539583
    ## 4  2009     4  -6.352083
    ## 5  2009     5  -2.414583
    ## 6  2009     6  -4.915625

###Carry Date-Time Data with Mutated Data
Now that we have the `harTemp.daily.09.11` data frame we may want to plot the
air temperature throughtout the year.  But what would be our x-axis?  

To have a clear and nicely formatted x-axis we need to retain the date-time
information. But we only want one `datetime` entry per air temperature
measurement. We can get this by asking R to output the first datetime entry for
each day for each year using the `summarize()` function.


    harTemp.daily.09.11 <- harMet15.09.11 %>%
      mutate(year3 = year(datetime)) %>%
      group_by(year3, jd) %>%
      summarize(mean_airt = mean(airt, na.rm = TRUE), datetime = first(datetime))
    
    str(harTemp.daily.09.11)

    ## Classes 'grouped_df', 'tbl_df', 'tbl' and 'data.frame':	1096 obs. of  4 variables:
    ##  $ year3    : num  2009 2009 2009 2009 2009 ...
    ##  $ jd       : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ mean_airt: num  -15.13 -9.14 -5.54 -6.35 -2.41 ...
    ##  $ datetime : POSIXct, format: "2009-01-01 00:15:00" "2009-01-02 00:15:00" ...
    ##  - attr(*, "vars")=List of 1
    ##   ..$ : symbol year3
    ##  - attr(*, "drop")= logi TRUE

Now we have a data frame with only values for year, Julian day, mean air temp,
and a date that could easily be plotted to show the mean daily temperature 
during the study period. 


##Challenge: Applying dplyr Skills
Calculate and save a data frame of the average air temperate for each month in
each year.  Name the new data frame "HarTemp.monthly.09.11".


