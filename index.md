## Exponential-Smoothing

We will be using the "forecast" package in R
```
library(forecast)
```

Consider the monthly sales of computer software packages of undergraduate college-level education curriculum development. The sale of these programs started in 1976. The data for January 1976 to December 1981 (n=72) are given below
```
sales <- c(2, 2, 4, 12, 0, 81, 58, 59, 25, 22, 87, 35, 22, 23, 10, 6, 7, 31, 132, 21, 48, 56, 87, 48, 69, 97, 155, 96, 62, 100, 61, 101, 79, 72, 49, 112, 104, 216, 115, 215, 178, 233, 239, 217, 196, 228, 164, 151, 194, 152, 114, 114, 151, 205, 292, 180, 213, 220, 193, 135, 317, 305, 298, 315, 231, 361, 353, 256, 257, 340, 438, 442)
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



### Simple Exponential 
