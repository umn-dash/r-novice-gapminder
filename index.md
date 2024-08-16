---
site: sandpaper::sandpaper_site
---

*An introduction to the R programming language for non-programmers using the gapminder data set*

The goal of this series of lessons is to teach novice programmers to write functional, useful code in the R programming language. R is commonly used in many disciplines for statistical analysis, and its huge volume of third-party packages make it highly versatile.

Many scientists who attend Software Carpentry workshops *use* R, but they don't all necessarily feel they *understand* it. Here, we attempt to give attendees a strong foundation in the fundamentals of R by thinking of it like a human language, one with its own grammar, punctuation, nouns, verbs, questions, adverbs, adjectives, and so on. We cover all those things step by step, concept by concept, in the most logical order we can manage.

Note that this workshop focuses *only* on teaching the fundamentals of R and RStudio; it does *not* cover statistical analysis, graphic design, or programming or data science best practices. Those topics are best explored after you feel comfortable using the language!

The written lessons contain *far* more material than can be taught in a day (or even three). However, the material can be readily broken into several day-long or half-day chunks. Alternatively, separate workshops can be used to cover the "Welcome to R" and "tidyverse" content.

::: prereq
## Prerequisites

These lessons assume you have R and RStudio installed on your computer.

-   [Download and install the latest version of R](https://www.r-project.org/) for your operating system.

-   [Download and install RStudio](https://www.rstudio.com/products/rstudio/download/#download).

RStudio is an application (an integrated development environment, or IDE) that facilitates the use of R and offers a number of nice features. You will need the free Desktop version for your computer for these lessons.

This lesson also assumes you understand that computers store data and information inside of files and that files are organized into folders (directories). Specific files and folders can be referenced by programs like R using their names or with file paths. No other general computing or programming knowledge is assumed.
:::

A variety of third-party packages are used throughout this workshop. These are not necessarily the best such packages, but they are ones many researchers use and find useful.

Interested users can download and install these packages by running the following command inside of R, if they choose, but this needn't be done prior to attendance:

```{r installpack}
install.packages("dpyr", "ggplot2", "gapminder", "tidyr")
```
