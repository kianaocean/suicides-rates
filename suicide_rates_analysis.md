
*I wrote this document as my final project in my Programming R for
Analytics class at Carnegie Mellon. The purpose of the assignment was to
use R to analyze a dataset and to include commentary along with our
analysis.*

### Introduction

I chose to analyze the Suicide Rates Overview dataset, which contains
worldwide suicide data from 1985-2016. The data was found on Kaggle, and
is available
[here](https://www.kaggle.com/russellyates88/suicide-rates-overview-1985-to-2016).
This data was compiled from four other datasets, and was built to find
signals correlated to increased suicide rates among different groups
throughout the world and across the socio-economic spectrum.

Relevant fields include country, year, gender, age group, suicide rate
(normalized for population size), Human Development Index rating, GDP
per capita, and generation.

Information on the human development index can be found
[here](http://hdr.undp.org/en/content/human-development-index-hdi). In
short, HDI is a summary measure of average achievement in key dimensions
of human development: a long and healthy life, being knowledgeable, and
having a decent standard of living. The HDI is the geometric mean of
normalized indices for each of the three dimensions. HDI is ranked on a
scale from 0 to 1.0, with 1.0 being the highest human development.
([Source](https://worldpopulationreview.com/country-rankings/hdi-by-country))

I am interested in this dataset because suicide is a serious problem,
particularly in the United States. It is a also a problem that has
gotten worse in recent years in the U.S., with suicide rates on the rise
since 1999. I thought it would be interesting to dig into U.S. rates and
compare the U.S. to other countries with similar GDP/capita and HDI. The
Covid-19 pandemic has made mental health and suicide an even more urgent
topic, with at least one CDC survey in 2020 finding that suicidal
thoughts have increased during the pandemic.
([Source](https://www.washingtonpost.com/health/2020/11/23/covid-pandemic-rise-suicides/))
In addition, I was interested in this data because suicide can often be
a taboo topic that people avoid talking about. I think it’s important to
raise awareness and encourage discussion around this serious topic that
affects so many, and examining the data is one important way to do that.
I think a worldwide dataset in particular will be interesting to get a
sense of how the U.S. compares to other countries.

My ideas for analysis of these data include the following:

-   Compare US rates of suicide to other countries, especially those
    with similar HDI and GDP per capita

-   How rates change by generation over time

-   Which age groups/genders are most affected - I know in the U.S.
    there is concern about the high suicide rates among young men in
    particular - does this vary by country?

-   Is there a link between HDI and higher suicide rates? I would expect
    the suicide rates to be lower for countries with higher HDI

### Data Load

``` r
suicides <- read.csv("master.csv", stringsAsFactors=TRUE)
# Rename columns
colnames(suicides) <- c("country", "year", "sex", "age", "suicides.count", "population", 
                        "suicides.rate", "country.year", "hdi", "gdp.annual", "gdp.capita", "generation")
# Factor ages and generation so they appear in a logical order on charts
# Source: https://sebastiansauer.github.io/ordering-bars/
suicides$age <- factor(suicides$age, levels = c("5-14 years", "15-24 years", "25-34 years", "35-54 years", "55-74 years", "75+ years"))
suicides$generation <- factor(suicides$generation, levels = c("G.I. Generation", "Silent", "Boomers", "Generation X", "Millenials", "Generation Z"))
```

After importing the data, I find that there are 27,820 rows and 12
columns. Let’s examine the structure and first few rows of this data.

``` r
str(suicides)
```

    ## 'data.frame':    27820 obs. of  12 variables:
    ##  $ country       : Factor w/ 101 levels "Albania","Antigua and Barbuda",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ year          : int  1987 1987 1987 1987 1987 1987 1987 1987 1987 1987 ...
    ##  $ sex           : Factor w/ 2 levels "female","male": 2 2 1 2 2 1 1 1 2 1 ...
    ##  $ age           : Factor w/ 6 levels "5-14 years","15-24 years",..: 2 4 2 6 3 6 4 3 5 1 ...
    ##  $ suicides.count: int  21 16 14 1 9 1 6 4 1 0 ...
    ##  $ population    : int  312900 308000 289700 21800 274300 35600 278800 257200 137500 311000 ...
    ##  $ suicides.rate : num  6.71 5.19 4.83 4.59 3.28 2.81 2.15 1.56 0.73 0 ...
    ##  $ country.year  : Factor w/ 2321 levels "Albania1987",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ hdi           : num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ gdp.annual    : Factor w/ 2321 levels "1,002,219,052,968",..: 727 727 727 727 727 727 727 727 727 727 ...
    ##  $ gdp.capita    : int  796 796 796 796 796 796 796 796 796 796 ...
    ##  $ generation    : Factor w/ 6 levels "G.I. Generation",..: 4 2 4 1 3 1 2 3 1 4 ...

``` r
kable(head(suicides))
```

| country | year | sex    | age         | suicides.count | population | suicides.rate | country.year | hdi | gdp.annual    | gdp.capita | generation      |
|:--------|-----:|:-------|:------------|---------------:|-----------:|--------------:|:-------------|----:|:--------------|-----------:|:----------------|
| Albania | 1987 | male   | 15-24 years |             21 |     312900 |          6.71 | Albania1987  |  NA | 2,156,624,900 |        796 | Generation X    |
| Albania | 1987 | male   | 35-54 years |             16 |     308000 |          5.19 | Albania1987  |  NA | 2,156,624,900 |        796 | Silent          |
| Albania | 1987 | female | 15-24 years |             14 |     289700 |          4.83 | Albania1987  |  NA | 2,156,624,900 |        796 | Generation X    |
| Albania | 1987 | male   | 75+ years   |              1 |      21800 |          4.59 | Albania1987  |  NA | 2,156,624,900 |        796 | G.I. Generation |
| Albania | 1987 | male   | 25-34 years |              9 |     274300 |          3.28 | Albania1987  |  NA | 2,156,624,900 |        796 | Boomers         |
| Albania | 1987 | female | 75+ years   |              1 |      35600 |          2.81 | Albania1987  |  NA | 2,156,624,900 |        796 | G.I. Generation |

One important detail to note is that HDI only appears in the data for
certain years for some countries, and not at all for other countries. In
total there are 19,456 observations where HDI appears as NA. I account
for this when computing mean HDI, by excluding the NA observations. In
the structure table you can see that the string variables have been
imported as factors and the other variables are numeric. The variable
gdp.annual has been imported as a factor, even though it is really a
numeric variable. I do not use this variable in my analysis so we don’t
need to worry about this.

### Exploratory Data Analysis

First it will be helpful to understand some of the key variables, at the
aggregate and country level.

The first series of histograms (and bar chart for year) shows that the
suicide rates and GDP per capita are heavily right skewed, while the
distribution of HDI ratings are slightly left skewed, with nearly all
countries having at least an HDI of 0.5. The count of observations by
year show that there is relatively little data for the last year
included, 2016.

``` r
par(mfrow=c(2, 2))
hist(suicides$suicides.rate, breaks=50, main="Histogram of Suicide Rates")
barplot(table(suicides$year), main="Observation Count by Year")
hist(suicides$hdi, main="Histogram of HDI Ratings")
hist(suicides$gdp.capita, main="Histogram of GDP per Capita")
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

Next I wanted to look at the countries with the highest suicide rates.
Below is a table of the top 50 countries sorted in descending order by
mean suicide rate. The United States is 37 on the list, with a rate of
13.8 per 100,000 people compared to Lithuania, which has the top rate of
40.4 suicides per 100,000 people.

``` r
summary.country <- ddply(suicides, ~ country, summarize, suicides.rate = (sum(suicides.count)/sum(population))*100000, hdi = mean(hdi, na.rm = TRUE), gdp.capita = mean(gdp.capita))
kable(head(arrange(summary.country, -suicides.rate), 50)) # I used this site for sorting tables http://www.cookbook-r.com/Manipulating_data/Sorting/
```

| country             | suicides.rate |       hdi | gdp.capita |
|:--------------------|--------------:|----------:|-----------:|
| Lithuania           |     41.182219 | 0.8035000 |  9280.5496 |
| Russian Federation  |     32.777207 |       NaN |  6518.8148 |
| Sri Lanka           |     30.483939 | 0.6380000 |   904.2727 |
| Belarus             |     30.344685 | 0.7712000 |  3333.9048 |
| Hungary             |     29.717558 | 0.8038750 |  9370.0516 |
| Latvia              |     28.471011 | 0.7842500 |  8961.0952 |
| Kazakhstan          |     26.898614 | 0.7408889 |  5329.1154 |
| Slovenia            |     26.360477 | 0.8565000 | 18642.2381 |
| Estonia             |     25.964524 | 0.8231250 | 11376.0952 |
| Ukraine             |     24.870397 | 0.7135000 |  1867.5357 |
| Finland             |     23.728620 | 0.8588889 | 35468.2759 |
| Japan               |     21.920580 | 0.8613000 | 36397.5484 |
| Belgium             |     20.692535 | 0.8607000 | 32066.7419 |
| Guyana              |     20.645284 | 0.6082857 |  1674.9200 |
| Austria             |     20.534084 | 0.8475000 | 34261.7801 |
| Croatia             |     20.090406 | 0.7872500 | 10355.8702 |
| France              |     19.699277 | 0.8486000 | 31481.4667 |
| Mongolia            |     19.514770 |       NaN |  4145.0000 |
| Republic of Korea   |     19.316652 |       NaN | 14801.2581 |
| Serbia              |     19.168363 | 0.7532857 |  4471.2778 |
| Suriname            |     18.609416 | 0.7076667 |  4351.9643 |
| Switzerland         |     17.478698 | 0.9090000 | 62981.7619 |
| Czech Republic      |     16.541431 | 0.8386667 | 12369.5466 |
| Cuba                |     16.449413 | 0.7413750 |  4351.1667 |
| Poland              |     16.058786 | 0.8030000 |  8146.4583 |
| Bulgaria            |     15.688543 | 0.7426000 |  3640.4333 |
| Uruguay             |     15.627650 | 0.7465556 |  7622.0714 |
| Luxembourg          |     15.116007 | 0.8511000 | 68798.3871 |
| Sweden              |     14.921211 | 0.8866667 | 41357.5754 |
| Germany             |     14.384192 | 0.8817778 | 35164.2308 |
| New Zealand         |     13.951828 | 0.8756667 | 22279.4828 |
| Denmark             |     13.635851 | 0.8986250 | 49299.9091 |
| Norway              |     13.277792 | 0.9210000 | 57319.6000 |
| Iceland             |     13.178076 | 0.8635000 | 39274.7539 |
| Canada              |     13.021090 | 0.8811111 | 30887.4828 |
| Trinidad and Tobago |     12.927656 | 0.7198571 |  8829.0370 |
| Australia           |     12.926599 | 0.9127500 | 32776.4000 |
| United States       |     12.838459 | 0.8916000 | 39269.6129 |
| Romania             |     12.655418 | 0.7547778 |  4791.4491 |
| Slovakia            |     12.015160 | 0.8075714 | 10526.0000 |
| Mauritius           |     11.654527 | 0.7079000 |  5506.6597 |
| Ireland             |     11.533274 | 0.8626000 | 34230.8667 |
| Kyrgyzstan          |     11.365394 | 0.6232222 |   720.7308 |
| Singapore           |     10.705368 | 0.8530000 | 38050.2581 |
| Netherlands         |     10.693551 | 0.8847000 | 35714.6702 |
| Chile               |      9.563284 | 0.7740000 |  7493.0645 |
| El Salvador         |      9.370110 | 0.6257778 |  2550.6667 |
| Cabo Verde          |      9.288357 |       NaN |  4124.0000 |
| Portugal            |      9.209641 | 0.7840000 | 14176.2963 |
| Puerto Rico         |      8.580785 |       NaN | 18352.6452 |

I was also curious about any correlation between a country’s suicide
rate and its HDI and GDP per capita. The two scatterplots below
illustrate that there is no clear relationship between suicide rate and
HDI, while there is a slight negative relationship between suicide rate
and GDP per capita. Intuitively, this makes sense, because I would
expect people in poorer countries to be more concerned about their
financial situation and for that to have a negative impact on their
mental health and suicidal thoughts. However, I did expect there to be a
stronger relationship between suicide rate and HDI.

``` r
par(mfrow=c(1, 2))
plot(summary.country$hdi, summary.country$suicides.rate)
plot(summary.country$gdp.capita, summary.country$suicides.rate)
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Next I look at the distribution of suicides by year. These initial
boxplots show me that each year there are a large number of extremely
high outliers - even after adjusting for population. This confirms what
we saw earlier in the histogram, which was very right skewed.

``` r
boxplot(suicides$suicides.count ~ suicides$year,
        ylim = c(0, 1000),
        main = "Boxplot of Number of Suicides by Year - Close Up")
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
boxplot(suicides$suicides.rate ~ suicides$year,
        ylim = c(0, 150),
        main = "Boxplot of Suicides per 100k Pop. by Year - Close Up")
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-7-2.png)<!-- -->

### Multiple Regression Model

Given that the data contains one outcome variable I am interested in
(suicide rate) and several potential explanatory variables, I thought it
would be interesting to develop a multiple regression model to see if
there are statistically significant impacts of the explanatory variables
on the outcome. Below, I regress suicide rate on year, sex, age group,
HDI, GDP/capita, and generation.

The results show that there are many statistically significant variables
in the data, including year, gender, age group, HDI, GDP/capita, and
whether an individual is part of the Silent generation. One of the most
surprising results to me is that HDI is positively correlated with
suicide rates, implying that as a country’s HDI increases, suicide rates
also tend to increase (controlling for other factors).

One thing to note is that the adjusted R-squared value is only 0.3344.
This is on the low side, but it makes sense, given that there are likely
many other factors that are not included in the data that could affect a
country’s suicide rate, such as inequality, unemployment rates, climate,
and levels of social isolation.

``` r
suicides.lm <- subset(suicides, select=c("year", "sex", "age", "suicides.rate", "hdi", "gdp.capita", "generation"))
model <- lm(suicides.rate ~ year + sex + age + hdi + gdp.capita + generation, data = suicides.lm)
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = suicides.rate ~ year + sex + age + hdi + gdp.capita + 
    ##     generation, data = suicides.lm)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -35.493  -7.673  -1.925   4.694 162.402 
    ## 
    ## Coefficients:
    ##                          Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)             4.390e+02  8.377e+01   5.241 1.64e-07 ***
    ## year                   -2.374e-01  4.307e-02  -5.511 3.67e-08 ***
    ## sexmale                 1.411e+01  3.098e-01  45.549  < 2e-16 ***
    ## age15-24 years          9.490e+00  7.915e-01  11.991  < 2e-16 ***
    ## age25-34 years          1.249e+01  9.152e-01  13.647  < 2e-16 ***
    ## age35-54 years          1.567e+01  1.328e+00  11.794  < 2e-16 ***
    ## age55-74 years          1.692e+01  1.912e+00   8.850  < 2e-16 ***
    ## age75+ years            2.355e+01  2.249e+00  10.471  < 2e-16 ***
    ## hdi                     4.259e+01  2.676e+00  15.918  < 2e-16 ***
    ## gdp.capita             -1.108e-04  1.079e-05 -10.277  < 2e-16 ***
    ## generationSilent       -3.612e+00  9.624e-01  -3.753 0.000176 ***
    ## generationBoomers      -2.375e+00  1.407e+00  -1.688 0.091421 .  
    ## generationGeneration X -2.121e+00  1.936e+00  -1.095 0.273348    
    ## generationMillenials   -1.456e+00  2.469e+00  -0.590 0.555231    
    ## generationGeneration Z  8.981e-01  2.993e+00   0.300 0.764167    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 14.16 on 8349 degrees of freedom
    ##   (19456 observations deleted due to missingness)
    ## Multiple R-squared:  0.3355, Adjusted R-squared:  0.3344 
    ## F-statistic: 301.1 on 14 and 8349 DF,  p-value: < 2.2e-16

### The United States Among Its Peers

I thought it would be interesting to compare suicide rates in the United
States with rates in a handful of other countries with similar HDI
ratings and GDP per capita. To do this, I first create a scatterplot
where each point is a country and on the x-axis is mean HDI and on the
y-axis is mean GDP per capita (across the entire time period). By
overlaying the country name over each point, I identify where the United
States falls and then select 7 of the countries that are closest to the
U.S. with regards to both GDP/capita and HDI over the period. These
countries are Sweden, Iceland, Japan, Netherlands, Germany, Canada, and
Australia.

``` r
country.hdi.gdp <- ddply(suicides, ~ country, summarize, mean.hdi = mean(hdi, na.rm=TRUE), mean.gdp.cap = mean(gdp.capita))
# I used this site to understand how to overlay labels on the scatterplot https://r-graphics.org/recipe-scatter-labels
country.sp <- ggplot(country.hdi.gdp, aes(x=mean.hdi, y=mean.gdp.cap)) +
  geom_point()
country.sp +
  geom_text(aes(label = country), size = 2, nudge_y = 1500) +
  ylab("Mean GDP per Capita") +
  xlab("Mean HDI") +
  ggtitle("GDP per Capita and HDI, by Country")
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

Next, I subset the data to only include these countries, and then plot
the mean rate over time. The chart shows that most of these countries
have similar trends of decreasing rates until about 2000, when rates
either start to increase or remain steady. The exception is Japan, which
has a higher rate than the rest of the countries throughout most of the
period. The United States falls in the middle of most of these countries
throughout the period, although in recent years (since 2010) the United
States has higher rates relative to many of the other countries.

``` r
country.subset <- subset(suicides, select=c("country", "year", "sex", "age", "suicides.count", "population", 
                        "suicides.rate", "country.year", "hdi", "gdp.annual", "gdp.capita", "generation"), 
                         subset=(country %in% c("United States", "Sweden", "Iceland", "Japan", "Netherlands",
                                                "Germany", "Canada", "Australia")))
country.year <- ddply(country.subset, ~ country + year, summarize, mean.rate = (sum(suicides.count)/sum(population))*100000)
line.country <- ggplot(data=country.year,
                  aes(y=mean.rate, x=year, color=country))
cbPalette <- c("#999999", "#E69F00", "#56B4E9", "#009E73", "#F0E442", "#0072B2", "#D55E00", "#CC79A7")
line.country + geom_line() +
  ylab("Mean Suicide Rate per 100k Pop.") +
  xlab(NULL) +
  guides(fill = guide_legend(title = NULL)) +
  ggtitle("Suicide Rates 1985-2016 by Country") +
  scale_color_manual(values=cbPalette)
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

Since rates for many of these countries overlap throughout the period, I
thought it would be useful to look at the mean rates for just these 8
countries in a table format. The first table shows that the United
States actually has the second lowest rate among its peer countries.
However, the chart indicates that since 2000 the United States’ rate has
been trending upwards and in the period since 2010 it looks like the
rate is higher than most of the other countries. The second table shows
that this is the case. Looking at the mean rates in the most recent
years (since 2010), we can see that the U.S. now has the second highest
rate among its peers.

``` r
summary.country.subset1 <- ddply(country.subset, ~ country, summarize, suicides.rate = (sum(suicides.count)/sum(population))*100000)
summary.country.subset2 <- ddply(subset(country.subset, subset=(year >= 2010)), ~ country, summarize, suicides.rate = (sum(suicides.count)/sum(population))*100000)
kable(arrange(summary.country.subset1, -suicides.rate), col.names=c("Country", "Mean Suicide Rate (1985-2016")) # I used this site for sorting tables http://www.cookbook-r.com/Manipulating_data/Sorting/
```

| Country       | Mean Suicide Rate (1985-2016 |
|:--------------|-----------------------------:|
| Japan         |                     21.92058 |
| Sweden        |                     14.92121 |
| Germany       |                     14.38419 |
| Iceland       |                     13.17808 |
| Canada        |                     13.02109 |
| Australia     |                     12.92660 |
| United States |                     12.83846 |
| Netherlands   |                     10.69355 |

``` r
kable(arrange(summary.country.subset2, -suicides.rate), col.names=c("Country", "Mean Suicide Rate (since 2010)")) # I used this site for sorting tables http://www.cookbook-r.com/Manipulating_data/Sorting/
```

| Country       | Mean Suicide Rate (since 2010) |
|:--------------|-------------------------------:|
| Japan         |                       21.81839 |
| United States |                       13.97668 |
| Iceland       |                       13.67422 |
| Sweden        |                       13.05217 |
| Germany       |                       12.91681 |
| Australia     |                       12.36475 |
| Canada        |                       11.93725 |
| Netherlands   |                       11.39316 |

I run a quick ANOVA test for these countries to see if the mean rates
since 2010 are statistically different. First I drop Japan, since it is
a bit of an outlier compared to the other countries. The results are
statistically significant, indicating there are some meaningful
differences in the mean rates of suicide across these 7 countries.

``` r
subset.anova1 <- subset(country.year, subset=(country != "Japan" & year >= 2010))
anova1 <- aov(mean.rate ~ country, data = subset.anova1)
summary(anova1)
```

    ##             Df Sum Sq Mean Sq F value  Pr(>F)   
    ## country      6  31.89   5.315   3.943 0.00395 **
    ## Residuals   36  48.53   1.348                   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

However, when I run the ANOVA test for just the countries closest to the
United States (Iceland, Sweden, Germany, and Australia) there is no
statistically significant difference in the mean suicide rates since
2010.

``` r
subset.anova2 <- subset(country.year, subset=(country %in% c("United States", "Iceland", "Sweden", "Germany", "Australia") & year >= 2010))
anova2 <- aov(mean.rate ~ country, data = subset.anova2)
summary(anova2)
```

    ##             Df Sum Sq Mean Sq F value Pr(>F)
    ## country      4  10.19   2.548   1.631  0.195
    ## Residuals   27  42.19   1.562

Since suicide is a problem that disproportionately affects men, I wanted
to break down the rates for each of these countries by gender. The
results show that across every country, suicide rates are much worse for
men than women. An important area of further research and policy
generation should consider why this is the case and how to help address
mental health and suicide among men.

``` r
country.gender <- ddply(country.subset, ~ country + sex, summarize, mean.rate = (sum(suicides.count)/sum(population))*100000)
bar.country.gender <- ggplot(country.gender, aes(y=country, x=mean.rate, fill=country))
bar.country.gender + geom_bar(stat="identity") + facet_grid(rows=vars(sex)) +
  guides(fill = FALSE) + 
  xlab("Mean Suicide Rate per 100k Pop. (1985-2016)") +
  ylab(NULL)
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

### Focusing on the United States

Since we are in the United States, I wanted to do some deeper analysis
specifically on the United States observations, to see if the data
informs us about any patterns specific to suicide in the U.S.

``` r
suicides.us <- filter(suicides, country == "United States")
kable(head(suicides.us))
```

| country       | year | sex    | age         | suicides.count | population | suicides.rate | country.year      |   hdi | gdp.annual        | gdp.capita | generation      |
|:--------------|-----:|:-------|:------------|---------------:|-----------:|--------------:|:------------------|------:|:------------------|-----------:|:----------------|
| United States | 1985 | male   | 75+ years   |           2177 |    4064000 |         53.57 | United States1985 | 0.841 | 4,346,734,000,000 |      19693 | G.I. Generation |
| United States | 1985 | male   | 55-74 years |           5302 |   17971000 |         29.50 | United States1985 | 0.841 | 4,346,734,000,000 |      19693 | G.I. Generation |
| United States | 1985 | male   | 25-34 years |           5134 |   20986000 |         24.46 | United States1985 | 0.841 | 4,346,734,000,000 |      19693 | Boomers         |
| United States | 1985 | male   | 35-54 years |           6053 |   26589000 |         22.77 | United States1985 | 0.841 | 4,346,734,000,000 |      19693 | Silent          |
| United States | 1985 | male   | 15-24 years |           4267 |   19962000 |         21.38 | United States1985 | 0.841 | 4,346,734,000,000 |      19693 | Generation X    |
| United States | 1985 | female | 35-54 years |           2105 |   27763000 |          7.58 | United States1985 | 0.841 | 4,346,734,000,000 |      19693 | Silent          |

I was first curious about the breakdown in rates by age group. This is
visualized in the bar chart below (broken down by gender). I was
surprised that for men, rates were so much higher among the older
population of people aged 75 and above. Interestingly, this pattern does
not hold for women, with women ages 35-54 having the highest rates of
suicide.

``` r
us.age <- ddply(suicides.us, ~ age + sex, summarize, mean.rate = (sum(suicides.count)/sum(population))*100000)
plot.us <- ggplot(data=us.age,
                  aes(y=mean.rate, x=age, fill=sex))
plot.us + geom_bar(stat="identity", position="dodge") +
  ylab("Mean Suicide Rate per 100k Pop.") +
  xlab(NULL) +
  guides(fill = guide_legend(title = NULL)) +
  ggtitle("United States Suicide Rates by Age Group 1985-2016")
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

The results I saw on age made me curious what the plot would look like
if we looked instead at generation. In this chart, the G.I. Generation
has the highest rates for men. This makes sense based on the previous
plot since they would have mostly been in the 75+ age category from
1985-2016. Millennials have relatively low rates, but they were in the
5-14 years and 15-24 years age categories during the period, so it makes
sense that their rates would be lower.

``` r
us.generation <- ddply(suicides.us, ~ generation + sex, summarize, mean.rate = (sum(suicides.count)/sum(population))*100000)
plot.us.generation <- ggplot(data=us.generation,
                  aes(y=mean.rate, x=generation, fill=sex))
plot.us.generation + geom_bar(stat="identity", position="dodge") +
  ylab("Mean Suicide Rate per 100k Pop.") +
  xlab(NULL) +
  guides(fill = guide_legend(title = NULL)) +
  ggtitle("United States Suicide Rates by Generation 1985-2016")
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

I wanted to understand the suicide trends over time, and since I found
the rates vary so much by gender, I wanted to separate out male and
female rates. The line graph below shows the trendline in the United
States. We can observe a downward trend until 2000, when the rates start
to increase for both men and women.

``` r
us.time <- ddply(suicides.us, ~ year + sex, summarize, mean.rate = (sum(suicides.count)/sum(population))*100000)
plot.us.time <- ggplot(data=us.time,
                  aes(y=mean.rate, x=year, color=sex))
plot.us.time + geom_line() +
  ylab("Mean Suicide Rate per 100k Pop.") +
  xlab(NULL) +
  guides(fill = guide_legend(title = NULL)) +
  ggtitle("United States Suicide Rates 1985-2016")
```

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

Given this trend, I wanted to further break out the data by age group to
see if there were underlying trends specific to certain age categories.

For women, it looks like each age category follows a similar trend over
time, increasing gradually starting in 2000. One exception is the 75+
age category, where the rate has generally decreased since 1985.

For men, there is a sharp decrease in the 75+ year age group starting
around 1990, leveling out around 2000. This age group still has the
highest rates, but it is now closer in magnitude to the other male age
categories. Since 2000, there have been increases in the male rates,
especially in the middle to older age categories (35-74 years). There
has also been an increase starting around 2010 for the 25-34 years male
age category.

When thinking about policies in the United States, it would be important
to understand why suicides have been increasing in these demographics in
recent years. I would also be interested in analyzing how these trends
differ by race, ethnicity, and socioeconomic status to see if we could
further understand what may be driving these trends in order to develop
smart policy proposals.

``` r
us.summary <- ddply(suicides.us, ~ age + sex + year, summarize, 
                    mean.rate = (sum(suicides.count)/sum(population))*100000)
plot.us <- ggplot(data=us.summary,
                  aes(y=mean.rate, x=year, color=age))
plot.us + geom_line(stat="identity", position="dodge") +
  ylab("Mean Suicide Rate per 100k Pop.") +
  xlab(NULL) +
  guides(fill = guide_legend(title = NULL)) +
  ggtitle("United States Suicide Rates by Age Group 1985-2016") +
  facet_grid(~sex)
```

    ## Warning: Width not defined. Set with `position_dodge(width = ?)`

![](suicide_rates_analysis_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->
