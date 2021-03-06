---
title: "Unit testing"
author: "Laurent Gatto"
---

These exercises were written by Martin Morgan and Laurent Gatto for a
[Bioconductor Developer Day workshop](http://bioconductor.org/help/course-materials/2013/BioC2013/developer-day-debug/).

# Introduction

> Whenever you are tempted to type something into a print statement
> or a debugger expression, write it as a test instead -- Martin Fowler

**Why unit testing?**

- Writing code to test code;
- anticipate bugs, in particular for edge cases;
- anticipate disruptive updates;
- document and test observed bugs using specific tests.

Each section provides a function that supposedly works as expected,
but quickly proves to misbehave. The exercise aims at first writing
some dedicated testing functions that will identify the problems and
then update the function so that it passes the specific tests. This
practice is called unit testing and we use the RUnit package for
this.

See the
[Unit Testing How-To](http://bioconductor.org/developers/how-to/unitTesting-guidelines/)
guide for details on unit testing using the
[`RUnit`](http://cran.r-project.org/web/packages/RUnit/index.html)
package. The
[`testthat`](http://cran.r-project.org/web/packages/testthat/) is
another package that provides unit testing infrastructure. Both
packages can conveniently be used to automate unit testing within
package testing. 

# Example

## Subsetting

### Problem

This function should return the elements of `x` that are in `y`.


```r
## Example
isIn <- function(x, y) {
    sel <- match(x, y)
    y[sel]
}

## Expected
x <- sample(LETTERS, 5)
isIn(x, LETTERS)
```

```
## [1] "V" "I" "Q" "K" "N"
```
But


```r
## Bug!
isIn(c(x, "a"), LETTERS)
```

```
## [1] "V" "I" "Q" "K" "N" NA
```

### Solution

Write a unit test that demonstrates the issue


```r
## Unit test:
library("RUnit")
test_isIn <- function() {
    x <- c("A", "B", "Z")
    checkIdentical(x, isIn(x, LETTERS))
    checkIdentical(x, isIn(c(x, "a"), LETTERS))

}

test_isIn()
```

```
## Error in checkIdentical(x, isIn(c(x, "a"), LETTERS)): FALSE 
## 
```

Update the buggy function until the unit test succeeds


```r
## updated function
isIn <- function(x, y) {
    sel <- x %in% y
    x[sel]
}

test_isIn() ## the bug is fixed and monitored
```

```
## [1] TRUE
```

## The `testthat` syntax

`expect_that(object_or_expression, condition)` with conditions
- equals: `expect_that(1+2,equals(3))` or `expect_equal(1+2,3)`
- gives warning: `expect_that(warning("a")`, `gives_warning())`
- is a: `expect_that(1, is_a("numeric"))` or `expect_is(1,"numeric")`
- is true: `expect_that(2 == 2, is_true())` or `expect_true(2==2)`
- matches: `expect_that("Testing is fun", matches("fun"))` or `expect_match("Testing is fun", "f.n")`
- takes less: `than expect_that(Sys.sleep(1), takes_less_than(3))`

and

```r
test_that("isIn function", {
    x <- c("A", "B", "Z")
    expect_identical(x, isIn(x, LETTERS))
    expect_identical(x, isIn(c(x, "a"), LETTERS))
})
```

## Batch unit testing

```r
library("testthat")
test_dir("./unittests/")
test_file("./unittests/test_foo.R")
```

# Exercises

## Column means

## Problem

The `col_means` function computes the means of all numeric columns in
a data frame (example from *Advanced R*, to illustrate defensive
programming).


```r
col_means <- function(df) {
  numeric <- sapply(df, is.numeric)
  numeric_cols <- df[, numeric]
  data.frame(lapply(numeric_cols, mean))
}

## Expected
col_means(mtcars)

## Bugs
col_means(mtcars[, "mpg"])
col_means(mtcars[, "mpg", drop = FALSE])
col_means(mtcars[, 0])
col_means(mtcars[0, ])
col_means(as.list(mtcars))
```

## Character matching

### Problem

What are the exact matches of `x` in `y`?


```r
isExactIn <- function(x, y)
    y[grep(x, y)]

## Expected
isExactIn("a", letters)

## Bugs
isExactIn("a", c("abc", letters))
isExactIn(c("a", "z"), c("abc", letters))
```

<!-- ### Solution -->

<!-- ```{r} -->
<!-- ## Unit test: -->
<!-- library("RUnit") -->
<!-- test_isExactIn <- function() { -->
<!--     checkIdentical("a", isExactIn("a", letters)) -->
<!--     checkIdentical("a", isExactIn("a", c("abc", letters))) -->
<!--     checkIdentical(c("a", "z"), isExactIn(c("a", "z"), c("abc", letters))) -->
<!-- } -->

<!-- test_isExactIn() -->

<!-- ## updated function: -->
<!-- isExactIn <- function(x, y) -->
<!--     x[x %in% y] -->

<!-- test_isExactIn() -->
<!-- ``` -->

## If conditions with length > 1

### Problem

If `x` is greater than `y`, we want the difference of their
squares. Otherwise, we want the sum.


```r
ifcond <- function(x, y) {
    if (x > y) {
        ans <- x*x - y*y
    } else {
        ans <- x*x + y*y
    } 
    ans
}

## Expected
ifcond(3, 2)
ifcond(2, 2)
ifcond(1, 2)

## Bug!
ifcond(3:1, c(2, 2, 2))
```

<!-- ### Solution -->

<!-- ```{r} -->
<!-- ## Unit test: -->
<!-- library("RUnit") -->
<!-- test_ifcond <- function() { -->
<!--     checkIdentical(5, ifcond(3, 2)) -->
<!--     checkIdentical(8, ifcond(2, 2)) -->
<!--     checkIdentical(5, ifcond(1, 2)) -->
<!--     checkIdentical(c(5, 8, 5), ifcond(3:1, c(2, 2, 2))) -->
<!-- } -->

<!-- test_ifcond() -->

<!-- ## updated function: -->
<!-- ifcond <- function(x, y) -->
<!--     ifelse(x > y, x*x - y*y, x*x + y*y) -->

<!-- test_ifcond() -->
<!-- ``` -->

## Know your inputs

### Problem

Calculate the euclidean distance between a single point and a set of
other points.


```r
## Example
distances <- function(point, pointVec) {
    x <- point[1]
    y <- point[2]
    xVec <- pointVec[,1]
    yVec <- pointVec[,2]
    sqrt((xVec - x)^2 + (yVec - y)^2)
}

## Expected
x <- rnorm(5)
y <- rnorm(5)

(m <- cbind(x, y))
(p <- m[1, ])

distances(p, m)

## Bug!
(dd <- data.frame(x, y))
(q <- dd[1, ])

distances(q, dd)
```

<!-- ### Solution -->

<!-- ```{r} -->
<!-- ## Unit test: -->
<!-- library("RUnit") -->
<!-- test_distances <- function() { -->
<!--     x <- y <- c(0, 1, 2) -->
<!--     m <- cbind(x, y) -->
<!--     p <- m[1, ] -->
<!--     dd <- data.frame(x, y) -->
<!--     q <- dd[1, ] -->
<!--     expct <- c(0, sqrt(c(2, 8))) -->
<!--     checkIdentical(expct, distances(p, m)) -->
<!--     checkIdentical(expct, distances(q, dd)) -->
<!-- } -->

<!-- test_distances() -->

<!-- ## updated function -->
<!-- distances <- function(point, pointVec) { -->
<!--     point <- as.numeric(point) -->
<!--     x <- point[1] -->
<!--     y <- point[2] -->
<!--     xVec <- pointVec[,1] -->
<!--     yVec <- pointVec[,2] -->
<!--     dist <- sqrt((xVec - x)^2 + (yVec - y)^2) -->
<!--     return(dist) -->
<!-- } -->

<!-- test_distances() -->
<!-- ``` -->

## Iterate on 0 length

### Problem

Calculate the square root of the absolute value of a set of numbers.


```r
sqrtabs <- function(x) {
    v <- abs(x)
    sapply(1:length(v), function(i) sqrt(v[i]))
}

## Expected
all(sqrtabs(c(-4, 0, 4)) == c(2, 0, 2))

## Bug!
sqrtabs(numeric())
```

<!-- ### Solution -->

<!-- ```{r} -->
<!-- ## Unit test: -->
<!-- library(RUnit) -->
<!-- test_sqrtabs <- function() { -->
<!--     checkIdentical(c(2, 0, 2), sqrtabs(c(-4, 0, 4))) -->
<!--     checkIdentical(numeric(), sqrtabs(numeric())) -->
<!-- } -->
<!-- test_sqrtabs() -->

<!-- ## updated function: -->
<!-- sqrtabs <- function(x) { -->
<!--   v <- abs(x) -->
<!--   sapply(seq_along(v), function(i) sqrt(v[i])) -->
<!-- } -->
<!-- test_sqrtabs()                          # nope! -->

<!-- sqrtabs <- function(x) { -->
<!--   v <- abs(x) -->
<!--   vapply(seq_along(v), function(i) sqrt(v[i]), 0) -->
<!-- } -->
<!-- test_sqrtabs()                          # yes! -->
<!-- ``` -->

# Unit testing in a package 

## In a package

1. Create a directory `./mypackage/tests`.
2. Create the `testthat.R` file

```r
library("testthat")
library("mypackage")
test_check("sequences")
```

3. Create a sub-directory `./mypackage/tests/testthat` and include as
   many unit test files as desired that are named with the `test_`
   prefix and contain unit tests.

4. Suggest the unit testing package in your `DESCRIPTION` file:

```
Suggests: testthat
```

## Example from the `sequences` package

From the `./sequences/tests/testthat/test_sequences.R` file:

### Object creation and validity

We have a fasta file and the corresponding `DnaSeq` object.

1. Let's make sure that the `DnaSeq` instance is valid, as changes in
   the class definition might have altered its validity.

2. Let's verify that `readFasta` regenerates and identical `DnaSeq`
   object given the original fasta file.

```r
test_that("dnaseq validity", {
  data(dnaseq)
  expect_true(validObject(dnaseq))
})

test_that("readFasta", {
  ## loading _valid_ dnaseq
  data(dnaseq)
  ## reading fasta sequence
  f <- dir(system.file("extdata", package = "sequences"),
           pattern="fasta", full.names = TRUE)
  xx <- readFasta(f[1])
  expect_true(all.equal(xx, dnaseq))
})
```

### Multiple implementations

Let's check that the R, C and C++ (via `Rcpp`) give the same result

```r
test_that("ccpp code", {
  gccountr <-
    function(x) tabulate(factor(strsplit(x, "")[[1]]))
  x <- "AACGACTACAGCATACTAC"
  expect_true(identical(gccount(x), gccountr(x)))
  expect_true(identical(gccount2(x), gccountr(x)))
})
```

## Exercise

Choose any data package of your choice and write a unit test that
tests the validity of all the its data. 

Tips 

- To get all the data distributed with a package, use `data(package = "packageName")`


```r
library("pRolocdata")
data(package = "pRolocdata")
```

- To test the validity of an object, use `validObject`




```r
data(andy2011)
validObject(andy2011)
```

```
## [1] TRUE
```

- Using the `testthat` syntax, the actual test for that data set would be 


```r
library("testthat")
expect_true(validObject(andy2011))
```


## Testing coverage in a package

The [covr](https://github.com/jimhester/covr) package:

![package coverage](./figs/covr.png)

We can use `type="all"` to examine the coverage in unit tests, examples and vignettes. This can
also be done interactively with Shiny:


```r
library(covr)
coverage <- package_coverage("/path/to/package/source", type="all")
shine(coverage)
```

[Coverage for all Bioconductor packages](https://codecov.io/github/Bioconductor-mirror).
