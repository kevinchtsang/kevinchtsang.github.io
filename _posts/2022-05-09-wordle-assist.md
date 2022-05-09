---
layout: post
title: "Wordle Assistant"
author: "Kevin Tsang"
# categories: journal
# tags: [documentation,sample]
image: wordle_assist.png
---

## Try it yourself

You can run this app locally by running the following code in R.

```
shiny::runGitHub("wordle_assist", "kevinchtsang")
```

Type in your word attempts and change the colours to match (grey - no match, yellow - in word, green - correct position) by clicking on the tiles.

## Introduction
Wordle is a word game where players try to guess a five-letter word within six tries. If the letter is not present in the target word, the game will mark the letter as grey. If the letter is present in the target word but not in the exact position, the game will mark the letter as yellow. If the letter is in the exact position as the target word, the game will mark the letter green.

This Wordle Assistant app is a tool to help search all possible English words given the pattern and attempts the player has made. The user will need to match the grid in the assistance tool with the grid in their Wordle game. Then all the possible words will be listed below, listed alphabetically.

![app screenshot](https://raw.githubusercontent.com/kevinchtsang/wordle_assist/main/wordle_assist_example1.png)

This assistance tool was developed using the Scrabble dictionary available via the [`words`](https://cran.r-project.org/web/packages/words/index.html) package, which may not match exactly to the specific dictionary of the Wordle implementation you play.

## Code
The app is written using R and Shiny, making use of the `tidyverse` package to search and filter the dictionary.

Code is available [here](https://github.com/kevinchtsang/wordle_assist)