---
title: "CS2"
author: "David Nguyen, Austin Simeone, John Rodgers, Hannah Kosinovsky"
date: "December 4, 2018"
output: 
  html_document: 
    keep_md: yes
---




```r
library(reshape2)
library(ggplot2)
library(pander)
```


```r
# load data from file in the data directory
data <- read.csv("./Data/API_17_DS2_en_csv_v2_10226244.csv", skip=4)
# generate rows for values for each year
data2 <- reshape(data, varying = 5:62, sep="", direction='long')
# rename columns
colnames(data2) <- c("Country","Code","Indicator","IndicatorCode","Value","Year","Id")
# isolate survival rate data into dataframe
survial.percent.male <- data2[data2$Indicator == "Survival to age 65, male (% of cohort)",]
# isolate literacy rate data into dataframe
literacy.youth.male <- data2[data2$Indicator == "Literacy rate, youth male (% of males ages 15-24)",]
# merge survival rate and literacy rate dataframes
survival.v.litrate.raw <- merge(survial.percent.male, literacy.youth.male, by=c("Code", "Year"))
# isolate specific variables from raw merged data
survival.v.literate <- survival.v.litrate.raw[c(3,1,2,6,11)]
# rename columns
colnames(survival.v.literate) <- c("Country","Code","Year","survival.percent","literate.rate")
# exclude na values
survival.v.literate <- na.omit(survival.v.literate)
# generate log of survival percent
survival.v.literate$log.survival.percent <- log(survival.v.literate$survival.percent)
# generate log of literate rate
survival.v.literate$log.literate.rate <- log(survival.v.literate$literate.rate)
```


```r
# plot surivival rate versus literate rate
ggplot(survival.v.literate, aes(x=log.literate.rate, y=log.survival.percent)) + geom_point(shape=1) + geom_smooth(method=lm)
```

![](CS2_files/figure-html/plot-1.png)<!-- -->


```r
# generate linear model of survival rate vs literate rate
s.v.l.lm <- lm(log.survival.percent ~ log.literate.rate, data=survival.v.literate)
# generate residuals
s.v.l.m.res <- resid(s.v.l.lm)
# plot residuals
plot(survival.v.literate$log.literate.rate, s.v.l.m.res, ylab="Residuals",xlab="log.literate.rate", main="Residuals vs log of literate rate")
```

![](CS2_files/figure-html/residuals-1.png)<!-- -->


```r
s.v.l.m.stud <- rstudent(s.v.l.lm)
hist(s.v.l.m.stud, freq=FALSE, main="Distribution of Studentized Residuals", xlab="Studentized Residuals")
xfit <- seq(min(s.v.l.m.stud)-1,max(s.v.l.m.stud)+1,length=40)
yfit <- dnorm(xfit)
lines(xfit,yfit)
```

![](CS2_files/figure-html/studentresiduals-1.png)<!-- -->

```r
par(mfrow=c(1,2))
qqnorm(s.v.l.m.stud)
qqline(s.v.l.m.stud)
```

![](CS2_files/figure-html/qqplot-1.png)<!-- -->

```r
pander(summary(s.v.l.lm))
```


--------------------------------------------------------------------
        &nbsp;           Estimate   Std. Error   t value   Pr(>|t|) 
----------------------- ---------- ------------ --------- ----------
    **(Intercept)**       0.1257     0.08567      1.467     0.1424  

 **log.literate.rate**    0.8935     0.01923      46.47       0     
--------------------------------------------------------------------


-------------------------------------------------------------
 Observations   Residual Std. Error   $R^2$   Adjusted $R^2$ 
-------------- --------------------- ------- ----------------
     2205             0.1509          0.495       0.4947     
-------------------------------------------------------------

Table: Fitting linear model: log.survival.percent ~ log.literate.rate
$\hat{\mu}(log(Survival Percent)|log(Literate Rate)) = 0.126 + 0.894 * log(Literate Rate)$

About 49.5% of the variation in the log of Survival Rate is explained by the log of the Literate Rate.