
## Daily Portuguese COVID-19 Data

**Last updated: Sat 11 Jul 2020 (03:25:16 UTC \[+0000\])**

  - Data available from **26 Feb 2020** until **10 Jul 2020** (136
    days).

### Download User Friendly Version

  - Download the cleaned and user friendly data from:
    **[covid19pt\_DSSG\_Long.csv](https://raw.githubusercontent.com/CEAUL/Dados_COVID-19_PT/master/data/covid19pt_DSSG_Long.csv)**
      - `data`: Date (Portuguese spelling).
      - `origVars`: Variable name taken from source data.
      - `origType`: Orginal variable count type.
      - `other`: Other types of `origVars`.
      - `symptoms`: Recorded COVID-19 symptoms.
      - `sex`: Gender (`F` - Females, `M` - Males, `All` - Females &
        Males).
      - `ageGrp`: Age groups in years (`desconhecidos` - unknown).
      - `ageGrpLower`: Lower limit of age group (useful for sorting).
      - `ageGrpUpper`: Upper limit of age group.
      - `region`: Portuguese Regions
      - `value`: Numeric value.
      - `valueUnits`: Units for the variable `value`.
  - Download the original unprocessed data (json to CSV):
    **[covid19pt\_DSSG\_Orig.csv](https://raw.githubusercontent.com/CEAUL/Dados_COVID-19_PT/master/data/covid19pt_DSSG_Orig.csv)**

### Source

For more information about the data and variables see:
**<https://github.com/dssg-pt/covid19pt-data>**

The original data were downloaded from an API provide by VOST
**<https://covid19-api.vost.pt/Requests/get_entry/>**

## Example Usage

### Read in the data

Using the `data.table` package to process the data.

``` r
# Load Libraries
library(data.table)
suppressPackageStartupMessages(library(here)) # library(here)

# Read in data as a data.frame and data.table object.
CV <- fread(here("data", "covid19pt_DSSG_Long.csv"))
str(CV)
## Classes 'data.table' and 'data.frame':   11424 obs. of  12 variables:
##  $ data       : chr  "2020-02-26" "2020-02-27" "2020-02-28" "2020-02-29" ...
##  $ origVars   : chr  "cadeias_transmissao" "cadeias_transmissao" "cadeias_transmissao" "cadeias_transmissao" ...
##  $ origType   : chr  "cadeias" "cadeias" "cadeias" "cadeias" ...
##  $ other      : chr  "cadeias_transmissao" "cadeias_transmissao" "cadeias_transmissao" "cadeias_transmissao" ...
##  $ symptoms   : chr  "" "" "" "" ...
##  $ sex        : chr  "All" "All" "All" "All" ...
##  $ ageGrpLower: chr  "" "" "" "" ...
##  $ ageGrpUpper: chr  "" "" "" "" ...
##  $ ageGrp     : chr  "" "" "" "" ...
##  $ region     : chr  "Portugal" "Portugal" "Portugal" "Portugal" ...
##  $ value      : num  NA NA NA NA NA NA NA NA NA 5 ...
##  $ valueUnits : chr  "" "" "" "" ...
##  - attr(*, ".internal.selfref")=<externalptr>

# Order data by original variable name and date.
setkeyv(CV, c("origVars", "data"))

# Convert data to a data object in dataset and add a change from previous day variable.
CV[, data := as.Date(data, format = "%Y-%m-%d")][
  , dailyChange := value - shift(value, n=1, fill=NA, type="lag"), by = origVars][
    grepl("^sintomas", origVars), dailyChange := NA]
```

### Overall Number of Deaths (daily) by Sex

``` r
library(ggplot2)
library(magrittr)

# Change the ggplot theme.
theme_set(theme_bw())

CV[origType=="obitos" & sex %in% c("F", "M") & ageGrp==""] %>%
  ggplot(aes(x=data, y=dailyChange, fill=as.factor(sex))) +
  geom_bar(stat = "identity") +
  scale_x_date(date_labels = "%b-%Y") +
  theme(legend.position = "bottom") +
  labs(
    title = "COVID-19 Portugal: Number Daily Deaths",
    x = "Date",
    y = "Number of Deaths",
    fill = "Sex")
## Warning: Removed 56 rows containing missing values (position_stack).
```

<img src="README_figs/README-deathsbySex-1.png" width="672" />

### Recorded Number of Confirmed COVID-19 Cases by Region

``` r
CV[origType=="confirmados" & ageGrp=="" & region!="Portugal"] %>%
  ggplot(., aes(x=data, y=value, colour=region)) +
  geom_line() +
  scale_x_date(date_labels = "%b-%Y") +
  scale_y_log10() +
  theme(legend.position = "bottom") +
  labs(
    title = "COVID-19 Portugal: Number of Confirmed Cases",
    x = "Date",
    y = "Number of Confirmed Cases",
    colour = "Region")
## Warning: Transformation introduced infinite values in continuous y-axis
## Warning: Removed 120 row(s) containing missing values (geom_path).
```

<img src="README_figs/README-casesbyRegion-1.png" width="672" />

<hr>

## Issues & Notes

### Use and interpret with care.

The data are provided as is. Any quality issues or errors in the source
data will be reflected in the user friend data.

Please **create an issue** to discuss any errors, issues, requests or
improvements.

### Calculated change between days can be negative (`dailyChange`).

``` r
CV[dailyChange<0 & !(origType %in% c("vigilancia", "internados"))][
  , .(data, origType, origVars, value, dailyChange)]
##            data    origType              origVars value dailyChange
##   1: 2020-03-08     cadeias   cadeias_transmissao     4          -1
##   2: 2020-06-13 confirmados     confirmados_0_9_f   423          -1
##   3: 2020-03-24 confirmados   confirmados_10_19_f    35          -1
##   4: 2020-03-24 confirmados   confirmados_40_49_f   224          -2
##   5: 2020-03-19 confirmados   confirmados_60_69_f    35         -14
##  ---                                                               
## 152: 2020-04-04      obitos    obitos_arsalentejo     0          -1
## 153: 2020-05-23      obitos      obitos_arscentro   230          -3
## 154: 2020-07-03      obitos      obitos_arscentro   248          -1
## 155: 2020-06-20      obitos              obitos_f   768          -1
## 156: 2020-05-21 transmissao transmissao_importada   767          -3
```
