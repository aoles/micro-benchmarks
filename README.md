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
    ##              expr  min     lq    mean median     uq    max neval cld
    ##     names(mtcars)  535  560.0  739.82  592.5  690.5   3835   100   a
    ##  colnames(mtcars) 1382 1534.5 5051.87 1596.0 1725.5 295572   100   a

    identical(names(mtcars), colnames(mtcars))

    ## [1] TRUE

Or even better, use the list representation:

    list = as.list(mtcars)
    microbenchmark(names(list), names(mtcars))

    ## Unit: nanoseconds
    ##           expr min  lq   mean median    uq   max neval cld
    ##    names(list)  89 103 248.91    109 127.0 12312   100  a 
    ##  names(mtcars) 519 534 733.18    578 683.5  9894   100   b

    identical(names(list), names(mtcars))

    ## [1] TRUE

Add elements to list
--------------------

Use double brackets when adding/setting elements of list.

    list = list()
    microbenchmark({list["a"] = 1}, {list[["a"]] = 1})

    ## Unit: nanoseconds
    ##                     expr  min     lq    mean median     uq   max neval cld
    ##    {     list["a"] = 1 } 1233 1318.5 1638.92 1446.5 1581.5 14788   100   a
    ##  {     list[["a"]] = 1 }  748  926.0 1229.74 1009.0 1170.5 16690   100   a

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
    ##  unlist(strsplit(x, ", ", fixed = TRUE)) 2.080 2.2300 2.50111 2.3525 2.6845  6.666   100   a
    ##    strsplit(x, ", ", fixed = TRUE)[[1L]] 1.456 1.6085 2.21879 1.6885 1.8335 17.805   100   a

`sprintf` vs `paste`
--------------------

`sprintf` can be almost 2x faster~

    ext = "pdf"

    microbenchmark(
        paste(".*\\.", ext, "$", sep=""),
        paste0(".*\\.", ext, "$"),
        sprintf(".*\\.%s$", ext)
    )

    ## Unit: microseconds
    ##                                  expr   min    lq    mean median     uq    max neval cld
    ##  paste(".*\\\\.", ext, "$", sep = "") 2.858 3.276 4.51862 3.4625 3.6540 57.283   100   b
    ##           paste0(".*\\\\.", ext, "$") 2.659 2.976 3.45894 3.0815 3.2805 17.671   100   b
    ##            sprintf(".*\\\\.%s$", ext) 1.478 1.881 2.10084 1.9915 2.1745  9.147   100  a

`sapply(..., USE.NAMES=FALSE)` vs. `unlist(lapply(...))`
--------------------------------------------------------

    x = letters
    microbenchmark(
      sapply(x, nchar, USE.NAMES=FALSE),
      unlist(lapply(x, nchar))
    )

    ## Unit: microseconds
    ##                                 expr    min     lq     mean  median      uq     max neval cld
    ##  sapply(x, nchar, USE.NAMES = FALSE) 51.270 53.426 60.60040 56.2080 66.9300 115.998   100   b
    ##             unlist(lapply(x, nchar)) 39.901 42.836 48.45184 45.1715 53.5155  81.054   100  a

Interleaving two vectors
------------------------

    length = 10
    a <- letters[1:length]
    b <- LETTERS[1:length]

    microbenchmark(
      unlist(mapply(function(x, y) c(x, y), a, b, SIMPLIFY=FALSE, USE.NAMES=FALSE)),
      c(rbind(a, b)),
      {
        v = vector(mode = "character", length = 2*length)
        idx <- 2*1:10
        v[idx-1] = a
        v[idx] = b
      },
      {
        idx <- order(c(seq_along(a), seq_along(b)))
        c(a,b)[idx]
      }
    )

    ## Unit: microseconds
    ##                                                                                                               expr
    ##                             unlist(mapply(function(x, y) c(x, y), a, b, SIMPLIFY = FALSE,      USE.NAMES = FALSE))
    ##                                                                                                     c(rbind(a, b))
    ##  {     v = vector(mode = "character", length = 2 * length)     idx <- 2 * 1:10     v[idx - 1] = a     v[idx] = b }
    ##                                               {     idx <- order(c(seq_along(a), seq_along(b)))     c(a, b)[idx] }
    ##     min      lq     mean  median      uq     max neval  cld
    ##  16.305 18.8595 22.86391 19.8355 22.3355 114.557   100    d
    ##   2.111  2.7080  3.62464  3.1655  3.6545  14.456   100 a   
    ##   4.486  5.3545  6.74521  6.1155  6.4835  24.078   100  b  
    ##  14.600 15.7380 18.34568 16.5050 17.3940  57.736   100   c

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
