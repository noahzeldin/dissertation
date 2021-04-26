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

# Stats

## Durations - General

### Total Duration in Minutes

``` r
dur_total_min <- sum(dur_tib$duration) / 60
```

Total duration in minutes: 38.58.

### Duration by Piece

``` r
dur_piece <- dur_tib %>% 
    group_by(piece_no) %>%
    summarize(duration_min = (sum(duration) / 60)) %>% 
    mutate(prop_of_dur = (duration_min/sum(duration_min)),
           duration_min = round(duration_min, digits = 2),
           prop_of_dur = round(prop_of_dur, digits = 3))
```

Output of `dur_piece`:

| piece\_no | duration\_min | prop\_of\_dur |
| :-------- | ------------: | ------------: |
| 1         |          4.91 |         0.127 |
| 2b        |          2.07 |         0.054 |
| 3a        |          3.83 |         0.099 |
| 3b        |          0.49 |         0.013 |
| 4         |          4.12 |         0.107 |
| 5         |          3.92 |         0.102 |
| 6a        |          0.06 |         0.002 |
| 6b        |          0.13 |         0.003 |
| 6c        |          0.24 |         0.006 |
| 7a        |          1.62 |         0.042 |
| 7b        |          0.04 |         0.001 |
| 8a        |          2.32 |         0.060 |
| 8b        |          3.12 |         0.081 |
| 9         |          4.82 |         0.125 |
| 10        |          1.96 |         0.051 |
| 11        |          0.60 |         0.016 |
| 12a       |          0.47 |         0.012 |
| 12b       |          0.90 |         0.023 |
| 13a       |          0.10 |         0.003 |
| 13b       |          0.47 |         0.012 |
| 14        |          2.37 |         0.061 |

Resort to find longest pieces.

``` r
dur_piece %>% 
    arrange(desc(duration_min))
```

    ## # A tibble: 21 x 3
    ##    piece_no duration_min prop_of_dur
    ##    <fct>           <dbl>       <dbl>
    ##  1 1                4.91       0.127
    ##  2 9                4.82       0.125
    ##  3 4                4.12       0.107
    ##  4 5                3.92       0.102
    ##  5 3a               3.83       0.099
    ##  6 8b               3.12       0.081
    ##  7 14               2.37       0.061
    ##  8 8a               2.32       0.06 
    ##  9 2b               2.07       0.054
    ## 10 10               1.96       0.051
    ## # ... with 11 more rows

<!-- ##### NEW with Lubridate - FIX! -->

<!-- ```{r} -->

<!-- dur_piece_lubridate <- dur_piece %>%  -->

<!--     separate(duration_min, sep = "\\.", into = c("min", "sec")) %>%  -->

<!--     mutate(min = as.numeric(min), -->

<!--            sec = as.numeric(sec) # PROBLEM - removes zeros -->

<!--            ) %>%  -->

<!--     mutate(sec = round(sec * .6)) %>%  -->

<!--     mutate(duration = (minutes(min) + seconds(sec)), .keep = "unused") %>%  -->

<!--     relocate(duration, .after = piece_no) -->

<!-- dur_piece_lubridate -->

<!-- ``` -->

### Duration by Category

``` r
dur_tib %>% 
    group_by(category) %>%
    summarize(duration = sum(duration)) %>% 
    mutate(prop_of_dur = (duration/sum(duration)))
```

    ## # A tibble: 4 x 3
    ##   category duration prop_of_dur
    ##   <fct>       <dbl>       <dbl>
    ## 1 1          1403.       0.606 
    ## 2 2            28.2      0.0122
    ## 3 3           791.       0.342 
    ## 4 4            92.0      0.0398

<!-- No contest: two biggest categories are 1. choir with orchestra and 3. tenor solo -->

<!-- close to 95 % -->

### Duration by Subcategory

``` r
dur_subcategory <- dur_tib %>% 
    group_by(subcategory) %>%
    summarize(duration = sum(duration)) %>% 
    mutate(duration_min = (duration/60)) %>% 
    mutate(prop_of_dur = (duration/sum(duration)))
```

Output of `dur_subcategory`:

| subcategory |   duration | duration\_min | prop\_of\_dur |
| :---------- | ---------: | ------------: | ------------: |
| 1a          | 1305.46453 |    21.7577421 |     0.5640358 |
| 1b          |   97.50000 |     1.6250000 |     0.0421256 |
| 2b          |   28.18182 |     0.4696970 |     0.0121762 |
| 3a          |  230.00000 |     3.8333333 |     0.0993732 |
| 3b          |  235.23697 |     3.9206162 |     0.1016359 |
| 3c          |  326.07852 |     5.4346419 |     0.1408847 |
| 4a          |   37.22431 |     0.6204052 |     0.0160830 |
| 4b          |    3.60000 |     0.0600000 |     0.0015554 |
| 4c          |   14.54545 |     0.2424242 |     0.0062845 |
| 4d          |   36.67471 |     0.6112452 |     0.0158456 |

<!-- arranged descending -->

<!-- ```{r} -->

<!-- dur_subcategory %>%  -->

<!--     arrange(desc(prop_of_dur)) -->

<!-- ``` -->

<!-- Exactly what I expected:    -->

<!-- * largest proportion = 1a. mixed choir with orchestra -->

<!-- * followed by 3. tenor solo with accompaniment -->

<!-- * then 1b. = male choir with orchestra -->

<!-- ** actually, only one piece = Streiklied -->

## Meter and Tempo

### Rates of Change

Construct table with following stats for each piece:

  - duration

  - # of meter changes

  - rate of meter changes in seconds = duration / \# of meter changes

  - rate of meter changes in bars

  - # of tempo changes

  - rate of tempo changes in seconds = duration / \# of meter changes

  - rate of tempo changes in bars

<!-- end list -->

``` r
dur_meter_tempo_piece <- dur_tib %>% 
    group_by(piece_no) %>% 
    summarize(meter_ch_count = sum(meter_ch_count),
              tempo_ch_count = sum(tempo_ch_count),
              duration = sum(duration)) %>% 
    mutate(meter_ch_rate_sec = round(duration / meter_ch_count, 
                                     digits = 2),
           tempo_ch_rate_sec = round(duration / tempo_ch_count, 
                                     digits = 2),
           duration = round(duration, digits = 2)) %>% 
    relocate(meter_ch_rate_sec, .after = meter_ch_count) %>% 
    relocate(tempo_ch_rate_sec, .after = tempo_ch_count)

dur_meter_tempo_piece
```

    ## # A tibble: 21 x 6
    ##    piece_no meter_ch_count meter_ch_rate_s~ tempo_ch_count tempo_ch_rate_s~
    ##    <fct>             <dbl>            <dbl>          <dbl>            <dbl>
    ##  1 1                     9            32.8               1           295.  
    ##  2 2b                    9            13.8               1           124.  
    ##  3 3a                    1           230                 4            57.5 
    ##  4 3b                   12             2.47              1            29.6 
    ##  5 4                    18            13.7               1           247.  
    ##  6 5                     7            33.6               9            26.1 
    ##  7 6a                    1             3.6               1             3.6 
    ##  8 6b                    4             1.9               1             7.62
    ##  9 6c                    1            14.6               1            14.6 
    ## 10 7a                    3            32.5               1            97.5 
    ## # ... with 11 more rows, and 1 more variable: duration <dbl>

Output of `dur_meter_tempo_piece`:

| piece\_no | meter\_ch\_count | meter\_ch\_rate\_sec | tempo\_ch\_count | tempo\_ch\_rate\_sec | duration |
| :-------- | ---------------: | -------------------: | ---------------: | -------------------: | -------: |
| 1         |                9 |                32.76 |                1 |               294.86 |   294.86 |
| 2b        |                9 |                13.82 |                1 |               124.35 |   124.35 |
| 3a        |                1 |               230.00 |                4 |                57.50 |   230.00 |
| 3b        |               12 |                 2.47 |                1 |                29.61 |    29.61 |
| 4         |               18 |                13.73 |                1 |               247.22 |   247.22 |
| 5         |                7 |                33.61 |                9 |                26.14 |   235.24 |
| 6a        |                1 |                 3.60 |                1 |                 3.60 |     3.60 |
| 6b        |                4 |                 1.90 |                1 |                 7.62 |     7.62 |
| 6c        |                1 |                14.55 |                1 |                14.55 |    14.55 |
| 7a        |                3 |                32.50 |                1 |                97.50 |    97.50 |
| 7b        |                1 |                 2.40 |                1 |                 2.40 |     2.40 |
| 8a        |                1 |               138.95 |                1 |               138.95 |   138.95 |
| 8b        |                1 |               187.13 |                7 |                26.73 |   187.13 |
| 9         |                8 |                36.14 |                1 |               289.09 |   289.09 |
| 10        |                5 |                23.48 |                1 |               117.38 |   117.38 |
| 11        |                2 |                18.00 |                1 |                36.00 |    36.00 |
| 12a       |                5 |                 5.64 |                1 |                28.18 |    28.18 |
| 12b       |                1 |                54.29 |                1 |                54.29 |    54.29 |
| 13a       |                1 |                 5.85 |                1 |                 5.85 |     5.85 |
| 13b       |                2 |                14.21 |                1 |                28.42 |    28.42 |
| 14        |                5 |                28.46 |                1 |               142.29 |   142.29 |

<!-- sorted by **meter_ch_count**   -->

<!-- ```{r} -->

<!-- dur_meter_tempo_piece %>%  -->

<!--     arrange(desc(meter_ch_count)) %>%  -->

<!--     select(-tempo_ch_count, -tempo_ch_rate_sec) -->

<!-- ``` -->

5 pieces with greatest number of meter changes.

``` r
meter_ch_count_piece_top_5 <- dur_meter_tempo_piece %>% 
    arrange(desc(meter_ch_count)) %>% 
    select(-tempo_ch_count, -tempo_ch_rate_sec) %>% 
    slice(1:5)

knitr::kable(meter_ch_count_piece_top_5)
```

| piece\_no | meter\_ch\_count | meter\_ch\_rate\_sec | duration |
| :-------- | ---------------: | -------------------: | -------: |
| 4         |               18 |                13.73 |   247.22 |
| 3b        |               12 |                 2.47 |    29.61 |
| 1         |                9 |                32.76 |   294.86 |
| 2b        |                9 |                13.82 |   124.35 |
| 9         |                8 |                36.14 |   289.09 |

<!-- sorted by **meter_ch_rate_sec** ASCENDING   -->

<!-- ```{r} -->

<!-- dur_meter_tempo_piece %>%  -->

<!--     filter(meter_ch_count != 1) %>%  -->

<!--     select(-tempo_ch_count, -tempo_ch_rate_sec) %>%  -->

<!--     arrange(meter_ch_rate_sec) -->

<!-- ``` -->

5 pieces with quickest rates of meter changes.

``` r
meter_ch_rate_piece_top_5 <- dur_meter_tempo_piece %>% 
    filter(meter_ch_count != 1) %>% 
    select(-tempo_ch_count, -tempo_ch_rate_sec) %>% 
    arrange(meter_ch_rate_sec) %>% 
    slice(1:5)

knitr::kable(meter_ch_rate_piece_top_5)
```

| piece\_no | meter\_ch\_count | meter\_ch\_rate\_sec | duration |
| :-------- | ---------------: | -------------------: | -------: |
| 6b        |                4 |                 1.90 |     7.62 |
| 3b        |               12 |                 2.47 |    29.61 |
| 12a       |                5 |                 5.64 |    28.18 |
| 4         |               18 |                13.73 |   247.22 |
| 2b        |                9 |                13.82 |   124.35 |

<!-- sorted by **tempo_ch_count**   -->

<!-- ```{r} -->

<!-- dur_meter_tempo_piece %>%   -->

<!--     select(-meter_ch_count, -meter_ch_rate_sec) %>%  -->

<!--     arrange(desc(tempo_ch_count)) -->

<!-- ``` -->

3 pieces with greatest number of tempo changes.

``` r
tempo_ch_count_and_rate_piece <- dur_meter_tempo_piece %>% 
    filter(tempo_ch_count != 1) %>% 
    select(-meter_ch_count, -meter_ch_rate_sec) %>% 
    arrange(tempo_ch_rate_sec)

knitr::kable(tempo_ch_count_and_rate_piece)
```

| piece\_no | tempo\_ch\_count | tempo\_ch\_rate\_sec | duration |
| :-------- | ---------------: | -------------------: | -------: |
| 5         |                9 |                26.14 |   235.24 |
| 8b        |                7 |                26.73 |   187.13 |
| 3a        |                4 |                57.50 |   230.00 |

### Most common meters

Create table with meter, non\_of\_mm and duration.

``` r
dur_meter <- dur_tib %>% 
    unite(meter_1, meter_2, col = "meter", sep = "_") %>% 
    select(piece_no, no_of_mm, meter, duration) %>% 
    relocate(meter, .after = piece_no) %>% 
    filter(meter != "0_0") # remove the pick-up from piece_no 10
```

Most common meters by no\_of\_mm.

``` r
dur_meter %>% 
    group_by(meter) %>% 
    count(no_of_mm) %>% 
    mutate(no_of_mm = no_of_mm * n) %>% 
    select(-n) %>% 
    summarize(no_of_mm = sum(no_of_mm)) %>% 
    arrange(desc(no_of_mm))
```

    ## # A tibble: 8 x 2
    ##   meter no_of_mm
    ##   <chr>    <dbl>
    ## 1 2_4        568
    ## 2 3_4        235
    ## 3 3_2        198
    ## 4 2_2        187
    ## 5 6_4         45
    ## 6 4_2         40
    ## 7 4_4         20
    ## 8 1_4          5

Most common meters by duration.

``` r
dur_meter %>% 
    group_by(meter) %>% 
    summarize(duration = sum(duration)) %>% 
    arrange(desc(duration))
```

    ## # A tibble: 8 x 2
    ##   meter duration
    ##   <chr>    <dbl>
    ## 1 2_4     635.  
    ## 2 3_2     624.  
    ## 3 3_4     467.  
    ## 4 2_2     228.  
    ## 5 4_2     203.  
    ## 6 6_4     112.  
    ## 7 4_4      41.9 
    ## 8 1_4       3.17

#### Grouped by duple vs. triple

``` r
dur_meter_groups <- dur_meter %>% 
    group_by(meter) %>% 
    summarize(duration = sum(duration)) %>% 
    separate(meter, sep = "_", into = c("meter_1", "meter_2")) %>% 
    mutate(meter_1 = as.factor(meter_1))

dur_meter_groups$meter_1 <- dur_meter_groups$meter_1 %>% 
    fct_relabel(~ c("other", "duple", "triple", "duple", "duple"))

dur_meter_groups <- dur_meter_groups %>% 
    select(-meter_2) %>% 
    rename(group = meter_1) %>% 
    group_by(group) %>% 
    summarize(duration = sum(duration)) %>% 
    mutate(prop_dur = duration / sum(duration),
           perc_dur = round(prop_dur*100))

dur_meter_groups
```

    ## # A tibble: 3 x 4
    ##   group  duration prop_dur perc_dur
    ##   <fct>     <dbl>    <dbl>    <dbl>
    ## 1 other      3.17  0.00137        0
    ## 2 duple   1220.    0.527         53
    ## 3 triple  1091.    0.471         47

Interesting: pretty evenly split.

##### 6/4

However, need to account for 6/4, which is compound duple.

Check which mm. are in 6/4.

``` r
gen_tib %>% 
  filter(meter_1 == 6, meter_2 == 4) 
```

    ## # A tibble: 46 x 83
    ##       id piece_no measure texture texture_value voices groupings soprano  alto
    ##    <int> <fct>      <dbl> <chr>           <dbl>  <dbl> <chr>       <dbl> <dbl>
    ##  1   196 4              3 m                 1        1 tb              0     0
    ##  2   206 4             13 p                 2        2 none            0     0
    ##  3   207 4             14 p                 2        2 none            0     0
    ##  4   209 4             16 h                 1.5      2 tb              0     0
    ##  5   221 4             28 p                 2        1 sa_tb           1     1
    ##  6   222 4             29 p                 2        2 sa_tb           1     1
    ##  7   223 4             30 p                 2        2 sa_tb           1     1
    ##  8   224 4             31 p                 2        2 sa_tb           1     1
    ##  9   225 4             32 p                 2        1 sa_tb           0     0
    ## 10   226 4             33 na                0        0 na              0     0
    ## # ... with 36 more rows, and 74 more variables: tenor <dbl>, bass <dbl>,
    ## #   parts_active <dbl>, ratio_voices_parts <dbl>, rests_s1 <dbl>,
    ## #   rests_s2 <dbl>, rests_a1 <dbl>, rests_a2 <dbl>, rests_t1 <dbl>,
    ## #   rests_t2 <dbl>, rests_b1 <dbl>, rests_b2 <dbl>, rests_all <dbl>,
    ## #   quarters_s1 <dbl>, quarters_s2 <dbl>, quarters_a1 <dbl>, quarters_a2 <dbl>,
    ## #   quarters_t1 <dbl>, quarters_t2 <dbl>, quarters_b1 <dbl>, quarters_b2 <dbl>,
    ## #   quarters_sum <dbl>, notes_s1 <dbl>, notes_s2 <dbl>, notes_a1 <dbl>,
    ## #   notes_a2 <dbl>, notes_t1 <dbl>, notes_t2 <dbl>, notes_b1 <dbl>,
    ## #   notes_b2 <dbl>, notes_sum <dbl>, notes_sum_pairings <dbl>, tones_s1 <dbl>,
    ## #   tones_s2 <dbl>, tones_a1 <dbl>, tones_a2 <dbl>, tones_t1 <dbl>,
    ## #   tones_t2 <dbl>, tones_b1 <dbl>, tones_b2 <dbl>, tones_sum <dbl>,
    ## #   tones <chr>, tones_count <dbl>, spoken <dbl>, acapella <dbl>,
    ## #   meter_1 <dbl>, meter_2 <dbl>, quarters_per_bar <dbl>, tempo <dbl>,
    ## #   duration <dbl>, dur_choir <dbl>, dur_spoken <dbl>, dur_acapella <dbl>,
    ## #   dur_s1 <dbl>, dur_s2 <dbl>, dur_a1 <dbl>, dur_a2 <dbl>, dur_t1 <dbl>,
    ## #   dur_t2 <dbl>, dur_b1 <dbl>, dur_b2 <dbl>, `meter 1` <dbl>, `meter 2` <dbl>,
    ## #   `quarters per bar` <dbl>, dm_s1 <dbl>, dm_s2 <dbl>, dm_a1 <dbl>,
    ## #   dm_a2 <dbl>, dm_t1 <dbl>, dm_t2 <dbl>, dm_b1 <dbl>, dm_b2 <dbl>,
    ## #   dm_sum <dbl>, dmc <dbl>

Compute percent of duration that is 6/4.

``` r
dur_6_4_perc <- round((dur_meter %>% 
    filter(meter == "6_4") %>% 
    select(duration) %>% 
    sum() / 
  dur_meter_groups %>% 
    select(duration) %>% 
    sum()) * 100, digits = 1)
```

percent of duration of duple that is 6/4.

``` r
dur_6_4_duple_perc <- round((dur_meter %>% 
    filter(meter == "6_4") %>% 
    select(duration) %>% 
    sum() / 
  dur_meter_groups %>% 
    filter(group == "duple") %>% 
    select(duration) %>% 
    sum()) * 100, digits = 1)
```

``` r
# duration of duple meters
dur_meter_groups %>% 
  filter(group == "duple") %>% 
  select(duration) -
  # subtract duration of 6/4 measures
  dur_meter %>% 
    filter(meter == "6_4") %>% 
    select(duration) %>% 
    sum() - 
  # subtract duration of triple meters
  dur_meter_groups %>% 
  filter(group == "triple") %>% 
  select(duration)
```

    ##   duration
    ## 1  16.6928

Almost even.
