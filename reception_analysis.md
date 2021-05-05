The Reception of *The Measures Taken* and *The Mother* in the
Political-Aesthetic Space of the Weimar Republic
================
Noah Zeldin
4/27/2021

  - [Introductory Remarks](#introductory-remarks)
  - [Load Packages](#load-packages)
  - [Importation](#importation)
  - [Quanteda Set-Up](#quanteda-set-up)
      - [Create Corpora from Data Sets](#create-corpora-from-data-sets)
          - [General Corpora](#general-corpora)
          - [Corpora Grouped by Piece](#corpora-grouped-by-piece)
          - [Corpora of Title Texts](#corpora-of-title-texts)
      - [Corpus Summary - Article Lengths,
        etc.](#corpus-summary---article-lengths-etc.)
      - [Dictionaries and Additional
        Stopwords](#dictionaries-and-additional-stopwords)
      - [Tokenize and Filter Corpora](#tokenize-and-filter-corpora)
      - [Document-Feature Matrices
        (dfm)](#document-feature-matrices-dfm)
          - [Group dfm’s by General Political Orientation
            (GPO)](#group-dfms-by-general-political-orientation-gpo)
          - [Create GPO grouped dfm’s for each
            piece](#create-gpo-grouped-dfms-for-each-piece)
          - [Less Specific Groupings](#less-specific-groupings)
          - [By GPO](#by-gpo)
          - [By Piece](#by-piece)
      - [Convert general (reduced) corpus and dfm to dataframes for
        later
        use](#convert-general-reduced-corpus-and-dfm-to-dataframes-for-later-use)
      - [Create corpus, dfm, ca, etc. of each piece w/o unknown GPO (for
        CA)](#create-corpus-dfm-ca-etc.-of-each-piece-wo-unknown-gpo-for-ca)
  - [FactoMineR Set-Up (for CA)](#factominer-set-up-for-ca)
  - [Keywords In Context (KWIC) and Keyword
    Exploration](#keywords-in-context-kwic-and-keyword-exploration)
      - [Create function to link KWIC with
        data](#create-function-to-link-kwic-with-data)
      - [Explore various KWIC](#explore-various-kwic)
          - [Special Case: LEHRLERN in rightwing articles on The
            Mother](#special-case-lehrlern-in-rightwing-articles-on-the-mother)
      - [Additional Terms Post-Processing
        (DFMs)](#additional-terms-post-processing-dfms)
  - [Dates of Publication with
    Counts](#dates-of-publication-with-counts)
      - [Create Tables](#create-tables)
          - [General](#general)
          - [By Piece](#by-piece-1)
      - [Date Ranges, Time Lengths,
        etc.](#date-ranges-time-lengths-etc.)
          - [Time Length in Years](#time-length-in-years)
          - [Range / Interval](#range-interval)
          - [Proportion of Articles Published near
            Premieres](#proportion-of-articles-published-near-premieres)
      - [Articles on The Measures Taken published in 1932 (with full
        dates)](#articles-on-the-measures-taken-published-in-1932-with-full-dates)
  - [Visualizations](#visualizations)
      - [Visual Summary of Reduced
        Corpus](#visual-summary-of-reduced-corpus)
          - [Very Long Articles](#very-long-articles)
      - [Wordclouds](#wordclouds)
      - [Word Frequency](#word-frequency)
      - [Correspondence Analysis](#correspondence-analysis)

# Introductory Remarks

Below is the annotated set-up for my quantitative analysis of the
Weimar-era reception of Brecht and Eisler’s *The Measures Taken* and
*The Mother*, which is included in the second chapter of my
dissertation. This analysis was conducted in R and relies heavily on the
[quanteda](https://quanteda.io/) package (for general manipulation, word
counts, etc.) and [FactoMineR](http://factominer.free.fr/index.html)
package (for correspondence analysis). (Also, I have tried to use
[tidyverse](https://www.tidyverse.org/) syntax as consistently as
possible.) The data set will be made available to researchers upon
request.

The **main goals** of this analysis are:

1.  to determine the most-discussed aspects of each work (e.g. musical
    vs.  theatrical components, politics)
    
      - In the dissertation, I compare these results to the works’
        postwar reception.

2.  to determine if there is a measurable correspondence between
    aesthetics and politics
    
      - Hence the notion of the “political-aesthetic space.”

**NB: Further refinements to the code are forthcoming.** The final
version will be included in the supplementary materials of the
dissertation.

In the near future, I will upload the corresponding write-up included in
ch. 2 of my dissertation to this repository.

# Load Packages

``` r
library(tidyverse)
library(readxl)
library(ggplot2)
library(quanteda)
library(FactoMineR)
library(lubridate)
```

# Importation

NB: Several articles had to be removed because of their distortionary
effects. This resulted in three versions of the data, as shown below.

Main data set containing all articles (`data_main`):

``` r
data_main <- read_excel("reception_analysis_data.xlsx", sheet = "all_docs_lowercase")
```

`data_main` w/o articles on the Erfurt performance *The Measures Taken*
(`data_no_erfurt`):

``` r
data_no_erfurt <- data_main %>% 
    filter(date != "24.01.1933") %>% 
    filter(date != "25.01.1933") %>% 
    filter(date != "26.01.1933") %>% 
    filter(date != "27.01.1933") %>% 
    filter(newspaper != "Internationaler_Revolutionärer_Theater-Bund_(Bulletin)")
```

  - 9 articles removed from original data set

(`data_main`).

`data_main` without *Measures Taken* articles on Erfurt performance or
Ferdinand Junghans, “Das Kartenhaus stürzt\!,” *Neue Preussische
Kreuz-Zeitung*, n.d. (`data_reduced`):

``` r
data_reduced <- data_no_erfurt %>% 
    filter(author != "Ferdinand Junghans")
```

  - 10 articles removed from original data set (`data_main`).

# Quanteda Set-Up

## Create Corpora from Data Sets

In addition to the main corpora, I create additional corpora, e.g. based
on piece, to provide greater flexibility for later analysis, for
instance, finding and/or counting instances of a keyword in articles on
*The Mother*.

### General Corpora

General Corpus from `data_main` (`corp`):

``` r
corp <- corpus(data_main, text_field = "text")
```

Corpus from `data_no_erfurt` (`corp_no_erfurt`):

``` r
corp_no_erfurt <- corpus(data_no_erfurt, text_field = "text")
```

Corpus from `data_reduced` (`corp_reduced` ):

``` r
corp_reduced <- corpus(data_reduced, text_field = "text")
```

### Corpora Grouped by Piece

*Measures Taken* Corpus (`mt_corp`):

``` r
mt_corp <- corpus_subset(corp, piece == "measures")
```

*Measures Taken* Corpus without articles on Erfurt performance
(`mt_corp_no_erfurt`):

``` r
mt_corp_no_erfurt <- corpus_subset(corp_no_erfurt, piece == "measures")
```

  - Again, the articles on the Erfurt performance were removed because
    of their distortionary effect.

*Mother* Corpus (`mother_corp`):

``` r
mother_corp <- corpus_subset(corp, piece == "mother")
```

### Corpora of Title Texts

This allows for later analysis of the texts of the article titles,
e.g. keyword frequencies.

Title Corpus (all articles) (`corp_title`):

``` r
corp_title <- corpus(data_main, text_field = "title")
```

*Mother* Title Corpus (`mother_corp_title`):

``` r
mother_corp_title <- corpus_subset(corp_title, piece == "mother")
```

## Corpus Summary - Article Lengths, etc.

``` r
corp_reduced_summary <- textstat_summary(corp_reduced) %>% 
  as_tibble

corp_reduced_summary$document <- corp_reduced_summary$document %>% 
        str_replace("text", "") 
    
corp_reduced_summary <- corp_reduced_summary %>% 
  mutate(document = as.numeric(document)) %>% 
  rename(article = document) %>% 
  left_join(data_main, by = "article") %>% 
  select(-c(text, other_metadata:Comp_Doc))
```

## Dictionaries and Additional Stopwords

General Dictionary

  - This dictionary contains what I perceived to be the most important
    keywords in the corpus and those on which I wished to focus in my
    analysis. As I explain in ch. 2, I read through all of the articles
    in the corpus prior to conducting this analysis. As a result, I had
    a clear idea of those terms on which I would focus.

<!-- end list -->

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

Regular Expressions Dictionary

  - In the case of pedagogical terms, it was necessary to use regular
    expressions. As I explain in ch. 2, **LEHRLERN** is the only keyword
    defined in these dictionaries that can be considered an
    interpretative combination of different words, rather than a mere
    grouping of synonyms. **LEHRLERN** groups together words relating to
    the following terms (the regular expressions should cover all forms
    of the terms: nouns, verbs, adjectives and adverbs).
    
      - “teaching” \[*lehren*\]
    
      - “learning” \[*lernen* (and past particple, *gelernt*) and
        *erlernen*\]
    
      - “instructing” \[*belehren*\]
    
      - “pedagogical” \[*pädagogisch*\]
    
      - “didactic” \[*didaktisch*\]

<!-- end list -->

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

  - These stopwords are not part of the set of German stopwords provided
    in **quanteda**.

<!-- end list -->

``` r
sw_add <- c("dass", "wurde", "schon", "mehr", "ganz*", "immer", "gibt", "ja",
            "müssen", "kommt", "sei", "tun")
```

## Tokenize and Filter Corpora

Create function for tokenizing and removing stopwords, punctuation and
other symbols:

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

Tokenize and filter corpora (results in tokens object, labeled `_toks`):

``` r
# general / ungrouped
gen_toks <- tokenize_and_remove_stopwords(corp)

# general / ungrouped NO ERFURT
gen_toks_no_erfurt <- tokenize_and_remove_stopwords(corp_no_erfurt)

# general / ungrouped REDUCED
gen_toks_reduced <- tokenize_and_remove_stopwords(corp_reduced)

# Measures Taken
mt_toks <- tokenize_and_remove_stopwords(mt_corp)

# Measures Taken NO ERFURT
mt_toks_no_erfurt <- tokenize_and_remove_stopwords(mt_corp_no_erfurt)

# Mother
mother_toks <- tokenize_and_remove_stopwords(mother_corp)

# Mother TITLE
mother_title_toks <- tokenize_and_remove_stopwords(mother_corp_title)
```

## Document-Feature Matrices (dfm)

Create function for converting tokenized items from previous section
(ending in `_toks`) to dfm and applying dictionaries:

``` r
convert_to_dfm_and_apply_dictionaries <- function(i) {
    dfm(i) %>% 
        dfm_lookup(dict_gen, exclusive = FALSE) %>% 
        dfm_lookup(dict_regex, valuetype = "regex", exclusive = FALSE)
}
```

Convert and apply dictionaries to tokens objects, using above-defined
function:

``` r
# general / ungrouped
gen_dfm <- convert_to_dfm_and_apply_dictionaries(gen_toks)
# general / ungrouped NO ERFURT
gen_dfm_no_erfurt <- convert_to_dfm_and_apply_dictionaries(gen_toks_no_erfurt)
# general / ungrouped REDUCD
gen_dfm_reduced <- convert_to_dfm_and_apply_dictionaries(gen_toks_reduced)
# Measures Taken 
mt_dfm <- convert_to_dfm_and_apply_dictionaries(mt_toks)
# Measures Taken NO ERFURT
mt_dfm_no_erfurt <- convert_to_dfm_and_apply_dictionaries(mt_toks_no_erfurt)
# Mother
mother_dfm <- convert_to_dfm_and_apply_dictionaries(mother_toks)
# ADDITION 12.17.20: Mother TITLE
mother_title_dfm <- convert_to_dfm_and_apply_dictionaries(mother_title_toks)
```

### Group dfm’s by General Political Orientation (GPO)

**GPO** or “general political orientation” is a variable included in the
original data set and is used extensively in the later analyses. It must
be added back int dfm’s, using the `groups` function in **quanteda**.

Here, we group the data by both piece and GPO, which results in 8 groups
total.

``` r
grouped_dfm <- dfm_group(gen_dfm,
                         groups = c("piece",
                                    "general_political_orientation"))

# same but NO ERFURT
grouped_dfm_no_erfurt <- dfm_group(gen_dfm_no_erfurt,
                                   groups = c("piece",
                                             "general_political_orientation"))
```

### Create GPO grouped dfm’s for each piece

NB: Only the third of these (`mother_dfm_gpo`) is used in the analysis
but all three have been kept for consistency.

``` r
# Measures Taken
mt_dfm_gpo <- dfm_group(mt_dfm,
                          groups = "general_political_orientation")

# Measures Taken NO ERFURT
mt_dfm_gpo_no_erfurt <- dfm_group(mt_dfm_no_erfurt,
                                    groups = "general_political_orientation")

# Mother
mother_dfm_gpo <- dfm_group(mother_dfm,
                            groups = "general_political_orientation")
```

### Less Specific Groupings

NB: The first two, which contain articles related to the Erfurt
performance of *The Measures Taken*, have been created for the sake of
consistency but are not used in the later analyses.

``` r
# by piece
piece_dfm <- dfm_group(gen_dfm,
                       groups = "piece")

# by GPO
gpo_dfm <- dfm_group(gen_dfm,
                     groups = "general_political_orientation")


# less specific groupings NO ERFURT

# by piece
piece_dfm_no_erfurt <- dfm_group(gen_dfm_no_erfurt,
                                 groups = "piece")

# by GPO
gpo_dfm_no_erfurt <- dfm_group(gen_dfm_no_erfurt,
                               groups = "general_political_orientation")
```

### By GPO

Not currently used in the analysis but leaving for the moment.

``` r
# Left
left_sub <- dfm_subset(grouped_dfm, 
                       general_political_orientation == "Left")

# Right
right_sub <- dfm_subset(grouped_dfm, 
                        general_political_orientation == "Right")

# Center
center_sub <- dfm_subset(grouped_dfm, 
                         general_political_orientation == "Center")

# Unknown
unknown_sub <- dfm_subset(grouped_dfm,
                          general_political_orientation == "Unknown")
```

### By Piece

Not currently used in the analysis but leaving for the moment.

``` r
# Measures Taken
mt_sub <- dfm_subset(grouped_dfm,
                            piece == "measures")

# Mother
mother_sub <- dfm_subset(grouped_dfm,
                         piece == "mother")
```

## Convert general (reduced) corpus and dfm to dataframes for later use

``` r
# convert gen_dfm_reduced to dataframe
gen_datafr_reduced <- convert(gen_dfm_reduced, to = "data.frame")

# convert corp_reduced to dataframe (contains GPOs + other metadata)
corp_reduced_datafr <- convert(corp_reduced, to = "data.frame")
```

## Create corpus, dfm, ca, etc. of each piece w/o unknown GPO (for CA)

This is necessary for the CA, where I exclude those articles for which
the GPO is unknown.

Create dfm without distortionary Erfurt articles or articles with
unknown GPO:

``` r
# broken up into 3 separate steps for legibility

corp_no_erfurt_or_unknown <- 
  corpus_subset(corp_no_erfurt, 
                ! general_political_orientation == "Unknown" )

toks_no_erfurt_or_unknown <- 
  tokenize_and_remove_stopwords(corp_no_erfurt_or_unknown)

dfm_no_erfurt_or_unknown <- 
  convert_to_dfm_and_apply_dictionaries(toks_no_erfurt_or_unknown)
```

Just GPO, not piece (not currently using):

``` r
gpo_dfm_no_erfurt_or_unknown <- dfm_group(dfm_no_erfurt_or_unknown,
                                          groups = "general_political_orientation")
```

Grouped by piece and GPO:

``` r
grouped_dfm_no_erfurt_or_unknown <- 
  dfm_group(dfm_no_erfurt_or_unknown, 
            groups = c("piece", "general_political_orientation"))
```

Only articles on *The Measures Taken*:

``` r
mt_corp_no_erfurt_or_unknown <- 
  corpus_subset(corp_no_erfurt,
                piece == "measures" &! 
                  general_political_orientation == "Unknown" )


mt_toks_no_erfurt_or_unknown <- 
  tokenize_and_remove_stopwords(mt_corp_no_erfurt_or_unknown)

mt_dfm_no_erfurt_or_unknown <- 
  convert_to_dfm_and_apply_dictionaries(mt_toks_no_erfurt_or_unknown)

mt_dfm_gpo_no_erfurt_or_unknown <- 
  dfm_group(mt_dfm_no_erfurt_or_unknown, 
            groups = "general_political_orientation")
```

Only articles on *The Mother*:

``` r
mother_corp_no_unknown <- 
  corpus_subset(corp, 
                piece == "mother" &! 
                  general_political_orientation == "Unknown" )

mother_toks_no_unknown <- tokenize_and_remove_stopwords(mother_corp_no_unknown)

mother_dfm_no_unknown <- 
  convert_to_dfm_and_apply_dictionaries(mother_toks_no_unknown)

mother_dfm_gpo_no_unknown <- 
  dfm_group(mother_dfm_no_unknown, 
            groups = "general_political_orientation")
```

# FactoMineR Set-Up (for CA)

Create function for converting dfm to dataframe and performing CA:

``` r
convert_to_dataframe_and_perform_ca <- function(i) {
    convert(i, to = "data.frame") %>% 
        CA(quali.sup = 1, graph = FALSE)
}
```

  - NB: I originally created this function, because I performed the CA
    on multiple versions of the data set. I may delete it in the future
    and integrate the necessary code in the step below.

Create dfm with proper group names (will appear in CA graph) and apply
above-defined function:

``` r
grouped_dfm_no_erfurt_or_unknown <- 
  dfm_group(grouped_dfm_no_erfurt_or_unknown,
            groups = c("Measures.Center", "Mother.Center", "Measures.Left",
                       "Mother.Left", "Measures.Right", "Mother.Right"))

grouped_ca_no_erfurt_or_unknown <- 
  convert_to_dataframe_and_perform_ca(grouped_dfm_no_erfurt_or_unknown)
```

# Keywords In Context (KWIC) and Keyword Exploration

## Create function to link KWIC with data

``` r
combine_kwic_with_data <- function(corpus, words, window) {
    
    i <- kwic(corpus, words, window = window) %>% 
        as_tibble() 
    
    i$docname <- i$docname %>% 
        str_replace("text", "") 
    
    i <- i %>% mutate(docname = as.numeric(docname)) %>% 
        rename(article = docname) %>% 
        left_join(
          data_reduced, 
          by = "article") %>% 
        select(-c(text, other_metadata:Comp_Doc))
    
    }
```

## Explore various KWIC

All of these keywords relate to claims made in ch. 2 of the
dissertation. (All words are written lowercase to reflect how they
appear post-processing.)

  - primitiv \[primitive\]

<!-- end list -->

``` r
primitiv_kwic <- 
  combine_kwic_with_data(corp, "primitiv*", 15)
```

  - langweilig \[boring\]

<!-- end list -->

``` r
langweilig_kwic <- 
  combine_kwic_with_data(corp, c("langeweile", "langweilig*"), 10)
```

  - langweilig + primitiv

<!-- end list -->

``` r
langweilig_primitiv_kwic <- 
  inner_join(langweilig_kwic, primitiv_kwic, by = "article")
```

  - proletarisch \[proletarian\]

<!-- end list -->

``` r
proletarisch_kwic <- 
  combine_kwic_with_data(corp, "proletarisch*", 10)
```

  - pudowkin
    \[[Pudovkin](https://en.wikipedia.org/wiki/Vsevolod_Pudovkin)\]

<!-- end list -->

``` r
pudowkin_kwic <- 
  combine_kwic_with_data(corp, "pudowkin*", 10)
```

  - kunst \[art\]

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

  - wissen \[knowledge\]

<!-- end list -->

``` r
wissen_kwic <- 
  combine_kwic_with_data(corp, "wissen*", 10)
```

  - lehrstück \[learning-piece\] - *The Mother*

<!-- end list -->

``` r
lehrstueck_mother_kwic <- 
  combine_kwic_with_data(corp, "lehrstück*", 10) %>% 
  filter(piece == "mother")
```

  - oratorium \[oratorio\] - *The Measures Taken*

<!-- end list -->

``` r
oratorium_mt_kwic <- 
  combine_kwic_with_data(corp, "oratori*", 30) %>% 
  filter(piece == "measures")
```

  - arbeitersänger \[worker-singer(s)\] - *The Measures Taken*

<!-- end list -->

``` r
arbeitersaenger_mt_kwic <- 
  combine_kwic_with_data(corp, "arbeitersänger*", 30) %>% 
  filter(piece == "measures")
```

  - stalin and stalinismus \[Stalinism\]

<!-- end list -->

``` r
combine_kwic_with_data(corp, "stalin*", 30) %>% 
  print()
```

    ## # A tibble: 3 x 16
    ##   article  from    to pre   keyword post  pattern title newspaper publisher
    ##     <dbl> <int> <int> <chr> <chr>   <chr> <fct>   <chr> <chr>     <chr>    
    ## 1      62    23    23 Gork~ Stalin  "- h~ stalin* "Die~ Germania  Zentrum  
    ## 2      66    55    55 Best~ Stalin  "und~ stalin* "Gor~ Sächsisc~ SPD      
    ## 3      83   662   662 und ~ Stalin~ "von~ stalin* "Ber~ Der_Aben~ SPD      
    ## # ... with 6 more variables: political_affiliation <chr>,
    ## #   general_political_orientation <chr>, date <chr>, author <chr>,
    ## #   complete_or_incomplete <chr>, piece <chr>

Only 3 articles on Mother.

  - cantata

<!-- end list -->

``` r
combine_kwic_with_data(corp, "Kantate*", 25) %>% 
  print()
```

    ## # A tibble: 0 x 16
    ## # ... with 16 variables: article <dbl>, from <int>, to <int>, pre <chr>,
    ## #   keyword <chr>, post <chr>, pattern <fct>, title <chr>, newspaper <chr>,
    ## #   publisher <chr>, political_affiliation <chr>,
    ## #   general_political_orientation <chr>, date <chr>, author <chr>,
    ## #   complete_or_incomplete <chr>, piece <chr>

No results.

### Special Case: LEHRLERN in rightwing articles on The Mother

As I explain in ch. 2, **LEHRLERN** is the only term in the keyword
diction (see above) that can be considered an interpretative combination
of different words, rather than a mere grouping of synonyms.
**LEHRLERN** groups together words relating to “teaching” \[*lehren*\],
“learning” \[*lernen*\] and “pedagogy.”

Create tibble:

``` r
lehrlern_kwic <- kwic(corp, dict_regex, window = 20, valuetype = "regex") %>% 
  as_tibble()

lehrlern_kwic$docname <- lehrlern_kwic$docname %>% 
    str_replace("text", "") 

lehrlern_kwic <- lehrlern_kwic %>% 
  mutate(docname = as.numeric(docname)) %>% 
  rename(article = docname) %>% 
  left_join(data_main, by = "article") %>% 
  select(-c(text, other_metadata:Comp_Doc))
```

Filter for *The Mother* and right GPO:

``` r
lehrlern_kwic_mother_right <- lehrlern_kwic %>% 
  filter(piece == "mother" & general_political_orientation == "Right")
```

## Additional Terms Post-Processing (DFMs)

LEHRLERN in Mother

``` r
lehrlern_mother_gpo_count <-  
  dfm_select(mother_dfm_gpo, pattern = "LEHRLERN") %>% 
  as_tibble()

lehrlern_mother_gpo_count
```

    ## # A tibble: 4 x 2
    ##   doc_id  LEHRLERN
    ##   <chr>      <dbl>
    ## 1 Center        29
    ## 2 Left          18
    ## 3 Right         15
    ## 4 Unknown       15

# Dates of Publication with Counts

## Create Tables

### General

All articles:

``` r
dates_tib <- corp_reduced_datafr %>% 
  as_tibble() %>% 
  select(article, date, piece) %>% 
  group_by(piece, date) %>% 
  count(date) %>% 
  arrange(desc(n))
```

Separate columns:

``` r
dates_tib_separate <- corp_reduced_datafr %>% 
  as_tibble() %>% 
    select(article, date, piece) %>% 
    filter(str_detect(date, "\\S{2}\\.\\S{2}\\.[:digit:]{4}")) %>% 
    separate(date, sep = "\\.", into = c("day", "month", "year"))
```

Make compatible with [Lubridate](https://lubridate.tidyverse.org/)
package:

``` r
dates_tib_lubridate <- dates_tib_separate %>% 
    mutate(day = as.numeric(day), 
           month = as.numeric(month),
           year = as.numeric(year)) %>%
    relocate(day, .after = month) %>% 
    relocate(year, .before = month) %>% 
    mutate(date = make_date(year, month, day)) %>% 
    select(article, date, piece) %>% 
  filter(!is.na(date)) %>% 
  arrange(date)
```

Percent of articles *without* full dates (i.e. excluded from Lubridate):

``` r
articles_perc_without_full_dates <- 
  (((nrow(dates_tib_separate) - 
     nrow(dates_tib_lubridate)) 
  / nrow(dates_tib_lubridate)) * 100) %>% 
  round() 
```

12% of articles do *not* have full dates.

Percent of articles *with* full dates (i.e. included for Lubridate):

``` r
articles_perc_with_full_dates <- 100 - articles_perc_without_full_dates
```

88% of articles have full dates.

### By Piece

#### Measures Taken

<!-- may not need this first one (non-Lubridate) -->

All

``` r
dates_mt <- dates_tib %>% 
  filter(piece == "measures") %>% 
  ungroup %>% 
  select(-piece)
```

Edited, for Lubridate

  - Chronological, w/o counts (saved for computing range, length, etc.
    for write-up):

<!-- end list -->

``` r
dates_mt_lubridate <- 
  dates_tib_lubridate %>% 
  filter(piece == "measures") %>% 
  arrange(date)
```

  - Counted and sorted descending:

<!-- end list -->

``` r
dates_tib_lubridate %>% 
  filter(piece == "measures") %>%
  count(date) %>% 
  arrange(desc(n))
```

    ## # A tibble: 20 x 2
    ##    date           n
    ##    <date>     <int>
    ##  1 1930-12-15    10
    ##  2 1930-12-16     2
    ##  3 1931-01-20     2
    ##  4 1930-06-04     1
    ##  5 1930-12-13     1
    ##  6 1930-12-20     1
    ##  7 1930-12-22     1
    ##  8 1930-12-24     1
    ##  9 1931-01-01     1
    ## 10 1931-01-03     1
    ## 11 1931-01-11     1
    ## 12 1931-01-14     1
    ## 13 1931-01-24     1
    ## 14 1931-05-09     1
    ## 15 1931-12-09     1
    ## 16 1932-02-19     1
    ## 17 1932-06-01     1
    ## 18 1932-09-24     1
    ## 19 1932-11-22     1
    ## 20 1932-11-25     1

#### Mother

All

``` r
dates_mother <- dates_tib %>% 
  filter(piece == "mother") %>% 
  ungroup %>% 
  select(-piece)
```

Edited, for Lubridate

  - Chronological, w/o counts (saved for computing range, length, etc.
    for write-up):

<!-- end list -->

``` r
dates_mother_lubridate <- 
  dates_tib_lubridate %>% 
  filter(piece == "mother") %>% 
  arrange(date)
```

  - Counted and sorted descending:

<!-- end list -->

``` r
dates_tib_lubridate %>% 
  filter(piece == "mother") %>%
  count(date) %>% 
  arrange(desc(n))
```

    ## # A tibble: 20 x 2
    ##    date           n
    ##    <date>     <int>
    ##  1 1932-01-18    20
    ##  2 1932-01-17     7
    ##  3 1932-01-19     7
    ##  4 1932-01-24     5
    ##  5 1932-01-23     4
    ##  6 1932-01-22     3
    ##  7 1932-01-25     3
    ##  8 1932-01-26     2
    ##  9 1932-01-27     2
    ## 10 1932-01-30     2
    ## 11 1931-12-23     1
    ## 12 1932-01-08     1
    ## 13 1932-01-13     1
    ## 14 1932-01-15     1
    ## 15 1932-01-20     1
    ## 16 1932-01-21     1
    ## 17 1932-01-29     1
    ## 18 1932-02-23     1
    ## 19 1932-03-05     1
    ## 20 1932-12-09     1

## Date Ranges, Time Lengths, etc.

### Time Length in Years

All Articles (edited for Lubridate):

``` r
difftime(tail(dates_tib_lubridate$date, 1),
         head(dates_tib_lubridate$date, 1)) %>% 
  time_length(unit = "year")
```

    ## [1] 2.516085

*The Measures Taken*

``` r
difftime(tail(dates_mt_lubridate$date, 1),
         head(dates_mt_lubridate$date, 1)) %>% 
  time_length(unit = "year")
```

    ## [1] 2.477755

*The Mother*

``` r
difftime(tail(dates_mother_lubridate$date, 1),
         head(dates_mother_lubridate$date, 1)) %>% 
  time_length(unit = "year")
```

    ## [1] 0.9637235

### Range / Interval

#### All

Save general start and end dates as variables for write-up:

``` r
articles_first_date <- head(dates_tib_lubridate$date, 1)

articles_last_date <- tail(dates_tib_lubridate$date, 1)
```

General dates interval:

``` r
interval(start = articles_first_date,
         end = articles_last_date)
```

    ## [1] 1930-06-04 UTC--1932-12-09 UTC

#### The Measures Taken

``` r
interval(start = head(dates_mt_lubridate$date, 1),
         end = tail(dates_mt_lubridate$date, 1))
```

    ## [1] 1930-06-04 UTC--1932-11-25 UTC

#### The Mother

``` r
interval(start = head(dates_mother_lubridate$date, 1),
         end = tail(dates_mother_lubridate$date, 1))
```

    ## [1] 1931-12-23 UTC--1932-12-09 UTC

### Proportion of Articles Published near Premieres

#### The Measures Taken

Save date of premiere as variable (cf.
[GBA](https://www.suhrkamp.de/werkausgabe/werke_grosse_kommentierte_berliner_und_frankfurter_ausgabe_30_baende_in_32_teilbaenden_und_ein_registerband_leinen_24.html)
3: 431):

``` r
mt_premiere_date <- ymd("1930-12-13")
```

Create interval for 1 week after premiere:

``` r
mt_premiere_week <- interval(start = mt_premiere_date,
         end = mt_premiere_date + dweeks(x = 1))
```

Percentage of articles published within 1 week of premiere:

``` r
articles_mt_premiere_week_perc <- 
  round(((dates_mt_lubridate %>% 
    filter(date %within% mt_premiere_week) %>% 
    nrow()) / nrow(dates_mt_lubridate)) * 100)
```

45% of articles on *The Measures Taken* were published within one week
of the premiere.

#### The Mother

Save date of premiere (cf.
[GBA](https://www.suhrkamp.de/werkausgabe/werke_grosse_kommentierte_berliner_und_frankfurter_ausgabe_30_baende_in_32_teilbaenden_und_ein_registerband_leinen_24.html)
3: 478):

``` r
mother_premiere_date <- ymd("1932-01-17")
```

Create interval for 1 week after premiere:

``` r
mother_premiere_week <- interval(start = mother_premiere_date,
         end = mother_premiere_date + dweeks(x = 1))
```

Percentage of articles published within 1 week of premiere:

``` r
articles_mother_premiere_week_perc <- 
  round(((dates_mother_lubridate %>% 
    filter(date %within% mother_premiere_week) %>% 
    nrow()) / nrow(dates_mother_lubridate)) * 100)
```

74% of articles on *The Mother* were published within one week of the
premiere.

## Articles on The Measures Taken published in 1932 (with full dates)

``` r
dates_tib_lubridate %>% 
    filter(piece == "measures" &
            year(date) == 1932) %>% 
    left_join(data_main, by = "article")
```

    ## # A tibble: 5 x 18
    ##   article date.x     piece.x title text  newspaper publisher political_affil~
    ##     <dbl> <date>     <chr>   <chr> <chr> <chr>     <chr>     <chr>           
    ## 1      35 1932-02-19 measur~ "Leh~ "Nic~ Literari~ Welt Ver~ Unknown         
    ## 2      50 1932-06-01 measur~ "„Di~ "1.\~ Der_Kämp~ KPD       KPD             
    ## 3      34 1932-09-24 measur~ "Die~ "Die~ Arbeiter~ Sozialde~ SPÖ             
    ## 4      36 1932-11-22 measur~ "Ers~ "In ~ General-~ Independ~ Unknown         
    ## 5      37 1932-11-25 measur~ "Kom~ "Am ~ Bayrisch~ Independ~ Unknown         
    ## # ... with 10 more variables: general_political_orientation <chr>,
    ## #   date.y <chr>, author <chr>, complete_or_incomplete <chr>,
    ## #   other_metadata <chr>, other_notes <chr>, AdK <chr>, AdK_Duplicate <chr>,
    ## #   Comp_Doc <chr>, piece.y <chr>

# Visualizations

Create color scheme:

``` r
colors_four <- c("#E41A1C", "#377EB8", "#4DAF4A", "#984EA3")
```

Create English labels for both works:

``` r
labels_english <- c(measures = "Measures Taken", mother = "Mother")
```

## Visual Summary of Reduced Corpus

Set-up for boxplot:

``` r
# 3 steps for set-up

# STEP 1
# create table with total number of tokens for each article
toks_grouped <- gen_datafr_reduced %>% 
    mutate(toks_sum = rowSums(.[ ,2:ncol(.)]), .after = 1) %>% 
    select(doc_id, toks_sum)

# NB: possible to use tidyverse syntax but much slower (including for reference)
# gen_datafr_reduced %>%
    # rowwise() %>% mutate(toks_sum = sum(c_across(2:ncol(.))), .after = 1)

# STEP 2
# add GPO column to gen_datafr_reduced (can't combine with above)
toks_grouped <- 
  left_join(toks_grouped, 
            corp_reduced_datafr[ , c("doc_id", 
                                     "general_political_orientation", 
                                     "piece")], 
            by = "doc_id", 
            copy = TRUE)

# STEP 3
# added for observation counts
toks_grouped <- toks_grouped %>% 
    add_count(piece, general_political_orientation)

# STEP 4
# must save GPO as factor and relevel, so that Left appears first
toks_grouped <- toks_grouped %>% 
    mutate(general_political_orientation = 
               as.factor(general_political_orientation))

toks_grouped$general_political_orientation <- 
    toks_grouped$general_political_orientation %>% fct_relevel("Left")
```

Code for boxplot:

``` r
toks_summary_boxplot <- toks_grouped %>% 
    ggplot(aes(x = piece, y = toks_sum, 
               fill = general_political_orientation)) +
    geom_boxplot(alpha = 0.7,
                 varwidth = FALSE) +
    stat_summary(fun = median, geom = "point", 
                 shape = 21, size = 4, color = "black", fill = "yellow2", 
                 alpha = 0.6) +
    stat_summary(fun = median, geom = "line", 
                 size = 1.2, color = "yellow2", alpha = 0.5, aes(group = 1)) +
    scale_fill_brewer(type = "seq", palette = "Set1") +
    geom_jitter(color = "darkgrey",
                size = 1.25, alpha = 0.95,
                show.legend = FALSE) + 
    coord_cartesian(xlim = NULL, ylim = c(0, 900)) +
    ylab("Tokens") + 
    scale_x_discrete(labels = c("Measures Taken", "Mother")) +
    xlab(NULL) +
    labs(fill = "Political Orientation",
         title = "Tokens per Article (Post-Processing)",
         subtitle = "Scaled for readability. Several outliers excluded.", 
         caption = "Yellow points denote the median tokens per article for each piece. NB: The positions of the scatter points do not correspond to political orientation.") +
    geom_text(aes(label = ..count.., 
                  y = ..prop..), 
              stat = "count",
              position = position_dodge(0.75)) +
    theme(axis.title.x = element_blank(),
          axis.ticks.x = element_blank(),
          panel.grid.major.x = element_blank(),
          panel.grid.major.y = element_line(color = "grey", linetype = 3),
          panel.grid.minor.y = element_line(color = "grey", linetype = 3),
          plot.caption.position = "plot",
          panel.background = element_rect(fill = "white", colour = 'black'))

toks_summary_boxplot
```

![](reception_analysis_files/figure-gfm/unnamed-chunk-80-1.png)<!-- -->

A couple of quick points, which I review in the write-up in greater
detail:

  - There are far more articles on *The Mother* than on *The Measures
    Taken*.

  - Articles on *The Measures Taken* tend to be longer than those on
    *The Mother*.

  - Articles on *The Measures Taken* are overwhelmingly from leftwing
    publications.

### Very Long Articles

The goal is to identify the very long articles in *Measures - Unknown*
category, because they stretch out the IQR.

``` r
corp_reduced_summary %>% 
    filter(general_political_orientation == "Unknown" &
               piece == "measures") %>% 
    arrange(desc(tokens)) %>% 
  select(article, tokens, title:piece)
```

    ## # A tibble: 10 x 11
    ##    article tokens title newspaper publisher political_affil~ general_politic~
    ##      <dbl>  <int> <chr> <chr>     <chr>     <chr>            <chr>           
    ##  1      35   2055 Lehr~ Literari~ Welt Ver~ Unknown          Unknown         
    ##  2      18   1894 Poli~ Der_Anbr~ Universa~ Unknown          Unknown         
    ##  3      42    822 Thea~ Heidelbe~ Independ~ Unknown          Unknown         
    ##  4      45    750 Brec~ Bergisch~ Vossen    Unknown          Unknown         
    ##  5      41    553 Welt~ Thüringi~ Unknown   Unknown          Unknown         
    ##  6      25    535 Anme~ Die_Lite~ Deutsche~ Unknown          Unknown         
    ##  7      19    439 Prob~ Der_Anbr~ Universa~ Unknown          Unknown         
    ##  8       7    365 Brec~ Münchner~ Independ~ Unknown          Unknown         
    ##  9      37    314 Komm~ Bayrisch~ Independ~ Unknown          Unknown         
    ## 10      43    194 Brec~ Dresdner~ Independ~ Unknown          Unknown         
    ## # ... with 4 more variables: date <chr>, author <chr>,
    ## #   complete_or_incomplete <chr>, piece <chr>

## Wordclouds

Grouped by Piece:

``` r
# must rename groups in English for word cloud
piece_dfm_no_erfurt_english <- dfm_group(piece_dfm_no_erfurt, 
                                         groups = c("Measures Taken", "Mother"))

textplot_wordcloud(piece_dfm_no_erfurt_english,
                   max_words = 100,
                   color = colors_four,
                   rotation = FALSE,
                   comparison = TRUE,
                   labelcolor = "black")
```

![](reception_analysis_files/figure-gfm/unnamed-chunk-82-1.png)<!-- -->

*Mother* - Grouped by GPO (no unknown):

``` r
wordcloud_mother_gpo <- 
  textplot_wordcloud(mother_dfm_gpo_no_unknown,
                   max_words = 100,
                   rotation = FALSE,
                   # must do manual order for colors, so that Left is red
                   color = c("#377EB8", "#E41A1C", "#4DAF4A", "#984EA3"),
                   comparison = TRUE,
                   labelcolor = "black")
```

![](reception_analysis_files/figure-gfm/unnamed-chunk-83-1.png)<!-- -->

## Word Frequency

By Piece:

``` r
# grouped by piece (weighted)
freq_piece <- dfm_weight(gen_dfm_reduced, scheme = "prop") %>% 
    textstat_frequency(n = 15, groups = "piece")

freq_piece_plot <- 
  ggplot(freq_piece, 
       aes(x = nrow(freq_piece):1, y = frequency, fill = group)) +
    geom_col() +
    facet_wrap(~ group, scales = "free", nrow = 2,
               labeller = labeller(group = labels_english)
               ) +
    coord_flip() +
    scale_x_continuous(breaks = nrow(freq_piece):1, 
                       labels = freq_piece$feature) +
    labs(x = NULL, y = "Relative Frequency") +
    scale_fill_manual(values = colors_four) + # prob could just use brewer, b/c only 2
    theme_bw() +
    theme(legend.position = "none",
          panel.grid.major.y = element_line(color = "grey", linetype = 3),
          panel.grid.minor.y = element_blank(),
          panel.grid.major.x = element_line(color = "grey", linetype = 3),
          panel.grid.minor.x = element_line(color = "grey", linetype = 3),
          plot.caption.position = "plot",
          panel.background = element_rect(fill = "white", colour = 'black'))

freq_piece_plot
```

![](reception_analysis_files/figure-gfm/unnamed-chunk-84-1.png)<!-- -->

By Piece + GPO:

``` r
# create weighted frequency table
freq_grouped_wt <- dfm_weight(gen_dfm_reduced, scheme = "prop") %>% 
    textstat_frequency(n = 10, 
                       groups = c("piece", "general_political_orientation")) 

# rename groups
freq_grouped_wt <- freq_grouped_wt %>% 
    mutate(group = str_replace_all(group, 
                                   c("measures.Left" = "Measures Taken | Left", 
                                     "measures.Center" = 
                                       "Measures Taken | Center", 
                                     "measures.Right" = 
                                       "Measures Taken | Right",
                                     "measures.Unknown" = 
                                       "Measures Taken | Unknown",
                                     "mother.Left" = "Mother | Left",
                                     "mother.Center" = "Mother | Center",
                                     "mother.Right" = "Mother | Right",
                                     "mother.Unknown" = "Mother | Unknown")))

# save group column as factor and reorder
freq_grouped_wt$group <- freq_grouped_wt$group %>% as.factor() 

freq_grouped_wt$group <- freq_grouped_wt$group %>% 
    fct_relevel("Measures Taken | Left") %>% 
    fct_relevel("Mother | Left", after = 4)

# code for graph
freq_piece_gpo_plot <- ggplot(freq_grouped_wt, 
       aes(x = nrow(freq_grouped_wt):1, y = frequency, fill = group)) +
    geom_col() +
    facet_wrap(~ group, scales = "free", nrow = 2) +
    coord_flip() +
    scale_x_continuous(breaks = nrow(freq_grouped_wt):1, 
                       labels = freq_grouped_wt$feature) +
    labs(x = NULL, y = "Relative Frequency") +
    scale_fill_manual(values = c(colors_four, colors_four)) + # repeat b/c need 8
    theme_bw() +
    theme(legend.position = "none",
          panel.grid.major.y = element_line(color = "grey", linetype = 3),
          panel.grid.minor.y = element_blank(),
          panel.grid.major.x = element_line(color = "grey", linetype = 3),
          panel.grid.minor.x = element_line(color = "grey", linetype = 3),
          plot.caption.position = "plot",
          panel.background = element_rect(fill = "white", colour = 'black'))

freq_piece_gpo_plot
```

![](reception_analysis_files/figure-gfm/unnamed-chunk-85-1.png)<!-- -->

## Correspondence Analysis

Main

``` r
ca_main <- plot(grouped_ca_no_erfurt_or_unknown,
                invisible = "row",
                col.quali.sup = "darkblue",
                selectCol = "contrib 20",
                autoLab = "y", 
                ylim = c(-0.75, 1.75), 
                unselect = 1,
                cex = 0.85)

ca_main <- ca_main + 
    labs(title = "The Political-Aesthetic Space of The Measures Taken and The Mother",
         subtitle = "(CA of pieces and political orientations with the top 20 contributing keywords.)") +
    theme_bw()

ca_main
```

![](reception_analysis_files/figure-gfm/unnamed-chunk-86-1.png)<!-- -->

*Measures Taken*

``` r
ca_measures <- plot(grouped_ca_no_erfurt_or_unknown,
     invisible = "row",
     col.quali.sup = "darkblue",
     selectCol = "contrib 20",
     autoLab = "y",
     xlim = c(0, 1.5),
     ylim = c(-1, 2), 
     unselect = 1,
     cex = 0.85)

ca_measures <- ca_measures + 
  labs(title = "The Political-Aesthetic Space of The Measures Taken",
       subtitle = "(Enlargement of positive x region of above.)") +
  theme_bw()

ca_measures
```

![](reception_analysis_files/figure-gfm/unnamed-chunk-87-1.png)<!-- -->

*Mother*

``` r
ca_mother <- plot(grouped_ca_no_erfurt_or_unknown,
     invisible = "row",
     col.quali.sup = "darkblue",
     selectCol = "contrib 20",
     autoLab = "y",
     xlim = c(-1.25, 0),
     ylim = c(-0.75, 0.25), 
     unselect = 1,
     cex = 0.85)

ca_mother <- ca_mother + 
  labs(title = "The Political-Aesthetic Space of The Mother",
       subtitle = "(Enlargement of negative x region of above.)") +
  theme_bw() 

ca_mother
```

![](reception_analysis_files/figure-gfm/unnamed-chunk-88-1.png)<!-- -->
