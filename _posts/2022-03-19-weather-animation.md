---
layout: post
title: "Turning Tables into Animations Using R"
author: "Kevin Tsang"
# categories: journal
# tags: [documentation,sample]
image: weather_viz.jpg
---

## Introduction

Presenting data with temporal and geographic features are often more
engaging through animations and maps. This post will give a walkthrough
on using R to extract the data from public sources, wrangle the data,
and create an animation of a map.

The data we will be using is the [daily weather summaries of
2020](https://digital.nmla.metoffice.gov.uk/SO_72b4d5a3-e5f0-41dc-a31d-1a3c8c4f1f59/)
provided by the [Met Office](https://www.metoffice.gov.uk/) (based in
the UK). The data is stored as tables within a PDF, which we will have
to parse using the `pdftools` package. The exact location (longitude and
latitude) of each [weather
station](https://www.metoffice.gov.uk/research/climate/maps-and-data/uk-synoptic-and-climate-stations)
will be referenced using web-scraping methods from the `rvest` package.
To source the country boundary data and coordinates, we will use the
`rnaturalearth` package, which sources the coordinates from [Natural
Earth](https://www.naturalearthdata.com/).

To manipulate and wrangle the data, we will use the `dplyr` and
`tidyverse` toolkit. For the plots, we will use the `ggplot2`,
`gganimate`, and `sf` package.

    library(tidyverse)

    ## Warning: package 'tidyverse' was built under R version 4.1.3

    ## Warning: package 'ggplot2' was built under R version 4.1.3

    library(pdftools)

    ## Warning: package 'pdftools' was built under R version 4.1.3

    library(stringr)
    library(ggplot2)
    library(lubridate)

    ## Warning: package 'lubridate' was built under R version 4.1.3

    library(janitor)

    ## Warning: package 'janitor' was built under R version 4.1.3

    library(fuzzyjoin)

    ## Warning: package 'fuzzyjoin' was built under R version 4.1.3

    library(gganimate)

    ## Warning: package 'gganimate' was built under R version 4.1.3

    library(robotstxt)
    library(rvest)

    library(sf)

    ## Warning: package 'sf' was built under R version 4.1.3

    library(rgeos)

    ## Warning: package 'rgeos' was built under R version 4.1.3

    library(rnaturalearth)

    ## Warning: package 'rnaturalearth' was built under R version 4.1.3

    library(rnaturalearthdata)

    ## Warning: package 'rnaturalearthdata' was built under R version 4.1.3

    library(rnaturalearthhires)

This post will cover:

-   PDF parsing
-   Web scraping
-   Data wrangling
-   Plotting maps with `sf`
-   Animating plots with `gganimate`

------------------------------------------------------------------------

## Parsing PDF Data

In this example, we will focus on using one month’s weather summary
(January 2020).

First, download the daily weather summary for January 2020,
“DWS\_2020\_01”, file from the [Met Office Digital Archive and Library
website](https://digital.nmla.metoffice.gov.uk/IO_8399517f-6890-44b7-8a48-e2880f78d511/)
and put it in your working directory.

Second, we will extract the information from the tables in the PDF file.
Let’s load the PDF file into R.

    dws_file_path <- "./DWS_2020_01.pdf"

    # import pdf as text
    dws_text <- pdf_text(dws_file_path)

    print(dws_text[7])

    ## [1] "Daily Weather Summary for Wednesday 01 January 2020\n\n\n\nSelected UK readings at (L) 0000 and (R) 1200 UTC\n\n     STATIONS                            0000 UTC                                     1200 UTC\n\nNO        SITE          PRESS    WDIR   WSPD   CLOUD   TEMP   TDEW   PRESS    WDIR   WSPD   CLOUD   TEMP   TDEW\n\n 1       Lerwick        1019.7   WSW     26      7      6.9    5.1   1012.2   WSW     33      7      8.5    7.1\n\n 2        Wick          1023.3   SW      11      0      4.3    0.9   1017.3   SSW     12      8      6.6    3.0\n\n 3      Stornoway       1022.9   SSW     20      7      7.9    5.4   1016.0   SSW     21      8      9.0    7.7\n\n 4   Loch Glascarnoch   1024.9    S      8       0      4.9   -0.9   1018.5    W      14      7      6.5    3.5\n\n 5      Aviemore        1026.6   SW      7       0      2.6   -1.4   1020.0   SSW     7       0      6.4    1.8\n\n 6        Dyce          1026.6   SSE     5       0      2.0   -3.6   1019.5   SSE     4       1      3.6    1.1\n\n 7    Tulloch Bridge    1029.2   SW      8       0      3.3    0.9   1022.6   SSW     9       8      5.7    2.8\n\n 8        Tiree         1027.2   SSW     13      8      7.8    5.8   1020.9    S      21      8      8.3    6.0\n\n 9      Leuchars        1029.0   WSW     9       7      1.5    0.7   1022.9    W      14      7      7.7    4.2\n\n10      Bishopton       1029.9    S      2       8      1.0    0.6   1024.1   SW      9       8      7.0    3.4\n\n11     Machrihanish     1029.0   SSW     7       8      8.3    6.0   1024.3   SSW     11      7      6.5    3.0\n\n12       Boulmer        1030.5    S      7       0      0.1   -0.8   1025.1   SSW     5       0      5.5    3.7\n\n13     Eskdalemuir      1031.5    SE     1       7     -2.5   -2.7   1026.1    S      8       8      4.7    3.9\n\n14      Aldergrove      1029.3    S      4       8      7.0    5.8   1024.7    S      8       8      5.8    3.1\n\n15        Shap          1031.5   ESE     9       8      2.5    2.0   1027.0   SW      12      7      4.7    2.9\n\n16       Leeming        1032.0   SSE     5       9      3.5    3.2   1027.1   SSE     9       0      5.5    3.2\n\n17     Ronaldsway       1029.9   NE      2       7      7.8    5.8   1026.3   SW      9       8      8.9    4.8\n\n18      Bridlington     1032.0   SSW     8       0      3.9    3.0   1027.1    -      0       0      6.0    3.5\n\n19       Crosby         1030.9    SE     5       8      7.4    5.6   1027.9   SSW     7       8      7.0    4.1\n\n20        Valley        1030.1   SSE     2       8      7.3    5.7   1027.4    S      14      5      7.8    4.8\n\n21     Waddington       1032.0   ESE     8       8      4.7    4.4   1028.7    S      10      8      4.3    2.7\n\n22      Shawbury        1030.6    E      4       8      7.1    5.7   1028.5   SSW     7       8      6.8    3.4\n\n23     Weybourne        1032.7   ESE     7       0      3.7    3.0   1029.5    S      5       7      5.2    3.0\n\n24       Bedford        1031.1   ENE     5       8      6.0    5.6   1029.6   SSW     4       8      4.8    2.8\n\n25      Aberporth       1029.8    E      3       8      6.4    4.2   1027.8    S      11      8      6.3    4.6\n\n26     Sennybridge      1030.2   NE      4       2      2.7    2.3   1028.8   SSE     4       8      5.1    4.0\n\n27      Wattisham       1032.3    SE     10      8      6.5    5.4   1029.8   SW      3       8      3.3    3.2\n\n28     Almondsbury      1029.8    E      6       8      6.6    3.9   1029.5    SE     2       8      6.2    6.0\n\n29      Heathrow        1031.0   ESE     6       0      3.2    2.0   1029.7   SSW     3       8      5.9    3.8\n\n30       Manston        1031.6   ESE     13      8      5.4    4.2   1030.0    S      3       8      4.7    3.0\n\n31       Chivenor       1028.8    E      5       8      6.9    3.9   1028.3    E      7       8      7.7    7.1\n\n32        Exeter        1028.9    E      4       8      7.1    4.6   1028.3    E      7       7      8.5    8.1\n\n33    Herstmonceux      1030.8    E      3       4      1.9    1.7   1029.7    E      1       8      5.4    5.0\n\n34        Hurn          1029.7    E      5       8      6.6    4.5   1029.3   ESE     4       8      7.4    6.4\n\n35      Camborne        1028.0    E      7       8      8.7    8.1   1028.3    S      1       7      8.9    7.0\n"

The `dws_text` variable is a list of text strings for each page of the
PDF. To filter out the irrelevant pages of information, we will use
`grepl` to look for the pages with “Selected UK readings”.

    # select tables for UK stations
    weather_tables <- dws_text[grepl("Selected UK readings at ",dws_text)]

    print(paste0("length(weather_tables) = ", length(weather_tables)))

    ## [1] "length(weather_tables) = 31"

This operation should reduce the length of the `dws_text` variable to
31, representing the 31 days in January. We call this shortened list
`weather_tables`, where each element of the list is a string of
information extracted from a weather table about a single day.

In order to transform the string into a table or data frame that we can
easily manipulate, we will define a function `extract_weather_table`.
The function takes the string containing a single weather table and
outputs a data frame with the weather table data. The function carries
out the following operations:

1.  Split the string by the `\n` characters.
2.  Split the strings by at least 2 space characters.
3.  Organise the data into 14 columns (the number of columns in the PDF
    file).
4.  Remove the first 10 rows of extracted data, which are not data of
    the weather table.
5.  Update the column names.
6.  Pivot the table into a long format, which is considered as a tidy
    format for R data wrangling. Meaning the data table will not have
    the information about 0000 UTC and 1200 UTC in separate columns, but
    the 0000 UTC will be in one row and 1200 UTC will be in another row.

<!-- -->

    extract_weather_table <- function(table_list){
      # input: single weather table in list format
      # output: single weather table in data.frame format
      
      # extract table
      weather_table <- unlist(strsplit(table_list, "\n"))
      weather_table <- str_split_fixed(weather_table," {2,}",14)
      weather_table_df <- data.frame(weather_table[11:nrow(weather_table),])

      # update column names
      colnames(weather_table_df) <- c(weather_table[9,1:2],
                                      paste(weather_table[9,3:8], "_0000"),
                                      paste(weather_table[9,9:14], "_1200"))
      weather_table_df <- weather_table_df %>%
        filter(SITE!="")
      
      # pivot and re-organise time
      weather_long <- pivot_longer(weather_table_df, cols = c(-"NO",-"SITE"))
      weather_long$TIME <- str_split_fixed(weather_long$name, "_", 2)[,2]
      weather_long$name <- str_split_fixed(weather_long$name, "_", 2)[,1]
      weather_wide <- pivot_wider(weather_long, 
                                  id_cols = c(NO, SITE,TIME), 
                                  names_from = name, 
                                  values_from = value)
      
      weather_table_df <- weather_wide
      weather_table_df$DATE <- parse_date_time(str_split(weather_table[1],"for ")[[1]][2], "AdbY")
      
      return(weather_table_df)
    }

Let’s see this in action for the 1st January 2020 weather table. Feel
free to compare `weather_table_2020_01_01` with the corresponding
weather table in the PDF.

    weather_table_2020_01_01 <- extract_weather_table(weather_tables[1])

    head(weather_table_2020_01_01)

    ## # A tibble: 6 x 10
    ##   NO    SITE      TIME  `PRESS ` `WDIR ` `WSPD ` `CLOUD ` `TEMP ` `TDEW `
    ##   <chr> <chr>     <chr> <chr>    <chr>   <chr>   <chr>    <chr>   <chr>  
    ## 1 " 1"  Lerwick   0000  1019.7   WSW     26      7        6.9     5.1    
    ## 2 " 1"  Lerwick   1200  1012.2   WSW     33      7        8.5     7.1    
    ## 3 " 2"  Wick      0000  1023.3   SW      11      0        4.3     0.9    
    ## 4 " 2"  Wick      1200  1017.3   SSW     12      8        6.6     3.0    
    ## 5 " 3"  Stornoway 0000  1022.9   SSW     20      7        7.9     5.4    
    ## 6 " 3"  Stornoway 1200  1016.0   SSW     21      8        9.0     7.7    
    ## # ... with 1 more variable: DATE <dttm>

We will now repeat the process for the other days of the month and join
them all into one data frame `month_weather_df`.

    for (table_i in 1:length(weather_tables)) {
      day_weather_df <- extract_weather_table(weather_tables[table_i])
      if (table_i == 1){
        month_weather_df <- day_weather_df
      } else {
        month_weather_df <- rbind(month_weather_df, day_weather_df)
      }
    }

    # clean names (for R format)
    month_weather_df <- month_weather_df %>%
      clean_names()

To save time in the future and not have to parse the PDF again, we can
save the data frame as a csv.

    # optional export
    write_csv(month_weather_df, "met_office_dws_2020_01.csv")

## Web Scraping

Using the name of weather station, we can obtain the exact location
(longitude and latitude) of the station using information from the [Met
Office
website](https://www.metoffice.gov.uk/research/climate/maps-and-data/uk-synoptic-and-climate-stations).
Instead of copy and pasting the information, we can automate the process
using web scraping.

First, check that the Met Office website allows web scraping using the
`robotstxt` package.

    website_path <- "https://www.metoffice.gov.uk/research/climate/maps-and-data/uk-synoptic-and-climate-stations"

    # check website allow bots
    paths_allowed(website_path)

    ##  www.metoffice.gov.uk

    ## [1] TRUE

Now we can scrape the website for the table of locations using the
`rvest` package.

    # scrape page
    page <- read_html(website_path)

    table_text <- page %>%
      html_nodes("tbody tr") %>%
      html_text()

`table_text` is a list where each element is a row of the table. It is a
bit untidy and will need to be organised into 4 columns: station name,
country, location, and station type. We will use a similar method of
parsing the strings in the PDF above.

    # become table
    stations_df <- data.frame(str_split_fixed(table_text, "\n", 4))
    colnames(stations_df) <- c("station_name", "country", "location", "station_type")

    # long lat
    split_location <- str_split_fixed(stations_df$location, ",", 2)
    stations_df$lat <- split_location[,1]
    stations_df$long <- split_location[,2]

    stations_df <- stations_df %>%
      mutate_all(str_replace_all, "[\r\n]" , "") %>%
      select(-location)

    head(stations_df)

    ##                  station_name         country station_type    lat    long
    ## 1            Guernsey Airport Channel Islands    Automatic 49.432  -2.598
    ## 2              Jersey Airport Channel Islands    Automatic 49.208  -2.196
    ## 3         Jersey Gorey Castle Channel Islands       Manual   49.2  -2.017
    ## 4            Jersey St Helier Channel Islands       Manual   49.2    -2.1
    ## 5 Jersey Trinity, States Farm Channel Islands       Manual 49.239  -2.093
    ## 6                   Albemarle         England    Automatic  55.02   -1.88

Checking `stations_df` with the information on the website, we can
confirm that the weather station locations table has been correctly
extracted. We can save this as a csv for future use.

    # optional export
    write_csv(stations_df, "met_office_station_locations.csv")

## Data Wrangling

Here, we will transform and cleanup the data frames.

If you have saved the daily weather summaries and station locations as a
csv, we can load these now. Otherwise, the data frames should already be
in your environment.

    # optional import
    month_weather_df <- read_csv("met_office_dws_2020_01.csv")
    stations_df <- read_csv("met_office_station_locations.csv")

Currently, the `stations_df` data frame contains the locations for Met
Office weather stations across the whole of the UK, not just the ones we
have the daily weather summaries. We will create a shortened data frame
`stations_small_df` that only includes these weather stations.
Furthermore, some of the station names are not exactly paired between
the two data frames, which we will have to tackle.

Using the `unique` function, we make a data frame that includes the
sites included in the daily weather summaries. We remove any duplicates
in the `stations_df` and only include the automatic weather stations.

    stations_small_df <- data.frame(unique(month_weather_df$site))
    colnames(stations_small_df) <- "station_name_dws"

    # remove duplicates
    location_clean_df <- stations_df[!duplicated(stations_df[c("lat","long")]),]

    location_clean_df <- location_clean_df %>%
      filter(station_type == "Automatic")

    # correct weather station name
    location_clean_df[location_clean_df$station_name == "Filton",]$station_name <- "Filton and Almondsbury"

Now we attach the longitudes and latitudes of the weather stations to
the `stations_small_df` data frame. Since not all station names have an
exact match between the two sources, we will use `fuzzyjoin` to join the
two data frames.

    # join coordinates
    stations_small_df <- regex_right_join(
      location_clean_df, 
      stations_small_df, 
      by = c(station_name="station_name_dws"))

    stations_small_df <- stations_small_df %>% select(-"station_name")
    colnames(stations_small_df)[5] <- "station_name"
    # colnames(stations_small_df)[grepl("station_name",colnames(stations_small_df))] <- "station_name"

    # reorder columns
    stations_small_df <- stations_small_df[,c(5,1,2,3,4)]

We can save the smaller, cleaned locations data frame for future use.

    # optional export
    write_csv(stations_small_df, "met_office_station_locations_small.csv")

## Plotting

Using the `rnaturalearth` package, we will extract the UK’s geometries
with the `ne_states` function.

    # UK geometries
    uk_sf <- ne_states(country = "united kingdom", returnclass = "sf")
    isle_of_man_sf <- ne_states(country = "isle of man", returnclass = "sf")

    uk_sf <- rbind(uk_sf,isle_of_man_sf)

NOTE: At the time of writing, the Wales regions of “West Wales and the
Valleys” and “East Wales” are the wrong way around.

    # correct east wales and west wales
    uk_sf$region <- plyr::mapvalues(
      uk_sf$region,
      from = c("West Wales and the Valleys",
               "East Wales"),
      to = c("East Wales",
             "West Wales and the Valleys")
    )

Using the `sf` package, we will transform the longitudes and latitudes
of the weather stations from the `stations_df` data frame into `sf`
format.

    stations_geom.df <- stations_small_df %>%
      st_as_sf(coords = c("long", "lat"), crs = 4326) %>%
      st_set_crs(4326)

Here, we will plot the locations of the weather stations on the map of
the UK.

    uk_stations_plot <- ggplot() + 
      geom_sf(data = uk_sf,
              aes(fill = region)) +
      geom_sf(data = stations_geom.df,
              color = "red", size = 2) +
      guides(fill=guide_legend(ncol=2)) +
      labs(
        title = "UK Weather Stations"
      )
    uk_stations_plot

<img src="weather_animation_files/figure-markdown_strict/unnamed-chunk-19-1.png" width="80%" />

### Cloud Coverage

As an example, we will display the daily cloud coverage across the UK.

The wrangling will look up the cloud coverage of each county at the
closest site (according the the middle point of each county).

First determine the closest weather station using the `st_distance`
function, which calculate the distance between coordinates.

    county_sf <- uk_sf

    closest <- list()
    for(i in 1:nrow(county_sf)){
      cent <- st_centroid(county_sf$geometry[i])
      closest[[i]] <- stations_geom.df$station_name[which.min(
        st_distance(cent, stations_geom.df$geometry))]
    }

    county_sf$closest_site <- unlist(closest)

Then, we will use `left_join` to look up the weather at each of the
coordinates for each day. The `day=day(date)` will set a new column
`day` that is the day of the month.

    weather_df <- left_join(month_weather_df, stations_small_df, by = c("site" = "station_name"))

    # average cloud cover per day
    weather_county_df <- weather_df %>%
      group_by(site, day=day(date)) %>%
      summarise(cloud = mean(as.numeric(cloud), na.rm=T))

    ## Warning: One or more parsing issues, see `problems()` for details

    ## `summarise()` has grouped output by 'site'. You can override using the `.groups` argument.

    weather_county_df <- full_join(county_sf[,c("closest_site","geometry")],
                                   weather_county_df,
                                   by = c("closest_site" = "site"))

We can represent the cloud coverage on the 1st January 2020 using a
varying transparency level (`alpha`) depending on cloud coverage.

    day_cloud_plot <- ggplot() + 
      geom_sf(data = filter(weather_county_df, day==1), 
              aes(alpha = cloud/9), 
              fill = "#5090aa",
              color = NA) +
      labs(title = "Cloud Coverage on 1 Jan 2020",
           caption = "Source: Met Office",
           alpha = "Cloud") +
      theme_void() +
      theme(plot.title = element_text(hjust = 0.5,
                                      size=22),
            plot.caption = element_text(size = 15),
            legend.position = "none")
    day_cloud_plot

![](weather_animation_files/figure-markdown_strict/unnamed-chunk-22-1.png)

Now we animate the cloud coverage in the UK for each day of January
2020, this is done using `transition_manual(day)` which sets a frame of
the animation based on the data segemented by `day` (the day of the
month).

    county_plot <- ggplot() + 
      geom_sf(data = weather_county_df, 
              aes(alpha = cloud/9), 
              fill = "#5090aa",
              color = NA) +
      labs(title = "Cloud Coverage on {current_frame} Jan 2020",
           caption = "Source: Met Office",
           alpha = "Cloud") +
      theme_void() +
      theme(plot.title = element_text(hjust = 0.5,
                                      size=22),
            plot.caption = element_text(size = 15),
            legend.position = "none") +
      transition_manual(day)

    animate(county_plot, fps = 4)

    ## nframes and fps adjusted to match transition

![](weather_animation_files/figure-markdown_strict/unnamed-chunk-23-1.gif)

We can save the plots and animations using `ggsave` and `anisave`.

    ggsave("day_cloud.png",day_cloud_plot)
    anim_save("Jan_cloud.gif", animate(county_plot, fps = 4))

## Summary

You have learnt to read and parse a PDF that includes tabled data,
scrape information from a website, transform the data and strings into
useful information, and display that information via an animated map. We
turned tables of numbers into an animation of the cloud coverage in the
UK over January 2020.

Thank you for reading!

------------------------------------------------------------------------

## About Author

[Kevin Tsang](https://github.com/kevinchtsang) is a final year PhD
student at University of Edinburgh applying machine learning, data
science, mHealth, and mathematics to asthma attack prediction.
