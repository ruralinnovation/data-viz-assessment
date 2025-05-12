```R
library(ggplot2)
library(tidycensus)
library(tidyverse)
library(tigris)
library(sf)
library(dplyr)
library(viridis)
options(tigris_use_cache = TRUE)
# Being a cybersecure person and importing my API key from an ignored file
census_api_key(readChar("api_key.txt",file.info("api_key.txt")$size))
```

    ── [1mAttaching core tidyverse packages[22m ─────────────────────────────────────────────────────────── tidyverse 2.0.0 ──
    [32m✔[39m [34mdplyr    [39m 1.1.4     [32m✔[39m [34mreadr    [39m 2.1.5
    [32m✔[39m [34mforcats  [39m 1.0.0     [32m✔[39m [34mstringr  [39m 1.5.1
    [32m✔[39m [34mlubridate[39m 1.9.4     [32m✔[39m [34mtibble   [39m 3.2.1
    [32m✔[39m [34mpurrr    [39m 1.0.4     [32m✔[39m [34mtidyr    [39m 1.3.1
    ── [1mConflicts[22m ───────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    [31m✖[39m [34mdplyr[39m::[32mfilter()[39m masks [34mstats[39m::filter()
    [31m✖[39m [34mdplyr[39m::[32mlag()[39m    masks [34mstats[39m::lag()
    [36mℹ[39m Use the conflicted package ([3m[34m<http://conflicted.r-lib.org/>[39m[23m) to force all conflicts to become errors
    To enable caching of data, set `options(tigris_use_cache = TRUE)`
    in your R script or .Rprofile.
    
    Linking to GEOS 3.13.0, GDAL 3.10.0, PROJ 9.5.1; sf_use_s2() is TRUE
    
    Loading required package: viridisLite
    
    To install your API key for use in future sessions, run this function with `install = TRUE`.
    



```R
library(devtools)
# Bringing in the plotting style
devtools::install_github("ruralinnovation/cori.charts")
library(cori.charts)
cori.charts::load_fonts() 
```

    Loading required package: usethis
    
    Skipping install of 'cori.charts' from a github remote, the SHA1 (8a1f5d6c) has not changed since last install.
      Use `force = TRUE` to force installation
    


## Importing and processing the data


```R
# Definitions of rural/nonrural can be fuzzy, we'll use CBSA
rural_defs <- read_csv("../data/cbsa_2020.csv")
rural_defs$GEOID = rural_defs$geoid # creating an uppercase for merging consistency
# B28002_007 = "Has a broadband internet subscription"
# B28002_001 = "Total households"
# "geometry = TRUE" pulls in the county shapes from tigris, allowing us to skip an additional merge.
# county <- get_acs(geography = "county", variables = c("B28002_001","B28002_007"), geometry = TRUE, year = 2023)

# county <- county %>% shift_geometry()
# county <- st_transform(county, crs=5070)
# st_write(county,'../data/counties-broadband-shape.shp',append=FALSE)
county <- st_read('../data/counties-broadband-shape.shp')
# county %>% filter(grepl('Conn', NAME))
```

    [1mRows: [22m[34m3234[39m [1mColumns: [22m[34m5[39m
    [36m──[39m [1mColumn specification[22m [36m───────────────────────────────────────────────────────────────────────────────────────────[39m
    [1mDelimiter:[22m ","
    [31mchr[39m (4): geoid, name, rural_def, is_rural
    [32mdbl[39m (1): year
    
    [36mℹ[39m Use `spec()` to retrieve the full column specification for this data.
    [36mℹ[39m Specify the column types or set `show_col_types = FALSE` to quiet this message.


    Reading layer `counties-broadband-shape' from data source 
      `/Users/benemery/cori/assessment/data/counties-broadband-shape.shp' 
      using driver `ESRI Shapefile'
    Simple feature collection with 6444 features and 5 fields
    Geometry type: MULTIPOLYGON
    Dimension:     XY
    Bounding box:  xmin: -3111747 ymin: -90959.31 xmax: 2258200 ymax: 3172568
    Projected CRS: NAD83 / Conus Albers



```R
# Loading multiple variables automatically formats them as long-form, creating multiple rows
# for each county. We pivot the data frame to wide-form so each county gets one row, and
# each variable has its own column.
county <- county %>%
  pivot_wider(names_from = variable, values_from = c(estimate,moe),values_fill = 0)


# Renaming columns for humans who don't memorize serial numbers.
colnames(county)[4] <- "households_est"
colnames(county)[5] <- "broadband_est"
colnames(county)[6] <- "households_moe"
colnames(county)[7] <- "broadband_moe"
# Create a new column that contains the amount of broadband access 
# expressed as a fraction of the total number of households.
county$broadband_frac = county$broadband_est/county$households_est


# The rural definitions seem to have Connecticut missing, so we'll need to rename the 
# merged df, and access the original later.
county_rurality <- merge(county, rural_defs[c('is_rural','GEOID')], by='GEOID')
```

## Generating plots
### 1. A simple plot comparing the two county categories


```R
# f<-ggplot(county, aes(is_rural,broadband_frac, fill=is_rural)) + #Uncomment to save figure
ggplot(county_rurality, aes(is_rural,broadband_frac, fill=is_rural)) +
    geom_violin(trim=FALSE)+
    geom_boxplot(width=0.1,outliers=FALSE)+
    theme_cori()+
    theme(legend.position="none")+
    scale_fill_cori(palette = "ctg2tlpu", reverse = TRUE)+
    labs(title = "Fraction of households with broadband access",
       subtitle = "By rurality",
       x = NULL,
       y = NULL,
       caption = "Source: US Census ACS")
# save_plot(f,"../export/broadband-violins.png",chart_width=6,chart_height=7,add_logo=FALSE) #Uncomment to save figure

```


    
![png](output_6_0.png)
    


As one would reasonably expect, rural counties, on average, have less broadband access. While the distributions do overlap substantially, it's notable that the rural median is close to the nonrural first quartile. The nonrural median fraction appears to by around 0.7, while the rural median fraction is closer to 0.6. By virtue of the distributional center being closer to the midpoint of the physical bounds, the rural distribution is closer to symmetric, while they both have some skew, with the longer tail extending into the lower values.

### A map of all the counties and their fractional broadband access
The following cell has all the lines commented out, because internet problems were preventing me from repeatedly running the `st_transform()` function. I instead exported the transformed shapefile after getting it to work and simply loaded it locally any time it was subsequently necessary.


```R
# The geometry is already there for us from the geometry = True tag earlier.

# But, we still need to rearrange the noncontiguous parts and specify the projection.
# county <- county %>% shift_geometry()
# county <- st_transform(county, crs=5070)
# st_write(county,'../data/counties-broadband-shape.shp',append=FALSE)
```


```R
# f <- ggplot(data=county)+
ggplot(data=county)+
      geom_sf(data = county, fill = NA, color = "gray80", size = 0.1) +
      geom_sf(
        data = filter(county), #is_rural == "Rural"), #If only plotting rural counties
        aes(fill = broadband_frac),
        color = NA
      ) +
    scale_fill_viridis(name = "fraction with access", na.value = "grey90") +
    labs(title = "Fraction of households with broadband",
        subtitle = "By county",
        caption = "Source: US Census ACS")+
    theme_cori_map()
# save_plot(f,"../export/broadband-2.png",chart_width=9,chart_height=7)
```


    
![png](output_10_0.png)
    


This is a rich visualization with more to be learned than I can process in this short time. The pieces that I notice will surely be a reflection of my personal and intellectual interests.

First, the northeast stands out as a region with anomolously extensive broadband access, especially in the band spanning from Massachussetts to DC. There's also a noticeable cluster of low access around Arkansas, Louisiana, Mississippi, and Alabama. Predictably, large portions of northern Alaska have virtually nonexistent broadband access. Access is also notably low in Puerto Rico, southern Virginia, the dividing line between Oregon/Idaho and California/Nevada, and on the northern half of the Arizona - New Mexico border. 

It's hard to avoid the overlap between these low-access areas and historical (in some cases, ongoing) oppression. The deep south continued relied on slavery for longer than other parts of the union, and implemented a series of legislation to prevent the distribution of rights and freedoms to Black Americans after emancipation. The eastern border of Arizona is contained by some of the most notoriously dangerous parts of Navajo Nation, where violence against indigenous women goes systemically unaddressed by the federal government. Puerto Rico, the site of historical toxic weapons testing and attempts at population control, is regularly referred to as a "colony" by its residents. Part of this historical throughline of subjugation is the deprioritization of these areas for the development of modern infrastructure. 


```R
library(rmarkdown)
convert_ipynb('county_broadband.ipynb','county_broadband.rmd')
```


```R
rmarkdown::render("county_broadband.rmd", "html_document")
```

    
    
    processing file: county_broadband.rmd
    


    1/19                  
    2/19 [unnamed-chunk-1]
    3/19                  
    4/19 [unnamed-chunk-2]
    5/19                  
    6/19 [unnamed-chunk-3]
    7/19                  
    8/19 [unnamed-chunk-4]
    9/19                  
    10/19 [unnamed-chunk-5]
    11/19                  
    12/19 [unnamed-chunk-6]
    13/19                  
    14/19 [unnamed-chunk-7]
    15/19                  
    16/19 [unnamed-chunk-8]
    17/19                  
    18/19 [unnamed-chunk-9]



```R

```
