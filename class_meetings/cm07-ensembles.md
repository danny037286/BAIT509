# BAIT 509 Class Meeting 07
Monday, March 19, 2018  

# Outline

- Bagging and Random Forests
- In-class exercises
- Boosting
- Implementations of ensemble methods
- Lab (work on Assignment 2)

# Motivation: why ensembles?

Because a classification or regression tree alone tends to be a poor competitor against other machine learning methods. In particular, they tend to be sensitive to the data: if fit to a separate training set, a completely different tree is prone to being fit. This is an embodiment of _high variance_.

Consider the hypothetical situation: collect $B$ data sets (of equal size), and fit a tree to all $B$ of them. These $B$ models are called an __ensemble__. Then if we want to predict on a new case (i.e., we've observed the predictors but not the response), take the average predictions of the $B$ trees in the case of regression, or the popular vote of the $B$ trees in the case of classification. This will reduce variance. 

But collecting $B$ data sets is not practical, and splitting our single data set into $B$ parts would not be a substitute, because we'd only be skyrocketing the variance of each tree before then reducing it in the ensemble.

# Bagging (= Bootstrap Aggregating)

We can emulate the above hypothetical situation with a technique called the __bootstrap__. If your data set has $n$ observations, then we can draw (for all intents and purposes) any number $B$ of data sets we like. To generate one data set, just randomly sample $n$ observations __with replacement__, and ta-da!

The $B$ data sets are related in some sense, so are not as good as having $B$ _independent_ data sets. But it still gives us something useful -- in fact, tremendously useful. Go ahead and fit a tree on each data set, and combine the results -- that's bagging in a nutshell.

Note that we deliberately tend to __overfit__ each tree in the ensemble, to get trees with low bias and high variance -- the variance of which will be reduced in the ensemble. 

This concept of bootstrap is very widespread -- it's not just used for trees, and not even just for machine learning. But for BAIT 509, trees are the only context you'll see bootstrap in.

## Size of $B$

Note that we can't overfit by increasing $B$, because this just results in new data sets being generated -- not fitting more and more models to a single data set. The error (MSE or error rate) will drop as $B$ increases, until it reaches a stable point where it no longer drops. Once this point is reached, increasing $B$ does not do us much good. This is a common approach for choosing $B$ -- no need for cross validation.

## Out of Bag (OOB) error

With bagging comes a unique opportunity to calculate the out-of-sample/test error _without having to partition the data_ into training and test sets (either through the validation set method, or cross validation). Here's how.

Remember how we obtain a single bootstrap data set: sample with replacement $n$ times. This means that, for each bootstrap sample, there will be some observations that were left out. These are called __out-of-bag__ (OOB) observations, and naturally form a test set!

Now, obtain an estimate of generalization error (such as MSE or error rate) like so: for each observation in your full data set, consider the trees for which this observation is OOB, and use those to form an ensemble prediction. Compare this against the true value to get the error for this observation. Repeat for all observations.

## Predictor Importance

We can use ensembles to determine the _importance_ of certain predictors over others. Recall that the addition of a stump to a tree reduces the training error. We can set up a "points system", where a reduction in MSE is "awarded" to the predictor responsible for the stump. Do this for all nodes in a tree to come up with a final score for each predictor -- the ones with the largest scores are most important.

# Random Forests

One problem with Bagging is that the trees in the ensemble tend to be _correlated_ -- that is, they share similarities.

Random forests attempt to fix this problem by modifying how a single tree in the ensemble is grown. Recall that, when making a new stump to grow a tree, we choose one predictor out of the total $p$ predictors. The idea behind random forests is to _restrict this choice to some random subset_ of $m$ predictors out of the $p$. A new batch of $m$ predictors is selected each time a stump is to be made. 

The result is an ensemble of trees that look "more random" -- they are said to be _decorrelated_. This prevents any one predictor from "dominating" the ensemble. And because the trees are less related, combining their predictions results in an overall better result. 

# In-class exercises

Answer the following questions, and upload your results to your github repo. Remember, your answers do not have to be correct to earn participation points!

- Bagging is a special case of random forests under which case?

> When $m=p$ -- that is, when the size of the predictor subset equals the total number of predictors. This means that, when sampling the predictors, we'd end up with all the predictors. 

- What are the hyperparameters we can control for random forests?

> Tree size, number of trees in the ensemble, number of predictors to subset at each branch ($m$).

- Suppose you have the following paired data of `(x,y)`: (1,2), (1,5), (2,0). Which of the following are valid bootstrapped data sets? Why/why not?
    1. (1,0), (1,2), (1,5) 
    2. (1,2), (2,0)
    3. (1,2), (1,2), (1,5)

> 1 is not valid, because (1,0) is not part of the data set (even though x=1 and y=0 are both in the data, they don't appear _paired_). 2 is not valid, because it only contains two observations (and should contain three, the size of the original data set). 3 is valid.

- For each of the above valid bootstapped data sets, which observations are out-of-bag (OOB)?

> For 3 above, the OOB observation is (2,0), because it's in the original sample, but not in the bootstrapped sample.

- You make a random forest consisting of four trees. You obtain a new observation of predictors, and would like to predict the response. What would your prediction be in the following cases?
    1. Regression: your trees make the following four predictions: 1,1,3,3.
    2. Classification: your trees make the following four predictions: "A", "A", "B", "C".
    
> For 1 (regression), we'd take the mean as the prediction, which is 2. For 2 (classification), we'd take the mode (or the "most popular vote"), which is "A". 

# Boosting

Boosting is another method, different from random forests and bagging, but still involves combining predictions of an ensemble. 

The details are beyond the scope of this course, so we will explain the main ideas. If you truly want a more comprehensive treatment, I suggest reading [this Kaggle blog post](http://blog.kaggle.com/2017/01/23/a-kaggle-master-explains-gradient-boosting/). 

## Basic boosting

(Also see the "Motivation" part of the above Kaggle blog -- this part is not overly technical). 

Let's look at a simple two-tree boosting ensemble for regression.

1. Fit a tree to your data
2. Compute the residuals (actual minus prediction).
3. Fit a second tree _to the residuals_.

To make a prediction on a new observation, do the following:

1. Feed the predictors into the first tree to get a "preliminary" prediction.
2. Feed the predictors into the second tree to get an _adjustment_.
3. Obtain a final prediction by adding the adjustment to the preliminary prediction.

This second tree captures patterns in the data that the first tree missed, which is why boosting is so useful.

Boosting, then, is a continuation of this, fitting trees in sequence.

## A conglomerate of "weak learners"

Boosting gradually improves predictions by learning on residuals. Because of this, there is no need to build a "strong" model for each iteration -- i.e., one that does well at prediction. 

Instead, we deliberately build _weak models_ to slowly get at the structure underyling the data. We therefore build low-depth trees for each member of the ensemble.

## Boosting for Classification

The adjustments made here are less interpretable, but do follow a similar logic. Instead of learning on residuals, a consecutive tree leans on classes _re-weighted_ so that observations that are incorrectly classified get a higher weight. This is called __adaboost__. 

## Learning Rate

It turns out that we obtain a more powerful prediction if we _slow down_ the "rate of learning". We introduce a "rate of learning" hyperparameter $\lambda$ between 0 and 1. Predictions from trees are multiplied by this amount before adjusting the prediction from the previous tree. 




# Ensembles in R

## `randomForest`

We use the `randomForest` package and the `randomForest` function in R to implement random forests (and thus also bagging for classification and regression trees). The syntax follows R's regression paradigm with `randomForest(response~predictors, data)`. Let's see an example with the `mtcars` data (a default data frame in R), predicting `mpg` (a regression problem):


```r
suppressPackageStartupMessages(library(randomForest))
set.seed(40)
my_fit <- randomForest(mpg ~ ., data=mtcars)
```

Note that the `.` stands for "all other variables in the data frame".

The `plot` function is useful for giving a quick-and-dirty plot of error vs. $B$:


```r
plot(my_fit)
```

![](cm07-ensembles_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

There's stability after around $B=200$ trees.

__Warning!__ The `predict` function works differently than usual. Usually, the following two outputs would be the same:


```r
yhat1 <- predict(my_fit)
yhat2 <- predict(my_fit, newdata=mtcars)
```

But they're not. Take a look:


```r
plot(yhat1, yhat2)
```

<img src="cm07-ensembles_files/figure-html/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

What's going on here? It turns out `predict(my_fit)` _without_ specifying `newdata` gives the _out of bag predictions_, whereas the entire ensemble is used to make predictions when the `newdata` argument is specified. So the OOB error and training errors, respectively, are:


```r
mean((yhat1 - mtcars$mpg)^2)
```

```
## [1] 5.374839
```

```r
mean((yhat2 - mtcars$mpg)^2)
```

```
## [1] 1.360225
```

Here are two key parameters you can change in the `randomForest` function:

- `mtry`: The number of predictors to sample at every stump iteration.
    - Set equal to the total number of predictors to perform bagging.
- `ntree`: The number of trees to fit. 

## Boosting

Boosting is not on your Assignment 2. For your reference, I'll indicate some useful implementations here.

- For regression, you can use the `gbm` function from the `gbm` R package to do boosting. But you can't do classification beyond 2 classes.
- For classification, I recommend the `AdaBoostClassifier` from the `sklearn.ensemble` library.
    - This library also contains `RandomForestClassifier` for random forest classification.
    - For plain decision trees (again for classification), the `sklearn.tree` library has a `DecisionTreeClassifier` method.
    
# Lab

You now have the remaining tools to complete Assignment 2. Members of the teaching team will be around until the end of class to help you complete your assignment. 

__Hint__: Your `Boston` dataset lives in the `MASS` package in R.
