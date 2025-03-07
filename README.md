
<!-- README.md is generated from README.Rmd. Please edit that file -->

# kdensity <img src="man/figures/logo.png" align="right" width="247" height="70" />

[![R build
status](https://github.com/JonasMoss/kdensity/workflows/R-CMD-check/badge.svg)](https://github.com/JonasMoss/kdensity/actions)
[![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/kdensity)](https://cran.r-project.org/package=kdensity)
[![DOI](https://zenodo.org/badge/120678148.svg)](https://zenodo.org/badge/latestdoi/120678148)

An `R` package for univariate kernel density estimation with parametric
starts and asymmetric kernels.

## News

`kdensity` is now linked to `univariateML`, meaning it supports the
approximately 30+ parametric starts from that package\!

## Overview

kdensity is an implementation of univariate kernel density estimation
with support for parametric starts and asymmetric kernels. Its main
function is `kdensity`, which is has approximately the same syntax as
`stats::density`. Its new functionality is:

  - `kdensity` has built-in support for many *parametric starts*, such
    as `normal` and `gamma`, but you can also supply your own. For a
    list of supported parametric starts, see the readme of
    [`univariateML`](https://github.com/JonasMoss/univariateML).
  - It supports several asymmetric kernels ones such as `gcopula` and
    `gamma` kernels, but also the common symmetric ones. In addition,
    you can also supply your own kernels.
  - A selection of choices for the bandwidth function `bw`, again
    including an option to specify your own.
  - The returned value is density function. This can be used for
    e.g. numerical integration, numerical differentiation, and point
    evaluations.

A reason to use `kdensity` is to avoid *boundary bias* when estimating
densities on the unit interval or the positive half-line. Asymmetric
kernels such as `gamma` and `gcopula` are designed for this purpose. The
support for parametric starts allows you to easily use a method that is
often superior to ordinary kernel density estimation.

Several `R` packages deal with kernel estimation. For an overview see
[Deng & Hadley Wickham
(2011)](https://vita.had.co.nz/papers/density-estimation.pdf). While no
other `R` package handles density estimation with parametric starts,
several packages supports methods that handle boundary bias.
[`evmix`](http://www.math.canterbury.ac.nz/~c.scarrott/evmix/) provides
a variety of boundary bias correction methods in the `bckden` function.
[`kde1d`](https://github.com/tnagler/kde1d) corrects for boundary bias
using transformed univariate local polynomial kernel density estimation.
[`logKDE`](https://github.com/andrewthomasjones/logKDE) corrects for
boundary bias on the half line using a logarithmic transform.
[`ks`](https://CRAN.R-project.org/package=ks) supports boundary
correction through the `kde.boundary` function, while
[`Ake`](https://CRAN.R-project.org/package=Ake) corrects for boundary
bias using tailored kernel functions.

## Installation

From inside `R`, use one of the following commands:

``` r
# For the CRAN release
install.packages("kdensity")
# For the development version from GitHub:
# install.packages("devtools")
devtools::install_github("JonasMoss/kdensity")
```

## Usage Example

Call the `library` function and use it just like `stats::density`, but
with optional additional arguments.

``` r
library("kdensity")
plot(kdensity(mtcars$mpg, start = "normal"))
```

## Description

Kernel density estimation with a *parametric start* was introduced by
Hjort and Glad in [Nonparametric Density Estimation with a Parametric
Start (1995)](https://projecteuclid.org/euclid.aos/1176324627). The idea
is to start out with a parametric density before you do your kernel
density estimation, so that your actual kernel density estimation will
be a correction to the original parametric estimate. The resulting
estimator will outperform the ordinary kernel density estimator in terms
of asymptotic integrated mean squared error whenever the true density is
close to your suggestion; and the estimator can be superior to the
ordinary kernel density estimator even when the suggestion is pretty far
off.

In addition to parametric starts, the package implements some
*asymmetric kernels*. These kernels are useful when modelling data with
sharp boundaries, such as data supported on the positive half-line or
the unit interval. Currently we support the following asymmetric
kernels:

  - Jones and Henderson’s *Gaussian copula KDE*, from Kernel-Type
    Density Estimation on the Unit Interval
    (2007)).
    This is used for data on the unit interval. The bandwidth selection
    mechanism described in that paper is implemented as well. This
    kernel is called `gcopula`.

  - Chen’s two *beta kernels* from Beta kernel estimators for density
    functions
    (1999).
    These are used for data supported on the on the unit interval, and
    are called `beta` and `beta_biased`.

  - Chen’s two *gamma kernels* from Probability Density Function
    Estimation Using Gamma Kernels
    (2000).
    These are used for data supported on the positive half-line, and are
    called `gamma` and `gamma_biased`.

These features can be combined to make asymmetric kernel densities
estimators with parametric starts, see the example below. The package
contains only one function, `kdensity`, in addition to the generics
`plot`, `points`, `lines`, `summary`, and `print`.

## Usage

The function `kdensity` takes some `data`, a kernel `kernel` and a
parametric start `start`. You can optionally specify the `support`
parameter, which is used to find the normalizing constant.

The following example uses the  data set. The black curve is a
gamma-kernel density estimate with a gamma start, the red curve a fully
parametric gamma density and and the blue curve an ordinary `density`
estimate. Notice the boundary bias of the ordinary `density` estimator.
The underlying parameter estimates are always maximum likelilood.

``` r
library("kdensity")
kde = kdensity(airquality$Wind, start = "gamma", kernel = "gamma")
plot(kde, main = "Wind speed (mph)")
lines(kde, plot_start = TRUE, col = "red")
lines(density(airquality$Wind, adjust = 2), col = "blue")
rug(airquality$Wind)
```

<img src="man/figures/README-example-1.png" width="750px" />

Since the return value of `kdensity` is a function, `kde` is callable
and can be used as any density function in `R` (such as `stats::dnorm`).
For example, you can do:

``` r
kde(10)
#> [1] 0.09980471
integrate(kde, lower = 0, upper = 1) # The cumulative distribution up to 1.
#> 1.27532e-05 with absolute error < 2.2e-19
```

You can access the parameter estimates by using `coef`. You can also
access the log likelihood (`logLik`), AIC and BIC of the parametric
start distribution.

``` r
coef(kde)
#> Maximum likelihood estimates for the Gamma model 
#>  shape    rate  
#> 7.1873  0.7218
logLik(kde)
#> 'log Lik.' 12.33787 (df=2)
AIC(kde)
#> [1] -20.67574
```

## How to Contribute or Get Help

If you encounter a bug, have a feature request or need some help, open a
[Github issue](https://github.com/JonasMoss/kdensity/issues). Create a
pull requests to contribute. This project follows a [Contributor Code of
Conduct](https://www.contributor-covenant.org/version/1/4/code-of-conduct/).

## References

  - [Hjort, Nils Lid, and Ingrid K. Glad. “Nonparametric density
    estimation with a parametric start.” The Annals of Statistics
    (1995): 882-904.](https://projecteuclid.org/euclid.aos/1176324627).

  - [Jones, M. C., and D. A. Henderson. “Miscellanea kernel-type density
    estimation on the unit interval.” Biometrika 94.4 (2007):977-984.]

  - [Chen, Song Xi. “Probability density function estimation using gamma
    kernels.” Annals of the Institute of Statistical Mathematics 52.3
    (2000):
    471-480.](https://link.springer.com/article/10.1023/A:1004165218295).

  - [Chen, Song Xi. “Beta kernel estimators for density functions.”
    Computational Statistics & Data Analysis 31.2 (1999):131-145.]