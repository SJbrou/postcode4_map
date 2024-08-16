# Draw Postcode4 Map

This repository contains an R script for visualizing postcode (4) maps using the `leaflet` package. The script integrates geographical data with numerical values to create interactive maps.

## Overview

The code provides a comprehensive approach to:
1. **Install and Load Dependencies**: Ensures necessary R packages are available.
2. **Preprocess GeoJSON Data**: Simplifies and stores geographical data in an `.RData` file for efficiency.
3. **Merge Data**: Combines geographical data with numerical values from an Excel file.
4. **Plot the Data**: Uses `leaflet` to generate interactive maps based on the data.

## Instructions

1. **Install and Load Dependencies**

   ```r
   # Clear environment and load dependencies
   rm(list = ls())
   if (!require("openxlsx")) install.packages("openxlsx")
   library(openxlsx)
   if (!require("dplyr")) install.packages("dplyr")
   library(dplyr)
   if (!require("sf")) install.packages("sf")
   library(sf)
   if (!require("leaflet")) install.packages("leaflet")
   library(leaflet)
   ```
   
2. **Preprocess GeoJSON Data (optional)**

  ```r
  # geojson_pc4 <- st_read("data/postcode4.geojson") 
  # geojson_pc4 <- st_simplify(geojson_pc4, dTolerance = 2) 
  # geojson_pc4$pc4_code <- as.numeric(geojson_pc4$pc4_code) 
  # save(geojson_pc4, file = "data/postcode4.RData") 
  ```
  
3. **Merge Data**

  ```r
  load("data/postcode4.RData")
  data <- read.xlsx("data/data.xlsx")
  data <- merge(geojson_pc4, data, by.x = "pc4_code", by.y = "Postcode", all.x = TRUE)
  data <- data %>% rename(Postcode = pc4_code)
  rm(geojson_pc4)
  ```
  
4. **Plot the Data**

  ```r
  toplot <- names(st_drop_geometry(data[, 8])) 
  pal <- colorNumeric(palette = "viridis", domain = range(data[[toplot]], na.rm = TRUE), na.color = "transparent")

  leaflet(data = data) %>%
    addProviderTiles(providers$OpenStreetMap) %>%
    addPolygons(
      fillColor = ~pal(data[[toplot]]),
      weight = 1,
      opacity = 1,
      color = "black",
      dashArray = "3",
      fillOpacity = 0.5,
      popup = ~paste0(
        "Postcode: ", data$Postcode, "<br>",
        "Gemeente: ", data$gem_name, "<br>",
        toplot, ": ", data[[toplot]]
      )
    ) %>%
    addLegend(
      pal = pal,
      values = data[[toplot]],
      title = toplot,
      position = "bottomright"
    ) %>%
    setView(
      lng = mean(st_coordinates(data)[, 1]),
      lat = mean(st_coordinates(data)[, 2]),
      zoom = 10
    )
  ```
  
5. **Function for Plotting**

  ```r
  draw_postcode4_map <- function(postcode4_path, data_path, column_to_plot) {
    # Function to draw postcode4 maps. 
    # Requires a RData containing the processed geojson data at postcode4_path
    # Requires the excel to have a column "Postcode" containing the postcode4 codes and corresponding values in column "column_to_plot"
    if (!require("readxl")) install.packages("readxl")
    if (!require("dplyr")) install.packages("dplyr")
    if (!require("sf")) install.packages("sf")
    if (!require("leaflet")) install.packages("leaflet")
    library(readxl)
    library(dplyr)
    library(sf)
    library(leaflet)
    
    # Load data
    load(postcode4_path)
    data <- read_excel(data_path)
    
    # Merge the shapefile data with the Excel data
    data <- merge(geojson_pc4, data, by.x = "pc4_code", by.y = "Postcode", all.x = TRUE)
    
    # Rename for clarity
    data <- data %>%
      rename(Postcode = pc4_code)
    
    # Define the color palette based on the selected column
    pal <- colorNumeric(palette = "viridis", domain = range(data[[column_to_plot]], na.rm = TRUE), na.color = "transparent")
    
    # Create the map
    map <- leaflet(data = data) %>%
      addProviderTiles(providers$OpenStreetMap) %>%
      addPolygons(
        fillColor = ~pal(data[[column_to_plot]]),
        weight = 1,
        opacity = 1,
        color = "black",
        dashArray = "3",
        fillOpacity = 0.5,
        popup = ~paste0(
          "Postcode: ", data$Postcode, "<br>",
          "Gemeente: ", data$gem_name, "<br>",
          column_to_plot, ": ", data[[column_to_plot]]
          )
      ) %>%
      addLegend(
        pal = pal,
        values = data[[column_to_plot]],
        title = column_to_plot,
        position = "bottomright"
      ) %>%
      setView(
        lng = mean(st_coordinates(data)[, 1]),
        lat = mean(st_coordinates(data)[, 2]),
        zoom = 10
      )
    
    # Return the map object
    return(map)
  }
  
  # Example usage:
  postcode4_map <- draw_postcode4_map("data/postcode4.RData", "data/data.xlsx", "population")
  postcode4_map  # This will display the map
  Note: Sample data is generated randomly and does not reflect real-world statistics.
  ```

**More Information**
  For more details and the complete code, please visit the <a href=https://sjbrou.github.io/postcode4_map/>GitHub repository</a>.