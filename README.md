
R/`drtmle`
==========

[![Travis-CI Build Status](https://travis-ci.org/benkeser/drtmle.svg?branch=master)](https://travis-ci.org/benkeser/drtmle) [![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/benkeser/drtmle?branch=master&svg=true)](https://ci.appveyor.com/project/benkeser/drtmle) [![Coverage Status](https://img.shields.io/codecov/c/github/benkeser/drtmle/master.svg)](https://codecov.io/github/benkeser/drtmle?branch=master) [![CRAN](http://www.r-pkg.org/badges/version/drtmle)](http://www.r-pkg.org/pkg/drtmle) [![CRAN downloads](https://cranlogs.r-pkg.org/badges/drtmle)](https://CRAN.R-project.org/package=drtmle) [![Project Status: Active - The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active) [![MIT license](http://img.shields.io/badge/license-MIT-brightgreen.svg)](http://opensource.org/licenses/MIT) [![DOI](https://zenodo.org/badge/75324341.svg)](https://zenodo.org/badge/latestdoi/75324341)

> Nonparametric estimators of the average treatment effect with doubly-robust confidence intervals and hypothesis tests

**Author:** [David Benkeser](https://www.benkeserstatistics.com/)

------------------------------------------------------------------------

Description
-----------

`drtmle` is an R package that computes marginal means of an outcome under fixed levels of a treatment. The package computes targeted minimum loss-based (TMLE) estimators that are doubly robust, not only with respect to consistency, but also with respect to asymptotic normality, as discussed in Benkeser, Carone, van der Laan & Gilbert, 2017 (*accepted Biometrika*; [working paper](http://biostats.bepress.com/ucbbiostat/paper356/)). This property facilitates construction of doubly-robust confidence intervals and hypothesis tests.

The package additionally includes methods for computing valid confidence intervals for an inverse probability of treatment weighted (IPTW) estimator of the average treatment effect when the propensity score is estimated via super learning, as discussed in [van der Laan, 2014](https://www.degruyter.com/downloadpdf/j/ijb.2014.10.issue-1/ijb-2012-0038/ijb-2012-0038.pdf).

------------------------------------------------------------------------

Installation
------------

Install the current stable release from [CRAN](https://cran.r-project.org/) via

``` r
install.packages("drtmle")
```

A developmental release may be installed from GitHub via [`devtools`](https://www.rstudio.com/products/rpackages/devtools/) with:

``` r
devtools::install_github("benkeser/drtmle")
```

------------------------------------------------------------------------

Usage
-----

### Doubly-robust inference for the average treatment effect

Suppose the data consist of a vector of baseline covariates (`W`), a multi-level treatment assignment (`A`), and a continuous or binary-valued outcome (`Y`). The function `drtmle` may be used to estimate *E*\[*E*(*Y* ∣ *A* = *a*<sub>0</sub>, *W*)\] for user-selected values of *a*<sub>0</sub> (via option `a_0`). The resulting targeted minimum loss-based estimates are doubly robust with respect to both consistency and asymptotic normality. The function computes doubly robust covariance estimates that can be used to construct doubly robust confidence intervals for marginal means and contrasts between means. A simple example on simulated data is shown below. We refer users to the vignette for more information and further examples.

``` r
# load packages
library(drtmle, quietly = TRUE)
#> drtmle: TMLE with doubly robust inference
#> Version: 1.0.2
library(SuperLearner, quietly = TRUE)
#> Super Learner
#> Version: 2.0-23-9000
#> Package created on 2017-07-20

# simulate simple data structure
set.seed(1234)
n <- 200
W <- data.frame(W1 = runif(n,-2,2), W2 = rbinom(n,1,0.5))
A <- rbinom(n, 1, plogis(-2 + W$W1 - 2*W$W2))
Y <- rbinom(n, 1, plogis(-2 + W$W1 - 2*W$W2 + A))

# estimate the covariate-adjusted marginal mean for A = 1 and A = 0
# here, we do not properly estimate the propensity score
fit1 <- drtmle(W = W, A = A, Y = Y, # input data
               a_0 = c(0, 1), # return estimates for A = 0 and A = 1
               SL_Q = "SL.npreg", # use kernel regression for E(Y | A = a, W)
               glm_g = "W1 + W2", # use misspecified main terms glm for E(A | W)
               SL_Qr = "SL.npreg", # use kernel regression to guard against
                                   # misspecification of outcome regression
               SL_gr = "SL.npreg" # use kernel regression to guard against
                                  # misspecification of propensity score
              )
# print the output
fit1
#> $est
#>            
#> 0 0.1403428
#> 1 0.2158854
#> 
#> $cov
#>              0            1
#> 0 0.0008075554 0.0002582092
#> 1 0.0002582092 0.0014881480

# get confidence intervals for marginal means
ci_fit1 <- ci(fit1)
# print the output
ci_fit1
#> $drtmle
#>     est   cil   ciu
#> 0 0.140 0.085 0.196
#> 1 0.216 0.140 0.291

# get confidence intervals for ate
ci_ate1 <- ci(fit1,contrast = c(-1, 1))
# print the output
ci_ate1
#> $drtmle
#>                   est    cil   ciu
#> E[Y(1)]-E[Y(0)] 0.076 -0.007 0.158
```

### Inference for super learner-based IPTW

The package additionally includes a function for computing valid confidence intervals about an inverse probability of treatment weight (IPTW) estimator when super learning is used to estimate the propensity score.

``` r
# fit iptw
fit2 <- adaptive_iptw(Y = Y, A = A, W = W, a_0 = c(0, 1),
                      SL_g = c("SL.glm", "SL.mean", "SL.step.interaction"),
                      SL_Qr = "SL.npreg")
#> Loading required package: nloptr
# print the output
fit2
#> $est
#>            
#> 0 0.1377524
#> 1 0.1943633
#> 
#> $cov
#>              0            1
#> 0 0.0007623723 0.0002181465
#> 1 0.0002181465 0.0106635990

# compute a confidence interval for margin means
ci_fit2 <- ci(fit2)
# print the output
ci_fit2
#> $iptw_tmle
#>     est    cil   ciu
#> 0 0.138  0.084 0.192
#> 1 0.194 -0.008 0.397

# compute a confidence interval for the ate
ci_ate2 <- ci(fit2, contrast = c(-1, 1))
# print the output
ci_ate2
#> $iptw_tmle
#>                   est    cil   ciu
#> E[Y(1)]-E[Y(0)] 0.057 -0.149 0.262
```

------------------------------------------------------------------------

Issues
------

If you encounter any bugs or have any specific feature requests, please [file an issue](https://github.com/benkeser/drtmle/issues).

------------------------------------------------------------------------

Citation
--------

After using the `drtmle` R package, please cite the following:

        @article{benkeser2017improved,
          year  = {2017},
          author = {Benkeser, David C and Carone, Marco and van der Laan, Mark J
            and Gilbert, Peter B},
          title = {Doubly-robust nonparametric inference on the average
            treatment effect},
          journal = {Biometrika}
        }

------------------------------------------------------------------------

License
-------

© 2016-2017 [David C. Benkeser](http://www.benkeserstatistics.com)

The contents of this repository are distributed under the MIT license. See below for details:

    The MIT License (MIT)

    Copyright (c) 2016-2017 David C. Benkeser

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
