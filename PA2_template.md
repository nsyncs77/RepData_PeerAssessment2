#Peer assessment 2


###Questions


**1.Across the United States, which types of events (as indicated in the EVTYPE variable) are most harmful with respect to population health?**


1.Loading and processing the data by regarding fatalities and injuries as related population health.

```r
storm <- read.csv("repdata-data-StormData.csv")
short <- storm[,c("BGN_DATE", "EVTYPE", "FATALITIES", "INJURIES")]
```


2.Converting to String from factor

```r
i <- sapply(short, is.factor)
short[i] <- lapply(short[i], as.character)
```


3.Building a function for extracting the year from BGN_DATE and add it as a new column 

```r
substrSpecific <- function(x, n, m){
  substr(x, nchar(x)-n+1, nchar(x)-m+1)
}
library("plyr")
mutate <- mutate(short, year = substrSpecific(short$BGN_DATE, 12, 9))
```


4.Accumulating the fatalities and injuries for each type of event in each year and add together as population health impact

```r
total <- ddply(mutate, .(year, EVTYPE), summarize, fatalsum = sum(FATALITIES), injuriesum = sum(INJURIES))
population <- mutate(total, impact = (fatalsum + injuriesum))
```


5.Accumulating the total impact for each type of event and filtering out impact<1000 for drawing the plot.

**The final result shows that Torando with total population health impact = 96980, is the most harmful with respect to population health.**


```r
library(ggplot2)
max <- ddply(population, .(EVTYPE), summarize, total = sum(impact))
summary(max$total)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0       0       0     158       0   96980
```

```r
final <- max[max$total>1000 , ]
g <- ggplot(final, aes(EVTYPE, total, fill = EVTYPE))
g+geom_bar(colour = "black", stat = "identity")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 


**2.Across the United States, which types of events have the greatest economic consequences?**

1.Loading and processing the data by regarding properties damage and crop damage as related economic impact.
Also, assuming M (million), K (thousand), H(hundred), B(billion) as the only allowable values in the xxxEXP features (crop and property).


```r
property <- storm[,c("BGN_DATE", "EVTYPE", "PROPDMG", "PROPDMGEXP")]
property$PROPDMGEXP <- tolower(property$PROPDMGEXP)
prop <- property[property$PROPDMGEXP=="k"|property$PROPDMGEXP=="m"|property$PROPDMGEXP=="h"|property$PROPDMGEXP=="b", ]

crop <- storm[,c("BGN_DATE", "EVTYPE", "CROPDMG", "CROPDMGEXP")]
crop$CROPDMGEXP <- tolower(crop$CROPDMGEXP)
crop <- crop[crop$CROPDMGEXP=="k"|crop$CROPDMGEXP=="m"|crop$CROPDMGEXP=="h"|crop$CROPDMGEXP=="b", ]
```


2.Converting to String from factor

```r
i <- sapply(prop, is.factor)
prop[i] <- lapply(prop[i], as.character)
i <- sapply(crop, is.factor)
crop[i] <- lapply(crop[i], as.character)
```


3.Building a function for extracting the year from BGN_DATE and add it as a new column 

```r
propmutate <- mutate(prop, year = substrSpecific(prop$BGN_DATE, 12, 9))
cropmutate <- mutate(crop, year = substrSpecific(crop$BGN_DATE, 12, 9))
```


4.Accumulating the property and crop damage for each type of event in each year 

```r
propsplit <- split(propmutate, propmutate$PROPDMGEXP)
cropsplit <- split(cropmutate, cropmutate$CROPDMGEXP)

for(i in 1:length(propsplit)){
    if(i==1){
        propsplit[[i]]$price = propsplit[[i]]$PROPDMG * 1000000000
    }else if(i==2){
        propsplit[[i]]$price = propsplit[[i]]$PROPDMG * 100
    }else if(i==3){
        propsplit[[i]]$price = propsplit[[i]]$PROPDMG * 1000
    }else{
        propsplit[[i]]$price = propsplit[[i]]$PROPDMG * 1000000
    }
}
for(i in 1:length(cropsplit)){
    if(i==1){
        cropsplit[[i]]$price2 = cropsplit[[i]]$CROPDMG * 1000000000
    }else if(i==2){
        cropsplit[[i]]$price2 = cropsplit[[i]]$CROPDMG * 1000
    }else{
        cropsplit[[i]]$price2 = cropsplit[[i]]$CROPDMG * 1000000
    }
}

bind <- rbind(propsplit[[1]], propsplit[[2]], propsplit[[3]], propsplit[[4]])
bind2 <- rbind(cropsplit[[1]], cropsplit[[2]], cropsplit[[3]])

proptotal <- ddply(bind, .(year, EVTYPE), summarize, propsum = sum(price))
croptotal <- ddply(bind2, .(year, EVTYPE), summarize, cropsum = sum(price2))
```


5.Accumulating the total economic impact (property damage + crop damage) for each type of event and filtering out impact<1 billion for drawing the plot.

**The final result shows that flood with 150 billion loss is the greatest economic consequence**

```r
propmax <- ddply(proptotal, .(EVTYPE), summarize, total = sum(propsum))
cropmax <- ddply(croptotal, .(EVTYPE), summarize, total2 = sum(cropsum))

merge = merge(propmax, cropmax, all=TRUE)
merge[is.na(merge$total), ]$total <- 0
merge[is.na(merge$total2), ]$total2 <- 0
economic <- mutate(merge, add = total + total2)
summary(economic$add)
```

```
##      Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
## 0.000e+00 1.512e+04 2.232e+05 1.108e+09 6.256e+06 1.503e+11
```

```r
economicfinal <- economic[economic$add>1000000000, ]
g <- ggplot(economicfinal, aes(EVTYPE, add, fill = EVTYPE))
g+geom_bar(colour = "black", stat = "identity")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

