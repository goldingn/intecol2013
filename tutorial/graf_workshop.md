GRaF R workshop - INTECOL 2013
=====


This file contains the code and outputs for a worked example of the ```GRaF``` R package for species distribution modelling.

First, we cover parameters and settings that can be used to fit GRaF models and then apply the model to some real-life data.


Getting started
----

First we'll create a 'true' underlying function and use it to fake some data to play with. We set the seed for the random number generator to ensure we can reproduce these figures later


```r
# true function
f <- function(x) pnorm(sin(x/4 + 3) + 0.5 * sin(x/2 - 10))

# set the RNG
set.seed(123)

# Sample presence/ absence at 100 different temperatures
n <- 100
temp <- runif(n, 10, 28)

PA <- rbinom(n, 1, f(temp))
```


Visualise this fake data


```r
temp_seq <- seq(min(temp), max(temp), len = 100)

# plot the true function
plot(f(temp_seq) ~ temp_seq, type = "l", ylim = c(0, 1), lwd = 2, col = "dark grey", 
    ylab = "probability of presence", xlab = "temperature")

# add the observed points
points(PA ~ temp, pch = 16, col = rgb(0, 0, 0, 0.5))
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 



Fitting a model
----

We can fit a simple ```GRaF``` model to this data using the ```graf``` function. Note that the second argument has to be a dataframe, not a vector or a matrix.

```r
# load GRaF (install it first if you don't already have it)
# install.packages('GRaF')
library(GRaF)
```

```
## Loading required package: dismo
```

```
## Loading required package: raster
```

```
## Loading required package: sp
```

```r
m1 <- graf(PA, data.frame(temp))
```


We can quickly plot the fitted model.


```r
plot(m1)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 


By default the ```plot``` method plots the modelled probability of presence along the first covariate and adds 95% credible intervals (the Bayesian interpretation of confidence intervals) as a shaded grey area and the data used to fit the model as rug plots. See ```?plot.graf``` for more options.

### Changing the lengthscales

GRaF can fit models of varying complexity (wiggliness of the predicted probability along a covariate) and this complexity is controlled by a set of model 'hyperparameters' (parameters which control other parameters) referred to as lengthscales. There is one lengthscale for each covariate in the model and they must be positive. A large lengthscale gives a model with low complexity (flatter) and a small lengthscale gives a model with high complexity (wigglier).

By default, ```graf``` makes a simple (pretty arbitrary) guess at what the correct lengthscales might be, but we can specify different lengthscales through the ```l``` argument to ```graf```. The lengthscale used to fit the model can be accessed from the model object.


```r
m1$l
```

```
## [1] 7.222
```


We can visualise models fitted with different values of ```l```


```r
# fit models with short and long lengthscales
m1.short <- graf(PA, data.frame(temp), l = 0.05)
m1.long <- graf(PA, data.frame(temp), l = 500)

par(mfrow = c(1, 3))
plot(m1.short, main = paste("lengthscale", m1.short$l))
plot(m1, main = paste("lengthscale", round(m1$l, 2)))
plot(m1.long, main = paste("lengthscale", m1.long$l))
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 


As well as visually assessing the fit of models with different lengthscales, we can make quantitative comparisons between models by calculating the deviance information criterion (DIC) of each model with the ```DIC``` function.


```r
DIC(m1.short)
```

```
##     DIC      pD 
## 112.494   9.354
```

```r
DIC(m1)
```

```
##     DIC      pD 
## 125.832   2.124
```

```r
DIC(m1.long)
```

```
##     DIC      pD 
## 128.923   1.091
```


DIC (like other other information criteria, such as AIC) trades off the ability of the model to fit the data against its complexity. With most other information criteria, model complexity is determined by the number of free parameters. In many Bayesian models the number of free parameters isn't well defined, so DIC instead calculates the number of 'effective' parameters. This number of effective parameters is what is given by the ```DIC``` function as ```pD```.

Here, the model ```m1.long``` has a ```pD``` of around 1 and so has similar complexity to a binomial regression model with a single regression parameter. ```m1.short``` has around 7 effective parameters, so has similar complexity to a seventh-order polynomial glm.

### Optimising the lengthscales

In practice, it's often preferable to automatically select the best lengthscale hyperparameters and ```graf``` does this by numerical optimisation.

Rather than minimising DIC, ```graf``` selects ```l``` using the model marginal likelihood. As with DIC, the marginal likelihood trades off complexity against fit to the data. As a result GRaF is able to fit complex models whilst preventing overfitting to the data in a similar way to regularization methods, like the lasso penalty used in MaxEnt.

GRaF also considers how realistic different lengthscales are for species distribution data, using a prior distribution to determine the realistic range of values that ```l``` could take. The prior distribution is discussed in [Golding & Purse (in preparation)](preprintlink) and its parameters of this prior distribution can be changed via the ```theta.prior.pars``` argument, though we won't discuss this in any more detail here.

We can fit a ```graf``` model with optimized lengthscales by specifying the ```opt.l``` argument. Note that this procedure fits lots of different models, so it takes longer to run. 


```r
m2 <- graf(PA, data.frame(temp), opt.l = TRUE)

# lengthscale
m2$l
```

```
## [1] 1.133
```

```r
# DIC and effective parameters
DIC(m2)
```

```
##     DIC      pD 
## 114.134   3.405
```

```r
# plot the fitted model
plot(m2)
lines(f(temp_seq) ~ temp_seq, lty = 2)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 



### Specifying the prior model

As well as the lengthscales, we need to provide GRaF with a prior model, giving our best *a priori* (i.e. before we look at the data) guess at the 'true' underlying model. The prior model acts in the same way as a prior distribution in other Bayesian models and can be used to incorporate existing knowledge about the species' ecology into the SDM.

By default, ```graf``` uses a flat mean function, reflecting a prior belief that the probability of presence is the same everywhere, regardless of the covariates. We can specify just about any prior model we like, by writing it as an R function.

For example, a previous study may have suggested that the probability of presence drops off rapidly above a threshold of 25 degrees:


```r
thresh <- function(x) ifelse(x$temp < 25, 0.7, 0.3)
# (plogis is the logit link function)
m3 <- graf(PA, data.frame(temp), opt.l = TRUE, prior = thresh)
```


or we can use the predict function of another model. Here we estimate the prior by first fitting a glm to the same data (from an orthodox Bayesian perspective this is cheating, but 'empirical Bayes' approaches like this can be very effective).


```r
m.lin <- glm(PA ~ temp, data = data.frame(temp), family = binomial)
lin <- function(temp) predict(m.lin, temp, type = "response")
m4 <- graf(PA, data.frame(temp), opt.l = TRUE, prior = lin)
```


We can show the mean function when using ```plot``` by setting ```prior = TRUE```.


```r
par(mfrow = c(1, 3))

plot(m2, prior = TRUE, main = "flat prior")
plot(m3, prior = TRUE, main = "threshold prior")
plot(m4, prior = TRUE, main = "linear prior")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 


As with all Bayesian models, where lots of data is available the prior has little effect, but where there is limited data it becomes important. Notice that the middle sections of the three curves above are similar, but at low and high temperatures they're pulled back towards their respective priors.

Note that it doesn't make sense to compare models with different priors by DIC.

### Measurement error

GRaF has a built in capacity to account for uncertainty in continuous environmental variables via a measurement error model. This can be useful where we know that environmental variables are measured with a fixed error, or where the covariates are an average across time or space and the standard deviations can be calculated from the same data.

To fit a measurement error model we simply need to supply ```graf``` with a matrix of standard deviations (with the same number of rows and columns as the continuous covariates) via the ```error``` argument. We can visualise the effect of incorporating measurement error by fitting models with different standard deviations:


```r
# create a matrix of sd = 1
error <- matrix(rep(1, n), ncol = 1)

m5.01 <- graf(PA, data.frame(temp), opt.l = TRUE, error = error * 0.1)

m5.05 <- graf(PA, data.frame(temp), opt.l = TRUE, error = error * 0.5)

m5.2 <- graf(PA, data.frame(temp), opt.l = TRUE, error = error * 2)

par(mfrow = c(1, 3))
plot(m5.01, main = "sd = 0.1")
plot(m5.05, main = "sd = 0.5")
plot(m5.2, main = "sd = 2")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 



### real-life example

Now we'll apply GRaF to analyse the distribution of short-finned eels (*Anguilla australis*) in New zealand, using data available in the dismo package.

First we'll load the data and subset it to simplify things. We'll keep two continuous variables (summer and winter temperatures) and discrete variable (whether there is a dam downstream). To make sure ````graf``` handles the discrete variable correctly, we'll make it a factor.



```r
data(Anguilla_train)
ang <- na.omit(Anguilla_train[, c(2:12, 14)])
ang$DSDam <- factor(ang$DSDam)
ang <- ang[1:100, ]
```



We fit a model, optimising the lengthscales. Note that when we optimise the lengthscales, factors are assigned an arbitrary low lengthscale of 0.01


```r
m.ang1 <- graf(ang$Angaus, ang[, -1], opt.l = TRUE)

par(mfrow = c(4, 3))
plot(m.ang1)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14.png) 




```r
par(mfrow = c(2, 2), mar = c(1, 1, 1, 1))
plot3d(m.ang1, c(1, 11), theta = -60)
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15.png) 

Levels of the discrete variable DSDam are considered to be independent, like in a maximum likelihood regression model. Therefore they have no lengthscale.


```r
m.ang1$l
```

```
##  [1]  3.551 12.426 10.848 13.168 11.919 12.181 12.722 12.693 13.421  0.010
## [11]  7.746
```




```r
data(Anguilla_grids)

# remove loc sed since there are lots of NAs
pred.dat <- data.frame(Anguilla_grids[])

# make it match-up with the training dataset
pred.dat$DSDam <- factor(pred.dat$DSDam)
pred.dat <- pred.dat[, order(order(names(ang[, -1])))]

p <- predict(m.ang1, pred.dat)
head(p)
```

```
##      posterior mode lower 95% CI upper 95% CI
## [1,]             NA           NA           NA
## [2,]             NA           NA           NA
## [3,]             NA           NA           NA
## [4,]             NA           NA           NA
## [5,]             NA           NA           NA
## [6,]             NA           NA           NA
```



```r
pred.mat <- Anguilla_grids[[1:3]]
pred.mat[] <- p
plot(pred.mat, zlim = c(0, 1), col = topo.colors(10))
```

![plot of chunk unnamed-chunk-18](figure/unnamed-chunk-18.png) 


which parameters are important (w/ DIC)


prediction with uncertainty
