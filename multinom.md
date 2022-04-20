Multinomial model
================

Let’s create a multinomial logistic regression model to get some
inferences about what affects a player’s choice of offer.

## Imports

We will need the nnet package.

``` r
library(nnet)
```

## p-values

Lets make a function that will return p-values from the result of the
regression.

``` r
multinom_p <- function(model){
  model_summary <- summary(model)
  model_summary$coefficients
  z <- model_summary$coefficients / model_summary$standard.errors
  p <- (1 - pnorm(abs(z), 0, 1)) * 2
  return(p)
}
```

## Relevel

We will relevel the response so we can get the results for just low and high offers.

``` r
data$OfferChosen <- relevel(data$OfferChosen, "Middle")
```

## Model

Now, lets run the model.

``` r
model <- multinom(formula = OfferChosen ~ OC + CashBuilder + RP +
                            OC:CashBuilder + OC:RP + CashBuilder:RP +
                            Banked + Home + Banked:Home +
                            Male + Male:CashBuilder,
                  data)
```

### Coefficients 
```r
beta <- summary(model)$coefficients
beta
```

### p-values
```r
beta <- summary(model)$coefficients
multinom_p(model)
```
