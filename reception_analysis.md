The Reception of The Measures Taken and The Mother in the
Political-Aesthetic Space of the Weimar Republic
================
Noah Zeldin
4/27/2021

  - [Introductory Remarks](#introductory-remarks)
  - [Load Packages](#load-packages)
  - [Importation](#importation)
  - [Quanteda Set-Up](#quanteda-set-up)
      - [Create Corpora](#create-corpora)
          - [General Corpora](#general-corpora)
          - [Piece Corpora](#piece-corpora)
          - [Title Corpora - NO MASSNAHME
            YET](#title-corpora---no-massnahme-yet)
      - [Corpus Summary - Article Lengths,
        etc.](#corpus-summary---article-lengths-etc.)
      - [Dictionaries and Additional
        Stopwords](#dictionaries-and-additional-stopwords)
      - [Tokenize and Filter Corpora](#tokenize-and-filter-corpora)
      - [DFMs](#dfms)
          - [Add GPO to general/ungrouped dfm
            -](#add-gpo-to-generalungrouped-dfm--)
          - [Create GPO grouped dfm’s for each
            piece](#create-gpo-grouped-dfms-for-each-piece)
          - [Less Specific Groupings](#less-specific-groupings)
          - [By GPO](#by-gpo)
          - [By Piece](#by-piece)
      - [Create corpus, dfm, ca, etc. of each piece w/o
        unknown](#create-corpus-dfm-ca-etc.-of-each-piece-wo-unknown)
  - [FactoMineR Set-Up](#factominer-set-up)
      - [Create function for converting dfm to dataframe and performing
        CA](#create-function-for-converting-dfm-to-dataframe-and-performing-ca)
      - [Use this function on each dfm](#use-this-function-on-each-dfm)
  - [KWIC and Keyword Exploration](#kwic-and-keyword-exploration)
      - [Create function to link KWIC with
        data](#create-function-to-link-kwic-with-data)
      - [Explore various KWIC](#explore-various-kwic)
          - [Special Case: LEHRLERN in
            Mother/Right](#special-case-lehrlern-in-motherright)
          - [Combinations](#combinations)
      - [Additional Terms Post-Processing
        (DFMs)](#additional-terms-post-processing-dfms)

# Introductory Remarks

Below is the annotated set-up for my quantitative analysis of the
Weimar-era reception of Brecht and Eisler’s *The Measures Taken* and
*The Mother*, which is included in the second chapter of my
dissertation. This analysis was conducted in R and relies heavily on the
[quanteda](https://quanteda.io/) package (for general manipulation, word
counts, etc.) and [FactoMineR](http://factominer.free.fr/index.html)
package (for correspondence analysis). (Also, I have tried to use
[tidyverse](https://www.tidyverse.org/) syntax as consistently as
possible.) Data sets will be made available to researchers upon request.

**NB: Further refinements to the coding are forthcoming.** This was my
first serious attempt at coding in R, so certain portions are still a
bit clunky. (Note too that many lines use the German titles for the two
pieces. All instances of this will be changed to English for
consistency.)

# Load Packages

``` r
library(tidyverse)
library(readxl)
library(quanteda)
library(quanteda.textmodels)
library(readtext)
library(tidytext)
library(ggplot2)
library(scales)
library(igraph)
library(ggraph)
library(FactoMineR)
library(RColorBrewer)
library(lubridate)
```

# Importation

NB: Several articles had to be removed because of their distortionary
effects. This resulted in multiple versions of the data, as shown below.
(This will be cleaned up in the near future.)

Main Spreadsheet

``` r
spreadsheet <- read_excel("all_documents_updated_9.10.xlsx", sheet = "All")
```

Spreadsheet w/o Massnahme Erfurt

``` r
spreadsheet_no_erfurt <- spreadsheet[-(c(38:45, 49)),]
```

Spreadshet w/or Massnahme Erfurt or “Kartenhaus”

``` r
spreadsheet_reduced <- spreadsheet[-(c(38:45, 49, 65)),]
```

# Quanteda Set-Up

## Create Corpora

### General Corpora

General Corpus

``` r
corp <- corpus(spreadsheet, text_field = "Text")
```

General Corpus w/o Massnahme Erfurt

``` r
corp_no_erfurt <- corpus(spreadsheet_no_erfurt, text_field = "Text")
```

General Corpus Reduced

``` r
corp_reduced <- corpus(spreadsheet_reduced, text_field = "Text")
```

### Piece Corpora

Massnahme Corpus

``` r
mass_corp <- corpus_subset(corp, Piece == "Massnahme")
```

Massnahme Corpus No Erfurt

``` r
mass_corp_no_erfurt <- corpus_subset(corp_no_erfurt, Piece == "Massnahme")
```

Mutter Corpus

``` r
mutter_corp <- corpus_subset(corp, Piece == "Mutter")
```

### Title Corpora - NO MASSNAHME YET

Title Corpus

``` r
corp_title <- corpus(spreadsheet, text_field = "Title")
```

Mutter Title Corpus

``` r
mutter_corp_title <- corpus_subset(corp_title, Piece == "Mutter")
```

## Corpus Summary - Article Lengths, etc.

``` r
corp_reduced_summary <- textstat_summary(corp_reduced) %>% 
  as_tibble

corp_reduced_summary$document <- corp_reduced_summary$document %>% 
        str_replace("text", "") 
    
corp_reduced_summary <- corp_reduced_summary %>% mutate(document = as.numeric(document)) %>% 
        rename(Article = document) %>% # this seems to work better than including in above step
        left_join(spreadsheet, by = "Article") %>% 
        select(-c(Text, Other_Metadata:Comp_Doc))
```

## Dictionaries and Additional Stopwords

General Dictionary

``` r
dict_gen <- dictionary(list(revolution = "revolution*",
                            bertolt = c("bert*", "bertolt*"),
                            theater = c("theater*", "theatralisch*"),
                            episch = "episch*",
                            musik = "musik*",
                            drama = "drama*",
                            politisch = "politi*",
                            kommunismus = "kommunis*",
                            chor = c("chor*", "chöre*"), 
                            proletarisch = "prolet*",
                            marxismus = "marx*",
                            primitiv = "primitiv*",
                            lehrstück = "lehrstück*",
                            brecht = "brecht*",
                            eisler = "eisler*",
                            gorki = "gorki*",
                            bürgerlich = "bürgerlich*",
                            propaganda = "propagand*",
                            kunst = c("kunst", "künstler*"),
                            lied = c("lied*", "song*"),
                            arbeitersänger = "arbeitersänger*", 
                            arbeiterchor = "arbeiterch*", 
                            bolschewismus = c("bolschewis*",
                                              "kulturbolschewis*"), 
                            langweilig = c("langweilig*", 
                                           "langeweile")))
```

Regex Dictionary

``` r
dict_regex <- dictionary(list(lehrlern = c("lehr(?!s)[a-z]+", 
                                           "belehr(?!s)[a-z]+", 
                                           "lern[a-z]+",
                                           "erlern[a-z]+", 
                                           "gelern[a-z]+", 
                                           "pädagog[a-z]+", 
                                           "didakt[a-z]+"))) 
```

Additional Stopwords

``` r
sw_add <- c("dass", "wurde", "schon", "mehr", "ganz*", "immer", "gibt", "ja",
            "müssen", "kommt", "sei", "tun")
```

## Tokenize and Filter Corpora

Create function for tokenizing and removing stopwords:

``` r
tokenize_and_remove_stopwords <- function(i) {
    tokens(i,
           remove_punct = TRUE,
           remove_numbers = TRUE,
           remove_symbols = TRUE) %>% 
        tokens_remove(c(stopwords("de"), sw_add)) %>% 
        tokens_remove(valuetype = "regex", "[0-9]+") %>% 
        tokens_keep(min_nchar = 3) 
}
```

Tokenize and filter corpora:

``` r
# general / ungrouped
gen_toks <- tokenize_and_remove_stopwords(corp)
# general / ungrouped NO ERFURT
gen_toks_no_erfurt <- tokenize_and_remove_stopwords(corp_no_erfurt)
# general / ungrouped REDUCED
gen_toks_reduced <- tokenize_and_remove_stopwords(corp_reduced)
# Massnahme
mass_toks <- tokenize_and_remove_stopwords(mass_corp)
# Massnahme NO ERFURT
mass_toks_no_erfurt <- tokenize_and_remove_stopwords(mass_corp_no_erfurt)
# Mutter
mutter_toks <- tokenize_and_remove_stopwords(mutter_corp)
# ADDITION 12.17.20 - Mutter TITLE
mutter_title_toks <- tokenize_and_remove_stopwords(mutter_corp_title)
```

## DFMs

Create function for converting toks to dfm and applying dictionaries:

``` r
convert_to_dfm_and_apply_dictionaries <- function(i) {
    dfm(i) %>% 
        dfm_lookup(dict_gen, exclusive = FALSE) %>% 
        dfm_lookup(dict_regex, valuetype = "regex", exclusive = FALSE)
}
```

Convert and apply dictionaries to corpora:

``` r
# general / ungrouped
gen_dfm <- convert_to_dfm_and_apply_dictionaries(gen_toks)
# general / ungrouped NO ERFURT
gen_dfm_no_erfurt <- convert_to_dfm_and_apply_dictionaries(gen_toks_no_erfurt)
# general / ungrouped REDUCD
gen_dfm_reduced <- convert_to_dfm_and_apply_dictionaries(gen_toks_reduced)
# Massnahme 
mass_dfm <- convert_to_dfm_and_apply_dictionaries(mass_toks)
# Massnahme NO ERFURT
mass_dfm_no_erfurt <- convert_to_dfm_and_apply_dictionaries(mass_toks_no_erfurt)
# Mutter
mutter_dfm <- convert_to_dfm_and_apply_dictionaries(mutter_toks)
# ADDITION 12.17.20: Mutter TITLE
mutter_title_dfm <- convert_to_dfm_and_apply_dictionaries(mutter_title_toks)
```

### Add GPO to general/ungrouped dfm -

<!-- PROB NEED TO CLEAN UP -->

NB: **GPO** stands for “genearl political orientation.”

``` r
# convert gen_dfm[_reduced] to dataframe
gen_datafr_reduced <- convert(gen_dfm_reduced, to = "data.frame")
# convert corp_reduced to dataframe - this has GPOs + other metadata
corp_reduced_datafr <- convert(corp_reduced, to = "data.frame")


# add GPO column to gen_datafr_reduced
gen_datafr_reduced_gpo <- left_join(gen_datafr_reduced, 
                                    corp_reduced_datafr[ , c("doc_id",    "Generalized_Political_Orientation")], 
                                    by = "doc_id", # 8.01 - now do this
                                    # by = c("document" = "doc_id"), # used to work
                                    copy = TRUE)


# eliminate doc col to perform CA (otherwise 2 quali.sup)
gen_datafr_reduced_gpo_2 <- gen_datafr_reduced_gpo[ , -1]


# ** group dfm's ####

# create grouped dfm: piece.gpo = 8 groups
grouped_dfm <- dfm_group(gen_dfm,
                         groups = c("Piece",
                                    "Generalized_Political_Orientation"))
# same but NO ERFURT
grouped_dfm_no_erfurt <- dfm_group(gen_dfm_no_erfurt,
                                   groups = c("Piece",
                                             "Generalized_Political_Orientation"))
```

### Create GPO grouped dfm’s for each piece

``` r
# Massnahme
mass_dfm_gpo <- dfm_group(mass_dfm,
                          groups = "Generalized_Political_Orientation")
# Massnahme NO ERFURT
mass_dfm_gpo_no_erfurt <- dfm_group(mass_dfm_no_erfurt,
                                    groups = "Generalized_Political_Orientation")
# Mutter
mutter_dfm_gpo <- dfm_group(mutter_dfm,
                            groups = "Generalized_Political_Orientation")
```

### Less Specific Groupings

``` r
# by piece
piece_dfm <- dfm_group(gen_dfm,
                       groups = "Piece")
# by GPO
gpo_dfm <- dfm_group(gen_dfm,
                     groups = "Generalized_Political_Orientation")

# less specific groupings NO ERFURT
# by piece
piece_dfm_no_erfurt <- dfm_group(gen_dfm_no_erfurt,
                                 groups = "Piece")
# by GPO
gpo_dfm_no_erfurt <- dfm_group(gen_dfm_no_erfurt,
                               groups = "Generalized_Political_Orientation")
```

### By GPO

``` r
# Left
left_sub <- dfm_subset(grouped_dfm, 
                       Generalized_Political_Orientation == "Left")
# Right
right_sub <- dfm_subset(grouped_dfm, 
                        Generalized_Political_Orientation == "Right")
# Center
center_sub <- dfm_subset(grouped_dfm, 
                         Generalized_Political_Orientation == "Center")
# Unknown
unknown_sub <- dfm_subset(grouped_dfm,
                          Generalized_Political_Orientation == "Unknown")
```

### By Piece

``` r
# Massnahme
massnahme_sub <- dfm_subset(grouped_dfm,
                            Piece == "Massnahme")
# Mutter
mutter_sub <- dfm_subset(grouped_dfm,
                         Piece == "Mutter")
```

## Create corpus, dfm, ca, etc. of each piece w/o unknown

<!-- from exp_7.24 -> need to integrate better, just being lazy -->

General - just GPO

``` r
corp_no_erfurt_or_unknown <- corpus_subset(corp_no_erfurt, 
                                           ! Generalized_Political_Orientation == "Unknown" )

toks_no_erfurt_or_unknown <- tokenize_and_remove_stopwords(corp_no_erfurt_or_unknown)

dfm_no_erfurt_or_unknown <- convert_to_dfm_and_apply_dictionaries(toks_no_erfurt_or_unknown)
```

Just GPO, not piece

``` r
gpo_dfm_no_erfurt_or_unknown <- dfm_group(dfm_no_erfurt_or_unknown,
                                          groups = "Generalized_Political_Orientation")
```

General - Piece + GPO

``` r
grouped_dfm_no_erfurt_or_unknown <- dfm_group(dfm_no_erfurt_or_unknown,
                                              groups = c("Piece", "Generalized_Political_Orientation"))
```

Measures Taken

``` r
mass_corp_no_erfurt_or_unknown <- corpus_subset(corp_no_erfurt, 
                                                Piece == "Massnahme" &! Generalized_Political_Orientation == "Unknown" )


mass_toks_no_erfurt_or_unknown <- tokenize_and_remove_stopwords(mass_corp_no_erfurt_or_unknown)

mass_dfm_no_erfurt_or_unknown <- convert_to_dfm_and_apply_dictionaries(mass_toks_no_erfurt_or_unknown)

mass_dfm_gpo_no_erfurt_or_unknown <- dfm_group(mass_dfm_no_erfurt_or_unknown,
                                               groups = "Generalized_Political_Orientation")
```

Mother

``` r
mutter_corp_no_unknown <- corpus_subset(corp, 
                                        Piece == "Mutter" &! Generalized_Political_Orientation == "Unknown" )


mutter_toks_no_unknown <- tokenize_and_remove_stopwords(mutter_corp_no_unknown)

mutter_dfm_no_unknown <- convert_to_dfm_and_apply_dictionaries(mutter_toks_no_unknown)

mutter_dfm_gpo_no_unknown <- dfm_group(mutter_dfm_no_unknown,
                                       groups = "Generalized_Political_Orientation")
```

# FactoMineR Set-Up

## Create function for converting dfm to dataframe and performing CA

``` r
convert_to_dataframe_and_perform_ca <- function(i) {
    convert(i, to = "data.frame") %>% 
        CA(quali.sup = 1, graph = FALSE)
}
```

## Use this function on each dfm

I think this is the only one I need

``` r
# piece + GPO = 8 groups NO ERFURT OR UNKNOWN
grouped_ca_no_erfurt_or_unknown <- convert_to_dataframe_and_perform_ca(grouped_dfm_no_erfurt_or_unknown)
```

NEW with English - 1.02.21

``` r
grouped_dfm_no_erfurt_or_unknown_english <- 
  dfm_group(grouped_dfm_no_erfurt_or_unknown,
            groups = c("Measures.Center", "Mother.Center", "Measures.Left",
                       "Mother.Left", "Measures.Right", "Mother.Right"))

grouped_ca_no_erfurt_or_unknown_english <- 
  convert_to_dataframe_and_perform_ca(grouped_dfm_no_erfurt_or_unknown_english)
```

# KWIC and Keyword Exploration

## Create function to link KWIC with data

``` r
combine_kwic_with_data <- function(corpus, words, window) {
    
    i <- kwic(corpus, words, window = window) %>% 
        as_tibble() 
    
    i$docname <- i$docname %>% 
        str_replace("text", "") 
    
    i <- i %>% mutate(docname = as.numeric(docname)) %>% 
        rename(Article = docname) %>% # this seems to work better than including in above step
        left_join(
          spreadsheet_reduced, # CHECKING 12.29.20 - seems to have worked better than normal spreadsheet with corp_reduced
          by = "Article") %>% 
        select(-c(Text, Other_Metadata:Comp_Doc))

    }
```

## Explore various KWIC

All of these keywords relate to claims made in ch. 2.

  - primitiv

<!-- end list -->

``` r
primitiv_kwic <- 
  combine_kwic_with_data(corp, "primitiv*", 15)
```

  - langweilig

<!-- end list -->

``` r
langweilig_kwic <- 
  combine_kwic_with_data(corp, c("langeweile", "langweilig*"), 10)
```

  - proletarisch

<!-- end list -->

``` r
proletarisch_kwic <- 
  combine_kwic_with_data(corp, "proletarisch*", 10)
```

  - pudowkin

<!-- end list -->

``` r
pudowkin_kwic <- 
  combine_kwic_with_data(corp, "pudowkin*", 10)
```

  - kunst

<!-- end list -->

``` r
kunst_kwic <- 
  combine_kwic_with_data(corp, c("kunst", "künstler*"), 10)
```

  - agitprop

<!-- end list -->

``` r
agitprop_kwic <- 
  combine_kwic_with_data(corp, "agitprop*", 10)
```

  - bildung

<!-- end list -->

``` r
bildung_kwic <- 
  combine_kwic_with_data(corp, "bildung", 10)
```

  - wissen

<!-- end list -->

``` r
wissen_kwic <- 
  combine_kwic_with_data(corp, "wissen*", 10)
```

  - lehrstück MUTTER

<!-- end list -->

``` r
lehrstueck_mutter_kwic <- 
  combine_kwic_with_data(corp, "lehrstück*", 10) %>% 
  filter(Piece == "Mutter")
```

  - Oratorium MASSNAHME

<!-- end list -->

``` r
oratorium_mass_kwic <- 
  combine_kwic_with_data(corp, "oratori*", 30) %>% 
  filter(Piece == "Massnahme")
```

  - Arbeitersänger MASSNAHME

<!-- end list -->

``` r
arbeitersaenger_mass_kwic <- 
  combine_kwic_with_data(corp, "arbeitersänger*", 30) %>% 
  filter(Piece == "Massnahme")
```

  - Stalin and Stalinismus

<!-- end list -->

``` r
combine_kwic_with_data(corp, "stalin*", 30) %>% 
  print()
```

    ## # A tibble: 3 x 16
    ##   Article  from    to pre   keyword post  pattern Title Newspaper Publisher
    ##     <dbl> <int> <int> <chr> <chr>   <chr> <fct>   <chr> <chr>     <chr>    
    ## 1      62    23    23 Gork~ Stalin  "- h~ stalin* "Die~ Germania  Zentrum  
    ## 2      66    55    55 Best~ Stalin  "und~ stalin* "Gor~ Sächsisc~ SPD      
    ## 3      83   662   662 und ~ Stalin~ "von~ stalin* "Ber~ Der_Aben~ SPD      
    ## # ... with 6 more variables: Political_Affiliation_or_Orientation <chr>,
    ## #   Generalized_Political_Orientation <chr>, Date <chr>, Author <chr>,
    ## #   Complete_or_Incomplete <chr>, Piece <chr>

Only 3 articles on Mutter.

  - Cantata

<!-- end list -->

``` r
combine_kwic_with_data(corp, "Kantate*", 25) %>% 
  print()
```

    ## # A tibble: 0 x 16
    ## # ... with 16 variables: Article <dbl>, from <int>, to <int>, pre <chr>,
    ## #   keyword <chr>, post <chr>, pattern <fct>, Title <chr>, Newspaper <chr>,
    ## #   Publisher <chr>, Political_Affiliation_or_Orientation <chr>,
    ## #   Generalized_Political_Orientation <chr>, Date <chr>, Author <chr>,
    ## #   Complete_or_Incomplete <chr>, Piece <chr>

No results.

### Special Case: LEHRLERN in Mother/Right

NB: Can’t use FN, b/c can’t specify valuetype Here, adaptation of FN

1.  create tibble

<!-- end list -->

``` r
lehrlern_kwic <- kwic(corp, dict_regex, window = 20, 
                      valuetype = "regex") %>% # must specify to used dict
  as_tibble()

lehrlern_kwic$docname <- lehrlern_kwic$docname %>% 
    str_replace("text", "") 

lehrlern_kwic <- lehrlern_kwic %>% mutate(docname = as.numeric(docname)) %>% 
    rename(Article = docname) %>% # this seems to work better than including in above step
    left_join(spreadsheet, by = "Article") %>% 
    select(-c(Text, Other_Metadata:Comp_Doc))
```

2.  filter for Mutter & Right

<!-- end list -->

``` r
lehrlern_kwic_mutter_right <- lehrlern_kwic %>% 
  filter(Piece == "Mutter" & 
               Generalized_Political_Orientation == "Right")
```

### Combinations

langweilig + primitiv

``` r
langweilig_primitiv_kwic <- 
  inner_join(langweilig_kwic, primitiv_kwic, by = "Article")
```

## Additional Terms Post-Processing (DFMs)

LEHRLERN in Mutter

``` r
lehrlern_mutter_gpo_count <-  
  dfm_select(mutter_dfm_gpo, pattern = "LEHRLERN") %>% 
  as_tibble()

lehrlern_mutter_gpo_count
```

    ## # A tibble: 4 x 2
    ##   doc_id  LEHRLERN
    ##   <chr>      <dbl>
    ## 1 Center        29
    ## 2 Left          18
    ## 3 Right         15
    ## 4 Unknown       15