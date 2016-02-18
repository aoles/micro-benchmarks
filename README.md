-   [`data.frame` column names](#dataframe-column-names)
-   [Session info](#session-info)

`data.frame` column names
-------------------------

Use `names` not `colnames` to access column names in `data.frames`.

    microbenchmark(names(mtcars), colnames(mtcars))

    ## Unit: nanoseconds
    ##              expr  min     lq    mean median     uq    max neval cld
    ##     names(mtcars)  494  528.5 3288.18  574.0  670.5 268100   100   a
    ##  colnames(mtcars) 1282 1404.0 2686.26 1626.5 1811.0  51153   100   a

    identical(names(mtcars), colnames(mtcars))

    ## [1] TRUE

Session info
------------

    sessionInfo()

    ## R version 3.2.3 (2015-12-10)
    ## Platform: x86_64-pc-linux-gnu (64-bit)
    ## Running under: Fedora 18 (Spherical Cow)
    ## 
    ## locale:
    ##  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C              
    ##  [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8    
    ##  [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8   
    ##  [7] LC_PAPER=en_US.UTF-8       LC_NAME=C                 
    ##  [9] LC_ADDRESS=C               LC_TELEPHONE=C            
    ## [11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C       
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] microbenchmark_1.4-2.1 BiocStyle_1.9.3       
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_0.12.3      knitr_1.12.3     magrittr_1.5     MASS_7.3-45     
    ##  [5] splines_3.2.3    munsell_0.4.3    colorspace_1.2-6 lattice_0.20-33 
    ##  [9] multcomp_1.4-3   stringr_1.0.0    plyr_1.8.3       tools_3.2.3     
    ## [13] grid_3.2.3       gtable_0.1.2     TH.data_1.0-7    htmltools_0.3   
    ## [17] yaml_2.1.13      survival_2.38-3  digest_0.6.9     ggplot2_2.0.0   
    ## [21] formatR_1.2.1    codetools_0.2-14 evaluate_0.8     rmarkdown_0.9.2 
    ## [25] sandwich_2.3-4   stringi_1.0-1    scales_0.3.0     mvtnorm_1.0-5   
    ## [29] zoo_1.7-12
