---
title: "ANOVA Tests"
author: "Alyssa Rolfe"
date: "March 17, 2019"
---

----------------------------


The Analysis of Variance (ANOVA) is commonly seen in biological papers and has a lot of practical uses. Compared to the t-test that is used to compare the means of two groups, the ANOVA allows us to look at more 3 or more groups for comparison. You should be familiar with the fact that there is a one-way and two-way ANOVA. While they are similar, they are used to ask slightly different statistical questions. You can review these differences and the assumptions of the tests below.   

## The One-Way ANOVA

The one-way ANOVA looks at the effect of an independent variable on the dependent variable.

**Assumptions:**  
1. Normality: The data is normally distributed  
2. Equal Variance: the variance between groups should be the same  
3. Independent Samples  
4. The dependent variable should be continuous  

## The Two-Way ANOVA

The two-way ANOVA looks at the effect of two independent variables on the dependent variable.

**Assumptions:**  
1. Normality: The data is normally distributed  
2. Equal Variance: the variance between groups should be the same  
3. Independent Samples  
4. The dependent variable should be continuous  
5. The independent variables are categorical and independent factors

### Libraries Used for this Project  


{% highlight r %}
library(tidyverse)
library(car)
library(stats)
{% endhighlight %}

### Generate Some Data

Here we will generate a dummy experiment and some associated data for demonstration. Think of this as an experiment looking at rodent weights in response to diet. We also information about the sex of the animals in question. With this variety of data we can test hypotheses using either a one-way or two-way ANOVA depending on the particular experimental question.


{% highlight r %}
weight = round(rnorm(120, mean = 27 ),2) # rnorm will generate random normally distributed data
sex = rep(c("M", "F"), 60)
diet = rep(c("HF", "N", "LF"), 40)

df = data.frame(weight, sex, diet)
# normally tibbles are great, but in this case we do want our strings as factors so we will just go with the standard base R data.frame()
head(df)
{% endhighlight %}



{% highlight text %}
##   weight sex diet
## 1  26.47   M   HF
## 2  26.25   F    N
## 3  24.78   M   LF
## 4  26.68   F   HF
## 5  27.66   M    N
## 6  26.24   F   LF
{% endhighlight %}

Look at the structure of the data to understand what we are working with.


{% highlight r %}
str(df)
{% endhighlight %}



{% highlight text %}
## 'data.frame':	120 obs. of  3 variables:
##  $ weight: num  26.5 26.2 24.8 26.7 27.7 ...
##  $ sex   : Factor w/ 2 levels "F","M": 2 1 2 1 2 1 2 1 2 1 ...
##  $ diet  : Factor w/ 3 levels "HF","LF","N": 1 3 2 1 3 2 1 3 2 1 ...
{% endhighlight %}

We also want to make sure that this is a balanced experiment (ie equal number of observations for each categorical variable). We can do this with a contingency table which returns the counts for us.


{% highlight r %}
table(df$sex, df$diet)
{% endhighlight %}



{% highlight text %}
##    
##     HF LF  N
##   F 20 20 20
##   M 20 20 20
{% endhighlight %}


### Visualize the Data  


{% highlight r %}
ggplot(data = df,
       aes(x = diet, y = weight, fill = sex)) +
  geom_boxplot()
{% endhighlight %}

![plot of chunk unnamed-chunk-31](/figure/unnamed-chunk-31-1.svg)

### Check the Assumptions  

**Assumptions:**  
1. Normality: The data is normally distributed  
2. Equal Variance: the variance between groups should be the same  
3. Independent Samples  
4. The dependent variable should be continuous

**Normality: The data is normally distributed**

Using the histogram you can look to see if the weight data (our dependent variable) is normally distributed. This should look like the classic bell curve distribution. It doesn't have to be perfect, but close is good.


{% highlight r %}
hist(df$weight)
{% endhighlight %}

![plot of chunk unnamed-chunk-32](/figure/unnamed-chunk-32-1.svg)

These results were expected since we generated normal data. For comparison let's generate a non-normal set of data and run the same tests.


{% highlight r %}
weight_non_normal = round(runif(120, max = 35, min = 14),2)
hist(weight_non_normal)
{% endhighlight %}

![plot of chunk unnamed-chunk-33](/figure/unnamed-chunk-33-1.svg)

We can also use a Q-Q (quantile-quantile) Plot for visual inspection of normality. Much like the histogram, the Q-Q plot is a visual tool to get a general idea of the data. The actual plot generated is composed of the theoretical normal distribution quantiles (x-axis) and the sample quantiles (y-axis). If our data is normally distributed we should get something that looks approximately like a straight line.


{% highlight r %}
par(mfrow=c(1,2))
qqPlot(df$weight, main="Q-Q Plot Normal Data") # normal data
{% endhighlight %}



{% highlight text %}
## [1] 18 22
{% endhighlight %}



{% highlight r %}
qqPlot(weight_non_normal, main="Q-Q Plot Non-Normal Data") # non-normal data
{% endhighlight %}

![plot of chunk unnamed-chunk-34](/figure/unnamed-chunk-34-1.svg)

{% highlight text %}
## [1] 21 61
{% endhighlight %}

We can also perform a true statistical test called the Shapiro-Wilk test. This tests if our data came from a normally distributed population. The hypotheses for this test are as follows:     

H0: The sample came from a normally distributed population  
H1: The sample came from a population that is not normally distributed  


{% highlight r %}
shapiro.test(df$weight) # normal data
{% endhighlight %}



{% highlight text %}
##
## 	Shapiro-Wilk normality test
##
## data:  df$weight
## W = 0.99165, p-value = 0.6878
{% endhighlight %}



{% highlight r %}
shapiro.test(weight_non_normal) # non-normal data
{% endhighlight %}



{% highlight text %}
##
## 	Shapiro-Wilk normality test
##
## data:  weight_non_normal
## W = 0.95151, p-value = 0.0002793
{% endhighlight %}

For the normal data we get a p-value > 0.05, and thus we cannot reject the null hypothesis that the data come from a normally distributed population. For the non-normal data we get a p-value < 0.05 which means we do reject the null hypothesis.   

In practical interpretation terms:  
* p-value > 0.05 = normal distribution  
* p-value < 0.05 = non-normal distribution  

**Equal Variance: the variance between groups should be the same**

To test if our variance between groups is equal we will use the Bartlett's test. Another option is the Levene's test, and you might see this used as well. Both are acceptable but it should be noted that the Bartlett's test should not be used with non-normal data. Levene's test is not as sensitive to non-normality and can be used to examine non-parametric data.    


{% highlight r %}
bartlett.test(weight ~ interaction(sex, diet), data =df)
{% endhighlight %}



{% highlight text %}
##
## 	Bartlett test of homogeneity of variances
##
## data:  weight by interaction(sex, diet)
## Bartlett's K-squared = 3.6223, df = 5, p-value = 0.605
{% endhighlight %}



{% highlight r %}
leveneTest(weight ~ sex*diet, data = df)
{% endhighlight %}



{% highlight text %}
## Levene's Test for Homogeneity of Variance (center = median)
##        Df F value Pr(>F)
## group   5  0.7421 0.5935
##       114
{% endhighlight %}

For both of these tests you will notice that the p-values are different, however they are both > 0.05. This means that we can assume the variances are equal.  

### Perform the One-Way ANOVA Test  

For the one-way ANOVA our sample experimental question is: does diet effect rodent weight?  

Hypotheses of our test are as follows:

H0: The means of all the diets are equal  
H1: The mean of at least one diet is different


{% highlight r %}
aov = aov(weight ~ diet, data = df) # perform the test
summary(aov) # print the results
{% endhighlight %}



{% highlight text %}
##              Df Sum Sq Mean Sq F value Pr(>F)
## diet          2   0.52  0.2582   0.256  0.774
## Residuals   117 117.80  1.0069
{% endhighlight %}



{% highlight r %}
print(model.tables(aov,"means"),digits=3) # print the summary of means table
{% endhighlight %}



{% highlight text %}
## Tables of means
## Grand mean
##          
## 27.21642
##
##  diet
## diet
##    HF    LF     N
## 27.28 27.13 27.24
{% endhighlight %}

Because the  p-value Pr(>F) is greater than the standard 0.05 threshold for significance, we fail to reject the null hypotheses above. More simply put...the mean weight of the animals is not different between the different diets.  

### Perform the Two-Way ANOVA Test  

If we want to look at the interaction between both of the independent variables (diet and sex) on weight we would need to use a two-way ANOVA. This is very similar in execution to the one-way ANOVA, but there are additional hypotheses that are being tested.  

Hypotheses of our test are as follows:

H0: The means of all the diets are equal  
H1: The mean of at least one diet is different

H0: The means of the sex groups are equal  
H1: The means of the sex groups are different

H0: There is no interaction between diet and sex  
H1: There is an interaction between diet and sex



{% highlight r %}
aov2 = aov(weight ~ sex * diet, data = df)
summary(aov2)
{% endhighlight %}



{% highlight text %}
##              Df Sum Sq Mean Sq F value Pr(>F)
## sex           1   1.05  1.0509   1.054  0.307
## diet          2   0.52  0.2582   0.259  0.772
## sex:diet      2   3.08  1.5424   1.547  0.217
## Residuals   114 113.67  0.9971
{% endhighlight %}

Because all p-values Pr(>F) are greater than the standard 0.05 threshold for significance, we fail to reject all three of the null hypotheses above.

### Tukey's Post-Hoc Test  

To look at the specific differences, we need to use a post-hoc test. To do this the Tukey's HSD (honestly significant difference) test is used. What it effectively does is perform a set of pairwise comparisons on the means. It should be noted that While the Tukey's test does have it's own set of assumptions, they are in line with the assumptions of the ANOVA. This means if the assumptions of the ANOVA were met, the Tukey's assumptions are met as well.  


{% highlight r %}
TukeyHSD(aov2)
{% endhighlight %}



{% highlight text %}
##   Tukey multiple comparisons of means
##     95% family-wise confidence level
##
## Fit: aov(formula = weight ~ sex * diet, data = df)
##
## $sex
##          diff        lwr       upr     p adj
## M-F 0.1871667 -0.1739852 0.5483186 0.3067599
##
## $diet
##         diff        lwr       upr     p adj
## LF-HF -0.153 -0.6832299 0.3772299 0.7725718
## N-HF  -0.034 -0.5642299 0.4962299 0.9872995
## N-LF   0.119 -0.4112299 0.6492299 0.8553084
##
## $`sex:diet`
##              diff        lwr       upr     p adj
## M:HF-F:HF  0.0435 -0.8718401 0.9588401 0.9999929
## F:LF-F:HF -0.0745 -0.9898401 0.8408401 0.9998976
## M:LF-F:HF -0.1880 -1.1033401 0.7273401 0.9911629
## F:N-F:HF  -0.3280 -1.2433401 0.5873401 0.9038181
## M:N-F:HF   0.3035 -0.6118401 1.2188401 0.9291363
## F:LF-M:HF -0.1180 -1.0333401 0.7973401 0.9990274
## M:LF-M:HF -0.2315 -1.1468401 0.6838401 0.9774122
## F:N-M:HF  -0.3715 -1.2868401 0.5438401 0.8471428
## M:N-M:HF   0.2600 -0.6553401 1.1753401 0.9626433
## M:LF-F:LF -0.1135 -1.0288401 0.8018401 0.9991944
## F:N-F:LF  -0.2535 -1.1688401 0.6618401 0.9664753
## M:N-F:LF   0.3780 -0.5373401 1.2933401 0.8374321
## F:N-M:LF  -0.1400 -1.0553401 0.7753401 0.9977868
## M:N-M:LF   0.4915 -0.4238401 1.4068401 0.6286119
## M:N-F:N    0.6315 -0.2838401 1.5468401 0.3488681
{% endhighlight %}

As expected none of our pairwise comparisons were significant, because our ANOVA indicated that they were not initially significant. Despite our uninteresting results, this is the the general method you would use for a two-way ANOVA with a post-hoc test for pairwise comparisons.   

As a final note, pay attention to the fact that the p-values for the Tukey's test are actually adjusted p-values. This is because the test controls for family-wise error rates. The more comparisons we make, the higher the probability that we will find false positives (type-I error). The Tukey's test takes into account the number of comparisons and attempts to minimize such type-I errors. This means it is effectively minimizing the probability that the researcher will find a comparison that is incorrectly significant. While small p-values are great, it is even more important to come to the most accurate conclusions.  

![The Easiest Way to Remember the Difference in Error Types](https://i.imgur.com/UikGNPX.jpg)

### Conclusions  

While this is clearly not a deep statistical discussion of ANOVAs, this is enough information to at least get you started using R to perform this test and associated tests. In this case our data wasn't very interesting and it met all the test assumptions, but this isn't how things work in reality. In another post I will go into more detail about what to do when the assumptions of the ANOVA are not met.
