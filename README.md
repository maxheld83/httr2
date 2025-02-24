
<!-- README.md is generated from README.Rmd. Please edit that file -->

# httr2

<!-- badges: start -->

[![R-CMD-check](https://github.com/r-lib/httr2/workflows/R-CMD-check/badge.svg)](https://github.com/r-lib/httr2/actions)
[![Codecov test
coverage](https://codecov.io/gh/r-lib/httr2/branch/master/graph/badge.svg)](https://codecov.io/gh/r-lib/httr2?branch=master)

<!-- badges: end -->

httr2 is a ground-up rewrite of [httr](https://httr.r-lib.org) that
provides a pipeable API (with an explicit request object) and solves
more problems that making API wrappers a pain (e.g. rate-limiting,
retries, OAuth, storing secrets needed for testing, …).

## Installation

You can install the development version from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("r-lib/httr2")
```

## Usage

``` r
library(httr2)
```

To use httr2, start by creating a request:

``` r
req <- request("https://r-project.org")
req
#> <httr2_request>
#> GET https://r-project.org
#> Body: empty
```

You can tailor this request with the `req_` family of functions:

``` r
# Add custom headers
req %>% req_headers("Accept" = "application/json")
#> <httr2_request>
#> GET https://r-project.org
#> Headers:
#> • Accept: 'application/json'
#> Body: empty

# Add a body, turning it into a POST
req %>% req_body_json(list(x = 1, y = 2))
#> <httr2_request>
#> POST https://r-project.org
#> Body: json encoded data

# Automatically retry if the request fails
req %>% req_retry(max_tries = 5)
#> <httr2_request>
#> GET https://r-project.org
#> Body: empty
#> Policies:
#> • retry_max_tries: 5

# Change the HTTP method
req %>% req_method("PATCH")
#> <httr2_request>
#> PATCH https://r-project.org
#> Body: empty
```

You can see what httr2 will send to the server with `req_dry_run()`:

``` r
req %>% req_dry_run()
#> GET / HTTP/1.1
#> Host: r-project.org
#> User-Agent: httr2/0.0.0.9000 r-curl/4.3.1 libcurl/7.64.1
#> Accept: */*
#> Accept-Encoding: deflate, gzip
```

And perform the request, getting back a response, with `req_perform()`:

``` r
resp <- req_perform(req)
resp
#> <httr2_response>
#> GET https://www.r-project.org/
#> Status: 200 OK
#> Content-Type: text/html
#> Body: In memory (6089 bytes)
```

The `resp_` functions make it easy to work with the response:

``` r
resp %>% resp_content_type()
#> [1] "text/html"
resp %>% resp_status_desc()
#> [1] "OK"
resp %>% resp_body_html()
#> {html_document}
#> <html lang="en">
#> [1] <head>\n<meta http-equiv="Content-Type" content="text/html; charset=UTF-8 ...
#> [2] <body>\n    <div class="container page">\n      <div class="row">\n       ...
```

## Major differences to httr

-   You can now create and modify a request without performing it. This
    means that there’s now a single function to perform and request and
    fetch the result: `req_perform()`. (If you want to handle the
    response as it streams in, instead use `req_stream()`). This
    replaces `httr::GET()`, `POST()`, `DELETE()`, `VERB()`, …. You can
    preview a request without sending it with `req_dry_run()`.

-   HTTP errors are automatically converted into R errors. Use
    `req_error()` to override the defaults (which turn all 4xx and 5xx
    responses into errors) or to add additional details to the error
    message.

-   You can automatically retry if the request fails or encounters a
    transient HTTP error (e.g. a 429 rate limit request). `req_retry()`
    defines the maximum number of retries, which errors are transient,
    and how long to wait between tries.

-   OAuth support has been totally overhauled to directly support many
    more flows and to make it much easier to both customise the built-in
    flows and create your own.

-   You can manage encrypted secrets often needed for testing with
    `secret_encrpyt()` and friends. You can obfuscate mildly
    confidential data (like many client secrets) with `obfuscate()`,
    preventing them being scraped from published code.

-   You can automatically cache all cacheable results with
    `req_cache()`. Relatively few API responses are cacheable, but when
    they are it typically makes a big difference.

## Acknowledgements

httr2 wouldn’t be possible without
[curl](https://jeroen.cran.dev/curl/),
[openssl](https://github.com/jeroen/openssl/),
[jsonlite](https://jeroen.cran.dev/jsonlite/), and
[jose](https://github.com/jeroen/jose/), which are all maintained by
[Jeroen Ooms](https://github.com/jeroen). A big thanks also go to [Jenny
Bryan](https://jennybryan.org) and [Craig
Citro](https://research.google/people/CraigCitro/) who have given me
much useful feedback on the both design of the internals and the user
facing API.
