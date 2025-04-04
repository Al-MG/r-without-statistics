---
output: html_document
editor_options: 
  chunk_output_type: console
---




# Crafting High-Quality Tables {#tables-chapter}

In his book *Fundamentals of Data Visualization*, Claus Wilke writes that tables are "an important tool for visualizing data." This statement might seem odd. Tables are often considered the opposite of data visualizations such as plots: a place to dump numbers for the few nerds who care to read them. But Wilke sees things differently. 

Tables should not be data dumps devoid of design. While bars, lines, and points in graphs are visualizations, so too are numbers in a table, and we should care about their appearance. As an example, take a look at the tables made by reputable news sources; data dumps these are not. Media organizations, whose job it is to communicate effectively, pay a lot of attention to table design. But elsewhere, because of tables’ apparent simplicity, Wilke writes, "they may not always receive the attention they need."

Many people use Microsoft Word to make tables, a strategy that has potential pitfalls. Wilke found that his version of Word included 105 built-in table styles. Of those, around 80 percent, including the default style, violated some key principle of table design. The good news is that R is a great tool for making high-quality tables. It has a number of packages for this purpose, and within these packages, several functions designed to make sure your tables follow important design principles. 
Moreover, if you’re writing reports in R Markdown (which you’ll learn about in Chapter \@ref(rmarkdown-chapter)), you can include code that will generate a table when you export your document. By working with a single tool to create tables, text, and other visualizations, you won’t have to copy and paste your data, lowering the risk of human error.

This chapter examines table design principles and shows you how to apply them to your tables using R’s `gt` package, one of the most popular table-making packages (and, as you’ll soon see, one that uses good design principles by default). These principles, and the code in this chapter, are adapted from Tom Mock’s blog post "10+ Guidelines for Better Tables in R." Mock works at Posit, the company that makes RStudio, and has become something of an R table connoisseur. We’ll walk through examples of Mock’s code to show how small tweaks can make a big difference.


## Creating a Data Frame {-}

We’ll begin by creating a data frame that we can use to make tables throughout this chapter. First, let’s load the packages we need. We’ll rely on the `tidyverse` package for general data manipulation functions, `gapminder` for the data we’ll use, `gt` to make the tables, and `gtExtras` to do some formatting on our tables:


```r
library(tidyverse)
library(gapminder)
library(gt)
library(gtExtras)
```

As we saw in Chapter \@ref(data-viz-chapter), the `gapminder` package provides country-level demographic statistics. To make a data frame for our table, let’s use just a few countries (the first four, in alphabetical order: Afghanistan, Albania, Algeria, and Angola) and three years (1952, 1972, and 1992). The `gapminder` data has many years, but we need only a few to demonstrate table-making principles. Here is the code to make the data frame called `gdp`: 


```r
gdp <- gapminder %>%
  filter(country %in% c("Afghanistan", "Albania", "Algeria", "Angola")) %>%
  select(country, year, gdpPercap) %>%
  mutate(country = as.character(country)) %>%
  pivot_wider(
    id_cols = country,
    names_from = year,
    values_from = gdpPercap
  ) %>%
  select(country, `1952`, `1972`, `1992`) %>%
  rename(Country = country)
```

Let’s see what `gdp` looks like:


```
#> # A tibble: 4 × 4
#>   Country      `1952`  `1972`  `1992`
#>   <chr>         <dbl>   <dbl>   <dbl>
#> 1 Afghanistan  779.45  739.98  649.34
#> 2 Albania     1601.1  3313.4  2497.4 
#> 3 Algeria     2449.0  4182.7  5023.2 
#> 4 Angola      3520.6  5473.3  2627.8
```

Now that we have some data, let’s use it to make a table.

## Table Design Principles {-}

Unsurprisingly, the principles of good table design are similar to those for data visualization more generally. In this section, we cover six of the most important.

### Principle One: Minimize Clutter {-}

As with data visualization, one of the most important principles of table design is to minimize clutter. One way we can do this is by removing unnecessary elements. A common source of clutter in tables is gridlines. Often, you see tables that look like Figure \@ref(fig:table-with-gridlines).

[F05001.png]

![(\#fig:table-with-gridlines)A table with gridlines everywhere](../temp/F05001.png){width=100%}

Having gridlines around every single cell in our table is unnecessary and creates visual clutter that distracts from the goal of communicating clearly. A table with minimal or even no gridlines (Figure \@ref(fig:table-horizontal-gridlines)) is a much more effective communication tool.

[F05002.png]

![(\#fig:table-horizontal-gridlines)A table with only horizontal gridlines](../temp/F05002.png){width=100%}

I mentioned that `gt` uses good table design principles by default, and this guideline is a great example of it. The second table, with minimal gridlines, requires just two lines of code. We pipe our `gdp` data into the `gt()` function, which creates a table:


```r
gdp %>%
  gt()
```

To add gridlines to every part of the example, we would have to add additional code. Here, the code that follows the `gt()` function adds gridlines:


```r
gdp %>%
  gt() %>%
  tab_style(
    style = cell_borders(
      side = "all",
      color = "black",
      weight = px(1),
      style = "solid"
    ),
    locations = list(
      cells_body(
        everything()
      ),
      cells_column_labels(
        everything()
      )
    )
  ) %>%
  opt_table_lines(extent = "none")
```

Since I don’t recommend taking this approach, I won’t walk through this code. However, if we wanted to remove additional gridlines, we could use the following: 


```r
gdp %>%
  gt() %>%
  tab_style(
    style = cell_borders(color = "transparent"),
    locations = cells_body()
  )
```

The `tab_style()` function uses a two-step approach. First, it identifies the style we want to modify (in this case, the borders); next, it tells the function where to apply these styles. Here, we tell `tab_style()` that we want to modify the borders using the `cell_borders()` function, making our borders transparent. Then, we say that we want this transformation to apply to the `cells_body()` location. Other options include `cells_column_labels()` for the first row.

Doing this gives us a table with no gridlines at all in the body (Figure \@ref(fig:table-no-body-gridlines)).

[F05003.png]

![(\#fig:table-no-body-gridlines)Figure 1-3	A table with gridlines only on the header row and the bottom](../temp/F05003.png){width=100%}

Let’s save this table as an object called `table_no_gridlines` so that we can add onto it later.



### Principle Two: Differentiate the Header from the Body {-}

While reducing clutter is an important goal, going too far can have negative consequences. A table with no gridlines at all can make it hard to differentiate between the header row and the table body. Take Figure \@ref(fig:table-no-gridlines-at-all), for example.

[F05004.png]

![(\#fig:table-no-gridlines-at-all)A table with all gridlines removed](../temp/F05004.png){width=100%}

We’ve already covered how to use appropriate gridlines. But by making the header row bold, we can make it stand out even more: 


```r
table_no_gridlines %>%
  tab_style(
    style = cell_text(weight = "bold"),
    locations = cells_column_labels()
  )
```

We start with the `table_no_gridlines` object (our saved table from earlier). Then, we apply our formatting with the `tab_style()` function using two steps. First, we say that we want to alter the text by using the `cell_text()` function to set the weight to bold. Second, we say we want this to happen only to the header row using the `cells_column_labels()` function. In Figure \@ref(fig:table-bolded-header), we can see what our table looks like with headers bolded.

[F05005.png]

![(\#fig:table-bolded-header)Table with header row bolded](../temp/F05005.png){width=100%}

Let’s save this table as `table_bold_header` in order to add additional formatting.




### Principle Three: Align Appropriately {-}

A third principle of high-quality table design is appropriate alignment. Specifically, numbers in tables should be right-aligned. Tom Mock explains why:

> Left-alignment or center-alignment of numbers impairs the ability to clearly compare numbers and decimal places. Right-alignment lets you align decimal places and numbers for easy parsing.

Let’s see this principle in action. In Figure \@ref(fig:table-cols-aligned-lcr), we’ve left-aligned the 1952 column, center-aligned the 1972 column, and right-aligned the 1992 column. You can see how much easier it is to compare the values in the 1992 column than in the other two columns. In both 1952 and 1972, it is much more difficult to compare the numeric values because the numbers in the same columns (the tens place, for example) are not in the same vertical position. In the 1992 column, however, the number in the tens place in Afghanistan (4) aligns with the number in the tens place in Albania (9) and all other countries. This vertical alignment makes it easier to scan the table.

[F05006.png]

![(\#fig:table-cols-aligned-lcr)Table with year columns aligned left, center, and right](../temp/F05006.png){width=100%}

As with other tables, we actually have to override the defaults to get the `gt` package to misalign the columns, as you can see in the following code. 


```r
table_bold_header %>%
  cols_align(
    align = "left",
    columns = 2
  ) %>%
  cols_align(
    align = "center",
    columns = 3
  ) %>%
  cols_align(
    align = "right",
    columns = 4
  )
```

By default, `gt` will right-align numeric values. Don’t change anything, and you’ll be golden.

Right alignment is best practice for numeric columns, but for text columns, use left alignment. As Jon Schwabish points out in his article "Ten Guidelines for Better Tables" in the *Journal of Benefit-Cost Analysis*, it’s much easier to read longer text cells when they are left aligned. To illustrate the benefit of left-aligning text, let’s add a country with a long name to the table. I’ve added Bosnia and Herzegovina and saved this as a data frame called `gdp_with_bosnia`. You’ll see that I’m using nearly the same code as I used to create the `gdp` data frame above:




```r
gdp_with_bosnia
#> # A tibble: 5 × 4
#>   Country                 `1952`  `1972`  `1992`
#>   <chr>                    <dbl>   <dbl>   <dbl>
#> 1 Afghanistan             779.45  739.98  649.34
#> 2 Albania                1601.1  3313.4  2497.4 
#> 3 Algeria                2449.0  4182.7  5023.2 
#> 4 Angola                 3520.6  5473.3  2627.8 
#> 5 Bosnia and Herzegovina  973.53 2860.2  2546.8
```

Now take the `gdp_with_bosnia` data frame and create a table with the country column center aligned. In the table in Figure \@ref(fig:table-country-centered), it’s hard to scan the country names, and that center-aligned column just looks a bit weird.

[F05007.png]

![(\#fig:table-country-centered)A table with country column center aligned](../temp/F05007.png){width=100%}

This is another example where we’ve had to change the `gt` defaults to mess things up. In addition to right-aligning numeric columns by default, `gt` left-aligns character columns. So, if we don’t touch anything, it will give us the alignment we’re looking for (Figure \@ref(fig:table-country-left)).

[F05008.png]

![(\#fig:table-country-left)A table with country column left aligned](../temp/F05008.png){width=100%}

If you ever do want to override the default alignments, you can use the `cols_align()` function. For example, here is how to make the table with the country names center aligned:


```r
gdp_with_bosnia %>%
  gt() %>%
  tab_style(
    style = cell_borders(color = "transparent"),
    locations = cells_body()
  ) %>%
  tab_style(
    style = cell_text(weight = "bold"),
    locations = cells_column_labels()
  ) %>%
  cols_align(
    columns = "Country",
    align = "center"
  )
```

Within this function, we use the `columns` argument to tell gt which columns to align and the `align` argument to select our alignment (`left`, `right`, or `center`).

### Principle Four: Use the Right Level of Precision {-}

In all of the tables we’ve made so far, we’ve used the data exactly as it came to us. The numeric columns, for example, extend their data to four decimal places. This is almost certainly too many. Having more decimal places makes a table harder to read, so you should always strike a balance between what Jon Schwabish describes as "necessary precision and a clean, spare table." 

Here is another way I’ve heard this principle described: If adding additional decimal places would change some action, keep them; otherwise, take them out. In my experience, people tend to leave too many decimal places in, putting too much importance on a very high degree of accuracy (and, in the process, reducing the legibility of their tables).

In our GDP table, we can use the `fmt_currency()` function to format the numeric values. The `gt` package has a whole series of functions for formatting values in tables, all of which start with `fmt_`. In the following code, we apply `fmt_currency()` to the 1952, 1972, and 1992 columns, then use the decimals argument to tell `fmt_currency()` to format the values with zero decimal places. After all, the difference between a GDP of \$799.4453 and \$779 is unlikely to lead to different decisions, so I’m comfortable with sacrificing precision for legibility:


```r
table_bold_header %>%
  fmt_currency(
    columns = c(`1952`, `1972`, `1992`),
    decimals = 0
  )
```

We end up with values formatted as dollars. The `fmt_currency()` function automatically adds a thousands-place comma to make the values even easier to read (Figure \@ref(fig:table-whole-numbers)).

[F05009.png]

![(\#fig:table-whole-numbers)A table with numbers rounded to whole numbers and dollar sign added](../temp/F05009.png){width=100%}

Now save your table for reuse as table_whole_numbers.




### Principle Five: Use Color Intentionally {-}

So far, our table hasn’t used any color. We’ll add some now to highlight outlier values. Especially for readers who want to scan your table, highlighting outliers with color can help significantly. Let’s make the highest value in the year 1952 a different color. To do this, we again use the `tab_style()` function: 


```r
table_whole_numbers %>%
  tab_style(
    style = cell_text(
      color = "orange",
      weight = "bold"
    ),
    locations = cells_body(
      columns = `1952`,
      rows = `1952` == max(`1952`)
    )
  )
```

This function uses `cell_text()` to both change the color of the text to orange and make it bold. Within the `cells_body()` function, we use the locations() function specify the columns and rows to which we want to apply our change. You can see that we’ve simply set the `columns` argument to the year whose values we’re changing. To set the rows, we need a more complicated formula. The code `rows = `1952` == max(`1952`)` causes the text transformation to occur in rows whose value is equal to the maximum value in that year. 

If we repeat this code for the 1972 and 1992 columns, we generate the result shown in Figure \@ref(fig:table-highlight-value). 

[F05010.png]

![(\#fig:table-highlight-value)A table with color added to show the highest value in each year](../temp/F05010.png){width=100%}

The `gt` package makes it straightforward to add color to highlight outlier values. 



### Principle Six: Add Data Visualization Where Appropriate {-}

Adding color to highlight outliers is one way to help guide the reader’s attention. Another way is to incorporate graphs into tables. Tom Mock developed an add-on package for `gt` called `gtExtras` that makes it possible to do just this. For example, in our table, we might want to show how the GDP of each country changes over time. To do that, we’ll add a new column that visualizes this trend using a *sparkline* (essentially, a simple line chart): 


```r
gdp_with_trend <- gdp %>%
  group_by(Country) %>%
  mutate(Trend = list(c(`1952`, `1972`, `1992`))) %>%
  ungroup()
```

The `gt_plt_sparkline()` function requires us to provide the values needed to make the sparkline in a single column. To accomplish this, we create a variable called `Trend`, using `group_by()` and `mutate()`, to hold a list of the values for each country. For Afghanistan, for example, Trend would contain 779.4453145, 739.9811058, and 649.3413952. We save this data as an object called `gdp_with_trend`.

Now we create our table as before, but add the `gt_plt_sparkline()` function to the end of the code. Within this function, we specify which column to use to create the sparkline (`Trend`):


```r
gdp_with_trend %>%
  gt() %>%
  tab_style(
    style = cell_borders(color = "transparent"),
    locations = cells_body()
  ) %>%
  tab_style(
    style = cell_text(weight = "bold"),
    locations = cells_column_labels()
  ) %>%
  fmt_currency(
    columns = c(`1952`, `1972`, `1992`),
    decimals = 0
  ) %>%
  tab_style(
    style = cell_text(
      color = "orange",
      weight = "bold"
    ),
    locations = cells_body(
      columns = `1952`,
      rows = `1952` == max(`1952`)
    )
  ) %>%
  tab_style(
    style = cell_text(
      color = "orange",
      weight = "bold"
    ),
    locations = cells_body(
      columns = `1972`,
      rows = `1972` == max(`1972`)
    )
  ) %>%
  tab_style(
    style = cell_text(
      color = "orange",
      weight = "bold"
    ),
    locations = cells_body(
      columns = `1992`,
      rows = `1992` == max(`1992`)
    )
  ) %>%
  gt_plt_sparkline(
    column = Trend,
    label = FALSE,
    palette = c("black", "transparent", "transparent", "transparent", "transparent")
  )
```

We set `label = FALSE` to remove text labels that `gt_plt_sparkline()` adds by default, then add a `palette` argument to make the sparkline black and all other elements of it transparent. (By default, the function will make different parts of the sparkline different colors.) The stripped-down sparkline in Figure \@ref(fig:table-sparkline) allows the reader to see the trend for each country at a glance.

[F05011.png]

![(\#fig:table-sparkline)A table with sparkline added to show trend over time](../temp/F05011.png){width=100%}

The `gtExtras` package can do way more than merely create sparklines. Its set of theme functions allow you to make your tables look like those published by *FiveThirtyEight*, *The New York Times*, *The Guardian*, and other news outlets. As an example, try removing the formatting we’ve applied so far and instead use the *gt_theme_538()* function to style the table:


```r
gdp %>%
  group_by(Country) %>%
  mutate(Trend = list(c(`1952`, `1972`, `1992`))) %>%
  ungroup() %>%
  gt() %>%
  tab_style(
    style = cell_text(
      color = "orange",
      weight = "bold"
    ),
    locations = cells_body(
      columns = `1952`,
      rows = `1952` == max(`1952`)
    )
  ) %>%
  tab_style(
    style = cell_text(
      color = "orange",
      weight = "bold"
    ),
    locations = cells_body(
      columns = `1972`,
      rows = `1972` == max(`1972`)
    )
  ) %>%
  tab_style(
    style = cell_text(
      color = "orange",
      weight = "bold"
    ),
    locations = cells_body(
      columns = `1992`,
      rows = `1992` == max(`1992`)
    )
  ) %>%
  fmt_currency(
    columns = c(`1952`, `1972`, `1992`),
    decimals = 0
  ) %>%
  gt_plt_sparkline(
    column = Trend,
    label = FALSE,
    palette = c("black", "transparent", "transparent", "transparent", "transparent")
  ) %>%
  gt_theme_538()
```

Take a look at tables on the FiveThirtyEight website, and you’ll see similarities to the one in Figure \@ref(fig:table-fivethirtyeight).

[F05012.png]

![(\#fig:table-fivethirtyeight)A table redone in FiveThirtyEight style](../temp/F05012.png){width=100%}

Add-on packages like `gtExtras` are common in the table-making landscape. If you’re working with the `reactable` package to make interactive tables, for example, you can also use the `reactablefmtr` to add interactive sparklines, themes, and more. You’ll learn more about making interactive tables in Chapter \@ref(websites-chapter). 

## Conclusion {-}

Many of the tweaks we made to our table are quite subtle. Changes like removing excess gridlines, bolding header text, right-aligning numeric values, and adjusting the level of precision can often go unnoticed, but if you skip them, your table will be far less effective. Our final product isn’t flashy, but it does communicate clearly.

We used the gt package to make our high-quality table, and as we’ve repeatedly seen, this package has good defaults built in. Often, you don’t need to change much in your code to make effective tables. But no matter which package you use, it’s essential to treat tables as worthy of just as much thought as other kinds of data visualization. 

In Chapter \@ref(rmarkdown-chapter), you’ll learn how to create reports using R Markdown, which can integrate your tables directly into the final document. What’s better than using just a few lines of code to make publication-ready tables?

## Learn More {-}

Consult the following resources to learn about table design principles and how to make high-quality tables with the `gt` package: 

"Ten Guidelines for Better Tables" by Jon Schwabish (*Journal of Benefit-Cost Analysis*, 2020), https://doi.org/10.1017/bca.2020.11

"10+ Guidelines for Better Tables in R" by Tom Mock (2020), https://themockup.blog/posts/2020-09-04-10-table-rules-in-r/

"Creating beautiful tables in R with {gt}" by Albert Rapp (2022), https://gt.albert-rapp.de/
