---
layout: post
title:  "Kaggle Otto Group Challenge - An initial Random Forest approach"
date: 2015-04-01
excerpt: "Thoughts and a simple approach to a Kaggle competition using Random Forest in Parallel"
image: "/images/kaggleotto1.jpg"
permalink: /blog/2015/04/01/Kaggle's-Otto-Group-Challenge-RandomForestExample
---


{% include advertisements.html %}


A couple of weeks ago, I decided to join [my 3rd Kaggle competition](https://www.kaggle.com/users/21458/feliperego): the [Otto Group Product Classification Challenge](https://www.kaggle.com/c/otto-group-product-classification-challenge). To me, participating on Kaggle competitions is a great way to learn and apply machine learning with different real-world datasets.

The Otto Group Product Classification Challenge is a competition sponsored by the Otto Group that asks participants to build a predictive model which is capable of classifying a list of more than 200,000 products with 93 features into their correct product categories.

The evaluation is done on the multi-class logarithmic loss metric (**logloss**). In the dataset, each product has been labelled with one true category and for each observation, you submit a predicted probability for each class.

This competition has gained massive popularity and as I'm writing this, there are more than 1,625 teams participating on the competition!

I've recently joined forces with my fellow team mate Saeh once again (we're doing ok on the top 25% of the leader board as of now). But my first submissions were done individually using simple Random Forest algorithms.

These algorithms were fairly straight-forward to implement in R and can be a good starting point for beating the competition's benchmark.

In this post, I share with you the most simplistic version of the algorithm I initially used (a few days ago it was putting me on a good spot on the leader board, but since then other approaches are working better). You'll note how simplistic the approach was, particularly around the resampling side of things, but nevertheless, it was a fun starting point!


{% include advertisements.html %}


After loading both the training and testing datasets up, I loaded all packages I was planning to use:


```r
# Load libraries

library(randomForest)
library(doParallel)
library(foreach)
library(caret)**
```

Note that we'll do some parallel processing to make use of the full power of the 8-core, 24GB RAM machine I'm using.

I then go on to create my *logloss*  function which will take the actuals and the predictions and in order to avoid the extremes of the log function, the predicted probabilities are replaced with max(min(p,1−10^−15),10^−15) in the function:


```r
# logloss evaluation metric

logloss = function(actual, predicted, eps=0.00001) {
  predicted = pmin(pmax(predicted, eps), 1-eps) - sum(actual*log(predicted))/nrow(actual)
}
```

I then create an internal train and test set from the training data for validation purpose:


```r
# Split data into internal tr_dt (training) and te_dt (testing) data (75/25)

trainRows = sample(1:nrow(dt), 0.75*nrow(dt))
tr_dt = dt[trainRows, ]
te_dt = dt[-trainRows, ]
```

As a starting point, here I use a simplistic resampling approach with the validation set and an arbitrary 75/25 split. Note that from this point on, I train my random forest straight on without any pre-processing or consideration for any further feature engineering:


```r
# call parallel processing
cl <- makeCluster(8)
registerDoParallel(cl)

# train a random forest on the internal training set
set.seed(12631)

rf = foreach(ntree=rep(1000,8), .combine=combine,
    	.multicombine=TRUE, .packages="randomForest") %dopar% {
			randomForest(x=tr_dt[ ,c(2:94)], y=tr_dt[ ,c(95)],
						data=tr_dt[,-1], ntree=ntree, do.trace=1, importance=TRUE,
						replace=TRUE, forest=TRUE)
}

# inspect the results of the model
rf

# stop 8-core parallel processing
stopCluster(cl)
```

Note from the above code chunk that we call *doParallel* for parallel processing using the 8-cores I've got available. This considerably speeds up my classifier algorithm. Here we are doing 1000 trees in parallel. After a few minutes running (I think it took roughly 10-15 min to finish) we inspect the results of the model and stop the cluster.

Next, we validate the model on the internal test set to get an idea of the error (we transform the test dataset to pass it onto the logloss function to calculate the error):


```r
# validate predictions on internal test set
rfPred = predict(rf, te_dt[,-1], type="prob")

# transform classes into dummy variables and compute log loss on rf from the internal test dataset
transf_classes <- dummyVars(~ target, data=te_dt, levelsOnly=TRUE)
classes1to9 <- predict(transf_classes,te_dt)
logloss(classes1to9, rfPred)
```

The final step is to create the submission file with the predicted probabilities across all 9 classes:


```r
# submission

test = read.csv("test.csv")
rfPredTest = predict(rf, test, type = "prob")
id = 1:nrow(test)
options("scipen"=100, "digits"=4)
submission = cbind(id, rfPredTest)
write.csv(submission, "submission.csv", row.names=FALSE, quote=FALSE)
```

The above algorithm yielded something like 0.57 on the leader board (now that would probably put you on somewhere in the middle of the leader board).

Note that so many improvements could have been made by thinking about the resampling process a bit further (maybe using the caret package?). There could have been further features created or some variable transformation considered.

These are only some of the things we (and I mean all competitors I'd imagine...) are thinking at the moment in the competition.


{% include advertisements.html %}

***

In this post I wanted to briefly illustrate the power and simplicity of using R to implement a very crude machine learning algorithm to tackle a real-world problem in light of the Otto Group's Kaggle competition.
