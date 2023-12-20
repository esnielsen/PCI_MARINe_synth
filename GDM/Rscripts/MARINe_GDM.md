MARINe_GDM
================
Erica Nielsen
2023-12-19

## Generalized Dissimarity Models Analyses for PCI

This markdown takes biodiversity data from MARINe rocky shore long term
monitoring & satellite environmental data to run GDMs to map community
structure for the present dat & 2050.

``` r
library(readxl)

#read point contact
pt_cont <- read_excel("C:/Users/erica.nielsen/Desktop/Synz/MARINe/large_biodiv_data/cbs_data_CA_2023.xlsx", 
    sheet = "point_contact_summary_data")

#read quadrat summary
quad <- read_excel("C:/Users/erica.nielsen/Desktop/Synz/MARINe/large_biodiv_data/cbs_data_CA_2023.xlsx", 
    sheet = "quadrat_summary_data")

#read swath data
swath <- read_excel("C:/Users/erica.nielsen/Desktop/Synz/MARINe/large_biodiv_data/cbs_data_CA_2023.xlsx", 
    sheet = "swath_summary_data")
```

The input data to make ‘site-pair’ tables for GDMs -\> Rows = sites,
Columns = spp abundance. To do this we need to do the following:

-combine pt_cont, quad, & swath tables into single table… problem -\>
quad & swath are density/m2 and pt_cont is % cov - we are just comparing
sites, so if %cov and density

``` r
#see if there's overlapping spp between three tables 
intersect(intersect(pt_cont$species_lump, quad$species_lump), swath$species_lump)
```

    ## character(0)

``` r
character(0) # so there's no species overlap between tables
```

    ## character(0)

The next problem is that we need one ‘spp_abund’ count for the present
day and we have multiple abundances per species (per year)– will try and
average spp abundaces over the years that the environmental data is
available for (I checked the air temp data from MARINe but it doesn’t
exist for all biodiv sites so will need to use other data)

- WorldClim Version 2 = 1970-2000, so not the best temporal range to
  match data –\> looking into air temp for same range, and this seems
  like a good option:

  <https://climatedataguide.ucar.edu/climate-data/airs-atmospheric-infrared-sounder-version-6-level-2>
  download here:
  <https://disc.gsfc.nasa.gov/datasets?page=1&source=AQUA%20AIRS,AQUA%20AMSU-A,AQUA%20HSB>
  date range=2002-09-01 to 2014-09-01 spatial range= xmin= -126, xmax=
  -116, ymin = 32, ymax= 42

- Bio-Oracle version 2 = 2000-2014

<!-- -->

    ## Warning: package 'dplyr' was built under R version 4.3.2

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    ## Warning: package 'ggplot2' was built under R version 4.3.2

![](MARINe_GDM_files/figure-gfm/filter%20years%20&%20check%20#%20years%20sampled-1.png)<!-- -->![](MARINe_GDM_files/figure-gfm/filter%20years%20&%20check%20#%20years%20sampled-2.png)<!-- -->![](MARINe_GDM_files/figure-gfm/filter%20years%20&%20check%20#%20years%20sampled-3.png)<!-- -->

-Looks like doing \>3 years per site would yield very low sample sizes…
think that we can try comparing 1 vs. 2 years sampling and see if
there’s difference

- What I could also do is not filter the data to 2000-2014, then apply
  3+ year filter and see if outputs differ (only thing then is that the
  climate data misses 2015-2023)

<!-- -->

    ## `summarise()` has grouped output by 'marine_site_name', 'latitude',
    ## 'longitude'. You can override using the `.groups` argument.
    ## `summarise()` has grouped output by 'marine_site_name', 'latitude',
    ## 'longitude'. You can override using the `.groups` argument.
    ## `summarise()` has grouped output by 'marine_site_name', 'latitude',
    ## 'longitude'. You can override using the `.groups` argument.

So now we have a ‘x-y species list’ dataframe (which is bioFormat=2 for
GDM, i.e. spp are rows and not columns)

Last step before we can make a ‘site-pair’ table is to get the
environmental predictor variables

``` r
library(sdmpredictors)
```

    ## Warning: package 'sdmpredictors' was built under R version 4.3.2
