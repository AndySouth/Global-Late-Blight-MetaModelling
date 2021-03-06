Reformat Patrick's Data
================

Reformat Patrick's Weather Data
-------------------------------

### Load required libraries

``` r
require("readr")
```

    ## Loading required package: readr

``` r
require("lubridate")
```

    ## Loading required package: lubridate

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
require("forecast")
```

    ## Loading required package: forecast

    ## Loading required package: zoo

    ## 
    ## Attaching package: 'zoo'

    ## The following objects are masked from 'package:base':
    ## 
    ##     as.Date, as.Date.numeric

    ## Loading required package: timeDate

    ## This is forecast 7.3

### Import data file

``` r
# import and rename columns on import, ignoring current names in file
station_data <-
  readr::read_csv(
    "~/Downloads/temp_humidity_Nakuru.csv",
    col_names = c("stationID", "datetime", "temperature", "relativeHumidity"),
    skip = 1
  )
```

    ## Parsed with column specification:
    ## cols(
    ##   stationID = col_character(),
    ##   datetime = col_datetime(format = ""),
    ##   temperature = col_integer(),
    ##   relativeHumidity = col_integer()
    ## )

``` r
# check the dataframe
summary(station_data)
```

    ##   stationID            datetime                    temperature      
    ##  Length:35001       Min.   :2012-12-31 22:00:00   Min.   :-9999.00  
    ##  Class :character   1st Qu.:2014-04-07 03:00:00   1st Qu.:   16.00  
    ##  Mode  :character   Median :2015-02-20 06:00:00   Median :   18.00  
    ##                     Mean   :2015-02-21 06:26:07   Mean   :   18.31  
    ##                     3rd Qu.:2016-01-24 08:00:00   3rd Qu.:   22.00  
    ##                     Max.   :2017-02-01 18:00:00   Max.   :   55.00  
    ##                                                   NA's   :27        
    ##  relativeHumidity
    ##  Min.   :  4.0   
    ##  1st Qu.: 50.0   
    ##  Median : 71.0   
    ##  Mean   : 66.7   
    ##  3rd Qu.: 88.0   
    ##  Max.   :100.0   
    ##  NA's   :39

``` r
# set -9999.00 values to NA
station_data[station_data == -9999.00] <- NA
```

### Interpolate

For the SimCast model to work in R there can not be any missing values, we'll interpolate to fill them in.

``` r
t <- as.ts(station_data$temperature)
station_data$t <- forecast::na.interp(t)

rh <- as.ts(station_data$relativeHumidity)
station_data$rh <- forecast::na.interp(rh)

station_data <- as.data.frame(c(station_data, t, rh))
```

### Format and write to disk

Now that we've fixed the missing values, let's clean up and write the new file out to disk

``` r
# Extract date/time bits
station_data$year <- year(station_data$datetime)
station_data$month <- month(station_data$datetime)
station_data$day <- day(station_data$datetime)
station_data$hour <- hour(station_data$datetime)

# drop first six rows since they are .5 hourly not hourly
station_data <- station_data[6:nrow(station_data), ]

# drop the original data with missing values
station_data <- station_data[, -c(3:4)]

# rearrange into the order that the "01 - SimCast_Blight_Units.R expects"
station_data <-
  station_data[c("stationID",
                 "year",
                 "month",
                 "day",
                 "hour",
                 "t",
                 "rh")]

# write new file to disk in format for use with "01 - SimCast_Blight_Units.R expects"
write_tsv(station_data, path = "~/Downloads/Nakuru.txt")
```

Appendix
--------

``` r
sessionInfo()
```

    ## R version 3.3.2 (2016-10-31)
    ## Platform: x86_64-apple-darwin16.3.0 (64-bit)
    ## Running under: macOS Sierra 10.12.3
    ## 
    ## locale:
    ## [1] en_AU.UTF-8/en_AU.UTF-8/en_AU.UTF-8/C/en_AU.UTF-8/en_AU.UTF-8
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] forecast_7.3      timeDate_3012.100 zoo_1.7-14        lubridate_1.6.0  
    ## [5] readr_1.0.0      
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_0.12.9        knitr_1.15.1       magrittr_1.5      
    ##  [4] munsell_0.4.3      colorspace_1.3-2   lattice_0.20-34   
    ##  [7] quadprog_1.5-5     plyr_1.8.4         stringr_1.1.0     
    ## [10] tools_3.3.2        nnet_7.3-12        parallel_3.3.2    
    ## [13] grid_3.3.2         gtable_0.2.0       htmltools_0.3.5   
    ## [16] tseries_0.10-37    lazyeval_0.2.0     yaml_2.1.14       
    ## [19] assertthat_0.1     rprojroot_1.2      digest_0.6.12     
    ## [22] tibble_1.2         ggplot2_2.2.1      evaluate_0.10     
    ## [25] rmarkdown_1.3.9002 fracdiff_1.4-2     stringi_1.1.2     
    ## [28] scales_0.4.1       backports_1.0.5
