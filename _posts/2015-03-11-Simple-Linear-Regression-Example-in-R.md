---
layout: post
title: "Simple Linear Regression - An example using R"
date: 2015-03-11
published: false
status: process
---

Linear regression is a type of supervised statistical learning approach that is useful for predicting a quantitative response Y. It can take the form of a single regression problem (where you use only a single predictor variable X) or a multiple regression (when more than one predictor is used in the model). It is one of the simplest and most straightforward approaches available and it is a starting point for more advanced modelling and predictive exercises.

This example will illustrate the application of a linear regression exercise using one single predictor (Simple Linear Regression). Our primary focus will be on the inferential aspect of the statistical learning exercise rather than on the its predicitve aspect.

The dataset we'll used is called **Prestige** and comes from the car package `library(car)`. The **Prestige** dataset is a data frame with 102 rows and 6 columns. Each row is an observation that relates to an occupation. The columns relate to predictors such as average years of education, percentage of women in the occupation, prestige of the occupation, etc.

Let's load the required libraries and inspect the dataset:


```r
# load the required packages.
library(car) # Library containing the dataset.
library(ggplot2) # Library to create some nice looking graphs.
library(MASS) # Library for our box-cox transform down the end.
```
