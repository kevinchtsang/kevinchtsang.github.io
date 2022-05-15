---
layout: post
title: "Wordle Assistant"
author: "Kevin Tsang"
# categories: journal
# tags: [documentation,sample]
image: wordle_assist.png
---
<style>
/* mobile */
@media(max-width: 540px) {
  .desktop {
    display: none;
  }
  
  .mobile {
    display: block;
    border: none;
    width: 100%
    height: 500px;
  }
}

/* desktop */
@media(min-width: 541px) {
  .desktop {
    display: block;
    border: none;
    width: 150%;
    height: 800px;
  }
  
  .mobile {
    display: none;
  }
}
</style>

## Try it yourself

<iframe class="desktop" src="https://hx2j2u-kevin-tsang.shinyapps.io/wordle_assist/"> </iframe>
<iframe class="mobile" src="https://hx2j2u-kevin-tsang.shinyapps.io/wordle_assist/"> </iframe>



You can also run this app locally by running the following code in R.

```
shiny::runGitHub("wordle_assist", "kevinchtsang")
```

Type in your word attempts and change the colours to match (grey - no match, yellow - in word, green - correct position) by clicking on the tiles.

## Getting started with R
If you don't already have R installed on your computer, you can download RStudio (an IDE for R) for free [here](https://www.rstudio.com/products/rstudio/download/).

Then in the Console, you can run the following code to run the app.

```
install.packages("shiny")
shiny::runGitHub("wordle_assist", "kevinchtsang")
```

It will take a while for all the packages to be downloaded.

## Introduction
Wordle is a word game where players try to guess a five-letter word within six tries. If the letter is not present in the target word, the game will mark the letter as grey. If the letter is present in the target word but not in the exact position, the game will mark the letter as yellow. If the letter is in the exact position as the target word, the game will mark the letter green.

This Wordle Assistant app is a tool to help search all possible English words given the pattern and attempts the player has made. The user will need to match the grid in the assistance tool with the grid in their Wordle game. Then all the possible words will be listed below, listed alphabetically.

![app screenshot](https://raw.githubusercontent.com/kevinchtsang/wordle_assist/main/wordle_assist_example1.png)

This assistance tool was developed using the Scrabble dictionary available via the [`words`](https://cran.r-project.org/web/packages/words/index.html) package, which may not match exactly to the specific dictionary of the Wordle implementation you play.

## Code
The app is written using R and Shiny, making use of the `tidyverse` package to search and filter the dictionary.

Code is available [here](https://github.com/kevinchtsang/wordle_assist)