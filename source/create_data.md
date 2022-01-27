Create 2022 Data - Crime
================
Christopher Prener, Ph.D.
(January 27, 2022)

## Introduction

This notebook creates the crime data for the Spring 2022 final projects.

## Dependencies

This notebook requires:

``` r
# tidystl packages
library(compstatr)

# tidyverse packages
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(readr)

# other packages
library(here)
```

    ## here() starts at /Users/chris/GitHub/slu-soc5650/crime-data-sp22

``` r
library(testthat)
```

    ## 
    ## Attaching package: 'testthat'

    ## The following objects are masked from 'package:readr':
    ## 
    ##     edition_get, local_edition

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     matches

## Load Raw Data

First, we load year-list objects using `cs_load_year()`. The raw data
have been manually downloaded from SLMPD’s open data portal.

``` r
data2019_raw <- cs_load_year(here("data", "raw", "2019"))
data2018_raw <- cs_load_year(here("data", "raw", "2018"))
data2017_raw <- cs_load_year(here("data", "raw", "2017"))
data2016_raw <- cs_load_year(here("data", "raw", "2016"))
data2015_raw <- cs_load_year(here("data", "raw", "2015"))
```

## Prep Data

Next, we’ll validate and prepare our data.

### 2019

We validate the data to make sure it can be collapsed using
`cs_validate_year()`:

``` r
cs_validate(data2019_raw, year = "2019")
```

    ## [1] TRUE

Since the validation result is a value of `TRUE`, we can proceed to
collapsing the year-list object into a single tibble with
`cs_collapse()` and then stripping out crimes reported in 2018 for
earlier years using `cs_combine()`. We also strip out unfounded crimes
that remain using `cs_filter_count()`:

``` r
# collapse into single object
data2019_raw <- cs_collapse(data2019_raw)

# combine and filter
cs_combine(type = "year", date = 2019, data2019_raw) %>%
  cs_filter_count(var = count) -> data2019
```

The `data2019` object now contains only crimes reported in 2019.

### 2018

We’ll repeat the validation process with the 2018 data:

``` r
cs_validate(data2018_raw, year = "2018")
```

    ## [1] TRUE

Since the validation result is a value of `TRUE`, we can proceed to
collapsing the year-list object into a single tibble with
`cs_collapse()` and then stripping out crimes reported in 2018 for
earlier years using `cs_combine()`. We also strip out unfounded crimes
that remain using `cs_filter_count()`:

``` r
# collapse into single object
data2018_raw <- cs_collapse(data2018_raw)

# combine and filter
cs_combine(type = "year", date = 2018, data2019_raw, data2018_raw) %>%
  cs_filter_count(var = count) -> data2018
```

The `data2018` object now contains only crimes reported in 2018.

### 2017

We’ll repeat the validation process with the 2017 data:

``` r
cs_validate(data2017_raw, year = "2017")
```

    ## [1] FALSE

Since we fail the validation, we can use the `verbose = TRUE` option to
get a summary of where validation issues are occurring.

``` r
cs_validate(data2017_raw, year = "2017", verbose = TRUE)
```

    ## # A tibble: 12 × 8
    ##    namedMonth codedMonth valMonth codedYear valYear oneMonth varCount valVars
    ##    <chr>      <chr>      <lgl>        <int> <lgl>   <lgl>    <lgl>    <lgl>  
    ##  1 January    January    TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ##  2 February   February   TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ##  3 March      March      TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ##  4 April      April      TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ##  5 May        May        TRUE          2017 TRUE    TRUE     FALSE    NA     
    ##  6 June       June       TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ##  7 July       July       TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ##  8 August     August     TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ##  9 September  September  TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ## 10 October    October    TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ## 11 November   November   TRUE          2017 TRUE    TRUE     TRUE     TRUE   
    ## 12 December   December   TRUE          2017 TRUE    TRUE     TRUE     TRUE

The data for May 2017 do not pass the validation checks. We can extract
this month and confirm that there are too many columns in the May 2017
release. Once we have that confirmed, we can standardize that month and
re-run our validation.

``` r
# extract data and unit test column numbers
expect_equal(ncol(cs_extract_month(data2017_raw, month = "May")), 26)

# standardize months
data2017_raw <- cs_standardize(data2017_raw, month = "May", config = 26)

# validate data
cs_validate(data2017_raw, year = "2017")
```

    ## [1] TRUE

We now get a `TRUE` value for `cs_validate_year()` and can move on to
collapsing the 2017 and 2018 raw data objects to create a new object,
`data2017`, that contains all known 2017 crimes including those that
were reported or upgraded in 2018.

``` r
# collapse into single object
data2017_raw <- cs_collapse(data2017_raw)

# combine and filter
cs_combine(type = "year", date = 2017, data2019_raw, data2018_raw, data2017_raw) %>%
  cs_filter_count(var = count) -> data2017
```

The `data2017` object now contains only crimes reported in 2017.

### 2016

We’ll repeat the validation process with the 2016 data:

``` r
cs_validate(data2016_raw, year = "2016")
```

    ## [1] TRUE

Since the validation process passes, we can immediately move on to
creating our 2016 data object:

``` r
# collapse into single object
data2016_raw <- cs_collapse(data2016_raw)

# combine and filter
cs_combine(type = "year", date = 2016, data2019_raw, data2018_raw, data2017_raw, 
           data2016_raw) %>%
  cs_filter_count(var = count) -> data2016
```

The `data2016` object now contains only crimes reported in 2016.

### 2015

We’ll repeat the validation process with the 2015 data:

``` r
cs_validate(data2015_raw, year = "2015")
```

    ## [1] TRUE

Since the validation process passes, we can immediately move on to
creating our 2015 data object:

``` r
# collapse into single object
data2015_raw <- cs_collapse(data2015_raw)

# combine and filter
cs_combine(type = "year", date = 2015, data2019_raw, data2018_raw, data2017_raw, 
           data2016_raw, data2015_raw) %>%
  cs_filter_count(var = count) -> data2015
```

The `data2015` object now contains only crimes reported in 2015.

## Clean-up Enviornment

We can remove the `_raw` objects at this point:

``` r
rm(data2015_raw, data2016_raw, data2017_raw, data2018_raw, data2019_raw)
```

## Create Single Table

Next, we’ll create a single table of our data. We also subset columns to
reduce the footprint of the table.

``` r
bind_rows(data2015, data2016, data2017, data2018, data2019) %>%
  select(date_occur, crime, description, ileads_address, ileads_street, x_coord, y_coord) -> crimes

write_csv(crimes, here("data", "clean", "crimes.csv"))
```
