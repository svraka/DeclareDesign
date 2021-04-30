DeclareDesign: Declare and Diagnose Research Designs
================

<!-- README.md is generated from README.Rmd. Please edit that file -->

[![CRAN
status](https://www.r-pkg.org/badges/version/DeclareDesign)](https://cran.r-project.org/package=DeclareDesign)
[![CRAN RStudio mirror
downloads](https://cranlogs.r-pkg.org/badges/grand-total/DeclareDesign?color=green)](https://r-pkg.org/pkg/DeclareDesign)
[![Build
status](https://github.com/DeclareDesign/DeclareDesign/workflows/R-CMD-check/badge.svg)](https://github.com/DeclareDesign/DeclareDesign/actions)
[![Code
coverage](https://codecov.io/gh/DeclareDesign/DeclareDesign/branch/master/graph/badge.svg?token=3WzOK9Glc2)](https://codecov.io/gh/DeclareDesign/DeclareDesign)

DeclareDesign is a system for describing research designs in code and
simulating them in order to understand their properties. Because
DeclareDesign employs a consistent grammar of designs, you can focus on
the intellectually challenging part – designing good research studies –
without having to code up simulations from scratch.

## Installation

To install the latest stable release of **DeclareDesign**, please ensure
that you are running version 3.5 or later of R and run the following
code:

``` r
install.packages("DeclareDesign")
```

## Usage

Designs are declared by adding together design elements. Here’s a
minimal example that describes a 100 unit randomized controlled trial
with a binary outcome. Half the units are assigned to treatment and the
remainder to control. The true value of the average treatment effect is
0.05 and it will be estimated with the difference-in-means estimator.
The diagnosis shows that the study is unbiased but underpowered.

``` r
library(DeclareDesign)

design <-
  declare_model(
    N = 100, 
    potential_outcomes(Y ~ rbinom(N, size = 1, prob = 0.5 + 0.05 * Z))
  ) +
  declare_inquiry(ATE = 0.05) +
  declare_assignment(Z = complete_ra(N, m = 50), legacy = FALSE) +
  declare_measurement(Y = reveal_outcomes(Y ~ Z)) + 
  declare_estimator(Y ~ Z, model = lm_robust)

diagnosands <-
  declare_diagnosands(bias = mean(estimate - estimand),
                      power = mean(p.value <= 0.05))

diagnosis <- diagnose_design(design, diagnosands = diagnosands)
```

``` r
diagnosis
```

| Inquiry | Estimator |   Bias | SE(Bias) | Power | SE(Power) | n sims |
|:--------|:----------|-------:|---------:|------:|----------:|-------:|
| ATE     | estimator | -0.004 |    0.004 | 0.076 |     0.011 |    500 |

## Companion software

The core DeclareDesign package relies on four companion packages, each
of which is useful in its own right.

1.  [randomizr](https://declaredesign.org/r/randomizr/): Easy to use
    tools for common forms of random assignment and sampling.
2.  [fabricatr](https://declaredesign.org/r/fabricatr/): Imagine your
    data before you collect it.
3.  [estimatr](https://declaredesign.org/r/estimatr/): Fast estimators
    for social scientists.
4.  [DesignLibrary](https://declaredesign.org/r/designlibrary/):
    Templates to quickly adopt and adapt common research designs.

## Learning DeclareDesign

1.  To get started, have a look at this vignette on [the idea behind
    DeclareDesign](https://declaredesign.org/getting-started.html),
    which covers the main functionality of the software.

2.  You can also browse a
    [library](https://declaredesign.org/r/designlibrary/) of already
    declared designs, which relies on the `DesignLibrary` package. The
    library includes canonical designs that you can download, modify,
    and deploy.

3.  A fuller description of the philosophy underlying the software is
    described in this [paper](https://declaredesign.org/declare.pdf).

## Package structure

Each of these `declare_*()` functions returns a *function*.

1.  `declare_model()` (describes dimensions and distributions over the
    variables, including potential outcomes)
2.  `declare_inquiry()` (takes variables in the model and calculates
    estimand value)
3.  `declare_sampling()` (takes a population and selects a sample)
4.  `declare_assignment()` (takes a population or sample and adds
    treatment assignments)
5.  `declare_measurement()` (takes data and adds measured values)
6.  `declare_estimator()` (takes data produced by sampling, assignment,
    and measurement and returns estimates linked to inquiries)
7.  `declare_test()` (takes data produced by sampling, assignment, and
    measurement and returns the result of a test)

To *declare a design*, connect the components of your design with the +
operator.

Once you have declared your design, there are four core
post-design-declaration commands used to modify or diagnose your design:

1.  `diagnose_design()` (takes a design, returns simulations and
    diagnosis)
2.  `draw_data()` (takes a design and returns a single draw of the data)
3.  `draw_estimates()` (takes a design a returns a single simulation of
    estimates)
4.  `draw_inquiries()` (takes a design a returns a single simulation of
    estimands)

A few other features:

1.  A designer is a function that takes parameters (e.g., `N`) and
    returns a design. `expand_design()` is a function of a designer and
    parameters that returns a design.
2.  You can change the features of the design to be diagnosed with
    `declare_diagnosands()`.

------------------------------------------------------------------------

This project is generously supported by a grant from the [Laura and John
Arnold Foundation](http://www.arnoldfoundation.org) and seed funding
from [EGAP](http://egap.org).
