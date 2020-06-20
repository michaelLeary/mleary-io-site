---
title: World steel production and capacity
author: michael
date: '2020-06-20'
slug: world-steel-production-and-capacity
categories:
  - R
tags:
  - Steel
---

Time series data: Steel production and steel capacity world (millions metric tons)

####
1. Source for production data: [World Steel Association](https://www.worldsteel.org/media-centre/press-releases/2020/worldsteel-short-range-outlook-june-2020.html)
1. Source for capacity data: [OECD](http://www.oecd.org/sti/ind/steelcapacity.htm)

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r pkgsused, include=FALSE}
pkgs = c("readr", 
         "tidyxl", 
         "tidyverse",
         "unpivotr",
         "devtools", 
         "data.table", 
         "dplyr",
         "magrittr", 
         "forcats",
         "rattle",
         "forecast") 
#package names: readr for import as tibble; data.table for fast csv save; tidyverse for operations using dplyr's piping feature; rattle for format tibble column names; magrittr %<>% function pipes left hand data to right side and returns result to left hand overwriting original contents on left hand
install.packages(pkgs, repos="https://cran.r-project.org")
inst = lapply(pkgs, library, character.only = TRUE) #load packages
devtools::install_github("nacnudus/smungs")

```

```{r data, include=FALSE}
# 2 clear rows and columns - with columns and rows removed prior to Excel. 

dataxl <- ('/Users/michaelleary/Documents/GitHub/indiairon/data/worldsteel.xlsx')

all_cells6 <-
  xlsx_cells(dataxl, sheets = "importR") %>%
  dplyr::filter(!is_blank) %>%
  select(row, col, data_type, character, numeric) %>%
print()
```

```{r exceldata, include=FALSE}
# View the cells in their original positions on the spreadsheet
rectify(all_cells6)
```

```{r topbotdata, include=FALSE}
head(all_cells6)
tail(all_cells6)
```

```{r csvdata, include=FALSE}
datacsv <- read_csv('/Users/michaelleary/Documents/GitHub/indiairon/data/worldsteel.csv', ) # in tibble form
```

```{r tibstrdata, include=FALSE}
str(datacsv)
```

```{r datatidy, include=FALSE}
#steep production as time series
#make template variable
ds <- datacsv
ds
ds <- ds[-c(121:122),-c(1:2)]
class(ds)
str(ds)
dspctcap <- ds %>%
  mutate(prod_pct_of_cap = round(100 * production / capacity))
tail(dspctcap)

```


```{r tsdata, include=FALSE}
#convert ds data frame into a time series 
dsts <- ts(dspctcap, frequency=1, start=c(1900), end=c(2019))
dsts
```


```{r ts_dates, include=FALSE}
start(dsts)
end(dsts)
```

```{r extracttstimeobject, include=FALSE}
timedsts <- time(dsts)
timedsts
```

```{r adddatetsdata, include=FALSE}
#da <- cbind(timedsts, dsts ) 
#colnames(da)
#da
#colnames(da) <- paste0("b", 1:2)
#da
#tail(da)
```


```{r subsettsdata, include=FALSE}
tmp = window(dsts, start=c(2000), end=c(2019))
class(tmp)
tmp
plot.ts(tmp)
```

```{r capprodplot, echo=FALSE}
dstsplot <-dsts[,-3] 

plot(window(dstsplot, start=c(2000), end=c(2019)), plot.type = "single",
     main = "World Steel production and capacity",
     ylab = "Millions metric tons",
     col=c("blue", "red"),
     lty=1:2)
legend(2000,2400, legend=c("production", "capacity"), col=c("blue", "red"),
       lty=1:2)
```

```{r lagtsvar, include=FALSE}
#create lagged variable
lagdsts <- stats::lag(dsts,-3)
lagdsts
```

```{r leaddstsvar, include=FALSE}
# create lead varaible
leaddsts <- lead(dsts,3)
leaddsts
```

```{r mergedtsleadlagvar, include=FALSE}
# lead/lag on one table
tx <- cbind(dsts, leaddsts)
tx1 <- cbind(tx, lagdsts)
tx1
```


```{r ts_as_df, include=FALSE}
myDF <- as.data.frame(dsts)
lagDF <- lag(myDF, n=1)
lagDF
```

```{r tsplot, include=FALSE}
plot.ts(dsts[,2],
        col="blue",
        lwd=1,
        ylab="Production in millions tons",
        main="World Steel production 1900-2019")
```

```{r tailtsdata, echo-FALSE}
tail(dsts,20)
```

```{r tslogplot, include=FALSE}
logdsts <- log(dsts[,1])
plot.ts(logdsts)
```

```{r tsdifference, include=FALSE}
dstsdiff1 <- diff(dsts[,1], differences=1)
plot.ts(dstsdiff1)
head(dstsdiff1)
```

```{r tsdifference2, include=FALSE}
dstsdiff2 <- diff(dsts[,1], differences=2)
plot.ts(dstsdiff2)
```

```{r datasummary, include=FALSE}
summary(dsts[,1])
```

```{r diff1summary, include=FALSE}
summary(dstsdiff1)
```

```{r diff2summary, include=FALSE}
summary(dstsdiff2)
```
