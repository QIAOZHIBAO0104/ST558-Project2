ST558\_PROJECT2
================
Qiaozhi Bao
2020/10/6

# Introduction

## Describe the data

The [Online News Popularity data
set](%22https://archive.ics.uci.edu/ml/datasets/Online+News+Popularity%22)
was published two years ago to summarize a heterogeneous set of features
about articles published by Mashable in a period of two years. There are
61 variables in total from the data set above: 58 predictive attributes,
2 non-predictive and 1 goal field.More details and summarization will be
discussed later in this project. \#\# The purpose of Analysis  
The purpose of this analysis is to create two models(ensemble and not
ensemble) to generate the best predict of the response
attribute–shares.Our analysis will help to determine what kind of
content would be most popular. \#\# Methods For this project,I first
split the data into training set and test set,then I examine the data
with summary statistics and correlation plots to see the relationships
between predictive attributes and the relationship between predictive
attributes and response variables,then some meaningless variables were
moved. I then utilized the caret package to create two models.Tree-based
model chosen using leave one out cross validation.Boosted tree model
chosen using cross-validation.

# Data Study

## Description of the Used Data

``` r
# Load all libraries
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.3.2     ✓ purrr   0.3.4
    ## ✓ tibble  3.0.3     ✓ dplyr   1.0.2
    ## ✓ tidyr   1.1.2     ✓ stringr 1.4.0
    ## ✓ readr   1.3.1     ✓ forcats 0.5.0

    ## ── Conflicts ───────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(ggplot2)
library(randomForest)
```

    ## randomForest 4.6-14

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

``` r
library(caret)
```

    ## Loading required package: lattice

    ## 
    ## Attaching package: 'caret'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     lift

``` r
library(tree)
```

    ## Registered S3 method overwritten by 'tree':
    ##   method     from
    ##   print.tree cli

``` r
library(gbm)
```

    ## Loaded gbm 2.1.8

``` r
library(corrplot)
```

    ## corrplot 0.84 loaded

``` r
library(e1071)
set.seed(1)
```

``` r
# Read in data and removing the first two columns as they are not predictive variables.
news_pop <- read_csv('./OnlineNewsPopularity.csv') %>% select(-`url`,-`timedelta`)
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_double(),
    ##   url = col_character()
    ## )

    ## See spec(...) for full column specifications.

``` r
# First to see Monday data
Mon_data <- news_pop%>% filter(weekday_is_monday==1)
Mon_data <- Mon_data %>% select(!starts_with('weekday_is'))
# Check if we have missing values, answer is 'No'
sum(is.na(Mon_data))
```

    ## [1] 0

``` r
Mon_data
```

    ## # A tibble: 6,661 x 52
    ##    n_tokens_title n_tokens_content n_unique_tokens n_non_stop_words
    ##             <dbl>            <dbl>           <dbl>            <dbl>
    ##  1             12              219           0.664             1.00
    ##  2              9              255           0.605             1.00
    ##  3              9              211           0.575             1.00
    ##  4              9              531           0.504             1.00
    ##  5             13             1072           0.416             1.00
    ##  6             10              370           0.560             1.00
    ##  7              8              960           0.418             1.00
    ##  8             12              989           0.434             1.00
    ##  9             11               97           0.670             1.00
    ## 10             10              231           0.636             1.00
    ## # … with 6,651 more rows, and 48 more variables:
    ## #   n_non_stop_unique_tokens <dbl>, num_hrefs <dbl>, num_self_hrefs <dbl>,
    ## #   num_imgs <dbl>, num_videos <dbl>, average_token_length <dbl>,
    ## #   num_keywords <dbl>, data_channel_is_lifestyle <dbl>,
    ## #   data_channel_is_entertainment <dbl>, data_channel_is_bus <dbl>,
    ## #   data_channel_is_socmed <dbl>, data_channel_is_tech <dbl>,
    ## #   data_channel_is_world <dbl>, kw_min_min <dbl>, kw_max_min <dbl>,
    ## #   kw_avg_min <dbl>, kw_min_max <dbl>, kw_max_max <dbl>, kw_avg_max <dbl>,
    ## #   kw_min_avg <dbl>, kw_max_avg <dbl>, kw_avg_avg <dbl>,
    ## #   self_reference_min_shares <dbl>, self_reference_max_shares <dbl>,
    ## #   self_reference_avg_sharess <dbl>, is_weekend <dbl>, LDA_00 <dbl>,
    ## #   LDA_01 <dbl>, LDA_02 <dbl>, LDA_03 <dbl>, LDA_04 <dbl>,
    ## #   global_subjectivity <dbl>, global_sentiment_polarity <dbl>,
    ## #   global_rate_positive_words <dbl>, global_rate_negative_words <dbl>,
    ## #   rate_positive_words <dbl>, rate_negative_words <dbl>,
    ## #   avg_positive_polarity <dbl>, min_positive_polarity <dbl>,
    ## #   max_positive_polarity <dbl>, avg_negative_polarity <dbl>,
    ## #   min_negative_polarity <dbl>, max_negative_polarity <dbl>,
    ## #   title_subjectivity <dbl>, title_sentiment_polarity <dbl>,
    ## #   abs_title_subjectivity <dbl>, abs_title_sentiment_polarity <dbl>,
    ## #   shares <dbl>

As there is no missing value in our Monday data, we will step to split
data. By using sample(), with 70% of the data goes to the training set
(4,662 observations, Mon\_train) and 30% goes to the test set (1,999
observations, Mon\_test).

``` r
# Split Monday data,70% for training set and 30% for test set
set.seed(1)
train <- sample(1:nrow(Mon_data),size = nrow(Mon_data)*0.7)
test <- dplyr::setdiff(1:nrow(Mon_data),train)
train_data <-Mon_data[train,]
test_data <- Mon_data[test,]
train_data
```

    ## # A tibble: 4,662 x 52
    ##    n_tokens_title n_tokens_content n_unique_tokens n_non_stop_words
    ##             <dbl>            <dbl>           <dbl>            <dbl>
    ##  1             12              158           0.684             1.00
    ##  2              8              198           0.513             1.00
    ##  3             12              426           0.463             1.00
    ##  4             15              187           0.618             1.00
    ##  5              8              302           0.661             1.00
    ##  6             12              136           0.769             1.00
    ##  7             13              755           0.435             1.00
    ##  8             11              384           0.610             1.00
    ##  9              8              456           0.541             1.00
    ## 10             12             1056           0.438             1.00
    ## # … with 4,652 more rows, and 48 more variables:
    ## #   n_non_stop_unique_tokens <dbl>, num_hrefs <dbl>, num_self_hrefs <dbl>,
    ## #   num_imgs <dbl>, num_videos <dbl>, average_token_length <dbl>,
    ## #   num_keywords <dbl>, data_channel_is_lifestyle <dbl>,
    ## #   data_channel_is_entertainment <dbl>, data_channel_is_bus <dbl>,
    ## #   data_channel_is_socmed <dbl>, data_channel_is_tech <dbl>,
    ## #   data_channel_is_world <dbl>, kw_min_min <dbl>, kw_max_min <dbl>,
    ## #   kw_avg_min <dbl>, kw_min_max <dbl>, kw_max_max <dbl>, kw_avg_max <dbl>,
    ## #   kw_min_avg <dbl>, kw_max_avg <dbl>, kw_avg_avg <dbl>,
    ## #   self_reference_min_shares <dbl>, self_reference_max_shares <dbl>,
    ## #   self_reference_avg_sharess <dbl>, is_weekend <dbl>, LDA_00 <dbl>,
    ## #   LDA_01 <dbl>, LDA_02 <dbl>, LDA_03 <dbl>, LDA_04 <dbl>,
    ## #   global_subjectivity <dbl>, global_sentiment_polarity <dbl>,
    ## #   global_rate_positive_words <dbl>, global_rate_negative_words <dbl>,
    ## #   rate_positive_words <dbl>, rate_negative_words <dbl>,
    ## #   avg_positive_polarity <dbl>, min_positive_polarity <dbl>,
    ## #   max_positive_polarity <dbl>, avg_negative_polarity <dbl>,
    ## #   min_negative_polarity <dbl>, max_negative_polarity <dbl>,
    ## #   title_subjectivity <dbl>, title_sentiment_polarity <dbl>,
    ## #   abs_title_subjectivity <dbl>, abs_title_sentiment_polarity <dbl>,
    ## #   shares <dbl>

``` r
test_data
```

    ## # A tibble: 1,999 x 52
    ##    n_tokens_title n_tokens_content n_unique_tokens n_non_stop_words
    ##             <dbl>            <dbl>           <dbl>            <dbl>
    ##  1              9              255           0.605             1.00
    ##  2             13             1072           0.416             1.00
    ##  3             12              989           0.434             1.00
    ##  4             11               97           0.670             1.00
    ##  5             10              231           0.636             1.00
    ##  6              9             1248           0.490             1.00
    ##  7             10              187           0.667             1.00
    ##  8              9              274           0.609             1.00
    ##  9              8             1207           0.411             1.00
    ## 10             13             1248           0.391             1.00
    ## # … with 1,989 more rows, and 48 more variables:
    ## #   n_non_stop_unique_tokens <dbl>, num_hrefs <dbl>, num_self_hrefs <dbl>,
    ## #   num_imgs <dbl>, num_videos <dbl>, average_token_length <dbl>,
    ## #   num_keywords <dbl>, data_channel_is_lifestyle <dbl>,
    ## #   data_channel_is_entertainment <dbl>, data_channel_is_bus <dbl>,
    ## #   data_channel_is_socmed <dbl>, data_channel_is_tech <dbl>,
    ## #   data_channel_is_world <dbl>, kw_min_min <dbl>, kw_max_min <dbl>,
    ## #   kw_avg_min <dbl>, kw_min_max <dbl>, kw_max_max <dbl>, kw_avg_max <dbl>,
    ## #   kw_min_avg <dbl>, kw_max_avg <dbl>, kw_avg_avg <dbl>,
    ## #   self_reference_min_shares <dbl>, self_reference_max_shares <dbl>,
    ## #   self_reference_avg_sharess <dbl>, is_weekend <dbl>, LDA_00 <dbl>,
    ## #   LDA_01 <dbl>, LDA_02 <dbl>, LDA_03 <dbl>, LDA_04 <dbl>,
    ## #   global_subjectivity <dbl>, global_sentiment_polarity <dbl>,
    ## #   global_rate_positive_words <dbl>, global_rate_negative_words <dbl>,
    ## #   rate_positive_words <dbl>, rate_negative_words <dbl>,
    ## #   avg_positive_polarity <dbl>, min_positive_polarity <dbl>,
    ## #   max_positive_polarity <dbl>, avg_negative_polarity <dbl>,
    ## #   min_negative_polarity <dbl>, max_negative_polarity <dbl>,
    ## #   title_subjectivity <dbl>, title_sentiment_polarity <dbl>,
    ## #   abs_title_subjectivity <dbl>, abs_title_sentiment_polarity <dbl>,
    ## #   shares <dbl>

# Data Summarizations

## Response variable

First I plot the histogram of the response variable `shares` and found
it is a right-skewed distribution variable,then I performed
log-transformation on `shares` and plot histogram too.

``` r
# Histogram of the response variable
ggplot(data=train_data, aes(x=shares))+geom_histogram()
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](ST558_PROJECT2_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
train_data$shares <- log(train_data$shares)
ggplot(data=train_data, aes(x=shares))+geom_histogram()+ xlab('Log(shares)')
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](ST558_PROJECT2_files/figure-gfm/unnamed-chunk-6-2.png)<!-- --> \#\#
Predictor Variables  
I used the summary() function to calculate summary statistics for each
of the quantitative variables in Mon\_data.

``` r
summary(train_data)
```

    ##  n_tokens_title  n_tokens_content n_unique_tokens  n_non_stop_words
    ##  Min.   : 2.00   Min.   :   0.0   Min.   :0.0000   Min.   :0.0000  
    ##  1st Qu.: 9.00   1st Qu.: 248.0   1st Qu.:0.4738   1st Qu.:1.0000  
    ##  Median :10.00   Median : 397.5   Median :0.5427   Median :1.0000  
    ##  Mean   :10.42   Mean   : 538.2   Mean   :0.5308   Mean   :0.9691  
    ##  3rd Qu.:12.00   3rd Qu.: 711.0   3rd Qu.:0.6088   3rd Qu.:1.0000  
    ##  Max.   :18.00   Max.   :7764.0   Max.   :1.0000   Max.   :1.0000  
    ##  n_non_stop_unique_tokens   num_hrefs      num_self_hrefs      num_imgs     
    ##  Min.   :0.0000           Min.   :  0.00   Min.   : 0.000   Min.   : 0.000  
    ##  1st Qu.:0.6287           1st Qu.:  4.00   1st Qu.: 1.000   1st Qu.: 1.000  
    ##  Median :0.6939           Median :  7.00   Median : 3.000   Median : 1.000  
    ##  Mean   :0.6728           Mean   : 10.62   Mean   : 3.367   Mean   : 4.382  
    ##  3rd Qu.:0.7544           3rd Qu.: 13.00   3rd Qu.: 4.000   3rd Qu.: 3.000  
    ##  Max.   :1.0000           Max.   :162.00   Max.   :51.000   Max.   :93.000  
    ##    num_videos     average_token_length  num_keywords   
    ##  Min.   : 0.000   Min.   :0.000        Min.   : 1.000  
    ##  1st Qu.: 0.000   1st Qu.:4.475        1st Qu.: 6.000  
    ##  Median : 0.000   Median :4.656        Median : 7.000  
    ##  Mean   : 1.367   Mean   :4.536        Mean   : 7.153  
    ##  3rd Qu.: 1.000   3rd Qu.:4.840        3rd Qu.: 9.000  
    ##  Max.   :74.000   Max.   :6.513        Max.   :10.000  
    ##  data_channel_is_lifestyle data_channel_is_entertainment data_channel_is_bus
    ##  Min.   :0.00000           Min.   :0.0000                Min.   :0.0000     
    ##  1st Qu.:0.00000           1st Qu.:0.0000                1st Qu.:0.0000     
    ##  Median :0.00000           Median :0.0000                Median :0.0000     
    ##  Mean   :0.04719           Mean   :0.2059                Mean   :0.1695     
    ##  3rd Qu.:0.00000           3rd Qu.:0.0000                3rd Qu.:0.0000     
    ##  Max.   :1.00000           Max.   :1.0000                Max.   :1.0000     
    ##  data_channel_is_socmed data_channel_is_tech data_channel_is_world
    ##  Min.   :0.00000        Min.   :0.0000       Min.   :0.0000       
    ##  1st Qu.:0.00000        1st Qu.:0.0000       1st Qu.:0.0000       
    ##  Median :0.00000        Median :0.0000       Median :0.0000       
    ##  Mean   :0.05277        Mean   :0.1836       Mean   :0.2072       
    ##  3rd Qu.:0.00000        3rd Qu.:0.0000       3rd Qu.:0.0000       
    ##  Max.   :1.00000        Max.   :1.0000       Max.   :1.0000       
    ##    kw_min_min       kw_max_min       kw_avg_min        kw_min_max    
    ##  Min.   : -1.00   Min.   :     0   Min.   :   -1.0   Min.   :     0  
    ##  1st Qu.: -1.00   1st Qu.:   441   1st Qu.:  136.2   1st Qu.:     0  
    ##  Median : -1.00   Median :   651   Median :  230.5   Median :  1400  
    ##  Mean   : 26.82   Mean   :  1231   Mean   :  317.1   Mean   : 11822  
    ##  3rd Qu.:  4.00   3rd Qu.:  1000   3rd Qu.:  352.6   3rd Qu.:  7200  
    ##  Max.   :318.00   Max.   :298400   Max.   :29946.9   Max.   :690400  
    ##    kw_max_max       kw_avg_max       kw_min_avg       kw_max_avg    
    ##  Min.   :     0   Min.   :     0   Min.   :  -1.0   Min.   :     0  
    ##  1st Qu.:843300   1st Qu.:173315   1st Qu.:   0.0   1st Qu.:  3531  
    ##  Median :843300   Median :242336   Median : 994.2   Median :  4255  
    ##  Mean   :748229   Mean   :257156   Mean   :1086.4   Mean   :  5582  
    ##  3rd Qu.:843300   3rd Qu.:330765   3rd Qu.:1986.1   3rd Qu.:  5938  
    ##  Max.   :843300   Max.   :798220   Max.   :3602.1   Max.   :298400  
    ##    kw_avg_avg    self_reference_min_shares self_reference_max_shares
    ##  Min.   :    0   Min.   :     0            Min.   :     0           
    ##  1st Qu.: 2355   1st Qu.:   659            1st Qu.:  1100           
    ##  Median : 2832   Median :  1200            Median :  2800           
    ##  Mean   : 3074   Mean   :  3951            Mean   :  9970           
    ##  3rd Qu.: 3535   3rd Qu.:  2600            3rd Qu.:  7900           
    ##  Max.   :33536   Max.   :690400            Max.   :843300           
    ##  self_reference_avg_sharess   is_weekend     LDA_00            LDA_01       
    ##  Min.   :     0             Min.   :0    Min.   :0.01818   Min.   :0.01819  
    ##  1st Qu.:  1000             1st Qu.:0    1st Qu.:0.02517   1st Qu.:0.02504  
    ##  Median :  2168             Median :0    Median :0.03341   Median :0.03337  
    ##  Mean   :  6321             Mean   :0    Mean   :0.18670   Mean   :0.15456  
    ##  3rd Qu.:  5200             3rd Qu.:0    3rd Qu.:0.24603   3rd Qu.:0.17145  
    ##  Max.   :690400             Max.   :0    Max.   :0.91999   Max.   :0.91997  
    ##      LDA_02            LDA_03            LDA_04        global_subjectivity
    ##  Min.   :0.01819   Min.   :0.01819   Min.   :0.01818   Min.   :0.0000     
    ##  1st Qu.:0.02857   1st Qu.:0.02857   1st Qu.:0.02857   1st Qu.:0.3951     
    ##  Median :0.04000   Median :0.04000   Median :0.04001   Median :0.4512     
    ##  Mean   :0.21064   Mean   :0.21781   Mean   :0.23029   Mean   :0.4402     
    ##  3rd Qu.:0.32402   3rd Qu.:0.35340   3rd Qu.:0.39356   3rd Qu.:0.5047     
    ##  Max.   :0.92000   Max.   :0.91998   Max.   :0.92708   Max.   :1.0000     
    ##  global_sentiment_polarity global_rate_positive_words
    ##  Min.   :-0.38021          Min.   :0.00000           
    ##  1st Qu.: 0.05543          1st Qu.:0.02820           
    ##  Median : 0.11732          Median :0.03817           
    ##  Mean   : 0.11631          Mean   :0.03900           
    ##  3rd Qu.: 0.17457          3rd Qu.:0.04975           
    ##  Max.   : 0.55455          Max.   :0.12139           
    ##  global_rate_negative_words rate_positive_words rate_negative_words
    ##  Min.   :0.000000           Min.   :0.0000      Min.   :0.0000     
    ##  1st Qu.:0.009674           1st Qu.:0.6000      1st Qu.:0.1852     
    ##  Median :0.015303           Median :0.7059      Median :0.2857     
    ##  Mean   :0.016784           Mean   :0.6779      Mean   :0.2910     
    ##  3rd Qu.:0.021818           3rd Qu.:0.8000      3rd Qu.:0.3871     
    ##  Max.   :0.086168           Max.   :1.0000      Max.   :1.0000     
    ##  avg_positive_polarity min_positive_polarity max_positive_polarity
    ##  Min.   :0.0000        Min.   :0.00000       Min.   :0.000        
    ##  1st Qu.:0.3052        1st Qu.:0.05000       1st Qu.:0.600        
    ##  Median :0.3586        Median :0.10000       Median :0.800        
    ##  Mean   :0.3540        Mean   :0.09543       Mean   :0.757        
    ##  3rd Qu.:0.4121        3rd Qu.:0.10000       3rd Qu.:1.000        
    ##  Max.   :1.0000        Max.   :1.00000       Max.   :1.000        
    ##  avg_negative_polarity min_negative_polarity max_negative_polarity
    ##  Min.   :-1.0000       Min.   :-1.0000       Min.   :-1.000       
    ##  1st Qu.:-0.3306       1st Qu.:-0.7000       1st Qu.:-0.125       
    ##  Median :-0.2510       Median :-0.5000       Median :-0.100       
    ##  Mean   :-0.2581       Mean   :-0.5198       Mean   :-0.106       
    ##  3rd Qu.:-0.1833       3rd Qu.:-0.3000       3rd Qu.:-0.050       
    ##  Max.   : 0.0000       Max.   : 0.0000       Max.   : 0.000       
    ##  title_subjectivity title_sentiment_polarity abs_title_subjectivity
    ##  Min.   :0.0000     Min.   :-1.00000         Min.   :0.0000        
    ##  1st Qu.:0.0000     1st Qu.: 0.00000         1st Qu.:0.1500        
    ##  Median :0.1333     Median : 0.00000         Median :0.5000        
    ##  Mean   :0.2771     Mean   : 0.06694         Mean   :0.3391        
    ##  3rd Qu.:0.5000     3rd Qu.: 0.13636         3rd Qu.:0.5000        
    ##  Max.   :1.0000     Max.   : 1.00000         Max.   :0.5000        
    ##  abs_title_sentiment_polarity     shares      
    ##  Min.   :0.000                Min.   : 1.386  
    ##  1st Qu.:0.000                1st Qu.: 6.817  
    ##  Median :0.000                Median : 7.244  
    ##  Mean   :0.153                Mean   : 7.459  
    ##  3rd Qu.:0.250                3rd Qu.: 7.901  
    ##  Max.   :1.000                Max.   :13.389

``` r
correlation1 <- cor(train_data[,c(1:10,52)])
corrplot(correlation1,type='upper',tl.pos = 'lt')
corrplot(correlation1,type='lower',method = 'number',add = T,diag = F,tl.pos = 'n')
```

![](ST558_PROJECT2_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
correlation2 <- cor(train_data[,c(11:20,52)])
corrplot(correlation2,type='upper',tl.pos = 'lt')
corrplot(correlation2,type='lower',method = 'number',add = T,diag = F,tl.pos = 'n')
```

![](ST558_PROJECT2_files/figure-gfm/unnamed-chunk-7-2.png)<!-- -->

``` r
correlation3 <- cor(train_data[,c(21:30,52)])
```

    ## Warning in cor(train_data[, c(21:30, 52)]): the standard deviation is zero

``` r
corrplot(correlation3,type='upper',tl.pos = 'lt')
corrplot(correlation3,type='lower',method = 'number',add = T,diag = F,tl.pos = 'n')
```

![](ST558_PROJECT2_files/figure-gfm/unnamed-chunk-7-3.png)<!-- -->

``` r
correlation4 <- cor(train_data[,c(31:40,52)])
corrplot(correlation4,type='upper',tl.pos = 'lt')
corrplot(correlation4,type='lower',method = 'number',add = T,diag = F,tl.pos = 'n')
```

![](ST558_PROJECT2_files/figure-gfm/unnamed-chunk-7-4.png)<!-- -->

``` r
correlation5 <- cor(train_data[,c(41:51,52)])
corrplot(correlation5,type='upper',tl.pos = 'lt')
corrplot(correlation5,type='lower',method = 'number',add = T,diag = F,tl.pos = 'n')
```

![](ST558_PROJECT2_files/figure-gfm/unnamed-chunk-7-5.png)<!-- --> From
the correlation plot,I decided to remove some meaningless
variables:`kw_min_min`,`kw_avg_min`,`kw_min_avg`,`is_weekend` Also some
highly correlated variables will be removed too,then we will get a new
train set and test set

``` r
train_data <- train_data %>% select(!starts_with("LDA"),-is_weekend)
test_data <- test_data %>% select(!starts_with("LDA"),-is_weekend)
train_data <- train_data %>% select(!starts_with('kw'))
test_data <- train_data %>% select(!starts_with('kw'))
```

``` r
train_data
```

    ## # A tibble: 4,662 x 37
    ##    n_tokens_title n_tokens_content n_unique_tokens n_non_stop_words
    ##             <dbl>            <dbl>           <dbl>            <dbl>
    ##  1             12              158           0.684             1.00
    ##  2              8              198           0.513             1.00
    ##  3             12              426           0.463             1.00
    ##  4             15              187           0.618             1.00
    ##  5              8              302           0.661             1.00
    ##  6             12              136           0.769             1.00
    ##  7             13              755           0.435             1.00
    ##  8             11              384           0.610             1.00
    ##  9              8              456           0.541             1.00
    ## 10             12             1056           0.438             1.00
    ## # … with 4,652 more rows, and 33 more variables:
    ## #   n_non_stop_unique_tokens <dbl>, num_hrefs <dbl>, num_self_hrefs <dbl>,
    ## #   num_imgs <dbl>, num_videos <dbl>, average_token_length <dbl>,
    ## #   num_keywords <dbl>, data_channel_is_lifestyle <dbl>,
    ## #   data_channel_is_entertainment <dbl>, data_channel_is_bus <dbl>,
    ## #   data_channel_is_socmed <dbl>, data_channel_is_tech <dbl>,
    ## #   data_channel_is_world <dbl>, self_reference_min_shares <dbl>,
    ## #   self_reference_max_shares <dbl>, self_reference_avg_sharess <dbl>,
    ## #   global_subjectivity <dbl>, global_sentiment_polarity <dbl>,
    ## #   global_rate_positive_words <dbl>, global_rate_negative_words <dbl>,
    ## #   rate_positive_words <dbl>, rate_negative_words <dbl>,
    ## #   avg_positive_polarity <dbl>, min_positive_polarity <dbl>,
    ## #   max_positive_polarity <dbl>, avg_negative_polarity <dbl>,
    ## #   min_negative_polarity <dbl>, max_negative_polarity <dbl>,
    ## #   title_subjectivity <dbl>, title_sentiment_polarity <dbl>,
    ## #   abs_title_subjectivity <dbl>, abs_title_sentiment_polarity <dbl>,
    ## #   shares <dbl>

# First Model

## Tree based model chosen using leave one out cross validation

``` r
tree.method <- train(shares ~.,data = train_data,method='rpart',
                       preProcess = c("center","scale"),
                     trControl = trainControl(method ='LOOCV'))
```

``` r
tree.method$results
```

    ##           cp      RMSE    Rsquared       MAE
    ## 1 0.00853208 0.9669950 0.016740386 0.7327253
    ## 2 0.01015569 0.9778566 0.002148338 0.7590913
    ## 3 0.04065966 1.0251815 0.152051399 0.8103113

``` r
tree.method$bestTune
```

    ##           cp
    ## 1 0.00853208

``` r
pred.tree <- predict(tree.method,test_data)
postResample(pred.tree,test_data$shares)
```

    ##       RMSE   Rsquared        MAE 
    ## 0.93558824 0.06097103 0.69938961

# Second Model

## Boosted tree model chosen using cross-validation

``` r
# We will fit the model using repeated CV
boosted.method <- train(shares ~.,data = train_data,method = 'gbm',
                      trControl = trainControl(method = 'repeatedcv', number=5,repeats =2),
                      preProcess = c("center","scale"),
                      verbose = FALSE)
```

``` r
pred.boost <- predict(boosted.method,test_data)
boostRMSE <- sqrt(mean((pred.boost- test_data$shares)^2))
boostRMSE
```

    ## [1] 0.8810502