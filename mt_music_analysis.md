Measures Taken Formal Musical Analysis
================
Noah Zeldin
4/26/2021

# Preliminary Set-up

Load packages

``` r
library(tidyverse)
library(readxl)
library(ggforce)
library(scales)
library(lubridate)
library(mosaic) # may be unnecessary
```

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

from list create single data frame and save as tibble

  - again, probably combine into single command

<!-- end list -->

``` r
gen_df <- bind_rows(voice_analysis_list)

gen_tib <- as_tibble(gen_df)
```
