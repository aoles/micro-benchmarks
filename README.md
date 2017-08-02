-   [`data.frame` column names](#dataframe-column-names)
-   [Add elements to list](#add-elements-to-list)
-   [Coerce result of `strsplit` to a
    vector](#coerce-result-of-strsplit-to-a-vector)
-   [`sprintf` vs `paste`](#sprintf-vs-paste)
-   [`sapply(..., USE.NAMES=FALSE)` vs.
    `unlist(lapply(...))`](#sapply...-use.namesfalse-vs.-unlistlapply...)
-   [Interleaving two vectors](#interleaving-two-vectors)
-   [Insert elements into a vector](#insert-elements-into-a-vector)
-   [`tabulate` as a much faster alternative to
    `table`](#tabulate-as-a-much-faster-alternative-to-table)
-   [Session info](#session-info)

`data.frame` column names
-------------------------

Use `names` not `colnames` to access column names in `data.frames`.

    microbenchmark(names(mtcars), colnames(mtcars))

    ## Unit: nanoseconds
    ##              expr  min     lq    mean median     uq    max neval cld
    ##     names(mtcars)  530  562.0  631.39    594  653.5   2338   100   a
    ##  colnames(mtcars) 1389 1493.5 4565.93   1576 1656.5 294788   100   a

    identical(names(mtcars), colnames(mtcars))

    ## [1] TRUE

Or even better, use the list representation:

    list = as.list(mtcars)
    microbenchmark(names(list), names(mtcars))

    ## Unit: nanoseconds
    ##           expr min    lq   mean median    uq   max neval cld
    ##    names(list)  89  95.0 166.11  104.5 119.5  5507   100  a 
    ##  names(mtcars) 509 528.5 695.70  553.0 646.0 10128   100   b

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
    ##    {     l["a"] = 1 } 1122 1191.5 1333.22 1247.0 1340.5  6490   100   b
    ##  {     l[["a"]] = 1 }  654  733.0  994.24  830.5  899.0 15287   100  a

Coerce result of `strsplit` to a vector
---------------------------------------

Take the first element of the list rather than unlisting.

    x = paste(letters[1:3], collapse = ", ")

    microbenchmark(
      unlist(strsplit(x, ", ", fixed=TRUE)),
      strsplit(x, ", ", fixed=TRUE)[[1L]]
    )

    ## Unit: microseconds
    ##                                     expr   min    lq    mean median     uq    max neval cld
    ##  unlist(strsplit(x, ", ", fixed = TRUE)) 2.043 2.202 2.48234 2.3115 2.4625 10.022   100   b
    ##    strsplit(x, ", ", fixed = TRUE)[[1L]] 1.491 1.588 1.89903 1.6770 1.7555 20.505   100  a

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
    ##  paste(".*\\\\.", ext, "$", sep = "") 1484 1624.5 2062.28 1714.5 1798.5 32772   100   b
    ##           paste0(".*\\\\.", ext, "$") 1352 1440.5 1584.85 1513.0 1592.0  6480   100  ab
    ##            sprintf(".*\\\\.%s$", ext)  755  876.5 1008.43  906.5 1024.0  7763   100  a

`sapply(..., USE.NAMES=FALSE)` vs. `unlist(lapply(...))`
--------------------------------------------------------

    x = letters

    microbenchmark(
      sapply(x, nchar, USE.NAMES=FALSE),
      unlist(lapply(x, nchar))
    )

    ## Unit: microseconds
    ##                                 expr    min      lq     mean  median     uq     max neval cld
    ##  sapply(x, nchar, USE.NAMES = FALSE) 49.899 51.4890 55.96297 53.5370 55.272 120.106   100   b
    ##             unlist(lapply(x, nchar)) 38.819 41.8505 46.05364 43.5625 45.148 107.046   100  a

Interleaving two vectors
------------------------

The fastest approach is to use "the `rbind` trick".

    length = 1000L
    a <- rep_len(letters, length)
    b <- rep_len(LETTERS, length)

    r <- function(a, b) {
      as.vector(rbind(a, b))
    }

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
      r(a, b),
      f(a, b),
      g(a, b),
      h(a, b)
    )

    ## Unit: microseconds
    ##     expr      min        lq       mean    median        uq      max neval cld
    ##  r(a, b)   14.187   15.5645   37.31824   17.9040   23.2235 1752.796   100  a 
    ##  f(a, b)   23.910   25.7760   69.45247   27.6105   31.9575 4056.845   100  a 
    ##  g(a, b)   71.064   73.6400  111.75248   78.5730   92.8485 2738.708   100  a 
    ##  h(a, b) 1171.796 1301.3695 1456.34797 1365.8200 1453.6545 3857.610   100   b

Insert elements into a vector
-----------------------------

Fill-in a preallocated results vector.

    n = 1000
    lines = sample(letters, n, TRUE)
    idx = sort(sample(n, size = 10))

    f = function(lines, idx, txt = "whatever") {
      idx_length = length(idx)
      lines_length = length(lines)
      v = vector(mode = "character", length = lines_length+idx_length)
      idx <- idx + seq_len(idx_length)
      v[-idx] <- lines
      v[idx] <- txt
      v
    }

    g = function(lines, idx, txt = "whatever") {
      v = rep_len(NA_character_, length(lines))
      v[idx] = txt
      v = as.vector(rbind(lines, v))
      v[!is.na(v)]
    }

    identical(f(lines, idx), g(lines, idx))

    ## [1] TRUE

    microbenchmark(
      f(lines, idx),
      g(lines, idx)
    )

    ## Unit: microseconds
    ##           expr    min     lq     mean  median      uq      max neval cld
    ##  f(lines, idx) 10.974 11.799 64.70326 12.5175 12.8900 5227.229   100   a
    ##  g(lines, idx) 35.482 36.308 77.54713 36.8370 37.3295 4072.784   100   a

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
    ##                expr       min         lq        mean     median        uq        max neval cld
    ##            table(y) 32628.446 33585.6900 36170.89465 34998.3875 35889.015 147546.086   100   b
    ##         tabulate(y)   108.449   110.8085   128.50483   117.2525   133.003    717.008   100  a 
    ##  tabulate(y, nbins)    61.447    63.0080    69.05595    65.8845    74.127     87.238   100  a

Session info
------------

    sessionInfo()

    ## R version 3.4.1 (2017-06-30)
    ## Platform: x86_64-pc-linux-gnu (64-bit)
    ## Running under: Fedora 18 (Spherical Cow)
    ## 
    ## Matrix products: default
    ## BLAS: /home/oles/R/R-3.4.1/lib/libRblas.so
    ## LAPACK: /home/oles/R/R-3.4.1/lib/libRlapack.so
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
    ## [1] EBImage_4.19.6         microbenchmark_1.4-2.1
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_0.12.12        compiler_3.4.1      plyr_1.8.4          bitops_1.0-6       
    ##  [5] tools_3.4.1         digest_0.6.12       evaluate_0.10.1     tibble_1.3.3       
    ##  [9] gtable_0.2.0        lattice_0.20-35     png_0.1-7           rlang_0.1.1        
    ## [13] Matrix_1.2-10       yaml_2.1.14         parallel_3.4.1      mvtnorm_1.0-6      
    ## [17] stringr_1.2.0       knitr_1.16          htmlwidgets_0.9     fftwtools_0.9-8    
    ## [21] locfit_1.5-9.1      rprojroot_1.2       grid_3.4.1          jpeg_0.1-8         
    ## [25] survival_2.41-3     rmarkdown_1.6       multcomp_1.4-6      TH.data_1.0-8      
    ## [29] ggplot2_2.2.1       magrittr_1.5        backports_1.1.0     scales_0.4.1       
    ## [33] codetools_0.2-15    htmltools_0.3.6     splines_3.4.1       MASS_7.3-47        
    ## [37] BiocGenerics_0.22.0 abind_1.4-5         colorspace_1.3-2    tiff_0.1-5         
    ## [41] sandwich_2.4-0      stringi_1.1.5       RCurl_1.95-4.8      lazyeval_0.2.0     
    ## [45] munsell_0.4.3       zoo_1.8-0
