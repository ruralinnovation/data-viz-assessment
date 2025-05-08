# Data visualization developer assessment

This short skills assessment involves pulling public data, creating a chart + map, and writing a brief summary of your findings. It should take about an hour to complete, please don't spend much more than that. We want to be respectful of your time and don't expect perfection. If you run into any issues, don't hesitate to reach out!

## The task

For this assessment, you're going to be analyzing and visualizing broadband adoption data from the US Census Bureau's 2023 American Community Survey. 

## Steps

**1) Fork this repository and clone it to your local computer**

If you’d prefer to keep your assessment private, feel free to clone this repo, create a private repository on GitHub, push your work there, and add us as collaborators when you're ready.  

**2) Pull American Community Survey Data**

Use the [tidycensus](https://walker-data.com/tidycensus/articles/basic-usage.html) package (and, in particular, the `get_acs` function) to pull county data on household broadband adoption rates for 2023. You'll need to sign up for a [Census API Key](http://api.census.gov/data/key_signup.html). You can view available variables by calling `tidycensus::load_variables(2023, "acs5")`. The relevant table for adoption rates is table `B28002`: *Presence and Types of Internet Subscriptions in Household*. At CORI, we use variable `B28002_007` (Broadband such as cable, fiber optic or DSL) to identify whether a household has a broadband subscription.

**3) Visualize rural vs nonrural broadband adoption rates**

To determine which counties are rural, you'll need to employ a rural definition. CORI's [`ruraldefinitions`](https://github.com/ruralinnovation/ruraldefinitions) package is a helpful resource. Here's how to download it: 

```r
# install.packages("devtools")
devtools::install_github("ruralinnovation/ruraldefinitions")
```

We typically use the Office of Management and Budget's definition of Core-Based Statistical Areas when working with county-level data. You can access the 2020 version of this definition using `ruraldefinitions::cbsa_2020`.

Another helpful package for visualization is our theming library [cori.charts](https://github.com/ruralinnovation/cori.charts/). 

```r
# install.packages("devtools")
devtools::install_github("ruralinnovation/cori.charts")
```

[`cori.charts`](https://github.com/ruralinnovation/cori.charts) contains several useful functions for loading fonts, implementing consistent styling, and exporting graphics. Here's a few useful ones:

- `load_fonts` loads relevant Google Fonts (Lato) into your local environment.
- `theme_cori` applies standardized formatting (consistent margins, fonts, etc.)
- `theme_cori_horizontal_bars` extends theme_cori with some horizontal bar specific styling.
- `theme_cori_map` extends theme_cori with some map specific styling.
- `save_plot` exports images with a CORI logo and consistent formatting.

Produce a chart that compares rural and nonrural broadband adoption in 2023. Simple is OK! Feel free to reference our ["cookbook"](https://ruralinnovation.github.io/cori.charts/articles/cookbook.html) for chart examples. Save your chart as a PNG in the export subfolder using `cori.charts::save_plot`.

**4) Map broadband adoption rates by county**

Next, let’s map how broadband adoption rates vary by county. Start by using the `tigris::counties()` function to load county geometries, and then join this spatial data with your broadband adoption data.

When preparing spatial data for mapping in R, the following functions are often helpful:

- `sf::st_as_sf()` – Converts a data frame to an `sf` (simple features) object if it's not already in that format.
- `tigris::shift_geometry()` – Rescales and repositions Alaska and Hawaii.
- `sf::st_transform(data, crs = 5070)` – Transforms the coordinate reference system to Albers Equal Area Conic (EPSG:5070), our preference for US choropleth maps.


The data should now be ready for mapping! Produce a map using geom_sf and save a PNG file to the `export` subfolder. If using `cori.charts::save_plot`, sizing maps can be a bit finicky and you may need to play around with the `chart_height` parameter to get something that looks good.


**5) Create a Pull Request with your changes**

In the description of the pull request, write a brief (1-3 sentence) summary of what you found in the data and any methods worth noting.


## Misc notes

- This repository is based on our standard data project template. By convention:
  - R scripts go in the `R/` folder
  - Exported images (e.g., charts, maps) go in the `export/` folder
  - Data files (e.g., CSVs, Parquet files) go in the `data/` folder
- Please comment your code where it helps explain your process or decisions.

