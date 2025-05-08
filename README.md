# Data Visualization Developer Assessment

This short skills assessment involves pulling public data, creating a chart and map, and writing a brief summary of your findings. It should take about an hour to complete—please don’t spend much more than that. We want to be respectful of your time and don’t expect perfection. If you run into any issues, don’t hesitate to reach out!

## The Task

You’ll be analyzing and visualizing digital access data from the U.S. Census Bureau’s 2023 American Community Survey.

## Steps

### 1) Fork this repository and clone it locally

This will allow you to work in your own version of the project and submit your changes via pull request.

### 2) Pull broadband adoption data from the American Community Survey

Use the [`tidycensus`](https://walker-data.com/tidycensus/articles/basic-usage.html) package to pull county-level data on household broadband adoption rates for 2023. The function you'll use is `get_acs()`.

To view available variables, use:

`tidycensus::load_variables(2023, "acs5")`

The relevant table is `B28002`: *Presence and Types of Internet Subscriptions in Household*. We typically use:

- `B28002_007`: Broadband such as cable, fiber optic, or DSL

### 3) Compare rural vs. nonrural broadband adoption

To define which counties are rural, we recommend using our [`ruraldefinitions`](https://github.com/ruralinnovation/ruraldefinitions) package:

`devtools::install_github("ruralinnovation/ruraldefinitions")`

We typically use the Office of Management and Budget's Core-Based Statistical Area (CBSA) definition. You can access the 2020 version using:

`ruraldefinitions::cbsa_2020`

To style your charts, consider using our [`cori.charts`](https://github.com/ruralinnovation/cori.charts) package:

`devtools::install_github("ruralinnovation/cori.charts")`


Key functions:

- `load_fonts()` – Loads Google Fonts (e.g., Lato)
- `theme_cori()` – Applies standardized CORI chart theming
- `theme_cori_horizontal_bars()` – Bar chart-specific styling
- `save_plot()` – Consistently saves and sizes exported images

**Deliverable:** Create a chart comparing rural vs. nonrural broadband adoption in 2023. Save it as a PNG in the `export` subfolder using `save_plot()`.

You can reference our chart [cookbook](https://ruralinnovation.github.io/cori.charts/articles/cookbook.html) for examples.

### 4) Map broadband adoption rates by county

Use `tigris::counties()` to load county geometries and join them with your broadband data. You’ll likely find these functions useful:

- `sf::st_as_sf()` – Ensure joined data is a valid `sf` object
- `tigris::shift_geometry()` – Repositions Alaska and Hawaii
- `sf::st_transform(data, crs = 5070)` – Reprojects to Albers Equal Area (EPSG:5070), ideal for national mapping

**Deliverable:** Create a map of broadband adoption rates by county. Save it as a PNG in the `export` folder. If using `save_plot()`, adjust the `chart_height` argument as needed to size the image appropriately.

### 5) Create a Pull Request

Once complete, open a Pull Request with your changes. In the PR description, include:

- A brief (1–3 sentence) summary of your findings
- Any notable methods or decisions you made

## Misc Notes

- This repo was forked from our standard data project template.
  - R scripts → `R/`
  - Data files → `data/`
  - Exports (charts/maps) → `export/`
- Feel free to comment your code as you see fit.

## Final Tips

Your analysis doesn't need to be complex—we’re looking for clarity, thoughtful use of code, and basic proficiency in data visualization and geospatial analysis in R.

