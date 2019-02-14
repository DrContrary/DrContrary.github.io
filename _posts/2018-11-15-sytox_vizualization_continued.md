---
title: "Quantitative Image Analysis Data Visualization"
author: "Alyssa"
date: "November 14, 2018"

---


----------------------------


# Plotting and Analysis of Quantitative Image Analysis Data  

Last time we were working on some real data obtained from a quantitative image analysis experiment. This time we will pick up where we left off and look at a few more metrics and explore some more plotting options.   

**Goals**  

* read in the data from online repository  
* visual inspection of outliers    
* statistical outlier detection   
* additional graphing    


##Import the Tidy Data  

The original data set in this case was untidy and required a bit of work before we could do anything with it. To save some time and to prevent redundancy, I have provided a cleaned up version of the data for direct download. This data can be directly loaded in from my github repository using the RCurl package.


{% highlight r %}
library(dplyr)
library(RCurl)
library(ggplot2)
{% endhighlight %}


{% highlight r %}
#read in the file from github
urlBase <-'https://raw.githubusercontent.com/DrContrary/R4biology/master/Data%20Files/'
mkCon <- function(nm) { 	
  textConnection(getURL(paste(urlBase,nm,sep='/')))
}
sytox_data_full <- read.table(mkCon('sytox_data_full.csv'), sep=',',header=T,comment.char='', row.names = 1)
sytox_data_full$glucose = as.character(sytox_data_full$glucose)
{% endhighlight %}

# Generate the Statistics Table Again  

Now that we have the data, lets run our statistics again so that we have them for graphing. Remember the full dataset contains the full range of glucose treatments as well as treatment. Our summary statistics will include the mean percentage of dead cells as well as the standard deviation. For ease of reading, we also use the arrange function. In this case the function sorts our data by treatment.


{% highlight r %}
#generate the table of statisitcs
mean_stats_full = sytox_data_full %>%
  group_by(glucose, mye.treatment) %>%
  summarise(percent_dead_mean = mean(percent_dead), sd_percent = sd(percent_dead)) %>%
  arrange(mye.treatment)
mean_stats_full
{% endhighlight %}



{% highlight text %}
## # A tibble: 18 x 4
## # Groups:   glucose [9]
##    glucose mye.treatment percent_dead_mean sd_percent
##    <chr>   <fct>                     <dbl>      <dbl>
##  1 0       +_Mye                      1.78      0.458
##  2 1       +_Mye                      3.59      1.11
##  3 1.5     +_Mye                      2.93      1.19
##  4 2       +_Mye                      1.10      0.482
##  5 2.5     +_Mye                      2.55      1.57
##  6 3       +_Mye                      4.36      6.52
##  7 3.5     +_Mye                      1.07      0.939
##  8 4       +_Mye                      1.43      0.664
##  9 4.5     +_Mye                      1.05      0.573
## 10 0       No_Mye                     2.19      0.568
## 11 1       No_Mye                     3.87      1.60
## 12 1.5     No_Mye                     3.35      0.824
## 13 2       No_Mye                     4.26      1.63
## 14 2.5     No_Mye                     3.11      1.18
## 15 3       No_Mye                     2.61      1.25
## 16 3.5     No_Mye                     8.39      4.84
## 17 4       No_Mye                    36.6       9.67
## 18 4.5     No_Mye                    44.4      28.9
{% endhighlight %}


# Generate Boxplot in ggplot to Visually Detect Outliers

The type of boxplot that we are talking about here is the box and whisker plot. If you recall from statistics each point on the box shows a specific data summary point. While the whiskers can be defined a number of ways, the default output in ggplot is Tukey's style we will focus only on this definition. This method define the points as such:  
* lower hinge: 25th percentile (first quartile)    
* upper hinge: 75th percentile (third quartile)  
* middle hinge: 50th percentile (mean)  
* upper whisker: from upper hinge to largest value or 1.5 X IQR (inter-quartile range)  
* lower whisker: from lower hinge to smallest value or 1.5 X IQR (inter-quartile range)  

**Outliers** are those points outside the end of the whiskers.   


![](https://upload.wikimedia.org/wikipedia/commons/1/1a/Boxplot_vs_PDF.svg)


For more information you can reference the open source paper here (1978) <https://www.jstor.org/stable/2683468?origin=crossref&seq=1#page_scan_tab_contents>  

## Using geom_boxplot  


{% highlight r %}
#box plot to show data and outliers
ggplot(sytox_data_full,
       aes(x= glucose, y = percent_dead, fill = mye.treatment)) +
  geom_boxplot()
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/figure/unnamed-chunk-4-1.svg)

For more information on the different arguments you can use with the boxplot look here: <https://ggplot2.tidyverse.org/reference/geom_boxplot.html>  


# Grubbs Test for Outliers

For illustration purposes only, the Grubbs Test will be used to determine if some of the outliers shown on the boxplot can be removed. I say illustration only because while it may be statistically sound to remove outliers from a dataset, I am of the belief that they should not be removed from biological data. However, in some fields such as chemistry the removal of outliers is common. Additionally, in the context of machine learning, the removal of outliers is often required for proper training.  

Why the Grubbs Test?  

There are a number of outlier tests and each has a certain set of assumptions and should be used under certain situations. The Grubbs Test allows for the removal of only one data point and is well suited for data with few observations. For easy implementation of this test we will use the outliers package.  


{% highlight r %}
#install.packages("outliers")
library(outliers)
{% endhighlight %}

The function is defined as : grubbs.test(x, type = 10, opposite = FALSE, two.sided = FALSE)  


{% highlight r %}
otest = sytox_data_full %>%
  filter(glucose ==3) %>%
  filter(mye.treatment == "+_Mye")
grubbs.test(otest$percent_dead)
{% endhighlight %}



{% highlight text %}
##
## 	Grubbs test for one outlier
##
## data:  otest$percent_dead
## G = 1.7802000, U = 0.0096713, p-value = 0.001012
## alternative hypothesis: highest value 15.9722222222222 is an outlier
{% endhighlight %}

Based on the results, we can conclude that at 3.0g/L glucose with treatment there is one outlier present. Just looking at the boxplot, it is evident without testing that this point is well beyond the end of the whisker.


{% highlight r %}
otest
{% endhighlight %}



{% highlight text %}
##   timepoint mye.treatment glucose DAPI Sytox total_cells percent_dead
## 1      24hr         +_Mye       3 1190     9        1199    0.7506255
## 2      24hr         +_Mye       3 1491    23        1514    1.5191546
## 3      24hr         +_Mye       3 1620    41        1661    2.4683925
## 4      24hr         +_Mye       3 1690    19        1709    1.1117613
## 5      24hr         +_Mye       3  121    23         144   15.9722222
{% endhighlight %}

If we look at the raw data, it becomes clear that an  error likely occurred in the total cell counts since the DAPI numbers are significantly lower than the other observations. As a result the total percentage of dead cells is much higher for this sample.  

# Boxplots With and Without Outliers Shown  

You may have started to notice a pattern by now, that there are many ways to accomplish the same task in R. This is true as well for plotting multiple plots together. In a separate tutorial it would be useful to go through the multiple packages and methods for doing so, but for now we will simply use the cowplot package. It has a number of features that really shine when it comes to creating multi-panel graphs like those you see in publications.  


{% highlight r %}
#install.packages("cowplot")
library(cowplot)
{% endhighlight %}



{% highlight r %}
#box plot to show data and outliers
nooutlier =ggplot(sytox_data_full,
       aes(x= glucose, y = percent_dead, fill = mye.treatment)) +
  geom_boxplot(outlier.shape = NA)+
  ylab("Percentage of Dead Cells") +
  xlab("Glucose Concentration (g/L)") +
  labs(title = "Outliers Hidden")+
  scale_fill_discrete(name="Experimental Condition",
                    breaks=c("+_Mye", "No_Mye"),
                    labels=c("Treated", "Untreated"))+
  theme(legend.position="bottom")
woutlier =ggplot(sytox_data_full,
                  aes(x= glucose, y = percent_dead, fill = mye.treatment)) +
  geom_boxplot()+
  ylab("Percentage of Dead Cells") +
  xlab("Glucose Concentration (g/L)") +
  labs(title = "Outliers Shown")+
  scale_fill_discrete(name="Experimental Condition",
                      breaks=c("+_Mye", "No_Mye"),
                      labels=c("Treated", "Untreated"))+
  theme(legend.position="bottom")
plot_grid(woutlier, nooutlier, labels = "AUTO")
{% endhighlight %}

<img src="/figure/unnamed-chunk-9-1.svg" title="plot of chunk unnamed-chunk-9" alt="plot of chunk unnamed-chunk-9" style="display: block; margin: auto;" />

Showing these side by side demonstrates the subtle differences. The most important aspect to note is that the boxes in B are not different than those in A. The removal of the outlying points only occurs on the graph and does not effect the underlying data. As such the IQRs and whiskers remain unchanged. If your goal is to remove the outliers entirely, you must remove them from the dataset to propagate the changes in the boxes.  

## High Impact Journal Formatting  

While the boxplot may be a great way to present data, it isn't as common to see in many of your top tier medical and biological journals. Humans are creatures of habit, we have always used the bargraph so that is what we turn to when given a choice. However if you keep up with your reading, you might notice that there has been more of a push to publish what I will call "complete data". This means all of the data points are presented along with the mean and standard deviation. One method I have noticed recently is overlying individual points on top of your standard bar graph.  

Luckily because of the laying method for generating plots in ggplot, we can easily achieve this type of graph using the data we already have.


{% highlight r %}
library(ggplot2)
{% endhighlight %}

Because we are using two individual data sets (mean_stats_full and sytox_data_full) and plotting them on the same graph, we must set one as the default. Since our bars and error bars are from the same stats dataframe, we will use this as default and indicate this in the ggplot() argument.  

To overlay the points on top, we must pull the data from the full sytox_data_full dataframe. To do so we specify the data for geom_point and provide the respective x and y values in the aes() arguments.


{% highlight r %}
#high impact journal formatting
ggplot(mean_stats_full,
       aes(x= glucose, y = percent_dead_mean, fill = mye.treatment)) +
  geom_bar(stat = "identity", position = "dodge") +
  geom_point(data = sytox_data_full,
             aes(x= glucose, y = percent_dead, fill = mye.treatment),
             position = position_dodge(width = 0.9),
             shape = 21,    #shape with outline
             size = 3)+     #size of point
  geom_errorbar(aes(ymin=percent_dead_mean-sd_percent, ymax=percent_dead_mean+sd_percent),
                size=.3,    # Thinner lines
                width=.2,
                position = position_dodge(width = 0.9)) +

  ylab("Percentage of Dead Cells") +
  xlab("Glucose Concentration (g/L)") +
  labs(title = "Barplot of Data using ggplot2") +
  scale_fill_discrete(name="Experimental \n Condition",
                      breaks=c("+_Mye", "No_Mye"),
                      labels=c("Treated", "Untreated"))+
  theme_bw()
{% endhighlight %}

![plot of chunk unnamed-chunk-11](/figure/unnamed-chunk-11-1.svg)


**NOTE:** use "\n" to create a line break in a title  

Order matters in the outcome of the final plot. If you plot the errorbar layer before the geom_point layer your points will cover the black lines of the errorbars.
