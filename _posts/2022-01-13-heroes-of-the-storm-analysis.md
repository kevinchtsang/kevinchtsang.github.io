---
layout: post
title: "Heroes of the Storm Analysis Tool"
author: "Kevin Tsang"
# categories: journal
tags: [project]
image: heroes-of-the-storm-analysis.png
---

# Heroes of the Storm Analysis
Heroes of the Storm replay parsing and analysis, to take a deeper look at your games and make decisions backed by data. See where you are getting kills, who you are getting kills with, where you are getting ganked, and more. 

This pipeline is setup to process multiple games for data analysis, try to include as many relivant games for each map as possible. A data exploration tool (Shiny app) has been included to aid your analysis. 

Good luck; have fun

## Getting Started
You will need to put your replays in a folder. If you don't have any recent replays, [Heroes Lounge](https://heroeslounge.gg/) hosts amature HotS matches and individual match replays can be downloaded.

### Parsing Replays
First install the [heroprotocol](https://github.com/Blizzard/heroprotocol).
```
python -m pip install --upgrade heroprotocol
```

Or clone the repository
```
git clone https://github.com/Blizzard/heroprotocol.git
python -m pip install -r ./heroprotocol/heroprotocol/requirements.txt
```

Then use the [`HotS replays to dataframe.ipynb`](HotS%20replays%20to%20dataframe.ipynb) notebook to parse your replays and combine them into a single dataframe `hero_died_all.csv`. This can be analysed in any method of your choice.

[`HotS Analyse Multiple Games.ipynb`](HotS%20Analyse%20Multiple%20Games.ipynb) notebook is used to manually calibrate the coordinates of the parsed replays to the official Blizzard map overlay. It can also be a place to conduct data analysis using python.

[`shiny_hero_deaths.R`](shiny_hero_deaths.R) is a Shiny app to explore the data interactively.

### Data Exploration Interface
The plot is built using plotly which you can zoom and pan around the map. Hovering over a data point will provide more information.

All the filters are provided on the left, where you can choose the map and any players to focus on. Game phase can be selected to highlight the kills that happend during different phase of the game based on the levels of each team.

After choosing a player, you can choose between kills (close to the kill, not necessary last hit), deaths, both, or all data from their games (played_game)

If a map or player is missing, it means there is no available data for your choosen selection. Check the replays you have parsed.

Under Advanced options, you can change the bin width of the heatmap, change plot type between a heatmap (summary) and scatter plot (every point), and choose colour scale that best displays the data on different maps.

Additional information can be extracted using dplyr in the R file or in the `HotS replays to dataframe.ipynb` python file before the CSV is exported.