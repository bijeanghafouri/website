---
title: Survey raking in R with {anesrake}
author: "Bijean Ghafouri"
lastmod: 15/11/2021
output: 
  html_document:
  theme: cosmo
  highlight: tango
---

# {anesrake} R Package Tutorial 

Survey research is often faced with issues related to sample representativeness. The survey sample might be different in important ways from the true population. To adjust for these errors, we can use raking adjustment. Raking allows us to select variables in our sample that will be adjusted based on true population parameters.

The following tutorial shows how to use `{anesrake}`, an r package that computes the weights for you. 

The goal is to identify variables, often demographic, to weight on. The statistical software compares the variables in your sample to the population to compute the weights. 

For more information on the package, please refer to the following document: https://electionstudies.org/wp-content/uploads/2018/04/nes012427.pdf



Let's load the packages needed and import the data we will be working with. For this tutorial, we will be using a dataset of survey responses on the political donations. You can <a id="raw-url" href="https://raw.githubusercontent.com/bijeanghafouri/website/main/content/blog/Tutorials/anesrake/donations.csv">download the data here.</a> 

```r
# Load packages 
require(pacman)
p_load(tidyverse, anesrake, weights)

# Load data 
dat <- read_csv('donations.csv')
dat <- as.data.frame(dat)

# Set target variables as factors (important!)
dat$income <- as.factor(dat$income)
dat$education <- as.factor(dat$education)
```


# Data simulation
First, we need to find our population-level estimates. In some cases, you will have access to population-level data from which you can draw your point estimates. However, you are likely to not have direct access to these data. You will need to find these statistics from other sources. 

In this turorial, I use data from the United States census to find the proportions of each category in my variables. I find [population-level education levels here](https://www.census.gov/data/tables/2020/demo/educational-attainment/cps-detailed-tables.html), and [income levels here](https://www.statista.com/statistics/203183/percentage-distribution-of-household-income-in-the-us/). 

From census data, we can find population-level marginal proportions for the variables we will weight on. We will be weighting on two variables: income and education. However, you can (perhaps ideally) weight on more variables, including sex, ethnicity, etc. Note that the variable levels are somewhat arbitrary. In a real survey, you are most likely to categorize income and education in other ways. 


```r
# Set target variables 
income <- c('20k', '50k', '100k')
income_prop <- c(.35, .50, .15)
education <- c('highschool', 'college', 'graduate')
education_prop <- c(.376, .497, .127)
population <- data_frame(income, education, income_prop, education_prop)
```

```
## Warning: `data_frame()` was deprecated in tibble 1.1.0.
## ℹ Please use `tibble()` instead.
```


We identify the list of variables we want to weight on by creating a list of 'targets'. It is important to make sure that the variable names in the dataset and in the population are the same.

```r
# Create target list 
target <- with(population, list(
  income = weights::wpct(income, income_prop),
  education  = weights::wpct(education, education_prop)
))
```


Now that the population-level proportions are dealt with, we can take a look at our survey results. Using the `weights::wpct` function, let's estimate the proportions of our target variables in the survey. 

```r
# Weight estimation 
weights::wpct(dat$income)
```

```
##      100k       20k       50k 
## 0.2058824 0.4325260 0.3615917
```

```r
weights::wpct(dat$education)
```

```
##    college   graduate highschool 
##  0.5432526  0.3460208  0.1107266
```


How do our survey proportions compare to our population proportions?

```r
target$income
```

```
## 100k  20k  50k 
## 0.15 0.35 0.50
```

```r
target$education
```

```
##    college   graduate highschool 
##      0.497      0.127      0.376
```
Seems like we have an over-representation of respondents with an income between 50k and 100k, and an over-representation of graduate and college students. Well, this is what raking is for -- let's fix this! 



The `{anesrake}` R package uses the ANES weighting algorithm to provide weights to any sample. We computed all the necessary parameters above. All we need to do is plug-and-play. There are many function arguments we can specify that I do not include here. The  `weightvec` argument allows us to supply a vector of weights if we are using a dataset that already offers weights. We could also use the `filter` argument to supply a binary vector indicating which observations to include/exclude for weighting. For example, we may want to exclude observations where respondent did not provide an answer to the outcome question. 



```r
raking <- anesrake(target,                        # target list identified above
                    dat,                          # survey dataset 
                    dat$caseid,                   # unique identifier for each respondent (1:nrow(dat))
                    cap = 5,                      # Maximum value for any given weight
                    choosemethod = "total",       # How are parameters compared for selection (other options include 'average' and 'max')
                    type = "pctlim",              # What targets should be used to weight 
                    pctlim = 0.05                 # Threshold for deviation
                    )
```

```
## [1] "Raking converged in 26 iterations"
```

```r
raking_summary <- summary(raking)
raking_summary
```

```
## $convergence
## [1] "Complete convergence was achieved after 26 iterations"
## 
## $base.weights
## [1] "No Base Weights Were Used"
## 
## $raking.variables
## [1] "income"    "education"
## 
## $weight.summary
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  0.2557  0.5272  0.6540  1.0000  1.3487  5.0001 
## 
## $selection.method
## [1] "variable selection conducted using _pctlim_ - discrepancies selected using _total_."
## 
## $general.design.effect
## [1] 1.966204
## 
## $income
##       Target Unweighted N Unweighted %  Wtd N Wtd % Change in %  Resid. Disc.
## 100k    0.15          238    0.2058824  173.4  0.15 -0.05588235 -3.330669e-16
## 20k     0.35          500    0.4325260  404.6  0.35 -0.08252595 -2.331468e-15
## 50k     0.50          418    0.3615917  578.0  0.50  0.13840830  2.664535e-15
## Total   1.00         1156    1.0000000 1156.0  1.00  0.27681661  5.329071e-15
##       Orig. Disc.
## 100k  -0.05588235
## 20k   -0.08252595
## 50k    0.13840830
## Total  0.27681661
## 
## $education
##            Target Unweighted N Unweighted %    Wtd N Wtd % Change in %
## college     0.497          628    0.5432526  574.532 0.497  -0.0462526
## graduate    0.127          400    0.3460208  146.812 0.127  -0.2190208
## highschool  0.376          128    0.1107266  434.656 0.376   0.2652734
## Total       1.000         1156    1.0000000 1156.000 1.000   0.5305467
##             Resid. Disc. Orig. Disc.
## college    -5.551115e-17  -0.0462526
## graduate    1.387779e-16  -0.2190208
## highschool  0.000000e+00   0.2652734
## Total       1.942890e-16   0.5305467
```



We've not got our raking results. Let's look at the specifics for each target variable. 

```r
raking_summary$income
```

```
##       Target Unweighted N Unweighted %  Wtd N Wtd % Change in %  Resid. Disc.
## 100k    0.15          238    0.2058824  173.4  0.15 -0.05588235 -3.330669e-16
## 20k     0.35          500    0.4325260  404.6  0.35 -0.08252595 -2.331468e-15
## 50k     0.50          418    0.3615917  578.0  0.50  0.13840830  2.664535e-15
## Total   1.00         1156    1.0000000 1156.0  1.00  0.27681661  5.329071e-15
##       Orig. Disc.
## 100k  -0.05588235
## 20k   -0.08252595
## 50k    0.13840830
## Total  0.27681661
```

```r
raking_summary$education
```

```
##            Target Unweighted N Unweighted %    Wtd N Wtd % Change in %
## college     0.497          628    0.5432526  574.532 0.497  -0.0462526
## graduate    0.127          400    0.3460208  146.812 0.127  -0.2190208
## highschool  0.376          128    0.1107266  434.656 0.376   0.2652734
## Total       1.000         1156    1.0000000 1156.000 1.000   0.5305467
##             Resid. Disc. Orig. Disc.
## college    -5.551115e-17  -0.0462526
## graduate    1.387779e-16  -0.2190208
## highschool  0.000000e+00   0.2652734
## Total       1.942890e-16   0.5305467
```

We can also look at the general effect weighting had on our sample. We see that weighting caused a 96.6% increase in the variance. 

```r
raking_summary$general.design.effect
```

```
## [1] 1.966204
```


With these weights, we are able to attach the weights vector to our survey data.

```r
# Create weights vector and attach to dataset 
dat$weights <- raking$weightvec
```



What is the effect of weighting on our outcome? (In this example, the outcome refers to donations. 1 = respondent would donate to a political party, 0 = respondent would not donate to a political party)

```r
wpct(dat$yhat)
```

```
##         0         1 
## 0.8132635 0.1867365
```

```r
wpct(dat$yhat, dat$weights)
```

```
##         0         1 
## 0.8623287 0.1376713
```

If we compare the above result with our original outcome, we see that our sample overestimates the proportion of individuals that would donate to a political party. This is likely caused by an oversample of college graduates and high-income earners. 


There we go! We now know how to weight our survey samples with the `{anesrake}` R package.
