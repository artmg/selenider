
<!-- README.md is generated from README.Rmd. Please edit that file -->

# selenider

<!-- badges: start -->

[![R-CMD-check](https://github.com/ashbythorpe/selenider/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/ashbythorpe/selenider/actions/workflows/R-CMD-check.yaml)
[![Codecov test
coverage](https://codecov.io/gh/ashbythorpe/selenider/branch/main/graph/badge.svg)](https://app.codecov.io/gh/ashbythorpe/selenider?branch=main)
<!-- badges: end -->

selenider aims to make web testing and scraping in R much simpler, using
either [chromote](https://rstudio.github.io/chromote/) or
[RSelenium](https://www.selenium.dev/). It is inspired by Java’s
[Selenide](https://selenide.org/) and Python’s
[Selene](https://yashaka.github.io/selene/).

It provides the following features:

- All elements are lazy, meaning that they will only actually collect an
  element when absolutely necessary.
- Automatic implicit waiting, meaning that functions will automatically
  wait a certain amount of time (by default, 4 seconds) before
  performing some action on it.
- An API that is designed for use with the pipe operator (`|>` or
  `%>%`).
- A quick and concise way to specify test expectations.

## Installation

You can install the development version of selenider like so:

``` r
# install.packages("remotes")
remotes::install_github("ashbythorpe/selenider")
```

## Usage

``` r
library(selenider)
```

To open a session, and navigate to a website, use `open_url()`.

``` r
open_url("https://www.r-project.org/")
```

Use `s()` to select an element. By default, CSS selectors are used, but
other options are available.

``` r
r_header <- s("#r-project")

r_header
#> A selenider element selecting:
#> The first element with css selector "#r-project".

html_text(r_header)
#> [1] "R Project"
```

For example, an xpath can be used instead.

``` r
s(xpath = "//div/a")
#> A selenider element selecting:
#> The first element with xpath "//div/a".
```

Use `ss()` to select multiple elements

``` r
all_links <- ss("a")

all_links
#> A collection of selenider elements selecting:
#> The elements with css selector "a".

html_attr(all_links[[1]], "href")
#> [1] "/"
```

Use `html_element()` and `html_elements()` to find child elements of an
existing element. These can be chained with the pipe operator (`|>`) to
specify paths to elements.

``` r
s(".row") |>
  html_element("div") |>
  html_element("p") |>
  html_text()
#> [1] ""
```

Use `html_children()` and friends to find elements using their relative
position to another.

``` r
s("p") |>
  html_children()
#> A collection of selenider elements selecting:
#> • The first element with css selector "p".
#> • The direct children.

s("p") |>
  html_ancestors()
#> A collection of selenider elements selecting:
#> • The first element with css selector "p".
#> • The ancestors.
```

You can use `html_filter()` and `html_find()` to filter collections of
elements using a custom function. `html_find()` returns the first
matching element, while `html_filter()` returns all matching elements.

``` r
all_links <- ss("a")

ss("a") |>
  html_find(has_text("About R")) |>
  html_attr("href")
#> [1] "/about.html"

ss("p") |>
  html_filter(is_visible)
#> A collection of selenider elements selecting:
#> • The elements with css selector "p" matching the following condition:
#>   `is_visible`
```

selenider elements are *lazy*, meaning that when you specify the path to
an element or group of elements, they are not actually located in the
DOM until you *do* something with them.

There are three types of functions that force an element to be
collected: \* properties (e.g. `html_text()`) \* conditions
(e.g. `is_visible()`) \* actions (e.g. `click()`)

selenider provides a concise testing interface using the `html_expect()`
function. Provide an element, and one or more conditions, and the
function will wait until all the conditions are met. Conditions can be
functions or simple calls (e.g. `has_text("text")`)

``` r
s("#getting-started") |>
  html_expect(is_present) |>
  html_expect(has_text("Getting Started"))

s(".row") |>
  html_element("#r-foundation") |>
  html_expect(is_visible, is_enabled)

s("div[role='navigation']") |>
  html_element(".row") |>
  html_children() |>
  html_expect(has_at_least(2))

s("#help-with-r") |>
  html_expect(
    \(x) substring(html_text(x), 1, 1) == "H"
  )
```

Errors try to give as much information as possible.

``` r
s("#news") |>
  html_expect(has_text("Getting Started"))
#> Error in `html_expect()`:
#> ! Condition failed after waiting for 4 seconds:
#> `has_text("Getting Started")`
#> ℹ `x` does not have text "Getting Started".
#> ℹ Actual text: "News".
```

And (`&&`), or (`||`) and not (`!`) can be used as if the conditions
were logical values. Additionally, you can omit the first argument to
`html_expect()` (but all conditions must be calls).

``` r
s(".random-class") |>
  html_expect(!is_present)

s(".mt-timeline") |>
  html_expect(is_visible || is_enabled)

elem_1 <- s(".random-class")

elem_2 <- s("#getting-started")

# Test that either the first or second element exists
html_expect(is_present(elem_1) || is_present(elem_2))
```

<!-- TODO: Link to vignettes -->
