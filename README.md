-   [`data.frame` column names](#dataframe-column-names)
-   [Add elements to list](#add-elements-to-list)
-   [Coerce result of `strsplit` to a
    vector](#coerce-result-of-strsplit-to-a-vector)
-   [`sprintf` vs `paste`](#sprintf-vs-paste)
-   [`sapply(..., USE.NAMES=FALSE)` vs.
    `unlist(lapply(...))`](#sapply...-use.namesfalse-vs.-unlistlapply...)
-   [Interleaving two vectors](#interleaving-two-vectors)
-   [Session info](#session-info)

`data.frame` column names
-------------------------

Use `names` not `colnames` to access column names in `data.frames`.

    microbenchmark(names(mtcars), colnames(mtcars))

    ## Unit: nanoseconds
    ##              expr  min   lq    mean median     uq    max neval cld
    ##     names(mtcars)  474  517  608.20    563  642.0   2272   100   a
    ##  colnames(mtcars) 1328 1418 5758.38   1525 1695.5 374158   100   a

    identical(names(mtcars), colnames(mtcars))

    ## [1] TRUE

Or even better, use the list representation:

    list = as.list(mtcars)
    microbenchmark(names(list), names(mtcars))

    ## Unit: nanoseconds
    ##           expr min    lq   mean median    uq   max neval cld
    ##    names(list)  91  96.0 263.00  108.5 119.5 13860   100  a 
    ##  names(mtcars) 495 508.5 693.59  562.5 648.5 10851   100   b

    identical(names(list), names(mtcars))

    ## [1] TRUE

Add elements to list
--------------------

Use double brackets when adding/setting elements of list.

    list = list()
    microbenchmark({list["a"] = 1}, {list[["a"]] = 1})

    ## Unit: nanoseconds
    ##                     expr  min     lq    mean median     uq   max neval cld
    ##    {     list["a"] = 1 } 1188 1289.0 1593.06 1379.5 1536.5 17043   100   a
    ##  {     list[["a"]] = 1 }  711  840.5 1237.55  920.0 1044.5 15824   100   a

Coerce result of `strsplit` to a vector
---------------------------------------

Take the first element of the list rather than unlisting.

    x = paste(letters[1:3], collapse = ", ")

    microbenchmark(
      unlist(strsplit(x, ", ", fixed=TRUE)),
      strsplit(x, ", ", fixed=TRUE)[[1L]]
    )

    ## Unit: microseconds
    ##                                     expr   min     lq    mean median     uq    max neval cld
    ##  unlist(strsplit(x, ", ", fixed = TRUE)) 2.001 2.1105 2.69130 2.2130 2.4555 18.780   100   b
    ##    strsplit(x, ", ", fixed = TRUE)[[1L]] 1.324 1.5005 1.73837 1.5865 1.7225 11.459   100  a

`sprintf` vs `paste`
--------------------

`sprintf` can be almost 2x faster~

    ext = "pdf"

    microbenchmark(
        paste(".*\\.", ext, "$", sep=""),
        paste0(".*\\.", ext, "$"),
        sprintf(".*\\.%s$", ext)
    )

    ## Unit: nanoseconds
    ##                                  expr  min   lq    mean median   uq   max neval cld
    ##  paste(".*\\\\.", ext, "$", sep = "") 1506 1627 1993.99 1726.5 1869 13651   100   b
    ##           paste0(".*\\\\.", ext, "$") 1309 1444 1689.28 1549.5 1658 13216   100   b
    ##            sprintf(".*\\\\.%s$", ext)  736  900 1190.70  940.0 1022 12710   100  a

`sapply(..., USE.NAMES=FALSE)` vs. `unlist(lapply(...))`
--------------------------------------------------------

    x = letters
    microbenchmark(
      sapply(x, nchar, USE.NAMES=FALSE),
      unlist(lapply(x, nchar))
    )

    ## Unit: microseconds
    ##                                 expr    min     lq     mean  median     uq     max neval cld
    ##  sapply(x, nchar, USE.NAMES = FALSE) 49.924 52.009 58.10911 53.9000 63.467 117.371   100   b
    ##             unlist(lapply(x, nchar)) 40.009 41.725 45.52771 42.9345 49.612  63.608   100  a

Interleaving two vectors
------------------------

The fastest approach is to use the `c(rbind(...)` trick.

    length = 10L
    a <- letters[1:length]
    b <- LETTERS[1:length]

    f <- function(a, b) {
      v = vector(mode = "character", length = 2L*length)
        idx <- 2L*1:length
        v[idx-1L] = a
        v[idx] = b
        v
    }


    g <- function(a, b) {
      idx <- order(c(seq_along(a), seq_along(b)))
        c(a,b)[idx]
    }


    h <- function(a, b) {
      unlist(mapply(function(x, y) c(x, y), a, b, SIMPLIFY=FALSE, USE.NAMES=FALSE))
    }

    microbenchmark(
      c(rbind(a, b)),
      f(a, b),
      g(a, b),
      h(a, b)
    )

    ## Unit: microseconds
    ##            expr    min      lq     mean  median      uq    max neval  cld
    ##  c(rbind(a, b))  1.800  2.5760  3.06521  2.9955  3.2770 12.485   100 a   
    ##         f(a, b)  4.365  5.1740  6.85454  5.9000  6.4035 26.867   100  b  
    ##         g(a, b) 14.096 15.2775 17.93229 16.1320 17.3550 68.373   100   c 
    ##         h(a, b) 16.694 18.7015 22.03209 19.5500 23.9730 50.286   100    d

Session info
------------

    sessionInfo()

    ## R version 3.3.1 (2016-06-21)
    ## Platform: x86_64-pc-linux-gnu (64-bit)
    ## Running under: Fedora 18 (Spherical Cow)
    ## 
    ## locale:
    ##  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C               LC_TIME=en_US.UTF-8       
    ##  [4] LC_COLLATE=en_US.UTF-8     LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8   
    ##  [7] LC_PAPER=en_US.UTF-8       LC_NAME=C                  LC_ADDRESS=C              
    ## [10] LC_TELEPHONE=C             LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C       
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] microbenchmark_1.4-2.1
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_0.12.7      knitr_1.14.11    magrittr_1.5     MASS_7.3-45      splines_3.3.1   
    ##  [6] munsell_0.4.3    colorspace_1.2-7 lattice_0.20-34  multcomp_1.4-6   stringr_1.1.0   
    ## [11] plyr_1.8.4       tools_3.3.1      grid_3.3.1       gtable_0.2.0     TH.data_1.0-7   
    ## [16] htmltools_0.3.5  yaml_2.1.13      survival_2.39-5  assertthat_0.1   digest_0.6.10   
    ## [21] tibble_1.2       Matrix_1.2-7.1   ggplot2_2.1.0    codetools_0.2-15 evaluate_0.10   
    ## [26] rmarkdown_1.1    sandwich_2.3-4   stringi_1.1.2    scales_0.4.0     mvtnorm_1.0-5   
    ## [31] zoo_1.7-13
