
<!-- README.md is generated from README.Rmd. Please edit that file -->

# S7

<!-- badges: start -->

[![R-CMD-check](https://github.com/RConsortium/OOP-WG/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/RConsortium/OOP-WG/actions/workflows/R-CMD-check.yaml)
[![Codecov test
coverage](https://codecov.io/gh/RConsortium/OOP-WG/branch/master/graph/badge.svg)](https://codecov.io/gh/RConsortium/OOP-WG?branch=master)

<!-- badges: end -->

The S7 package is a new OOP system designed to be a successor to S3 and
S4. It has been designed and implemented collaboratively by the
RConsortium Object-Oriented Programming Working Group, which includes
representatives from R-Core, BioConductor, RStudio/tidyverse, and the
wider R community.

## Installation

The long-term goal of this project is to merge S7 in to base R. For now,
you can experiment by installing the in-development version from GitHub:

``` r
# install.packages("remotes")
remotes::install_github("rconsortium/OOP-WG")
```

## Usage

This section gives a very brief overview of the entirety of S7. Learn
more of the basics in `vignettte("S7")`, the details of method dispatch
in `vignette("dispatch")`, and compatibility with S3 and S4 in
`vignette("compatibility")`.

``` r
library(S7)
```

### Classes and objects

S7 classes have a formal definition, which includes a list of properties
and an optional validator. Use `new_class()` to define a class:

``` r
range <- new_class("range",
  properties = list(
    start = class_double,
    end = class_double
  ),
  validator = function(self) {
    if (length(self@start) != 1) {
      "@start must be length 1"
    } else if (length(self@end) != 1) {
      "@end must be length 1"
    } else if (self@end < self@start) {
      "@end must be greater than or equal to @start"
    }
  }
)
```

`new_class()` returns the class object, which is also the constructor
you use to create instances of the class:

``` r
x <- range(start = 1, end = 10)
x
#> <range>
#>  @ start: num 1
#>  @ end  : num 10
```

### Properties

The data possessed by an object is called its **properties**. Use `@` to
get and set properties:

``` r
x@start
#> [1] 1
x@end <- 20
x
#> <range>
#>  @ start: num 1
#>  @ end  : num 20
```

Properties are automatically validated against the type declared in
`new_class()` (`double` in this case), and with the class **validator**:

``` r
x@end <- "x"
#> Error: <range>@end must be <double>, not <character>
x@end <- -1
#> Error: <range> object is invalid:
#> - @end must be greater than or equal to @start
```

### Generics and methods

Like S3 and S4, S7 uses **functional OOP** where methods belong to
**generic** functions, and method calls look like all other function
calls: `generic(object, arg2, arg3)`. This style is called functional
because from the outside it looks like a regular function call, and
internally the components are also functions.

Use `new_generic()` to create a new generic: the first argument is the
generic name (used in error messages) and the second gives the arguments
used for dispatch. The third, and optional argument, supplies the body
of the generic. This is only needed if your generic has additional
arguments that aren’t used for method dispatch.

``` r
inside <- new_generic("inside", "x", function(x, y) {
  # Actually finds and calls the appropriate method
  S7_dispatch()
})
```

Once you have a generic, you can define a method for a specific class
with `method<-`:

``` r
# Add a method for our class
method(inside, range) <- function(x, y) {
  y >= x@start & y <= x@end
}
inside
#> <S7_generic> function (x, y)  with 1 methods:
#> 1: method(inside, range)

inside(x, c(0, 5, 10, 15))
#> [1] FALSE  TRUE  TRUE  TRUE
```

You can use `method<-` to register methods for base types on S7
generics:

``` r
method(inside, class_numeric) <- function(x, y) {
  y >= min(x) & y <= max(x)
}
```

And register methods for S7 classes on S3 or S4 generics:

``` r
method(format, range) <- function(x, ...) {
  paste0("[", x@start, ", ", x@end, "]")
}
format(x)
#> [1] "[1, 20]"

method(mean, range) <- function(x, ...) {
  (x@start + x@end) / 2
}
mean(x)
#> [1] 10.5
```
