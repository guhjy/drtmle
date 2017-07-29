
<!-- README.md is generated from README.Rmd. Please edit that file -->
R/`drtmle`
==========

[![Travis-CI Build Status](https://travis-ci.org/benkeser/drtmle.svg?branch=master)](https://travis-ci.org/benkeser/drtmle) [![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/benkeser/drtmle?branch=master&svg=true)](https://ci.appveyor.com/project/benkeser/drtmle) [![Coverage Status](https://img.shields.io/codecov/c/github/benkeser/drtmle/master.svg)](https://codecov.io/github/benkeser/drtmle?branch=master) [![CRAN](http://www.r-pkg.org/badges/version/drtmle)](http://www.r-pkg.org/pkg/drtmle) [![Project Status: Active - The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active) [![MIT license](http://img.shields.io/badge/license-MIT-brightgreen.svg)](http://opensource.org/licenses/MIT)

> Nonparametric estimators of the average treatment effect with doubly-robust confidence intervals and hypothesis tests

**Author:** [David Benkeser](https://www.benkeserstatistics.com/)

------------------------------------------------------------------------

Description
-----------

`drtmle` is an R package that computes marginal effect estimators for binary treatments on continuous and binary outcomes. The targeted minimum loss-based (TMLE) estimators are doubly robust, not only with respect to consistency, but also with respect to asymptotic normality, as discussed in Benkeser, Carone, van der Laan & Gilbert, 2017,(*accepted Biometrika*, [working paper](http://biostats.bepress.com/ucbbiostat/paper356/)). Therefore, it is possible to construct doubly-robust confidence intervals and hypothesis tests. Also included are methods for computing valid confidence intervals for an inverse probability of treatment weighted (IPTW) estimator of the average treatment effect when the propensity score is estimated via super learning, as discussed in [van der Laan, 2014](https://www.degruyter.com/downloadpdf/j/ijb.2014.10.issue-1/ijb-2012-0038/ijb-2012-0038.pdf).

------------------------------------------------------------------------

Installation
------------

-   Install the most recent *stable release*: `devtools::install_github("benkeser/drtmle")`

-   To contribute, install the *development version*: `devtools::install_github("benkeser/drtmle", ref = "develop")`

------------------------------------------------------------------------

Use
---

This package can be used to estimate covariate-adjusted marginal means under multiple discrete levels of a treatment. It may be used in situations where data consist of a vector of baseline covariates (`W`), a multi-level treatment assignment (`A`), and a continuous or binary-valued outcome (`Y`). The function `drtmle` may be used to estimate *E*\[*E*(*Y*|*A* = *a*<sub>0</sub>, *W*)\] for user-selected values of *a*<sub>0</sub> (via option `a_0`). The resulting targeted minimum loss-based estimates are doubly robust with respect to both consistency and asymptotic normality. The function computes doubly robust variance estimators that can be used to construct doubly robust confidence intervals for covariate-adjusted marginal means and contrasts between these means (via the `drconfint` function) for different levels of *a*<sub>0</sub>. Doubly robust hypothesis tests are also available (via `drtest` function). A simple example on simulated data is shown below.

``` r
# load package
library(drtmle)
#> drtmle: TMLE with doubly robust inference
#> Version: 0.0.0.9000

# simulate simple data structure
set.seed(1234)
n <- 200
W <- data.frame(W1 = runif(n,-2,2), W2 = rbinom(n,1,0.5))
A <- rbinom(n, 1, plogis(-2 + W$W1 - 2*W$W2))
Y <- rbinom(n, 1, plogis(-2 + W$W1 - 2*W$W2 + A))

# estimate the covariate-adjusted marginal mean for A = 1 and A = 0
# here, we do not properly estimate the propensity score
fit <- drtmle(W = W, A = A, Y = Y, # input data
              a_0 = c(0,1), # return estimates for A = 0 and A = 1
              SL_Q = "SL.npreg", # use kernel regression for E(Y | A=a, W)
              glm_g = "W1 + W2", # use misspecified main terms glm for E(A | W)
              SL_Qr = "SL.npreg", # use kernel regression to guard against
                                  # misspecification of outcome regression
              SL_gr = "SL.npreg" # use kernel regression to guard against 
                                 # misspecification of propensity score
              )
#> Warning in (function (a, Q, g, gr) : No sane fluctuation found. Proceeding
#> using current estimates.
# print the output
fit
#> $est
#>        [,1]
#> 0 0.1403429
#> 1 0.2158854
#> 
#> $cov
#>              0            1
#> 0 0.0008238590 0.0002365695
#> 1 0.0002365695 0.0015010560

# get confidence intervals for means
ci_fit <- confint(fit)
# print the output
ci_fit
#> $drtmle
#>     est   cil   ciu
#> 0 0.140 0.084 0.197
#> 1 0.216 0.140 0.292
```

------------------------------------------------------------------------

Issues
------

If you encounter any bugs or have any specific feature requests, please [file an issue](https://github.com/benkeser/survtmle/issues).

------------------------------------------------------------------------

Citation
--------

After using the `drtmle` R package, please cite the following:

        @article{benkeser2017improved,
          year  = {2017},
          author = {Benkeser, David C and Carone, Marco and van der Laan, Mark J and Gilbert, Peter B},
          title = {Doubly-robust nonparametric inference on the average treatment effect},
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
