# Data visualization developer assessment

This short skills assessment involves pulling public data, creating a chart, and writing a brief summary of your findings. It should take about an hour to complete. Please don't spend much more than this. We understand the time constraint and don't expect perfection. If you run into any issues, please don't hesitate to reach out!

## The task

For this assessment, you're going to be analyzing and visualizing digital access data from the US Census Bureau's 2023 American Community Survey. 

## Steps

**1) Fork this repository and clone it to your local computer**

**2) Pull American Community Survey Data**

Use the [`tidycensus`](https://walker-data.com/tidycensus/articles/basic-usage.html) package (and, in particular, the `get_acs` function) to pull county data on household broadband adoption rates for 2023. You can view available variables by calling `tidycensus::load_variables(2023, "acs5")`. The relevant table for adoption rates is table `B28002`: Presence and Types of Internet Subscriptions in Household. At CORI, we use variable `B28002_007` (Broadband such as cable, fiber optic or DSL) to identify whether a household has a broadband subscription.

**3) Visualize rural vs nonrural broadband adoption rates**

To determine which counties are rural, you'll need to employ a rural definition. CORI's `ruraldefinitions` package is a helpful resource. Here's how to download it: 

```r
# install.packages("devtools")
devtools::install_github("ruralinnovation/ruraldefinitions")
```

We typically use the Office of Management and Budget's Core-Based Statistical Area's definition when working with county-level data. You can access the 2020 version of definition like this `ruraldefinitions:cbsa_2020`.

Another useful package when visualizing data is our theming library [`cori.charts`](https://github.com/ruralinnovation/cori.charts/). 

```r
# install.packages("devtools")
devtools::install_github("ruralinnovation/cori.charts")
```

`cori.charts` contains several useful functions for loading fonts, implementing consistent styling, and exporting graphics. Here's a few useful ones:

`load_fonts` loads relevant Google Fonts (Lato) into your local environment.
`theme_cori` applies standardized formatting (consistent margins, fonts, etc.)
`theme_cori_horizontal_bars` extends `theme_cori` with some horizontal bar specific styling.
`theme_cori_map` extends `theme_cori` with some map specific styling.

Produce a chart that compares rural and nonrural broadband adoption in 2023. Simple is OK! Feel free to reference our ["cookbook"](https://ruralinnovation.github.io/cori.charts/articles/cookbook.html) for chart examples. Save your chart as a png in the `export` subfolder using `cori.charts::save_plot`.

**4) Map broadband adoption rates by county**

Next up, let's map how broadband adoption rates vary by county. To do so, we'll need to need to load county geometries using the `tigris::counties` function and join it with our broadband adoption data. When mapping data in R, I often use three functions. First, when joining spatial data with non-spatial data, you may need to convert the derived output to an sf object with `sf::st_as_sf`. Second, when mapping the USA, it's helpful to use `tigris::shift_geometry` to rescale Hawaii and Alaska. Third, depending on the data, it may be necessary to transform the projected coordinate system. I recommend using the Albers Equal Area Conic projection, which can be done with `st_transform(data, crs = 5070)`. 

The data should now be ready for mapping! Produce a map using `geom_sf` and save a png file to the `export` subfolder. If using `save_plot`, sizing maps can be a bit finicky and you may need to play around with the `chart_height` parameter to get something that looks good.


**5) Create a Pull Request with your changes**

In the description of the pull request, write a brief (1-3 sentence) summary of what you found in the data and any methods worth noting.

