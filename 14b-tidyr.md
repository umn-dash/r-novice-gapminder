---
title: Pivoting using tidyr
teaching: 20
exercises: 10
source: Rmd
editor_options: 
  markdown: 
    wrap: 72
---

::: objectives
-   Distinguish between "long" and "wide" formats (and
    "in-between" formats) for storing data in a table.
-   Recognize and explain the benefits and drawbacks of storing data in each format.
-   Use the `pivot` functions in the `tidyr` package to convert data
    from one format to another.
:::

::: questions
-   *How* am I storing data in my data sets?
-   How does R tend to expect me to store my data?
-   How can I convert (or *pivot*) my data from one form to another?
:::

## Introduction

**Important**: There is no one universal, "right" way to store data in a
table! There are many advantages and disadvantages to each way and, in
some ways, so long as your organizational system follows good data science storage conventions, is consistent, and works
for you and your team, then you're probably doing it "right."

That said, from a *computing* standpoint, there are two broad ways to
think about how data are stored in a table--in "wide" format or in
"long" format:

![When data are in a wide format (left-hand side above), *groups* within the data (such as teams) have their own rows, and everything we know about each group (such as stats) is listed in different columns in that group's row. When data are in long format (right-hand side above), each individual row belongs to a *single observation*, i.e one datum (such as one particular record of one statistic) and columns instead hold information about what groups each statistic belongs to. Same data, different organizations!](https://www.statology.org/wp-content/uploads/2021/12/wideLong1-1.png)

In "wide" format, each row is usually a grouping, site, individual, etc. for which multiple observations are recorded, and each column is everything you know about that entity, perhaps at several different time points.

In "long" format, each row is a single observation (even if you have several observations per entity), and the columns serve to clarify which entity each observations belongs to.

Notice that, regardless, the *exact same information* is being stored in the table--it's just *arrayed* differently!

This is actually a spectrum--data can be organized in a *super* long format, a *super* wide format, or somewhere in between (if the data set has enough complexity to allow it)!

Also, when I say "long" and "wide," I don't mean in the *physical* sense
of how much data you're storing. Obviously, if you have a lot of data,
you're going to have a lot of rows and/or columns potentially, so your data set might
*look* "long" even though it's *organized* in a "wide" format.

::: challenge
**Important**: One consequence of data organization is that it
influences the ease (or difficulty!) of using a programming language
like R to manipulate your data.

Consider: How would you use `dplyr`'s `mutate()` function to calculate a
"points per rebound" value for each team using the "wide" format data set shown
above? How about for the "long" format data set?

Then, consider: How would you use `dplyr`'s `summarize()` function to get an
average number of assists across all teams using the "wide" format data set shown
above? How about the "long" format data set?

::: solution
With the data in "wide" format, using `mutate()` to calculate "points
per rebound" for each team would be easy because the two values for each
team (`Points` and `Rebounds`) are in the same row. So, we'd do
something like this:


``` r
dataset %>% 
  mutate(pts_per_rebound = Points/Rebounds)
```

However, with the data in "long" format, it's not at all obvious how (or
even if) one could use `mutate()` to get what we want. The numbers we're
trying to slam together are in different rows now. This demonstrates
that, in some ways, `mutate()` is a "rowwise" tool--its designed for
operations that leverage data stored more "widely."

Similarly, with the data in "wide" format, using `summarize()` to
calculate an average number of `Assists` across all teams would be easy
because all the numbers we're trying to smash together are in the same
column. So, we'd do something like this:


``` r
dataset %>% 
  summarize(mean_assists = mean(Assists))
```

Yes, you can use `summarize()` without using `group_by()` if you have no
groups to group by!

However, once again, the "long" format version presents difficulties--there
are numbers in the `Value` column we'd need to avoid here to calculate our
average. So, it'd require us to use `filter()` first to remove these:


``` r
dataset %>% 
  filter(Variable == Assists) %>%  #Notice I have to filter by the Variable column...
  summarize(mean_assists = mean(Value)) #...But take the mean of the Value column.
```

So, `summarize()` *seems* like a tool also designed for "wide" format
data, but that's only because our wide-format data have *already* been grouped
by team. In fact, `group_by()` and `summarize()` are best thought of as
tools designed to work best with "long" format data because we need to be able to easily sub-divide our data into groups to then be able to summarize effectively and long formats have more columns for "grouping variables."

This whole exercise is designed to show that while, to *some* extent, data
organization is "personal preference," it also has implications for how
we manipulate our data (and how easy or hard it will be to do so).
:::
:::

While this is perhaps over-generalizing, I would say that **humans tend
to prefer reading and recording data in "wider" formats**. Notice that,
when we report data in tables or record data on data sheets, we tend to do so across rows rather than down columns. Recording data in long format, in particular, tends to feel tedious because it requires us to fill out "grouping variable" columns many times with the same information. 

However, **computers tend to be designed to "prefer" manipulating data
in "longer" formats** (regardless of what the previous example may have
led you to believe!). Computers don't "see" natural groupings in data
like humans can, so they count on having columns that clarify these
groups (like `continent` and `country` do in the gapminder dataset), and
those types of columns only exist in "longer" formats. In particular,
many `dplyr` operations will actually be easier if your data are in
"long" format, and `ggplot2` *especially* expects "long" format to
render plots with various groups correctly.

This may seem like a dilemma--we're torn between how *we'd* prefer the
data to look and how *R* would prefer them to look. But, remember, it's
all the same data, just arranged differently. So, it seems like it'd be
possible to "reshape" our data to suit both needs. That's, in
part, what the `tidyr` package's functions `pivot_longer()` and
`pivot_wider()` are for.

## Preparation and setup

Note: This lesson uses the **gapminder data set**. This data set can be
downloaded
[here](https://drive.google.com/file/d/1sBkIJlC_6fImNZhyV0XfsllyUD-hSmoS/view?usp=sharing).
Make sure to load the data set into your global environment before
continuing.


``` r
gap = read.csv("data/gapminder_data.csv", header = TRUE)
```

Also, this lesson will revolve around use of the optional "add-on"
package `tidyr`. `tidyr` is a package in the "tidyverse"--an array of
useful tools that are designed to look similar and work well together
and that make R much more powerful to use but also a lot different than
"Base R." Everything we do in this lesson can be done in "Base R"
instead, but it won't (typically) be as efficient, easy, or clear!

`tidyr`, like many add-on packages, is **not** installed with R. So, we
must install it before we can use it:


``` r
install.packages("tidyr") #Only run once!
```

You only need to install a package once, so there's no need to run the
above command more than once. However, packages are updated
occasionally. When updates are available, you can re-install new
versions using the same `install.packages()` function.

Once a package is installed, it still isn't "turned on" by default. So,
to turn on `tidyr` so that we can access its unique features, we use the
`library()` function:


``` r
library(tidyr)
```

The above command must be run every time you start up a new session of R
and want access to `tidyr`'s features!

We'll also need the `dplyr` package here so that we can use pipes. See the `dplyr` lesson for details!




## Minding the Gapminder

::: challenge
Let's look at the gapminder data set once more to remind ourselves of
its structure and contents:


``` r
head(gap)
```

``` output
      country year      pop continent lifeExp gdpPercap
1 Afghanistan 1952  8425333      Asia  28.801  779.4453
2 Afghanistan 1957  9240934      Asia  30.332  820.8530
3 Afghanistan 1962 10267083      Asia  31.997  853.1007
4 Afghanistan 1967 11537966      Asia  34.020  836.1971
5 Afghanistan 1972 13079460      Asia  36.088  739.9811
6 Afghanistan 1977 14880372      Asia  38.438  786.1134
```

Consider: Is the gapminder data set in "long" format or "wide" format?
Why do you say that?

::: solution
Sorry, this is *sort of* a trick question because the right answer is
"both" (or "neither").

On the one hand, the gapminder data set is not *as* long as it *could* be.
For example, right now, there are three columns containing numeric data
(`pop`, `lifeExp`, and `gdpPercap`). We *could* instead have a single
`Value` column to hold these data, as we had with our fake sports data
earlier, and a second column that lists what kind of `Value` each row
represents.

On the other hand, the gapminder data set is also not *as* wide as it
*could* be either. For example, right now, there are three "grouping
variables" (`country`, `year`, and `continent`). We *could* instead have
a single row per `country` and have separate
columns for the data from each year (i.e., `pop1952`, `pop1957`,
`pop1962`, etc.).

So, "data orientation" is a spectrum, and the gapminder data set exists
sort of in the middle of it! Hold onto this idea--in the rest of this lesson,
we're going to see how to **make** those longer and wider versions we
just imagined!
:::
:::

## PIVOT_LONGER

We'll begin by seeing how to make our data even longer than it is now by
combining all the values in the `pop`, `lifeExp`, and `gdpPercap`
columns into a single column called `Value`. We'll then add a second
column (officially called a "key" column) that clarifies which
`Statistic` is being reported in the `Value` column in that row.

The `tidyr` verb that corresponds with this goal is `pivot_longer()`. As
with most `dplyr` verbs (and `tidyverse` verbs in general!), the first
input to `pivot_longer()` is __always__ the data frame you are trying to
reshape (or "pivot"). In this case, that will be our gapminder data set.

After providing our data set, we will provide three more inputs in this particular case:

1.  The columns we are eliminating and collapsing into a single column.
    Here, that will be `pop`, `lifeExp`, and `gdpPercap`. We'll provide
    these inputs to the `cols` *parameter*.

2.  The name of the new column that will hold all the *values* that
    *used* to be held by the columns we're eliminating. So, our
    population, life expectancy, and GDP data in this case. We'll call this new
    column `Value`, and we'll provide this input to the `values_to`
    *parameter*.

3.  The name of the new "key" column that will clarify which statistic
    is being held in the `Value` column in a given row. We'll call this
    new column `Statistic`, and we'll provide this input to the
    `names_to` parameter.

Let's put all this together and see what it looks like!


``` r
gap_longer = gap %>% #Review the dplyr lesson if you forget how to use pipes!
  pivot_longer(cols = c(pop, lifeExp, gdpPercap), #We have to use c() here to bundle the column names together because these three columns are not consecutive.
               values_to = "Value", #Notice the quotation marks--these are essential here!
               names_to = "Statistic")
head(gap_longer)
```

``` output
# A tibble: 6 × 5
  country      year continent Statistic     Value
  <chr>       <int> <chr>     <chr>         <dbl>
1 Afghanistan  1952 Asia      pop       8425333  
2 Afghanistan  1952 Asia      lifeExp        28.8
3 Afghanistan  1952 Asia      gdpPercap     779. 
4 Afghanistan  1957 Asia      pop       9240934  
5 Afghanistan  1957 Asia      lifeExp        30.3
6 Afghanistan  1957 Asia      gdpPercap     821. 
```

The first thing I want you to notice (by looking over in the 'Environment Pane') is that our new data set, `gap_longer` is indeed much longer than before: it's now up to `5112` rows! It also has fewer columns: 5 instead of the 6 we started with. 

::: challenge

The other important thing to notice here is the contents of the `Statistic` column. Where did R get this column's contents from?

::: solution

It used the names of the old columns, the ones we got rid of! It figures if those names were good enough to serve as column names in the original data, they must be good enough to be "keys" in this new organization too!

:::
:::

## PIVOT_WIDER

Now, we'll go the other way--we'll make our gapminder data set wider than it already is. We'll make a data set that has a single row for each `country` (countries will be our groups) and we'll have a different column for every `year` x "Statistic" combo we have for each country.

The `tidyr` verb that corresponds with this goal is `pivot_wider()`, and it works very similarly to `pivot_longer()`. Besides our data frame (first input), we'll provide `pivot_wider()` two more inputs:

1.  The column(s) that we're going to eliminate by spreading their contents out over several new columns. In our new, wider data set, we're going to have a column for each `year` x "Statistic" combination, so we're going to eliminate the current `pop`, `lifeExp`, and `gdpPercap` columns as we know them (though their names will get used to make the names for the new columns we're creating). We'll provide them to the `values_from` *parameter*.
2.  The column that we're going to eliminate by instead inserting it into the names of the new columns we're creating. Here, that's going to be the `year` column (I promise this'll make more sense when you see it!). We'll provide this to the `names_from` *parameter*.

Let's see what this looks like!


``` r
gap_wider = gap %>% 
  pivot_wider(values_from = c(pop, lifeExp, gdpPercap),
              names_from = year)
head(gap_wider)
```

``` output
# A tibble: 6 × 38
  country     continent pop_1952 pop_1957 pop_1962 pop_1967 pop_1972 pop_1977
  <chr>       <chr>        <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>
1 Afghanistan Asia       8425333  9240934 10267083 11537966 13079460 14880372
2 Albania     Europe     1282697  1476505  1728137  1984060  2263554  2509048
3 Algeria     Africa     9279525 10270856 11000948 12760499 14760787 17152804
4 Angola      Africa     4232095  4561361  4826015  5247469  5894858  6162675
5 Argentina   Americas  17876956 19610538 21283783 22934225 24779799 26983828
6 Australia   Oceania    8691212  9712569 10794968 11872264 13177000 14074100
# ℹ 30 more variables: pop_1982 <dbl>, pop_1987 <dbl>, pop_1992 <dbl>,
#   pop_1997 <dbl>, pop_2002 <dbl>, pop_2007 <dbl>, lifeExp_1952 <dbl>,
#   lifeExp_1957 <dbl>, lifeExp_1962 <dbl>, lifeExp_1967 <dbl>,
#   lifeExp_1972 <dbl>, lifeExp_1977 <dbl>, lifeExp_1982 <dbl>,
#   lifeExp_1987 <dbl>, lifeExp_1992 <dbl>, lifeExp_1997 <dbl>,
#   lifeExp_2002 <dbl>, lifeExp_2007 <dbl>, gdpPercap_1952 <dbl>,
#   gdpPercap_1957 <dbl>, gdpPercap_1962 <dbl>, gdpPercap_1967 <dbl>, …
```

As the name correctly suggests, `gap_wider` is indeed much wider than our original gapminder data set: It has 38 columns instead of the original 6. We also have fewer rows: Just 142 (1 per `country`) compared to the original 1704. 

Importantly, our new columns have intuitive, predictable names: `gdpPercap_1972`, `pop_1992`, and `lifeExp1987`, for example. Hopefully this is making more sense now that you've seen it in action!

::: challenge

You've now seen `pivot_longer()` and `pivot_wider()`. Maybe you've noticed that they seem like "opposites?" They are! *They're designed to "undo" the other's work*, in fact!

So, use `pivot_wider()` to "rewind" `gap_longer` back to the organization of our original data set.

::: solution

This task is thankfully *relatively* easy. We tell R that it should pull the names for the new columns in our wider table from the `Statistic` column, then pull the values for those new columns from the old `Value` column:


``` r
gap_returned1 = gap_longer %>% 
  pivot_wider(names_from = Statistic,
              values_from = Value)
head(gap_returned1)
```

``` output
# A tibble: 6 × 6
  country      year continent      pop lifeExp gdpPercap
  <chr>       <int> <chr>        <dbl>   <dbl>     <dbl>
1 Afghanistan  1952 Asia       8425333    28.8      779.
2 Afghanistan  1957 Asia       9240934    30.3      821.
3 Afghanistan  1962 Asia      10267083    32.0      853.
4 Afghanistan  1967 Asia      11537966    34.0      836.
5 Afghanistan  1972 Asia      13079460    36.1      740.
6 Afghanistan  1977 Asia      14880372    38.4      786.
```

:::
:::

::: challenge

Now, use `pivot_longer()` to "rewind" `gap_wider` back to the organization of the original data set. This transformation will be a *little* more complicated, though; you'll need to specify slightly different inputs:

1.  For the `names_to` *parameter*, specify exactly `c(".value", "year")`. `".value"` is a special input value here that has a particular meaning--see if you can guess what it is! You can use `?pivot_longer()` to research the answer, if you'd like.
2.  You'll also need to specify `"_"` as an input for the `names_sep` *parameter*. See if you can guess why. 
3.  Lastly, you won't be needing to specify anything for the `values_to` parameter this time--`".value"` is taking care of the need to put anything there.

::: solution

Here's how we'd use `pivot_longer()` to "rewind" to our original data set:


``` r
gap_returned2 = gap_wider %>% 
  pivot_longer(cols = pop_1952:gdpPercap_2007,
               names_to = c(".value", "year"),
               names_sep = "_")
head(gap_returned2)
```

``` output
# A tibble: 6 × 6
  country     continent year       pop lifeExp gdpPercap
  <chr>       <chr>     <chr>    <dbl>   <dbl>     <dbl>
1 Afghanistan Asia      1952   8425333    28.8      779.
2 Afghanistan Asia      1957   9240934    30.3      821.
3 Afghanistan Asia      1962  10267083    32.0      853.
4 Afghanistan Asia      1967  11537966    34.0      836.
5 Afghanistan Asia      1972  13079460    36.1      740.
6 Afghanistan Asia      1977  14880372    38.4      786.
```
Our inputs for `names_to` were first telling R "the names of the new columns should come from the *first* parts of the names of the columns we're getting rid of." That was the `".values"` bit!

Then, they were telling R "the other column you should make should simply be called `year`."

Lastly, by saying `names_sep = "_"`, we were indicating that R should hack apart the old column names at the underscores (aka *separate* the old names at the `_`) to find the proper bits to use in the new column names. 

So, R pulled apart the old column names, creating `pop`, `lifeExp`, and `gdpPercap` columns from the front halves and then years to fill the new `year` column from the back halves of those old column names. Pretty incredible, huh??

Try to rephrase the above explanation in your own words.

:::
:::

::: keypoints
-   Use the `tidyr` package to reshape the organization of your data.
-   Use `pivot_longer()` to go towards a "longer" layout.
-   Use `pivot_wider()` to go towards a "wider" layout.
:::
