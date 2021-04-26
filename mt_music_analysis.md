Measures Taken Formal Musical Analysis
================
Noah Zeldin
4/26/2021

# Preliminary Set-up

Load packages

``` r
library(tidyverse)
```

    ## -- Attaching packages --------------------------------------- tidyverse 1.3.0 --

    ## v ggplot2 3.3.2     v purrr   0.3.4
    ## v tibble  3.0.4     v dplyr   1.0.2
    ## v tidyr   1.1.2     v stringr 1.4.0
    ## v readr   1.4.0     v forcats 0.5.0

    ## Warning: package 'tibble' was built under R version 4.0.3

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(readxl)
library(ggforce)
library(scales)
```

    ## 
    ## Attaching package: 'scales'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     discard

    ## The following object is masked from 'package:readr':
    ## 
    ##     col_factor

``` r
library(lubridate)
```

    ## Warning: package 'lubridate' was built under R version 4.0.3

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
library(mosaic) # may be unnecessary
```

    ## Registered S3 method overwritten by 'mosaic':
    ##   method                           from   
    ##   fortify.SpatialPolygonsDataFrame ggplot2

    ## 
    ## The 'mosaic' package masks several functions from core packages in order to add 
    ## additional features.  The original behavior of these functions should not be affected by this.

    ## 
    ## Attaching package: 'mosaic'

    ## The following object is masked from 'package:Matrix':
    ## 
    ##     mean

    ## The following object is masked from 'package:scales':
    ## 
    ##     rescale

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     count, do, tally

    ## The following object is masked from 'package:purrr':
    ## 
    ##     cross

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     stat

    ## The following objects are masked from 'package:stats':
    ## 
    ##     binom.test, cor, cor.test, cov, fivenum, IQR, median, prop.test,
    ##     quantile, sd, t.test, var

    ## The following objects are masked from 'package:base':
    ## 
    ##     max, mean, min, prod, range, sample, sum

# Importation

## Importation - Duration

check worksheets \* ensure that there is a sheet for each piece except
2a; should be 21

``` r
excel_sheets("Massnahme_durations_meter_changes.xlsx")
```

    ##  [1] "1 Vorspiel"                     "2b Lob der U.S.S.R."           
    ##  [3] "3a Rezitativ"                   "3b Sprechchor"                 
    ##  [5] "4 Lob der illegalen Arbeit"     "5 Gesang der Reiskahnschlepper"
    ##  [7] "6a"                             "6b Lenin-Zitat"                
    ##  [9] "6c Kanon über ein Lenin-Zitat"  "7a Streiklied"                 
    ## [11] "7b"                             "8a Rezitativ"                  
    ## [13] "8b Song von der Ware"           "9 Ändere die Welt"             
    ## [15] "10 Lob der Partei"              "11 Rezitativ"                  
    ## [17] "12a Rezitativ"                  "12b Wir sind der Abschaum"     
    ## [19] "13a  untitled "                 "13b untitled"                  
    ## [21] "14 Schlusschor"

import all sheets into a single list and specify column types

``` r
durations_sheets <- excel_sheets("Massnahme_durations_meter_changes.xlsx")

durations_list <- lapply(durations_sheets, 
                         function(x) 
                             read_excel("Massnahme_durations_meter_changes.xlsx", 
                                                  sheet = x, 
                                                          col_types = c(
                                                              "text", # piece_no
                                                              "numeric", # category
                                                              "text", # subcategory
                                                              "numeric", # segment
                                                              "numeric", # m_start
                                                              "numeric", # m_end
                                                              "numeric", # no_of_mm
                                                              "numeric", # meter_1
                                                              "numeric", # meter_2
                                                              "numeric", # meter_ch_count
                                                              "numeric", # quarters_per_bar
                                                              "numeric", # beats
                                                              "numeric", # tempo
                                                              "numeric", # tempo_ch_count
                                                              "numeric") # duration
                                                          ))
```

from list create single data frame and save as tibble

  - can probably combine into single command

<!-- end list -->

``` r
dur_df <- bind_rows(durations_list)

dur_tib <- as_tibble(dur_df)
```

## Importation - Voice Analysis

check worksheets \* ensure that there is a sheet for each choral piece

``` r
excel_sheets("Massnahme_choir_voice_analysis.xlsx")
```

    ##  [1] "1 Vorspiel"                     "2b Lob der U.S.S.R."           
    ##  [3] "3b Sprechchor"                  "4 Lob der illegalen Arbeit"    
    ##  [5] "5 Gesang der Reiskahnschlepper" "6a (untitled)"                 
    ##  [7] "6b Lenin-Zitat (Sprechchor)"    "6c Kanon über ein Lenin-Zitat" 
    ##  [9] "7a Streiklied"                  "7b (untitled)"                 
    ## [11] "8b Song von der Ware"           "9 Ändere die Welt"             
    ## [13] "10 Lob der Partei"              "11 Rezitativ"                  
    ## [15] "12b Wir sind der Abschaum"      "13a (untitled)"                
    ## [17] "13b (untitled)"                 "14 Schlusschor"

import all sheets into a single list and specify column types

``` r
voice_analysis_sheets <- excel_sheets("Massnahme_choir_voice_analysis.xlsx")

voice_analysis_list <- lapply(voice_analysis_sheets, 
                           function(x) read_excel("Massnahme_choir_voice_analysis.xlsx", 
                                                  sheet = x, 
                                                          col_types = c(
                                                              "text", # piece_no
                                                              "numeric", # measure
                                                              "text", # texture
                                                              "numeric", # voices
                                                              "text", # groupings
                                                              
                                                              "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", # soprano:ratio
                                                              
                                                              "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", # rests
                                                              
                                                              "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", # quarters
                                                              
                                                              "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", # notes
                                                              
                                                              "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "text", "numeric", # tones
                                                              
                                                              "numeric", "numeric", # spoken, acapella
                                                              
                                                              "numeric", "numeric", "numeric", "numeric", # meter etc.
                                                              
                                                              "numeric", "numeric", "numeric", "numeric", # general dur's
                                                              
                                                              "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric", "numeric" # dur's of voices
                                                              ) 
                                                              ))
```

    ## Warning in read_fun(path = enc2native(normalizePath(path)), sheet_i = sheet, :
    ## Expecting numeric in K24 / R24C11: got 'na'

    ## Warning in read_fun(path = enc2native(normalizePath(path)), sheet_i = sheet, :
    ## Expecting numeric in K25 / R25C11: got 'na'

    ## Warning in read_fun(path = enc2native(normalizePath(path)), sheet_i = sheet, :
    ## Expecting numeric in K26 / R26C11: got 'na'

    ## Warning in read_fun(path = enc2native(normalizePath(path)), sheet_i = sheet, :
    ## Expecting numeric in K27 / R27C11: got 'na'

    ## Warning in read_fun(path = enc2native(normalizePath(path)), sheet_i = sheet, :
    ## Expecting numeric in K50 / R50C11: got 'na'

from list create single data frame and save as tibble

  - again, probably combine into single command

<!-- end list -->

``` r
gen_df <- bind_rows(voice_analysis_list)

gen_tib <- as_tibble(gen_df)
```
