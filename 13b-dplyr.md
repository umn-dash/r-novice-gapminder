---
title: Data Frame Manipulation with dplyr
teaching: 30
exercises: 15
source: Rmd
---

::: objectives
-   SUBSET a data set to a specific subset of rows and/or columns.
-   SORT or RENAME columns.
-   MAKE NEW COLUMNS, perhaps using old columns as inputs.
-   Generate SUMMARIES of our data set.
-   Use PIPES to string multiple operations on the same data set together.
:::

::: questions
-   What are the most common types of operations performed on data sets?
-   How can you perform these operations efficiently and clearly?
:::

## Preparation and setup

Note: This lesson uses the **gapminder data set**. This data set can be downloaded [here](https://drive.google.com/file/d/1sBkIJlC_6fImNZhyV0XfsllyUD-hSmoS/view?usp=sharing). Make sure to load the data set into your global environment before continuing.


```r
gap = read.csv("data/gapminder_data.csv", header = TRUE)
```

Also, this lesson will revolve around use of the optional "add-on" package `dplyr`. `dplyr` is a package in the "tidyverse"--an array of useful tools that are designed to look similar and work well together and that make R much more powerful to use but also a lot different than "Base R." Everything we do in this lesson can be done in "Base R" instead, but it won't (typically) be as efficient, easy, or clear!

`dplyr`, like many add-on packages, is **not** installed with R. So, we must install it before we can use it:


```r
install.packages("dplyr") #Only run once!
```

You only need to install a package once, so there's no need to run the above command more than once. However, packages are updated occasionally. When updates are available, you can re-install new versions using the same `install.packages()` function.

Once a package is installed, it still isn't "turned on" by default. So, to turn on `dplyr` so that we can access its unique features, we use the `library()` function:


```r
library(dplyr)
```

The above command must be run every time you start up a new session of R and want access to `dplyr`'s features!

## Introduction

R users work with data frames A LOT...like, *a lot*. When working with data frames, we very often find ourselves wanting to perform a few specific actions, such as cutting certain rows or columns, renaming columns, or transforming columns into new ones.

We can do ALL of those things in "Base R" if we wanted to, and we've already seen ways to do some of those things in previous lessons! However, `dplyr` makes doing all these things easier, more concise, AND more intuitive. It does this by adding specific "verbs" (functions) that have a consistent syntax and structure (as well as intuitive names) and by allowing us to write our code in connected "sentences" using those verbs so that more things can happen in a single command without that command also getting harder to read.

In this lesson, we'll go through each of these commands, showing their individual uses for examining and manipulating the gapminder data set:


```r
head(gap) #To remind us what the columns are called and what data are contained in this data set
```

```{.output}
      country year      pop continent lifeExp gdpPercap
1 Afghanistan 1952  8425333      Asia  28.801  779.4453
2 Afghanistan 1957  9240934      Asia  30.332  820.8530
3 Afghanistan 1962 10267083      Asia  31.997  853.1007
4 Afghanistan 1967 11537966      Asia  34.020  836.1971
5 Afghanistan 1972 13079460      Asia  36.088  739.9811
6 Afghanistan 1977 14880372      Asia  38.438  786.1134
```

## SELECT and RENAME

What if we only wanted a subset of the columns in a data set? The `dplyr` verb corresponding to this desire is `select()`.

**Important**: Every verb in `dplyr`, such as `select()`, takes as its first input the data frame you are trying to manipulate. After that first input, every *subsequent* input you provide becomes another "thing" you want the function to do to that data frame. What I mean by that last part will hopefully become more clear with an example.

Suppose I wanted a version of the gapminder data set that included only the `country` and `year` columns. I could use `select()` to achieve that desire like this:


```r
gap_shrunk = select(gap, #The first input is ALWAYS the data frame to be manipulated.
               country, year) #All the rest of the inputs become "things to do". Here, that's specific columns to select, referenced by name.
head(gap_shrunk)
```

```{.output}
      country year
1 Afghanistan 1952
2 Afghanistan 1957
3 Afghanistan 1962
4 Afghanistan 1967
5 Afghanistan 1972
6 Afghanistan 1977
```

In the example above, I provided my data frame as the first input to `select()` and all the columns I wanted to *select* as subsequent inputs. As a result, I ended up with a shrunken version of the original data set, one that contains only those two columns.

In the indexing lesson, we learned some "tricks" that work with `select()` too. For example, you can select a sequence of columns that are consecutive by using the `:` operator and the names of the first and last columns in that sequence like so:


```r
gap_sequence = select(gap, 
                      pop:lifeExp) 
head(gap_sequence)
```

```{.output}
       pop continent lifeExp
1  8425333      Asia  28.801
2  9240934      Asia  30.332
3 10267083      Asia  31.997
4 11537966      Asia  34.020
5 13079460      Asia  36.088
6 14880372      Asia  38.438
```

We can also use the `-` operator to instead specify columns we want to *reject* instead of keep. For example, to retain every column *except* the `year` column, we could do this:


```r
gap_noyear = select(gap, 
                    -year)
head(gap_noyear)
```

```{.output}
      country      pop continent lifeExp gdpPercap
1 Afghanistan  8425333      Asia  28.801  779.4453
2 Afghanistan  9240934      Asia  30.332  820.8530
3 Afghanistan 10267083      Asia  31.997  853.1007
4 Afghanistan 11537966      Asia  34.020  836.1971
5 Afghanistan 13079460      Asia  36.088  739.9811
6 Afghanistan 14880372      Asia  38.438  786.1134
```

We can also use `select()` to *rearrange* columns by specifying the column names in the new order we want them to be in:


```r
gap_reordered = select(gap, 
                       year, country)
head(gap_reordered)
```

```{.output}
  year     country
1 1952 Afghanistan
2 1957 Afghanistan
3 1962 Afghanistan
4 1967 Afghanistan
5 1972 Afghanistan
6 1977 Afghanistan
```

What if wanted to rename some of our columns? The `dplyr` verb corresponding to this goal is `rename()`.

As with `select()` (and all `dplyr` verbs), `rename()`'s first input is the data frame we're manipulating. Each subsequent input needs to be an "instructions list," with the new name to the left of an `=` operator and the old column name to the right of it (what I like to call *"new = old" format*).

For example, to rename the `pop` column to "population," which I would personally find more intuitive, we would do the following:


```r
gap_renamed = rename(gap, 
                     population = pop) #New = old format
head(gap_renamed)
```

```{.output}
      country year population continent lifeExp gdpPercap
1 Afghanistan 1952    8425333      Asia  28.801  779.4453
2 Afghanistan 1957    9240934      Asia  30.332  820.8530
3 Afghanistan 1962   10267083      Asia  31.997  853.1007
4 Afghanistan 1967   11537966      Asia  34.020  836.1971
5 Afghanistan 1972   13079460      Asia  36.088  739.9811
6 Afghanistan 1977   14880372      Asia  38.438  786.1134
```

It's as simple as that! If we wanted to rename multiple columns, we could just add more subsequent inputs to the same `rename()` call.

## The magic of pipes

::: challenge
What if I wanted to *first* eliminate some columns and *then* rename some of the remaining columns? How would you go about doing that?

::: solution
Your *first* impulse might be to do this in two distinct commands, saving *intermediate objects* after each step, like so:


```r
gap_selected = select(gap, 
                      country:pop)
gap_remonikered = rename(gap_selected, 
               population = pop)
head(gap_remonikered)
```

```{.output}
      country year population
1 Afghanistan 1952    8425333
2 Afghanistan 1957    9240934
3 Afghanistan 1962   10267083
4 Afghanistan 1967   11537966
5 Afghanistan 1972   13079460
6 Afghanistan 1977   14880372
```
:::
:::

There's **nothing** *wrong* with this approach, but it's a little...tedious. Plus, if you don't pick really good names for each *intermediate object*, it can get confusing and hard for you and others to follow along with.

I hope you're thinking "I bet there's an easier way." And there is! We can combine these two discrete steps into one easy-to-read "sentence." The only catch is we have to use a strange operator called a *pipe* to do it.

`dplyr` pipes look like this: `%>%`. On Windows, the hotkey to render a pipe is control + shift + M, in case you'd rather type that then "%\>%"! On Mac, it's similar: Command + shift + M.

Pipes may look a little funny, but they do something *really* cool. They take the "thing" produced on their left (once all the operations over there are complete) and "pump" that thing into the operations on their right, specifically into the first available input slot.

This is easier to explain with an example, so let's see how to use pipes to perform the two operations we did above in a single command.


```r
gap.final = gap %>% 
  select(country:pop) %>% 
  rename(population = pop) 
head(gap.final)
```

```{.output}
      country year population
1 Afghanistan 1952    8425333
2 Afghanistan 1957    9240934
3 Afghanistan 1962   10267083
4 Afghanistan 1967   11537966
5 Afghanistan 1972   13079460
6 Afghanistan 1977   14880372
```

The command above says: "Take the `gap` data set and pump it into `select()`'s first input slot (where it belongs anyway). Then, do `select()`'s operations (which yield a new, manipulated data set) and pump *that* resulting data set into `rename()`'s first input slot. Then, when all that's done, save the result into an object called `gap.final`."

Hopefully, you can see how this might be easier to read and follow along with and also be more efficient! It also explains why every `dplyr` verb's first input slot is the data frame to be manipulated--it makes every verb ready to receive inputs from "upstream" via a pipe!

## FILTER, ARRANGE, and MUTATE

What if we only wanted to look at the data from a specific continent or a specific time span (i.e., we want just specific rows)? The `dplyr` verb corresponding to this action is `filter()` (see, I said the verbs have intuitive names!).

Each input given to `filter()` past the first (which is ALWAYS the data frame to be manipulated, as we've firmly established) is a *logical rule*, which is something we've seen before (e.g., when studying `if()`). Here, each *logical rule* will consist of the name of the column we'll check the values of, a *logical operator* (like `==` or `<=`), and a "threshold" to check against. If **all** the *logical rules* given for a particular row pass, we keep that row. Otherwise, we remove it from the new data frame we create.

For example, here's how we'd filter our data set to just rows where the value in the `continent` column is **exactly** `Europe`:


```r
gap_europe = gap %>% 
  filter(continent == "Europe") #The column to check values in, a logical operator, and the "value for comparison."
head(gap_europe)
```

```{.output}
  country year     pop continent lifeExp gdpPercap
1 Albania 1952 1282697    Europe   55.23  1601.056
2 Albania 1957 1476505    Europe   59.28  1942.284
3 Albania 1962 1728137    Europe   64.82  2312.889
4 Albania 1967 1984060    Europe   66.22  2760.197
5 Albania 1972 2263554    Europe   67.69  3313.422
6 Albania 1977 2509048    Europe   68.93  3533.004
```

As another example, here's how we'd filter our data set to just rows with data from before the year `1975`:


```r
gap_pre1975 = gap %>% 
  filter(year < 1975)
head(gap_pre1975)
```

```{.output}
      country year      pop continent lifeExp gdpPercap
1 Afghanistan 1952  8425333      Asia  28.801  779.4453
2 Afghanistan 1957  9240934      Asia  30.332  820.8530
3 Afghanistan 1962 10267083      Asia  31.997  853.1007
4 Afghanistan 1967 11537966      Asia  34.020  836.1971
5 Afghanistan 1972 13079460      Asia  36.088  739.9811
6     Albania 1952  1282697    Europe  55.230 1601.0561
```

::: challenge
What if we wanted data only from *between* the years 1970 and 1979? How would you achieve this goal using `filter()`? Hint: There are *at least* three ways you should be able to think of!

::: solution
Here are three different solutions. In the first, we use the `&` (and) operator to specify two rules that a value in the `year` column must satisfy:


```r
and_option = gap %>% 
  filter(year > 1969 & year < 1980)
```

However, I explained earlier that, with `filter()`, every input past the first is a *logical rule* a row must satisfy for that row to be kept. So, a comma between *logical rules* here is just as good as `&` is:


```r
comma_option = gap %>% 
  filter(year > 1969, year < 1980)
```

Or, if you would prefer it, you can always just stack multiple `filter()` calls back to back:


```r
stacked_option = gap %>% 
  filter(year > 1969) %>% 
  filter(year < 1980)
```

There's nothing right or wrong with any of these options, so which do you prefer? Why?
:::
:::

::: challenge
**Important**: When chaining dplyr verbs together, order matters! Why does the following code fail when we try to run it?


```r
this_will_fail = gap %>% 
  select(pop:lifeExp) %>% 
  filter(year < 1975)
```

::: solution
Recall that the `year` column is not one of the columns that exists in between the `pop` and `lifeExp` columns. So, the `year` column gets cuts by the `select()` call here *before* we get to the `filter()` call, so the `filter()` call then can't find that column to use it.

Considering that `dplyr` "sentences" can get long and complicated, it's good to remember to be thoughtful about the order you specify actions in!
:::
:::

What if we wanted to *sort* our data set by the values in one or more columns? The `dplyr` verb for this desire is `arrange()`.

Every input past the first given to `arrange()` is a column we want to sort by, with columns provided earlier taking "precedence" in the sorting over later ones.

For example, here's how we'd sort our data set by the `lifeExp` column:


```r
gap_sorted = gap %>% 
  arrange(lifeExp)
head(gap_sorted)
```

```{.output}
       country year     pop continent lifeExp gdpPercap
1       Rwanda 1992 7290203    Africa  23.599  737.0686
2  Afghanistan 1952 8425333      Asia  28.801  779.4453
3       Gambia 1952  284320    Africa  30.000  485.2307
4       Angola 1952 4232095    Africa  30.015 3520.6103
5 Sierra Leone 1952 2143249    Africa  30.331  879.7877
6  Afghanistan 1957 9240934      Asia  30.332  820.8530
```

This sorts the data in ascending order by default. If we want to reverse the sorting, we can use the `desc()` function:


```r
gap_sorted_down = gap %>% 
  arrange(desc(lifeExp))
head(gap_sorted_down)
```

```{.output}
          country year       pop continent lifeExp gdpPercap
1           Japan 2007 127467972      Asia  82.603  31656.07
2 Hong Kong China 2007   6980412      Asia  82.208  39724.98
3           Japan 2002 127065841      Asia  82.000  28604.59
4         Iceland 2007    301931    Europe  81.757  36180.79
5     Switzerland 2007   7554661    Europe  81.701  37506.42
6 Hong Kong China 2002   6762476      Asia  81.495  30209.02
```

::: challenge
I mentioned above that you can provide multiple inputs to `arrange()`, but it's a little hard to explain what this actually does, so let's try it:


```r
gap_2xsorted = gap %>% 
  arrange(year, continent)
head(gap_2xsorted)
```

```{.output}
       country year     pop continent lifeExp gdpPercap
1      Algeria 1952 9279525    Africa  43.077 2449.0082
2       Angola 1952 4232095    Africa  30.015 3520.6103
3        Benin 1952 1738315    Africa  38.223 1062.7522
4     Botswana 1952  442308    Africa  47.622  851.2411
5 Burkina Faso 1952 4469979    Africa  31.975  543.2552
6      Burundi 1952 2445618    Africa  39.031  339.2965
```

What did the command above do? Why? What would change if you reversed the order of `continent` and `year` in the call?

::: solution
This command *first* sorted the data set by the unique values in the `year` column. It then "broke ties," where two rows have the **same** value for `year`, by *then* sorting by unique values in the `continent` column. So, we get records for `Africa` sooner than we get records for `Asia` for the same year.

If we reversed the order of our two inputs, we'd instead get **all** records for `Africa` before getting **any** records for `Asia`. Within a `continent`, though, we'd get back earlier records before getting back more recent ones.

This mirrors the behavior of "multi-column sorting" functionality as it exists in programs like Microsoft Excel.
:::
:::

What if we wanted to make a new column using an old column's values as inputs? This is the kind of thing many of us would think to do in Microsoft Excel, where it isn't always easy or reproducible. Thankfully, we have the `dplyr` verb `mutate()` for this.

Every input to `mutate()` past the first is an "instructions list" of how to make a new column using one or more old columns, and these "instructions lists" also follow *"new = old" format*.

For example, here's how we would create a new column called `pop1K` that is made by dividing the `pop` column's values by 1000:


```r
gap_newcol = gap %>% 
  mutate(pop1K = round(pop / 1000)) #New = old format
head(gap_newcol)
```

```{.output}
      country year      pop continent lifeExp gdpPercap pop1K
1 Afghanistan 1952  8425333      Asia  28.801  779.4453  8425
2 Afghanistan 1957  9240934      Asia  30.332  820.8530  9241
3 Afghanistan 1962 10267083      Asia  31.997  853.1007 10267
4 Afghanistan 1967 11537966      Asia  34.020  836.1971 11538
5 Afghanistan 1972 13079460      Asia  36.088  739.9811 13079
6 Afghanistan 1977 14880372      Asia  38.438  786.1134 14880
```

You'll note that the old `pop` column is still around after this. If you want to get rid of it, you can provide `.keep = "unused"` as another input to `mutate()` and it will eliminate any old columns that were used to create new ones. Try it!

To keep things simple here, we won't show you this, but `mutate()`'s "instructions lists" can get *very* complicated if you need them to! So, stop here to briefly imagine how you could perform a more complicated `mutate()` call here than the one we showed you.

## GROUP_BY and SUMMARIZE

Perhaps the most common (and powerful) action we might want to take on a data set is generating summaries from it. "What's the mean of *this* column?" or "What's the median value for all the different groups in *that* column?", for example.

What if we wanted to calculate the mean life expectancy across all years by country for the gapminder data set?

We *could* use `filter()` to go country by country (one at a time), save the resulting data sets as *intermediate objects*, and then take a mean manually of the `lifeExp` column for each *intermediate object*...but what a **pain** that would be!

Thankfully, we don't have to; instead, we can just use `group_by()` and `summarize()`! Unlike other `dplyr` verbs we've met, these are a duo--they are **almost always** used together and, importantly, we *always* have to use `group_by()` first.

So, let's start by understanding what `group_by()` does. Each input given to `group_by()` past the first creates groupings in the data. Specifically, you provide one (or more) names of columns and R will find all the different values in that column (such as all the different unique country names) and subtly "bundle up" all the rows that belong to each group.

This is easier to show you than to explain, so let's try it:


```r
gap_grouped = gap %>% 
  group_by(country) #Bundle up all the rows that belong to each different value in this country column.
head(gap_grouped)
```

```{.output}
# A tibble: 6 × 6
# Groups:   country [1]
  country      year      pop continent lifeExp gdpPercap
  <chr>       <int>    <dbl> <chr>       <dbl>     <dbl>
1 Afghanistan  1952  8425333 Asia         28.8      779.
2 Afghanistan  1957  9240934 Asia         30.3      821.
3 Afghanistan  1962 10267083 Asia         32.0      853.
4 Afghanistan  1967 11537966 Asia         34.0      836.
5 Afghanistan  1972 13079460 Asia         36.1      740.
6 Afghanistan  1977 14880372 Asia         38.4      786.
```

When we actually look at the new data set we've created, it will *look* as though nothing has changed. And, in a lot of ways, nothing has! However, if you examine `gap_grouped` in your R Studio's 'Environment' Pane, you'll notice that `gap_grouped` is considered a "grouped data frame" now instead of a plain-old one.

We can see what that means by using the `str()` ("structure") function to peek "under the hood" at `gap_grouped`:


```r
str(gap_grouped)
```

```{.output}
gropd_df [1,704 × 6] (S3: grouped_df/tbl_df/tbl/data.frame)
 $ country  : chr [1:1704] "Afghanistan" "Afghanistan" "Afghanistan" "Afghanistan" ...
 $ year     : int [1:1704] 1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
 $ pop      : num [1:1704] 8425333 9240934 10267083 11537966 13079460 ...
 $ continent: chr [1:1704] "Asia" "Asia" "Asia" "Asia" ...
 $ lifeExp  : num [1:1704] 28.8 30.3 32 34 36.1 ...
 $ gdpPercap: num [1:1704] 779 821 853 836 740 ...
 - attr(*, "groups")= tibble [142 × 2] (S3: tbl_df/tbl/data.frame)
  ..$ country: chr [1:142] "Afghanistan" "Albania" "Algeria" "Angola" ...
  ..$ .rows  : list<int> [1:142] 
  .. ..$ : int [1:12] 1 2 3 4 5 6 7 8 9 10 ...
  .. ..$ : int [1:12] 13 14 15 16 17 18 19 20 21 22 ...
  .. ..$ : int [1:12] 25 26 27 28 29 30 31 32 33 34 ...
  .. ..$ : int [1:12] 37 38 39 40 41 42 43 44 45 46 ...
  .. ..$ : int [1:12] 49 50 51 52 53 54 55 56 57 58 ...
  .. ..$ : int [1:12] 61 62 63 64 65 66 67 68 69 70 ...
  .. ..$ : int [1:12] 73 74 75 76 77 78 79 80 81 82 ...
  .. ..$ : int [1:12] 85 86 87 88 89 90 91 92 93 94 ...
  .. ..$ : int [1:12] 97 98 99 100 101 102 103 104 105 106 ...
  .. ..$ : int [1:12] 109 110 111 112 113 114 115 116 117 118 ...
  .. ..$ : int [1:12] 121 122 123 124 125 126 127 128 129 130 ...
  .. ..$ : int [1:12] 133 134 135 136 137 138 139 140 141 142 ...
  .. ..$ : int [1:12] 145 146 147 148 149 150 151 152 153 154 ...
  .. ..$ : int [1:12] 157 158 159 160 161 162 163 164 165 166 ...
  .. ..$ : int [1:12] 169 170 171 172 173 174 175 176 177 178 ...
  .. ..$ : int [1:12] 181 182 183 184 185 186 187 188 189 190 ...
  .. ..$ : int [1:12] 193 194 195 196 197 198 199 200 201 202 ...
  .. ..$ : int [1:12] 205 206 207 208 209 210 211 212 213 214 ...
  .. ..$ : int [1:12] 217 218 219 220 221 222 223 224 225 226 ...
  .. ..$ : int [1:12] 229 230 231 232 233 234 235 236 237 238 ...
  .. ..$ : int [1:12] 241 242 243 244 245 246 247 248 249 250 ...
  .. ..$ : int [1:12] 253 254 255 256 257 258 259 260 261 262 ...
  .. ..$ : int [1:12] 265 266 267 268 269 270 271 272 273 274 ...
  .. ..$ : int [1:12] 277 278 279 280 281 282 283 284 285 286 ...
  .. ..$ : int [1:12] 289 290 291 292 293 294 295 296 297 298 ...
  .. ..$ : int [1:12] 301 302 303 304 305 306 307 308 309 310 ...
  .. ..$ : int [1:12] 313 314 315 316 317 318 319 320 321 322 ...
  .. ..$ : int [1:12] 325 326 327 328 329 330 331 332 333 334 ...
  .. ..$ : int [1:12] 337 338 339 340 341 342 343 344 345 346 ...
  .. ..$ : int [1:12] 349 350 351 352 353 354 355 356 357 358 ...
  .. ..$ : int [1:12] 361 362 363 364 365 366 367 368 369 370 ...
  .. ..$ : int [1:12] 373 374 375 376 377 378 379 380 381 382 ...
  .. ..$ : int [1:12] 385 386 387 388 389 390 391 392 393 394 ...
  .. ..$ : int [1:12] 397 398 399 400 401 402 403 404 405 406 ...
  .. ..$ : int [1:12] 409 410 411 412 413 414 415 416 417 418 ...
  .. ..$ : int [1:12] 421 422 423 424 425 426 427 428 429 430 ...
  .. ..$ : int [1:12] 433 434 435 436 437 438 439 440 441 442 ...
  .. ..$ : int [1:12] 445 446 447 448 449 450 451 452 453 454 ...
  .. ..$ : int [1:12] 457 458 459 460 461 462 463 464 465 466 ...
  .. ..$ : int [1:12] 469 470 471 472 473 474 475 476 477 478 ...
  .. ..$ : int [1:12] 481 482 483 484 485 486 487 488 489 490 ...
  .. ..$ : int [1:12] 493 494 495 496 497 498 499 500 501 502 ...
  .. ..$ : int [1:12] 505 506 507 508 509 510 511 512 513 514 ...
  .. ..$ : int [1:12] 517 518 519 520 521 522 523 524 525 526 ...
  .. ..$ : int [1:12] 529 530 531 532 533 534 535 536 537 538 ...
  .. ..$ : int [1:12] 541 542 543 544 545 546 547 548 549 550 ...
  .. ..$ : int [1:12] 553 554 555 556 557 558 559 560 561 562 ...
  .. ..$ : int [1:12] 565 566 567 568 569 570 571 572 573 574 ...
  .. ..$ : int [1:12] 577 578 579 580 581 582 583 584 585 586 ...
  .. ..$ : int [1:12] 589 590 591 592 593 594 595 596 597 598 ...
  .. ..$ : int [1:12] 601 602 603 604 605 606 607 608 609 610 ...
  .. ..$ : int [1:12] 613 614 615 616 617 618 619 620 621 622 ...
  .. ..$ : int [1:12] 625 626 627 628 629 630 631 632 633 634 ...
  .. ..$ : int [1:12] 637 638 639 640 641 642 643 644 645 646 ...
  .. ..$ : int [1:12] 649 650 651 652 653 654 655 656 657 658 ...
  .. ..$ : int [1:12] 661 662 663 664 665 666 667 668 669 670 ...
  .. ..$ : int [1:12] 673 674 675 676 677 678 679 680 681 682 ...
  .. ..$ : int [1:12] 685 686 687 688 689 690 691 692 693 694 ...
  .. ..$ : int [1:12] 697 698 699 700 701 702 703 704 705 706 ...
  .. ..$ : int [1:12] 709 710 711 712 713 714 715 716 717 718 ...
  .. ..$ : int [1:12] 721 722 723 724 725 726 727 728 729 730 ...
  .. ..$ : int [1:12] 733 734 735 736 737 738 739 740 741 742 ...
  .. ..$ : int [1:12] 745 746 747 748 749 750 751 752 753 754 ...
  .. ..$ : int [1:12] 757 758 759 760 761 762 763 764 765 766 ...
  .. ..$ : int [1:12] 769 770 771 772 773 774 775 776 777 778 ...
  .. ..$ : int [1:12] 781 782 783 784 785 786 787 788 789 790 ...
  .. ..$ : int [1:12] 793 794 795 796 797 798 799 800 801 802 ...
  .. ..$ : int [1:12] 805 806 807 808 809 810 811 812 813 814 ...
  .. ..$ : int [1:12] 817 818 819 820 821 822 823 824 825 826 ...
  .. ..$ : int [1:12] 829 830 831 832 833 834 835 836 837 838 ...
  .. ..$ : int [1:12] 841 842 843 844 845 846 847 848 849 850 ...
  .. ..$ : int [1:12] 853 854 855 856 857 858 859 860 861 862 ...
  .. ..$ : int [1:12] 865 866 867 868 869 870 871 872 873 874 ...
  .. ..$ : int [1:12] 877 878 879 880 881 882 883 884 885 886 ...
  .. ..$ : int [1:12] 889 890 891 892 893 894 895 896 897 898 ...
  .. ..$ : int [1:12] 901 902 903 904 905 906 907 908 909 910 ...
  .. ..$ : int [1:12] 913 914 915 916 917 918 919 920 921 922 ...
  .. ..$ : int [1:12] 925 926 927 928 929 930 931 932 933 934 ...
  .. ..$ : int [1:12] 937 938 939 940 941 942 943 944 945 946 ...
  .. ..$ : int [1:12] 949 950 951 952 953 954 955 956 957 958 ...
  .. ..$ : int [1:12] 961 962 963 964 965 966 967 968 969 970 ...
  .. ..$ : int [1:12] 973 974 975 976 977 978 979 980 981 982 ...
  .. ..$ : int [1:12] 985 986 987 988 989 990 991 992 993 994 ...
  .. ..$ : int [1:12] 997 998 999 1000 1001 1002 1003 1004 1005 1006 ...
  .. ..$ : int [1:12] 1009 1010 1011 1012 1013 1014 1015 1016 1017 1018 ...
  .. ..$ : int [1:12] 1021 1022 1023 1024 1025 1026 1027 1028 1029 1030 ...
  .. ..$ : int [1:12] 1033 1034 1035 1036 1037 1038 1039 1040 1041 1042 ...
  .. ..$ : int [1:12] 1045 1046 1047 1048 1049 1050 1051 1052 1053 1054 ...
  .. ..$ : int [1:12] 1057 1058 1059 1060 1061 1062 1063 1064 1065 1066 ...
  .. ..$ : int [1:12] 1069 1070 1071 1072 1073 1074 1075 1076 1077 1078 ...
  .. ..$ : int [1:12] 1081 1082 1083 1084 1085 1086 1087 1088 1089 1090 ...
  .. ..$ : int [1:12] 1093 1094 1095 1096 1097 1098 1099 1100 1101 1102 ...
  .. ..$ : int [1:12] 1105 1106 1107 1108 1109 1110 1111 1112 1113 1114 ...
  .. ..$ : int [1:12] 1117 1118 1119 1120 1121 1122 1123 1124 1125 1126 ...
  .. ..$ : int [1:12] 1129 1130 1131 1132 1133 1134 1135 1136 1137 1138 ...
  .. ..$ : int [1:12] 1141 1142 1143 1144 1145 1146 1147 1148 1149 1150 ...
  .. ..$ : int [1:12] 1153 1154 1155 1156 1157 1158 1159 1160 1161 1162 ...
  .. ..$ : int [1:12] 1165 1166 1167 1168 1169 1170 1171 1172 1173 1174 ...
  .. ..$ : int [1:12] 1177 1178 1179 1180 1181 1182 1183 1184 1185 1186 ...
  .. .. [list output truncated]
  .. ..@ ptype: int(0) 
  ..- attr(*, ".drop")= logi TRUE
```

If we look towards the bottom of the output above, we'll see that, for every unique value in the `country` column, there is now a list of all the *row numbers* of rows sharing that same country value (i.e., all the rows for `"Afghanistan"`, all the rows for `"Albania"`, etc.). In other words, R now knows that each row belongs to one specific group within the larger data set. So, when we then ask it to calculate a summary of our data set, it can do so for each group separately.

Let's see how that works by next examining `summarize()`. Each input past the first given to `summarize()` is an "instructions list" for how to generate a summary, with these "instructions lists" once again taking *new = old format* (see, I said these tools were designed to be consistent!).

For example, let's tell `summarize()` to calculate a mean life expectancy for every country and to call the new column holding those means `mean_lifeExp`:


```r
gap_summarized = gap_grouped %>%  #Make sure you put our grouped data set here!
  summarize(mean_lifeExp = mean(lifeExp)) #new = old format.
head(gap_summarized)
```

```{.output}
# A tibble: 6 × 2
  country     mean_lifeExp
  <chr>              <dbl>
1 Afghanistan         37.5
2 Albania             68.4
3 Algeria             59.0
4 Angola              37.9
5 Argentina           69.1
6 Australia           74.7
```

::: challenge
Consider: How many rows does `gap_summarized` now have? Why does it have so many fewer rows than `gap_grouped` did? And where did all the other columns go?

::: solution
`gap_summarized` only has 142 rows, whereas `gap_grouped` had 1704. The reason for this is that we summarized our data *by group*; we asked R to give us a __single value__ for each group in our data set. There are only 142 countries in the gapminder data set, so we end up with a single row for each country when we summarize.

But where did all the other columns go? Well, we didn't ask for summaries of those other columns. So, if there used to be 12 values of `pop` for a given country before summarization, but there's going to be just a single row for a given country after summarization, and we don't tell R how exactly to "collapse" those 12 values down to just one, it's more "responsible" for R to just drop those columns entirely rather than to guess how it should do that collapsing, right? 

:::
:::

If you want multiple summaries, you can simply provide multiple inputs to `summarize()`. For example, `n()` is a handy function for counting up the number of data points in each group prior to any summarizations occurring:


```r
gap_summarized = gap_grouped %>%  
  summarize(mean_lifeExp = mean(lifeExp),
            sample_sizes = n()) 
head(gap_summarized)
```

```{.output}
# A tibble: 6 × 3
  country     mean_lifeExp sample_sizes
  <chr>              <dbl>        <int>
1 Afghanistan         37.5           12
2 Albania             68.4           12
3 Algeria             59.0           12
4 Angola              37.9           12
5 Argentina           69.1           12
6 Australia           74.7           12
```

Here, all the values in our new column are `12` because we have exactly 12 records per country to start with, but if the number of records differed between countries, the above operation would have shown us that.

::: challenge
One more concept: I said earlier that you can provide multiple inputs to `group_by()`. What happens when we do that? Let's try it:


```r
gap_2xgrouped = gap %>% 
  group_by(continent, year)
str(gap_2xgrouped)
```

```{.output}
gropd_df [1,704 × 6] (S3: grouped_df/tbl_df/tbl/data.frame)
 $ country  : chr [1:1704] "Afghanistan" "Afghanistan" "Afghanistan" "Afghanistan" ...
 $ year     : int [1:1704] 1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
 $ pop      : num [1:1704] 8425333 9240934 10267083 11537966 13079460 ...
 $ continent: chr [1:1704] "Asia" "Asia" "Asia" "Asia" ...
 $ lifeExp  : num [1:1704] 28.8 30.3 32 34 36.1 ...
 $ gdpPercap: num [1:1704] 779 821 853 836 740 ...
 - attr(*, "groups")= tibble [60 × 3] (S3: tbl_df/tbl/data.frame)
  ..$ continent: chr [1:60] "Africa" "Africa" "Africa" "Africa" ...
  ..$ year     : int [1:60] 1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
  ..$ .rows    : list<int> [1:60] 
  .. ..$ : int [1:52] 25 37 121 157 193 205 229 253 265 313 ...
  .. ..$ : int [1:52] 26 38 122 158 194 206 230 254 266 314 ...
  .. ..$ : int [1:52] 27 39 123 159 195 207 231 255 267 315 ...
  .. ..$ : int [1:52] 28 40 124 160 196 208 232 256 268 316 ...
  .. ..$ : int [1:52] 29 41 125 161 197 209 233 257 269 317 ...
  .. ..$ : int [1:52] 30 42 126 162 198 210 234 258 270 318 ...
  .. ..$ : int [1:52] 31 43 127 163 199 211 235 259 271 319 ...
  .. ..$ : int [1:52] 32 44 128 164 200 212 236 260 272 320 ...
  .. ..$ : int [1:52] 33 45 129 165 201 213 237 261 273 321 ...
  .. ..$ : int [1:52] 34 46 130 166 202 214 238 262 274 322 ...
  .. ..$ : int [1:52] 35 47 131 167 203 215 239 263 275 323 ...
  .. ..$ : int [1:52] 36 48 132 168 204 216 240 264 276 324 ...
  .. ..$ : int [1:25] 49 133 169 241 277 301 349 385 433 445 ...
  .. ..$ : int [1:25] 50 134 170 242 278 302 350 386 434 446 ...
  .. ..$ : int [1:25] 51 135 171 243 279 303 351 387 435 447 ...
  .. ..$ : int [1:25] 52 136 172 244 280 304 352 388 436 448 ...
  .. ..$ : int [1:25] 53 137 173 245 281 305 353 389 437 449 ...
  .. ..$ : int [1:25] 54 138 174 246 282 306 354 390 438 450 ...
  .. ..$ : int [1:25] 55 139 175 247 283 307 355 391 439 451 ...
  .. ..$ : int [1:25] 56 140 176 248 284 308 356 392 440 452 ...
  .. ..$ : int [1:25] 57 141 177 249 285 309 357 393 441 453 ...
  .. ..$ : int [1:25] 58 142 178 250 286 310 358 394 442 454 ...
  .. ..$ : int [1:25] 59 143 179 251 287 311 359 395 443 455 ...
  .. ..$ : int [1:25] 60 144 180 252 288 312 360 396 444 456 ...
  .. ..$ : int [1:33] 1 85 97 217 289 661 697 709 721 733 ...
  .. ..$ : int [1:33] 2 86 98 218 290 662 698 710 722 734 ...
  .. ..$ : int [1:33] 3 87 99 219 291 663 699 711 723 735 ...
  .. ..$ : int [1:33] 4 88 100 220 292 664 700 712 724 736 ...
  .. ..$ : int [1:33] 5 89 101 221 293 665 701 713 725 737 ...
  .. ..$ : int [1:33] 6 90 102 222 294 666 702 714 726 738 ...
  .. ..$ : int [1:33] 7 91 103 223 295 667 703 715 727 739 ...
  .. ..$ : int [1:33] 8 92 104 224 296 668 704 716 728 740 ...
  .. ..$ : int [1:33] 9 93 105 225 297 669 705 717 729 741 ...
  .. ..$ : int [1:33] 10 94 106 226 298 670 706 718 730 742 ...
  .. ..$ : int [1:33] 11 95 107 227 299 671 707 719 731 743 ...
  .. ..$ : int [1:33] 12 96 108 228 300 672 708 720 732 744 ...
  .. ..$ : int [1:30] 13 73 109 145 181 373 397 409 517 529 ...
  .. ..$ : int [1:30] 14 74 110 146 182 374 398 410 518 530 ...
  .. ..$ : int [1:30] 15 75 111 147 183 375 399 411 519 531 ...
  .. ..$ : int [1:30] 16 76 112 148 184 376 400 412 520 532 ...
  .. ..$ : int [1:30] 17 77 113 149 185 377 401 413 521 533 ...
  .. ..$ : int [1:30] 18 78 114 150 186 378 402 414 522 534 ...
  .. ..$ : int [1:30] 19 79 115 151 187 379 403 415 523 535 ...
  .. ..$ : int [1:30] 20 80 116 152 188 380 404 416 524 536 ...
  .. ..$ : int [1:30] 21 81 117 153 189 381 405 417 525 537 ...
  .. ..$ : int [1:30] 22 82 118 154 190 382 406 418 526 538 ...
  .. ..$ : int [1:30] 23 83 119 155 191 383 407 419 527 539 ...
  .. ..$ : int [1:30] 24 84 120 156 192 384 408 420 528 540 ...
  .. ..$ : int [1:2] 61 1093
  .. ..$ : int [1:2] 62 1094
  .. ..$ : int [1:2] 63 1095
  .. ..$ : int [1:2] 64 1096
  .. ..$ : int [1:2] 65 1097
  .. ..$ : int [1:2] 66 1098
  .. ..$ : int [1:2] 67 1099
  .. ..$ : int [1:2] 68 1100
  .. ..$ : int [1:2] 69 1101
  .. ..$ : int [1:2] 70 1102
  .. ..$ : int [1:2] 71 1103
  .. ..$ : int [1:2] 72 1104
  .. ..@ ptype: int(0) 
  ..- attr(*, ".drop")= logi TRUE
```

How did R group together rows in this case?

Next, try generating mean life expectancies and sample sizes using `gap_2xgrouped` as an input. You'll get different values than we did before when using `gap_grouped`, and we'll also get a different number of rows in the resulting output. Why?

::: solution
Here's how we'd generate those summaries:


```r
gap_2xsummarized = gap_2xgrouped %>%  
  summarize(mean_lifeExp = mean(lifeExp),
            sample_sizes = n()) 
head(gap_2xsummarized)
```

```{.output}
# A tibble: 6 × 4
# Groups:   continent [1]
  continent  year mean_lifeExp sample_sizes
  <chr>     <int>        <dbl>        <int>
1 Africa     1952         39.1           52
2 Africa     1957         41.3           52
3 Africa     1962         43.3           52
4 Africa     1967         45.3           52
5 Africa     1972         47.5           52
6 Africa     1977         49.6           52
```

By specifying multiple columns to group by, what R did is find the rows belonging to each unique *combo* of the unique values in each column. That is, here, it found the rows that belong to each unique `continent` x `year` combo.

So, when we summarize, we're calculating summaries *across* different countries that belong to each unique `continent` for each unique `year`. Because there are differing numbers of countries in each `continent`, our sample sizes now differ between the continents--examine the final data frame and confirm that!
:::
:::

::: keypoints
-   Use the `dplyr` package to manipulate data frames in efficient, clear, and intuitive ways.
-   Use `select()` to retain specific variables when creating a new, smaller data frame.
-   Use `filter()` to subset data based on values in one or more columns.
-   Use `group_by()` and `summarize()` to generate summaries of data by groups.
-   Use `mutate()` to create new variables using old ones.
-   Use `rename()` to rename columns and `arrange()` to sort your data set by one or more columns.
-   Use pipes (`%>%`) to string `dplyr` verbs together into "sentences."
-   Remember that order matters when stringing together `dplyr` verbs!
:::
