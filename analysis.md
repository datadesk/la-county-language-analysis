Analyzing L.A. County’s changing languages in Census microdata
================

By [Ryan Menezes](https://twitter.com/ryanvmenezes/)

``` r
library(ipumsr)
library(tidyverse)
```

The IPUMS-USA extract comes with two files:

1.  A fixed-width data file in a compressed folder
2.  A data codebook in an XML format, which describes the data based on
    the [Data Documentation Initative](https://ddialliance.org/), or
    “DDI”

The `ipumsr` package can read in both.

``` r
ddi = read_ipums_ddi('data/usa_00011.xml')
data = read_ipums_micro(ddi)
```

    ## Use of data from IPUMS-USA is subject to conditions including that users should
    ## cite the data appropriately. Use command `ipums_conditions()` for more details.

## The codebook

The DDI details the variables in the extract.

``` r
info.variables = ddi$var_info
info.variables
```

    ## # A tibble: 14 x 10
    ##    var_name var_label var_desc val_labels code_instr start   end imp_decim
    ##    <chr>    <chr>     <chr>    <list>     <chr>      <dbl> <dbl>     <dbl>
    ##  1 YEAR     Census y… "YEAR r… <tibble [… <NA>           1     4         0
    ##  2 SAMPLE   IPUMS sa… "SAMPLE… <tibble [… <NA>           5    10         0
    ##  3 SERIAL   Househol… "SERIAL… <tibble [… "\nSERIAL…    11    18         0
    ##  4 CBSERIAL Original… "CBSERI… <tibble [… "\nCBSERI…    19    31         0
    ##  5 HHWT     Househol… "HHWT i… <tibble [… "\nHHWT i…    32    41         2
    ##  6 CLUSTER  Househol… CLUSTER… <tibble [… "\nCLUSTE…    42    54         0
    ##  7 STATEFIP State (F… "STATEF… <tibble [… <NA>          55    56         0
    ##  8 COUNTYF… County (… "COUNTY… <tibble [… "\nCOUNTY…    57    59         0
    ##  9 STRATA   Househol… "STRATA… <tibble [… "\nSTRATA…    60    71         0
    ## 10 GQ       Group qu… "GQ cla… <tibble [… <NA>          72    72         0
    ## 11 PERNUM   Person n… PERNUM … <tibble [… "\n\nPERN…    73    76         0
    ## 12 PERWT    Person w… "PERWT … <tibble [… "\nPERWT …    77    86         2
    ## 13 LANGUAGE Language… LANGUAG… <tibble [… <NA>          87    88         0
    ## 14 LANGUAG… Language… LANGUAG… <tibble [… <NA>          89    92         0
    ## # … with 2 more variables: var_type <chr>, rectypes <lgl>

The key column in this extract is
[LANGUAGE](https://usa.ipums.org/usa-action/variables/LANGUAGE), which
“reports the language that the respondent spoke at home, particularly
if a language other than English was spoken.” The codebook provides the
values for every LANGUAGED code.

``` r
info.variables %>% 
  filter(var_name == 'LANGUAGE') %>% 
  pull(val_labels) %>% 
  `[[`(1)
```

    ## # A tibble: 92 x 2
    ##      val lbl            
    ##    <dbl> <chr>          
    ##  1     0 N/A or blank   
    ##  2     1 English        
    ##  3     2 German         
    ##  4     3 Yiddish, Jewish
    ##  5     4 Dutch          
    ##  6     5 Swedish        
    ##  7     6 Danish         
    ##  8     7 Norwegian      
    ##  9     8 Icelandic      
    ## 10     9 Scandinavian   
    ## # … with 82 more rows

## The data

All of the data in the extract. Most of these come preselected with any
IPUMS extract.

``` r
data %>% head()
```

    ## # A tibble: 6 x 14
    ##    YEAR       SAMPLE SERIAL CBSERIAL  HHWT CLUSTER STATEFIP COUNTYFIP
    ##   <int>    <int+lbl>  <dbl>    <dbl> <dbl>   <dbl> <int+lb> <dbl+lbl>
    ## 1  1980 198001 [198… 192089       NA    20 1.98e12 6 [Cali…      2323
    ## 2  1980 198001 [198… 192089       NA    20 1.98e12 6 [Cali…      2323
    ## 3  1980 198001 [198… 192089       NA    20 1.98e12 6 [Cali…      2323
    ## 4  1980 198001 [198… 192089       NA    20 1.98e12 6 [Cali…      2323
    ## 5  1980 198001 [198… 192089       NA    20 1.98e12 6 [Cali…      2323
    ## 6  1980 198001 [198… 192090       NA    20 1.98e12 6 [Cali…      2323
    ## # … with 6 more variables: STRATA <dbl>, GQ <int+lbl>, PERNUM <dbl>,
    ## #   PERWT <dbl>, LANGUAGE <int+lbl>, LANGUAGED <int+lbl>

The data is already filtered down to California. Filter it down again to
just L.A. County, then keep only the relevant columns.

``` r
la.data = data %>% 
  filter(COUNTYFIP == 37) %>% 
  select(YEAR, LANGUAGE, PERWT)

la.data %>% head()
```

    ## # A tibble: 6 x 3
    ##    YEAR    LANGUAGE PERWT
    ##   <int>   <int+lbl> <dbl>
    ## 1  1980 1 [English]    20
    ## 2  1980 1 [English]    20
    ## 3  1980 1 [English]    20
    ## 4  1980 1 [English]    20
    ## 5  1980 1 [English]    20
    ## 6  1980 1 [English]    20

## Aggregating microdata into totals

Each line of a microdata file represents a person’s actual response to
the survey.
[PERWT](https://usa.ipums.org/usa-action/variables/PERWT#description_section)
is the approxmiation of how many people the line of data represents. It
needs to be aggregated and summed to get total counts for the language
for that year.

``` r
la.data.agg = la.data %>% 
  group_by(YEAR, LANGUAGE) %>% 
  summarise(PERWT = sum(PERWT))

la.data.agg %>% head()
```

    ## # A tibble: 6 x 3
    ## # Groups:   YEAR [1]
    ##    YEAR            LANGUAGE   PERWT
    ##   <int>           <int+lbl>   <dbl>
    ## 1  1980 0 [N/A or blank]     346500
    ## 2  1980 1 [English]         4888960
    ## 3  1980 2 [German]            47660
    ## 4  1980 3 [Yiddish, Jewish]   18900
    ## 5  1980 4 [Dutch]             13560
    ## 6  1980 5 [Swedish]            3780

Reformat the data, separating the labels from the code, plus add columns
for the percent of the population speaking that language in each year
and the county rank for that year.

``` r
la.languages = la.data.agg %>% 
  group_by(YEAR) %>% 
  mutate(percent = PERWT / sum(PERWT)) %>% 
  # take out "N/A or blank" before calculating rank
  filter(LANGUAGE != 0) %>% 
  mutate(rankinyear = rank(desc(percent))) %>% 
  ungroup(YEAR) %>% 
  transmute(
    year = YEAR,
    langcode = zap_labels(LANGUAGE),
    language = as.character(as_factor(LANGUAGE)),
    total = PERWT,
    percent, rankinyear
  )

la.languages %>% head()
```

    ## # A tibble: 6 x 6
    ##    year langcode language          total  percent rankinyear
    ##   <int>    <int> <chr>             <dbl>    <dbl>      <dbl>
    ## 1  1980        1 English         4888960 0.652             1
    ## 2  1980        2 German            47660 0.00636           7
    ## 3  1980        3 Yiddish, Jewish   18900 0.00252          12
    ## 4  1980        4 Dutch             13560 0.00181          16
    ## 5  1980        5 Swedish            3780 0.000504         28
    ## 6  1980        6 Danish             2820 0.000376         33

## Analysis

What were the top 10 languages spoken in 1980?

``` r
la.languages %>% 
  filter(year == 1980) %>% 
  arrange(-total) %>% 
  head(10)
```

    ## # A tibble: 10 x 6
    ##     year langcode language            total percent rankinyear
    ##    <int>    <int> <chr>               <dbl>   <dbl>      <dbl>
    ##  1  1980        1 English           4888960 0.652            1
    ##  2  1980       12 Spanish           1591000 0.212            2
    ##  3  1980       43 Chinese             77920 0.0104           3
    ##  4  1980       54 Filipino, Tagalog   68700 0.00917          4
    ##  5  1980       48 Japanese            58280 0.00778          5
    ##  6  1980       49 Korean              55000 0.00734          6
    ##  7  1980        2 German              47660 0.00636          7
    ##  8  1980       11 French              38800 0.00518          8
    ##  9  1980       28 Armenian            38600 0.00515          9
    ## 10  1980       10 Italian             34940 0.00466         10

What were the top 10 languages spoken in 2018?

``` r
la.languages %>% 
  filter(year == 2018) %>% 
  arrange(-total) %>% 
  head(10)
```

    ## # A tibble: 10 x 6
    ##     year langcode language                  total percent rankinyear
    ##    <int>    <int> <chr>                     <dbl>   <dbl>      <dbl>
    ##  1  2018        1 English                 4108477 0.407            1
    ##  2  2018       12 Spanish                 3741860 0.370            2
    ##  3  2018       43 Chinese                  401703 0.0398           3
    ##  4  2018       54 Filipino, Tagalog        236587 0.0234           4
    ##  5  2018       49 Korean                   181622 0.0180           5
    ##  6  2018       28 Armenian                 169458 0.0168           6
    ##  7  2018       31 Hindi and related         79249 0.00784          7
    ##  8  2018       29 Persian, Iranian, Farsi   72613 0.00719          8
    ##  9  2018       50 Vietnamese                69346 0.00686          9
    ## 10  2018       48 Japanese                  49664 0.00492         10

How have English and Spanish, far and away the top languages, changed
over time?

``` r
la.languages %>% 
  filter(language %in% c('English', 'Spanish')) %>% 
  ggplot(aes(year, percent * 100, color = language)) +
  geom_line() +
  geom_point() +
  xlab('Year') +
  ylab('Percent of county population') +
  ggtitle('English and Spanish speakers in L.A. County') +
  theme_minimal()
```

![](analysis_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

How about the other languages?

Start by keeping any language that has been in the top 10 for a
particular year.

``` r
ever.top.10 = la.languages %>% 
  filter(rankinyear <= 10) %>% 
  distinct(language) %>% 
  left_join(la.languages)

ever.top.10 %>% head()
```

    ## # A tibble: 6 x 6
    ##   language  year langcode   total percent rankinyear
    ##   <chr>    <int>    <int>   <dbl>   <dbl>      <dbl>
    ## 1 English   1980        1 4888960 0.652            1
    ## 2 English   1990        1 4436610 0.501            1
    ## 3 English   2000        1 4036798 0.424            1
    ## 4 English   2010        1 3930328 0.400            1
    ## 5 English   2018        1 4108477 0.407            1
    ## 6 German    1980        2   47660 0.00636          7

``` r
unique(ever.top.10$language)
```

    ##  [1] "English"                 "German"                 
    ##  [3] "Italian"                 "French"                 
    ##  [5] "Spanish"                 "Armenian"               
    ##  [7] "Chinese"                 "Japanese"               
    ##  [9] "Korean"                  "Filipino, Tagalog"      
    ## [11] "Persian, Iranian, Farsi" "Vietnamese"             
    ## [13] "Hindi and related"

This leaves 13 languages to look at

``` r
lang.yearly.barplot = ever.top.10 %>%
  # take out the two biggest
  filter(!language %in% c('English', 'Spanish')) %>% 
  arrange(-year, -total) %>% 
  # control ordering
  mutate(language = fct_inorder(language)) %>% 
  ggplot(aes(as_factor(year), percent * 100, fill = language)) +
  geom_bar(stat = 'identity', position = 'dodge') +
  scale_fill_brewer(palette = "Paired", name = 'Language') +
  xlab('Year') +
  ylab('Percent of county population') +
  ggtitle('All languages ever in the top 10 spoken in L.A. County') +
  theme_minimal()

lang.yearly.barplot
```

![](analysis_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

What has the top 10 looked like over time?

``` r
top.10.by.year = ever.top.10 %>%
  mutate(language = fct_inorder(language)) %>% 
  ggplot(aes(x = year, y = rankinyear, group = language)) +
  geom_point(aes(size = total), color = 'grey') +
  geom_line(color = 'grey') +
  geom_point(
    data = . %>% filter(rankinyear > 4),
    aes(color = language, size = total)
  ) +
  geom_line(
    data = . %>% filter(rankinyear > 4),
    aes(color = language)
  ) +
  scale_x_continuous(
    limits = c(1980, 2021),
    breaks = c(1980, 1990, 2000, 2010, 2018),
  ) +
  scale_colour_brewer(palette = 'Paired') +
  scale_y_reverse(
    breaks = 1:10,
    limits = c(10,1)
  ) +
  theme_minimal() +
  theme(legend.position = 'none') +
  ylab('Rank in year') +
  xlab('') + 
  ggtitle('Top 10 languages spoken in L.A. County')

top.10.by.year +
  # annotate with names at end
  geom_text(
    data = . %>% group_by(language) %>% filter(rankinyear <= 10) %>% filter(year == max(year)),
    aes(year + 0.76, rankinyear, label = str_replace_all(word(language), ',', '')),
    hjust = 'left',
    size = 3
  )
```

![](analysis_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

How have these languages risen and fallen in and out of the top 10?

``` r
top.10.by.year +
  scale_y_reverse(
    breaks = 1:25,
    minor_breaks = NULL
  ) +
  # annotation
  geom_text(
    data = . %>% group_by(language) %>% filter(year == max(year)),
    aes(year + 0.76, rankinyear, label = str_replace_all(word(language), ',', '')),
    hjust = 'left',
    size = 3
  )
```

![](analysis_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

Summarize the change in languages other than English and Spanish by
calculating the difference between now and 1980.

``` r
ever.top.10 %>% 
  filter(!language %in% c('English', 'Spanish')) %>% 
  filter(year == 1980 | year == 2018) %>% 
  arrange(-total) %>% 
  select(language, year, percent) %>% 
  mutate(
    percent = percent * 100,
    language = word(language),
    language = str_replace_all(language, ',', ''),
    language = fct_rev(fct_inorder(language))
  ) %>% 
  ggplot(aes(x = percent, y = language)) +
  geom_segment(
    data = . %>%
      pivot_wider(names_from = year, values_from = percent) %>% 
      mutate(netgain = (`2018` - `1980`) > 0),
    aes(x = `1980`, xend = `2018`, y = language, yend = language, color = netgain),
    arrow = arrow(length = unit(0.2, "cm"))
  ) +
  theme_minimal() +
  theme(legend.position = 'none')
```

![](analysis_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->