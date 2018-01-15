---
layout: post
title:  "Simple Excel Data Wrangling with R"
date: 2015-04-09
excerpt: "Today we are going to deal with some Microsoft Excel spreadsheet data in R."
image: "/images/excelwrangling1.jpg"
permalink: /blog/2015/04/09/Simple-Spreadsheet-Data-Wrangling-With-R
---




Today we are going to deal with some Microsoft Excel spreadsheet data in R. Excel spreadsheets are used everywhere by a variety of companies and employees from diverse areas with varied skill sets. We often have to deal with sparse and messy excel spreadsheets that contain some useful information we want to preferably extract with some code so we can then plug it in R to  create more robust statistical analysis and visualisation.

In today's example, we'll work with a couple of spreadsheets from the Employee Earnings and Hours survey conducted by the Australian Bureau of Statistics.

This survey is conducted every two years and designed to provide detailed statistics on the composition and distribution of employee earnings, hours paid for and the methods used to set employees' pay. Information is collected from a sample of employers about characteristics of both the employers (such as industry and sector) and their employees (such as occupation, type of employee, and method of setting pay). This information is used to provide comprehensive statistics about earnings and hours paid for, for various groups of employees, classified by for example industry, occupation or pay setting method.

For more information on the survey methodology and scope, see:  [Employee Earnings and Hours survey for 2014](http://www.abs.gov.au/AUSSTATS/abs@.nsf/Lookup/6306.0Explanatory%20Notes1May%202014?OpenDocument) and [Employee Earnings and Hours survey for 2012.](http://www.abs.gov.au/AUSSTATS/abs@.nsf/allprimarymainfeatures/46CA336671A196E4CA257DD400759253?opendocument)

We won't use the full range of data available in the survey and the idea here is not so much t analyse the survey itself but to demonstrate a way we can use R to wrangle data. We'll create a simple example by downloading the survey's results from 2014 and 2012 to have a comparative view. We'll go through some data munging and wrangling with R to set the data in the shape we need. We'll then go on to plug the data into [Tableau](http://public.tableau.com/s/) for some interactive visualisation.

The first step is to download the data. For this example we'll download [this file for 2014 data](http://www.abs.gov.au/ausstats/subscriber.nsf/log?openagent&63060do012_201405.xls&6306.0&Data%20Cubes&BC612754F1F40239CA257DD4007590FF&0&May%202014&22.01.2015&Latest) and [this file for 2012 data.](http://www.abs.gov.au/ausstats/subscriber.nsf/log?openagent&63060do008_201205.xls&6306.0&Data%20Cubes&CFDD56EAA013C56FCA257AFB000E41A9&0&May%202012&23.01.2013&Latest)

We set our working directory to where we saved our files (I created a folder called ABS_EarningsSurvey and saved the  files I downloaded in there) and load the required libraries. We'll use the package **XLConnect** to load excel spreadsheets into R with ease. You can find this package's full documentation [here.](http://cran.r-project.org/web/packages/XLConnect/XLConnect.pdf) We'll also use the packages **dplyr** and **reshape2**.

```r
# set working directory
setwd('~/ABS_EarningsSurvey')
```

``` r
# load required libraries
library(XLConnectJars)
library(dplyr)
library(reshape2)
```

The next step here is to load the files saved:

``` r 
# Load excel files downloaded
file2012 = readWorksheetFromFile('2012_EarningsSurveyByOccupation.xls',
                                 sheet='Table_1', region='A08:F360', 
                                 header=FALSE, keep=c(1,6))
names(file2012) = c('UnitGroupDescr', '2012')
file2012[is.na(file2012)] = 0

file2014 = readWorksheetFromFile('2014_EarningsSurveyByOccupation.xls',
                                 sheet='Table_1', region='A08:G363', 
                                 header=FALSE, keep=c(1,4))
names(file2014) = c('UnitGroupDescr', '2014')
file2014[is.na(file2014)] = 0
```

Note we call one file at a time. With *XLConnect*  we can ask for a specific sheet and the region (or range) in the worksheet we want the information from. In our code above, we will only bring data from cell A08 to cell F360. We also call a function to keep only specfic columns (columns 1 and 6 for file 2012 for example) from the results. These are the columns with want to keep in our data frame so by passing *keep* parameter we only retain the columns we'll use and the rest is discarded.

Note also that after we load the files, we rename the column names and also replace NAs with zeros.

Next step is to consolidate both files by merging them using the **merge** function from R base:

``` r
# Consolidate both files in one master file
consolFile = merge(file2012, file2014, by='UnitGroupDescr', 
					all.x=TRUE, all.y=TRUE)
consolFile[is.na(consolFile)] = 0
```

By passing **TRUE** in both *all.x* and *all.y* in the merge function, we are telling it to bring values of UnitGroupDescr from both the file2012 and file2014 and append them in the consolidated file without discarding anything. If you are familiar with SQL, it is kind of asking for a full outer join (bring everything avaialble from both tables!). It brings all values we are merging on from both files without constraining or anchoring on one particular file.

We then go on to create a new varaible in our consolidated data frame called UnitGroup. This new variable is essentially an offspring of the UnitGroupDescr variable,  bringing its first 4 digits. Next, we create a MinorGroup variable whic is essentially the first 3 digits fo the newly created UnitGroup variable.

Why are we creating these you may ask?

Well, these two files we loaded from the ABS website contain only UnitGroupDescr level data (4 digits and the occupation official name). Let's suppose now that you have another file you found that contains some higher level information about the occupations such as the occupation group and you wanted to somehow merge these two different sets of information in one file.

Let's demonstrate what we could do in this case. Let's load the newly found file that contains the occupation group level information we want to incorporate into the files. Let's call this new variable MinorGroup. The file that follows can be found [here.](http://www.abs.gov.au/AUSSTATS/subscriber.nsf/log?openagent&12200%20anzsco%20version%201.2%20-structure%20v2.xls&1220.0&Data%20Cubes&6A8A6C9AC322D9ABCA257B9E0011956C&0&2013,%20Version%201.2&05.07.2013&Latest)

``` r
anzsco = readWorksheetFromFile('ANZSCO_Class.xls',
  sheet='Table 4', region='C12:D522', header=FALSE)

anzsco = subset(anzsco, is.na(as.numeric(anzsco$Col1)) == FALSE )
names(anzsco) = c('MinorGroup', 'MinorGroupDescr')
```
Note from the above that we load the file we downloaded from the ABS website, store it in an R object called *anzsco*, specify a certain range in the worksheet and bring only the rows that contain numbers in Column1. We then finally rename the columns.

If you pay attention to how the spreadhseet looks like, you'll see how interestingly structured the data is. The data is in this crazy looking hierarchical form. But all we're interested is in column's C and D data points. But because the way the file is structured, we have to specify what sort of data in that range we want. We don;t care about nulls, NAs and non-nmeric data in column C. So we go on and tell subset to only bring the rows that have numeric data and is not NAs. We end up with a clean, structured file with only the information we want.

Next, we can create a final file that merges both the newly created anzsco data points and the both files from 2012 and 2014 data:

``` r
finalFile = merge(consolFile, anzsco, by='MinorGroup', all.x=TRUE, all.y=TRUE)
finalFile = finalFile[ ,c(1,6,5,2,3,4)]
finalFile = filter(finalFile, MinorGroupDescr != 'All occupations' )
finalFile = melt(finalFile,
  measure.vars=c('2012', '2014'),
	variable.name='Period', value.name='WeeklyEarnings')
finalFile$Period = paste0(finalFile$Period, '-01-01')
finalFile$Period = as.Date(finalFile$Period)
finalFile[is.na(finalFile)] = 0
```

The code chunk above contains the steps to to merge the files together, filter out unwanted data points, melt the final data to a long version, reset the date variable to a data format and finally replace NAs with zeros.

The final step is to simply save the file in a *.csv format to then load it into Tableau for some cool looking visualisation:

```r
write.csv(finalFile, 'WeeklyEarnings2012_2014ByOccupation.csv', row.names=FALSE)
```

You can see the product of this data wrangling work in a Tableau interactive dashboard [here.](http://feliperego.github.io/blog/2015/04/09/Employee-Average-Weekly-Earnings-Estimate/)


***

In this example we illustrated a few simple yet effective examples of dealing with somewhat unstructured spreadsheet data with R. We saw a few options to reshape and load excel spreadsheet data.
