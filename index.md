## Exponential-Smoothing

### Define problem and generate data
Consider the monthly sales of computer software packages of undergraduate college-level education curriculum development. The sale of these programs started in 1976. The data for January 1976 to December 1981 (n=72) are given below
```
sales <- c(2, 2, 4, 12, 0, 81, 58, 59, 25, 22, 87, 35, 22, 23, 10, 6, 7, 31, 132, 21, 48, 56, 87, 48, 69, 97, 155, 96, 62, 100, 61, 101, 79, 72, 49, 112, 104, 216, 115, 215, 178, 233, 239, 217, 196, 228, 164, 151, 194, 152, 114, 114, 151, 205, 292, 180, 213, 220, 193, 135, 317, 305, 298, 315, 231, 361, 353, 256, 257, 340, 438, 442)
```

We will be using the "forecast" package in R
```
library(forecast)
```

Prepare the data for time series analysis
```
> salestimeseries <- ts(sales, frequency=12, start=c(1976, 1))
> salestimeseries
     Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
1976   2   2   4  12   0  81  58  59  25  22  87  35
1977  22  23  10   6   7  31 132  21  48  56  87  48
1978  69  97 155  96  62 100  61 101  79  72  49 112
1979 104 216 115 215 178 233 239 217 196 228 164 151
1980 194 152 114 114 151 205 292 180 213 220 193 135
1981 317 305 298 315 231 361 353 256 257 340 438 442
```

Plot data
```
> plot(salestimeseries, xlab="Year", ylab="Sales", main="Monthly Sales of Computer Software Package")
```
![original resid dist](https://github.com/xinyix/Exponential-Smoothing/blob/master/sales.png?raw=true)


### Determine whether simple, double or triple exponential smoothing is appropriate for this data set
From a look at the plot, we found there is a upward trend and possible seasonality in our data. So the better choices are either double (considers trend element) or triple exponential smoothing (considers trend and seasonality). We will justify our belief by taking the first 5 year worth of data as training set, and the rest as testing set. We will see which of the three methods gives the smallest mean squared error.  

We first determine the optimal smoothing constant (when n=72) for each of the three methods
```
## Simple Exponential Smoothing
> single <- ses(salestimeseries, h=1, initial="simple", alpha=NULL)
> summary(single)

Forecast method: Simple exponential smoothing

Model Information:
Simple exponential smoothing 

Call:
 ses(y = salestimeseries, h = 1, initial = "simple", alpha = NULL) 

  Smoothing parameters:
    alpha = 0.5022 
    
    
## Double Exponential Smoothing
> double <- holt(salestimeseries, h=1, initial="simple", alpha=NULL, beta=NULL)
> summary(double)

Forecast method: Holt's method

Model Information:
Holt's method 

Call:
 holt(y = salestimeseries, h = 1, initial = "simple", alpha = NULL, beta = NULL) 

  Smoothing parameters:
    alpha = 0.4123 
    beta  = 0.0315 


## Triple Exponential Smoothing
> triple <- hw(salestimeseries, h=1, initial="simple", alpha=NULL, beta=NULL, gamma=NULL)
> summary(triple)

Forecast method: Holt-Winters' additive method

Model Information:
Holt-Winters' additive method 

Call:
 hw(y = salestimeseries, h = 1, initial = "simple", alpha = NULL, beta = NULL,  

 Call:
     gamma = NULL) 

  Smoothing parameters:
    alpha = 0.4123 
    beta  = 0.0315 
```

From the function calls above, we have alpha_ses=0.5022, alpha_holt=alpha_hw=0.4123. I also included a plot to illustrate the optimal smoothing constants graphically (R code in Appendix)
![original resid dist](https://github.com/xinyix/Exponential-Smoothing/blob/master/alphas.png?raw=true)

Now we perform one-step-ahead forecasts for simple, double and triple exponential smoothing, starting from taking the first five year of data for training, and calculate the calculate the forecast MSE
```
## Simple Exponential Smoothing
train <- sales[1:60]
forecasts <- vector(mode="numeric", length=12)
for (k in 1:12) {
	ses.obj <- ses(train, h=1, initial="simple", alpha=NULL)
	forecasts[k] <- as.numeric(ses.obj$mean)
	train <- sales[1:(60+k)]
}

err <- 0
for (i in 1:12) {
	err = err + (sales[60+i]-forecasts[i])^2
}
mse <- err/12
> mse
[1] 6502.202


## Double Exponential Smoothing 
train <- sales[1:60]
forecasts <- vector(mode="numeric", length=12)
for (k in 1:12) {
	holt.obj <- holt(train, h=1, initial="simple", alpha=NULL, beta=NULL)
	forecasts[k] <- as.numeric(holt.obj$mean)
	train <- sales[1:(60+k)]
}
err <- 0
for (i in 1:12) {
	err = err + (sales[60+i]-forecasts[i])^2
}
mse <- err/12

> mse
[1] 6193.2


## Triple Exponential Smoothing
train <- sales[1:60]
forecasts <- vector(mode="numeric", length=12)
for (k in 1:12) {
	hw.obj <- hw(train, h=1, initial="simple", alpha=NULL, beta=NULL, gamma=NULL)
	forecasts[k] <- as.numeric(hw.obj$mean)
	train <- sales[1:(60+k)]
}

err <- 0
for (i in 1:12) {
	err = err + (sales[60+i]-forecasts[i])^2
}
mse <- err/12
> mse
[1] 6193.2
```
Thus we can conclude in our case double and triple exponential smoothing perform better than simple exponential smoothing. We also notice the forecast MSE of double and triple exponential smoothing are the same. This is due to automatic parameter optimization of the Holt-Winters function, gamma is ignored. This means there are no significant seasonal component in our data.

## Appendix
```
## define a sequence of alphas
alphas <- seq(0.1, 0.99, length.out=100)
 
## for each method, define functions for calculating fitted MSE
SesMseOut <- function (x) {
	ses.obj <- ses(salestimeseries, h=1, initial="simple", alpha=x)
 	mse <- sum(ses.obj$residuals^2)/length(ses.obj$residuals)
 	return(mse)
} 

HoltMseOut <- function (x) {
	holt.obj <- holt(salestimeseries, h=1, initial="simple", alpha=x, beta=NULL)
 	mse <- sum(holt.obj$residuals^2)/length(holt.obj$residuals)
 	return(mse)
} 

HoltWintersMseOut <- function (x) {
	hw.obj <- hw(salestimeseries, h=1, initial="simple", alpha=x, beta=NULL, gamma=NULL)
 	mse <- sum(hw.obj$residuals^2)/length(hw.obj$residuals)
 	return(mse)
} 

## return values
ses.mse <- lapply(alphas, SesMseOut)
holt.mse <- lapply(alphas, HoltMseOut)
hw.mse <- lapply(alphas, HoltWintersMseOut)

## plot results
plot(alphas, ses.mse, ylab="Fitted MSE", ylim=c(2000, 4500), pch=18, type="b", col="red")
par(new=TRUE)
plot(alphas, holt.mse, ylab="", ylim=c(2000, 4500), pch=13, type="b", col="blue")
par(new=TRUE)
plot(alphas, hw.mse, ylab="", ylim=c(2000, 4500), pch=19, type="b", col="yellow")
legend(0.6, 4000, legend=c("ses", "holt", "hw"), col=c("red", "blue", "yellow"), lty=1:2)
 
```
