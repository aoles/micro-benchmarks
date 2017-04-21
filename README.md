-   [`data.frame` column names](#dataframe-column-names)
-   [Add elements to list](#add-elements-to-list)
-   [Coerce result of `strsplit` to a
    vector](#coerce-result-of-strsplit-to-a-vector)
-   [`sprintf` vs `paste`](#sprintf-vs-paste)
-   [`sapply(..., USE.NAMES=FALSE)` vs.
    `unlist(lapply(...))`](#sapply...-use.namesfalse-vs.-unlistlapply...)
-   [Interleaving two vectors](#interleaving-two-vectors)
-   [`tabulate` as a much faster alternative to
    `table`](#tabulate-as-a-much-faster-alternative-to-table)
-   [Session info](#session-info)

`data.frame` column names
-------------------------

Use `names` not `colnames` to access column names in `data.frames`.

    microbenchmark(names(mtcars), colnames(mtcars))

    ## Unit: nanoseconds
    ##              expr  min     lq    mean median   uq    max neval cld
    ##     names(mtcars)  523  555.0  635.95    581  674   2058   100   a
    ##  colnames(mtcars) 1398 1565.5 4553.64   1642 1800 281716   100   a

    identical(names(mtcars), colnames(mtcars))

    ## [1] TRUE

Or even better, use the list representation:

    list = as.list(mtcars)
    microbenchmark(names(list), names(mtcars))

    ## Unit: nanoseconds
    ##           expr min    lq   mean median  uq   max neval cld
    ##    names(list)  90  96.0 124.41  110.5 121  1065   100  a 
    ##  names(mtcars) 494 518.5 707.33  605.5 661 10281   100   b

    identical(names(list), names(mtcars))

    ## [1] TRUE

Add elements to list
--------------------

Use double brackets when adding/setting elements of list.

    l = list()

    microbenchmark(
      {l["a"] = 1},
      {l[["a"]] = 1}
    )

    ## Unit: nanoseconds
    ##                  expr  min     lq    mean median     uq   max neval cld
    ##    {     l["a"] = 1 } 1116 1198.5 1444.37 1262.5 1354.0 15958   100   b
    ##  {     l[["a"]] = 1 }  687  761.5  894.30  829.0  904.5  4736   100  a

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
    ##  unlist(strsplit(x, ", ", fixed = TRUE)) 2.353 2.5290 2.94063 2.6205 2.8045 21.122   100   b
    ##    strsplit(x, ", ", fixed = TRUE)[[1L]] 1.732 1.8395 1.99061 1.9040 2.0350  5.378   100  a

`sprintf` vs `paste`
--------------------

`sprintf` can be up to 2x faster.

    ext = "pdf"

    microbenchmark(
        paste(".*\\.", ext, "$", sep=""),
        paste0(".*\\.", ext, "$"),
        sprintf(".*\\.%s$", ext)
    )

    ## Unit: nanoseconds
    ##                                  expr  min     lq    mean median     uq   max neval cld
    ##  paste(".*\\\\.", ext, "$", sep = "") 1477 1674.0 2163.84 1842.5 1997.5 20190   100   b
    ##           paste0(".*\\\\.", ext, "$") 1346 1532.0 2201.95 1633.5 1829.0 11639   100   b
    ##            sprintf(".*\\\\.%s$", ext)  852  982.5 1304.60 1081.5 1219.0  6526   100  a

`sapply(..., USE.NAMES=FALSE)` vs. `unlist(lapply(...))`
--------------------------------------------------------

    x = letters

    microbenchmark(
      sapply(x, nchar, USE.NAMES=FALSE),
      unlist(lapply(x, nchar))
    )

    ## Unit: microseconds
    ##                                 expr    min     lq     mean  median      uq     max neval cld
    ##  sapply(x, nchar, USE.NAMES = FALSE) 54.178 56.896 60.81825 58.5395 60.4775 123.624   100   b
    ##             unlist(lapply(x, nchar)) 43.126 46.717 49.33468 48.0605 50.0660 103.364   100  a

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
    ##            expr    min      lq     mean  median      uq      max neval cld
    ##  c(rbind(a, b))  2.033  2.6950  3.27481  3.1815  3.4515   13.095   100   a
    ##         f(a, b)  2.358  2.9310 43.64800  3.4050  3.9855 3991.971   100   a
    ##         g(a, b) 23.592 24.5075 56.62345 25.0330 25.7850 3101.484   100   a
    ##         h(a, b) 17.013 19.3530 44.99752 20.3955 21.0155 2493.958   100   a

`tabulate` as a much faster alternative to `table`
--------------------------------------------------

For programming it typically makes more sense to call `tabulate` on a
well formed integer vector. If possible, specify `nbins`.

    library(EBImage)
    x = readImage(system.file('images', 'shapes.png', package='EBImage'))
    x = x[110:512,1:130]
    y = bwlabel(x)
    nbins = max(y)

    microbenchmark(
      table(y),
      tabulate(y),
      tabulate(y, nbins)
    )

    ## Unit: microseconds
    ##                expr       min        lq        mean    median         uq        max neval cld
    ##            table(y) 32800.918 34242.183 37759.15113 35392.569 36384.2835 142859.937   100   b
    ##         tabulate(y)   109.827   113.312   132.10748   120.545   134.0165    659.013   100  a 
    ##  tabulate(y, nbins)    63.007    65.842    72.44416    69.159    76.1415    126.470   100  a

Session info
------------

    sessionInfo()

    ## R version 3.4.0 alpha (2017-04-04 r72488)
    ## Platform: x86_64-pc-linux-gnu (64-bit)
    ## Running under: Fedora 18 (Spherical Cow)
    ## 
    ## Matrix products: default
    ## BLAS: /home/oles/R/R-alpha/lib/libRblas.so
    ## LAPACK: /home/oles/R/R-alpha/lib/libRlapack.so
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
    ## [1] EBImage_4.17.42        microbenchmark_1.4-2.1
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_0.12.10        compiler_3.4.0      plyr_1.8.4          tools_3.4.0        
    ##  [5] digest_0.6.12       evaluate_0.10       tibble_1.3.0        gtable_0.2.0       
    ##  [9] lattice_0.20-35     png_0.1-7           Matrix_1.2-9        yaml_2.1.14        
    ## [13] parallel_3.4.0      mvtnorm_1.0-6       stringr_1.2.0       knitr_1.15.1       
    ## [17] fftwtools_0.9-8     locfit_1.5-9.1      rprojroot_1.2       grid_3.4.0         
    ## [21] jpeg_0.1-8          survival_2.41-3     rmarkdown_1.4       multcomp_1.4-6     
    ## [25] TH.data_1.0-8       ggplot2_2.2.1       magrittr_1.5        backports_1.0.5    
    ## [29] scales_0.4.1        codetools_0.2-15    htmltools_0.3.5     splines_3.4.0      
    ## [33] MASS_7.3-47         BiocGenerics_0.21.3 abind_1.4-5         colorspace_1.3-2   
    ## [37] tiff_0.1-5          sandwich_2.3-4      stringi_1.1.5       lazyeval_0.2.0     
    ## [41] munsell_0.4.3       zoo_1.8-0
