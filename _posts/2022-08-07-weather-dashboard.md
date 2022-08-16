---
layout: post
title: "Weather Dashboard (Power BI)"
author: "Kevin Tsang"
# categories: journal
# tags: [documentation,R,Shiny]
image: dashboard-thumbnail.png
---

## Introduction
Power BI is a data visualisation tool. Here I produced a dashboard using the [daily weather summaries of 2021](https://digital.nmla.metoffice.gov.uk/SO_ad3e3b06-cbf4-4302-91e6-6195761050bd/) provided by the [Met Office](https://www.metoffice.gov.uk/) (based in the UK). By clicking on the different sites across the UK, you can see the weather as measured at the station.

![app screenshot](assets/img/weather_dashboard_heathrow.png)

![app demo](assests/img/powerbi_weather_dashboard.gif)

## Data Processing
There are two tables involved: one with the daily weather summaries and one with the locations of the Met Office stations. Refer to my [previous post](https://kevinchtsang.github.io/weather-animation/) for the process using R. The two tables are linked via `station_name` to `SITE`. This allows the interactive map to filter the data in the surrounding visuals.

The measure used to write the temperature card was as follows:

```
Max_Temp_Today = CALCULATE (
    FORMAT(MAX(met_office_weather[TEMP]), "0.0" & UNICHAR(176) & "C"),
    CALCULATETABLE (
        VALUES(met_office_weather[DATE]),
        met_office_weather[DATE]= MAX(met_office_weather[DATE])
    )
)
```

This filters the date to today's date, then calculates the maximum daily temperature, follow by Celsius formating.