![coreSim logo](img/coreSim_logo.png)

Version: 0.1.0 [![Build Status](https://travis-ci.org/christophergandrud/coreSim.svg?branch=master)](https://travis-ci.org/christophergandrud/coreSim)

> Core functionality for simulating quantities of interest from generalised linear 
models.

## Purpose

Using simulations to find quantities of interest and associated uncertainty
can be an effective way of showing substantively meaningful results from 
generalised linear models (GLM). 

This R package provides core functions that can serve as the back bone to other
packages for finding and plotting simulated quantities of interest from GLMs.

## Steps

1. Estimate your model using whatever GLM model fitting function you like 
(note: I've only tested `lm`, `glm`, and `survival`).

2. Simulate coefficients with `b_sim`.

3. Find your quantities of intest with `qi_builder`.

4. Present your results, e.g. by plotting the simulated quantities of 
interest.

## Example: Normal linear model 

Here is an example using data from the **car** package:


```r
library(coreSim)
library(car)

# Normal linear model
m1 <- lm(prestige ~ education + type, data = Prestige)
# Simulate coefficients
m1_sims <- b_sim(m1)

# Create fitted values
fitted_df_1 <- expand.grid(education = 6:16, typewc = 1)

# Find predicted outcomes (95% central interval, by default)
linear_qi <- qi_builder(b_sims = m1_sims, newdata = fitted_df_1)
```

```
## Note: model argument missing -> assuming normal linear model.
```

```r
head(linear_qi)
```

```
##   education typewc      qi_
## 1         6      1 21.00803
## 2         6      1 23.28625
## 3         6      1 22.20055
## 4         6      1 21.78379
## 5         6      1 17.61408
## 6         6      1 19.33084
```

## Example: Predicted probabilities from logistic regressions

By default `qi_builder` simply returns the linear systematic component, which 
in normal linear regression is simply the predicted y, i.e. 
$y = \alpha + \mathrm{\beta X}$.

`qi_builder` allows you to supply any function for creating quantities of 
interest that you would like. This function needs to simply be able to convert
a vector of linear systematic components to your quantity of interest. 

For example, to find predicted probabilities from a logistic regression model
create a function to turn the systematic component into the QI:


```r
pr_fun <- function(x) 1 / (1 + exp(x))
```

Then supply it to `qi_builder`'s `model` argument:


```r
# Download data
URL <- 'http://www.ats.ucla.edu/stat/data/binary.csv'
Admission <- read.csv(URL)
Admission$rank <- as.factor(Admission$rank)

# Estimate model
m2 <- glm(admit ~ gre + gpa + rank, data = Admission, family = 'binomial')

# Simulate coefficients
m2_sims <- b_sim(m2)

# Create fitted values
m2_fitted <- expand.grid(gre = seq(220, 800, by = 10), gpa = c(2, 4),
                         rank4 = 1)

# Find quantity of interest
logistic_qi <- qi_builder(m2_sims, m2_fitted, model = pr_fun)

head(logistic_qi)
```

```
##   gre gpa rank4       qi_
## 1 220   2     1 0.3637113
## 2 220   2     1 0.1920045
## 3 220   2     1 0.4539414
## 4 220   2     1 0.1655613
## 5 220   2     1 0.6036712
## 6 220   2     1 0.5079155
```



## See also

Christopher Gandrud (2015). simPH: An R Package for Illustrating Estimates from
Cox Proportional Hazard Models Including for Interactive and Nonlinear Effects.
Journal of Statistical Software, 65(3), 1-20.
<http://www.jstatsoft.org/v65/i03/>.

Gandrud, Christopher. Laron K. Williams and Guy D. Whitten (2015). dynsim: Dynamic
Simulations of Autoregressive Relationships. R package version 1.2.1.
<https://CRAN.R-project.org/package=dynsim>.

King, Gary, Michael Tomz, and Jason Wittenberg. 2000. "Making the Most of 
Statistical Analyses: Improving Interpretation and Presentation." American 
Journal of Political Science 44(2): 341-55.

