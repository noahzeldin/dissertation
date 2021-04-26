Measures Taken Formal Musical Analysis
================
Noah Zeldin
4/26/2021

# Load packages

``` r
library(tidyverse)
library(readxl)
library(ggforce)
library(scales)
library(lubridate)
library(mosaic) # may be unnecessary
```

# Importation

## Duration

Import all sheets into a single list and specify column types.

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

From list create single data frame and save as tibble.

``` r
dur_tib <- as_tibble(bind_rows(durations_list))
```

## Voice Analysis

Check worksheets and ensure that there is a sheet for each choral piece.

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

Import all sheets into a single list and specify column types.

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

From list create single data frame and save as tibble.

``` r
gen_tib <- as_tibble(bind_rows(voice_analysis_list))
```

# Cleaning and Manipulation: 1. Durations

Convert **piece\_no**, **category** and **subcategory** to factors.

``` r
dur_tib <- dur_tib %>% 
    mutate(piece_no = as_factor(piece_no)) %>% 
    mutate(category = as_factor(category)) %>% 
    mutate(subcategory = as_factor(subcategory))
```

<!-- Check to see if each piece is now a factor. -->

<!-- * NB: n col. refers to # of segments -->

<!-- ```{r} -->

<!-- dur_tib$piece_no %>%  -->

<!--     fct_count() -->

<!-- ``` -->

<!-- same for category -->

<!-- ```{r} -->

<!-- dur_tib$category %>%  -->

<!--     fct_count() -->

<!-- ``` -->

Relevel subcategories.

``` r
dur_tib$subcategory <- dur_tib$subcategory %>% 
    fct_relevel("1b", after = 1) %>% 
    fct_relevel("2b", after = 2) %>% 
    fct_relevel("3b", after = 4) %>% 
    fct_relevel("3c", after = 5)

dur_tib$subcategory %>% 
    fct_count()
```

    ## # A tibble: 10 x 2
    ##    f         n
    ##    <fct> <int>
    ##  1 1a       58
    ##  2 1b        9
    ##  3 2b        5
    ##  4 3a        4
    ##  5 3b       18
    ##  6 3c        8
    ##  7 4a       16
    ##  8 4b        1
    ##  9 4c        1
    ## 10 4d        4

# Cleaning and Manipulation: 2. Voice Analysis

## General Tibble: gen\_tib

Convert **piece\_no** to factor.

``` r
gen_tib <- gen_tib %>% 
    mutate(piece_no = as_factor(piece_no))
```

<!-- Check to see if each piece is now a factor. -->

<!-- * **Add something about n = # of mm. - and check.** -->

<!-- ```{r} -->

<!-- gen_tib$piece_no %>%  -->

<!--     fct_count() -->

<!-- ``` -->

Since 6c now follows 7a, must relevel (and check).

``` r
gen_tib$piece_no <- gen_tib$piece_no %>% 
    fct_relevel("6c", after = 7)

gen_tib$piece_no %>% 
    fct_count()
```

    ## # A tibble: 18 x 2
    ##    f         n
    ##    <fct> <int>
    ##  1 1       118
    ##  2 2b       49
    ##  3 3b       26
    ##  4 4        72
    ##  5 5       205
    ##  6 6a        3
    ##  7 6b        9
    ##  8 6c       16
    ##  9 7a       72
    ## 10 7b        2
    ## 11 8b      184
    ## 12 9        52
    ## 13 10       70
    ## 14 11       29
    ## 15 12b      57
    ## 16 13a       4
    ## 17 13b      17
    ## 18 14       96

Group by piece.

``` r
by_piece <- gen_tib %>% group_by(piece_no)
```

Add **rowid** column and make first column (called “id”).

``` r
gen_tib <- gen_tib %>% 
    rowid_to_column("id")
```

### Additional Calculations for Density

**NB: This section will in all likelihood be deleted, since it
ultimately did not figure into the final analysis.**

Translate texture to numeric value in new column:

  - monophony = 1

  - homophony = 1.5

  - antiphony = 1.5

  - polyphony = 2

<!-- end list -->

``` r
gen_tib <- gen_tib %>% 
    mutate(texture_value = case_when(texture == "na" ~ "0", 
                                   texture == "m" ~ "1", 
                                   texture == "h" ~ "1.5", 
                                   texture == "a" ~ "1.5", # added 12.26.20
                                   texture == "p" ~ "2",)) %>% 
    mutate(texture_value = as.numeric(texture_value)) %>% 
    relocate(texture_value, .after = texture)
```

Add density column for each voice.

``` r
gen_tib <- gen_tib %>% 
    mutate(dm_s1 = (notes_s1+tones_s1)/quarters_per_bar,
                      dm_s2 = (notes_s2+tones_s2)/quarters_per_bar,
                      dm_a1 = (notes_a1+tones_a1)/quarters_per_bar,
                      dm_a2 = (notes_a2+tones_a2)/quarters_per_bar,
                      dm_t1 = (notes_t1+tones_t1)/quarters_per_bar,
                      dm_t2 = (notes_t2+tones_t2)/quarters_per_bar,
                      dm_b1 = (notes_b1+tones_b1)/quarters_per_bar,
                      dm_b2 = (notes_b2+tones_b2)/quarters_per_bar)
```

Create a **dm\_sum** column from 8 individual **dm** columns.

``` r
gen_tib <- gen_tib %>% 
    rowwise() %>% 
    mutate(dm_sum = sum(c_across(dm_s1:dm_b2)))
```

With new **dm\_sum** column, create **dmc** column.

``` r
gen_tib <- gen_tib %>% 
    rowwise() %>% 
    mutate(dmc = (dm_sum/4)*texture_value)
```

Need to reconvert gen\_tib to tibble after performing `rowwise()`.

``` r
gen_tib <- gen_tib %>% as_tibble()
```

## Tibble for Choral Portions: gen\_tib\_sung

Filter out parts without choral singing. (This includes passages spoken
by the choir.) \* **CHECK IF NECESSARY TO ADD** `as_tibble()` **TO
END.**

``` r
gen_tib_sung <- gen_tib %>% 
    filter(texture != "na" & 
               parts_active > 0 & 
               spoken == 0) %>% 
    as_tibble() # may not be necessary
```

<!-- Check to see if any NAs remain. The following should return an **empty**  -->

<!-- table. -->

<!-- ```{r} -->

<!-- gen_tib_sung %>%  -->

<!--     filter(is.na(dm_s1), -->

<!--            is.na(dm_s2),  -->

<!--            is.na(dm_a1), -->

<!--            is.na(dm_a2), -->

<!--            is.na(dm_t1), -->

<!--            is.na(dm_t2), -->

<!--            is.na(dm_b1), -->

<!--            is.na(dm_b2)) %>%  -->

<!--     select(id, piece_no, measure, starts_with("notes_"), starts_with("tones_"), -->

<!--            quarters_per_bar, starts_with("dm_"))  -->

<!-- ``` -->

## Tibble for Pitch: pitch\_tib

``` r
pitch_tib <- gen_tib_sung %>% 
    select(id, piece_no, measure, tones, dur_choir, dmc) %>%
    relocate(tones, .after = dmc) %>% 
    mutate(
        c = str_count(tones, "c$|c\\,"),
        c_sharp_d_flat = str_count(tones, "c-sharp|d-flat"),
        d = str_count(tones, "d$|d\\,"),
        d_sharp_e_flat = str_count(tones, "d-sharp|e-flat"),
        e = str_count(tones, "e$|e\\,"),
        f = str_count(tones, "f$|f\\,"),
        f_sharp_g_flat = str_count(tones, "f-sharp|g-flat"),
        g = str_count(tones, "g$|g\\,"),
        g_sharp_a_flat = str_count(tones, "g-sharp|a-flat"),
        a = str_count(tones, "a$|a\\,"),
        a_sharp_b_flat = str_count(tones, "a-sharp|b-flat"),
        b = str_count(tones, "b$|b\\,"),    
                ) %>% 
    rowwise %>% 
    mutate(tones_sum = sum(c_across(c:b))) 
```
