Introduction to Merging Data Tables in R
================
Your Name Here

### 1. Introduction

There are many situations where data sets are broken into multiple
tables, and multiple reasons that this might make sense. Sometimes it is
easier to collect data in different pieces, other times it is to reduce
the file size. Regardless of the reason for splitting data sets into
multiple tables, they should always be formatted in such a way that
there is at least one common column between the tables so that they can
be merged as needed. In this tutorial, we will explore how to use the
`join` functions described in Chapter 13 of the R4DS text to merge these
data tables.

All datasets needed for this tutorial are stored in the “data”
subdirectory of the **inclass265** repository.

**Note**: In the **tidyverse**, most tabular sets of data are **data
frames** stored as **tibbles**. Data frames can store objects of
different classes (e.g. some columns can be text and other columns can
be integers). A synonymous term is **data table**, which is used by some
textbooks and other languages, such as the structured query language
(SQL).

To begin, let’s ensure that the **tidyverse** packages are loaded.

``` r
library(tidyverse)
```

### 2. Example data tables

To illustrate the different `join` functions we will use a small example
of a customer database. We will focus on two tables: **orders** which
contains the order number, customer ID, and date of the order; and
**customers** which contains the customer ID and customer name. These
are intentionally small data tables so that is easier to see how the
`join` statements are working.

``` r
orders <- read_csv("data/orders.csv")
```

    ## Rows: 4 Columns: 3
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): date
    ## dbl (2): order, id
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
orders
```

    ## # A tibble: 4 × 3
    ##   order    id date  
    ##   <dbl> <dbl> <chr> 
    ## 1     1     4 Jan-01
    ## 2     2     8 Feb-01
    ## 3     3    42 Apr-15
    ## 4     4    50 Apr-17

``` r
customers <- read_csv("data/customers.csv")
```

    ## Rows: 6 Columns: 2
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): name
    ## dbl (1): id
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
customers
```

    ## # A tibble: 6 × 2
    ##      id name   
    ##   <dbl> <chr>  
    ## 1     4 Tukey  
    ## 2     8 Wickham
    ## 3    15 Mason  
    ## 4    16 Jordan 
    ## 5    23 Patil  
    ## 6    42 Cox

### 3. Joining data tables

The the `dplyr` package provides six different `join` functions, each
merges two data tables together in a different way. The best way to
understand these functions is to see how each works in our small
example.

All of the functions have the same three arguments:

| argument | explanation                                                                                       |
|----------|---------------------------------------------------------------------------------------------------|
| `x`      | the first (left) table to join                                                                    |
| `y`      | the second (right) table to join                                                                  |
| `by`     | a character vector of variables to join by, notice that the column name must always be in quotes. |

#### inner_join

`inner_join` creates a new table which is restricted to cases where the
values of “by variable” exist in *both* data sets. *All* columns from
both data sets are returned for these cases.

``` r
inner_join(x = orders, y = customers, by = "id")
```

    ## # A tibble: 3 × 4
    ##   order    id date   name   
    ##   <dbl> <dbl> <chr>  <chr>  
    ## 1     1     4 Jan-01 Tukey  
    ## 2     2     8 Feb-01 Wickham
    ## 3     3    42 Apr-15 Cox

**Remark**

In this `inner_join`, we lose the row for order 4 from the **orders**
table because customer 50 does not appear in the **customers** data
table. We also lose lose orders 15, 16 and 23 from the **customers**
table. Note the columns order, date and name are also returned. Any
excess variables included after the join can be removed via a `select()`
command if needed.

#### left_join

`left_join` returns all cases from the `x` data table, regardless of
whether there are matching values of the `by` variable(s) in `y`. I
think of this as a very common kind of “lookup.” If the lookup table
doesn’t include a value for the `by` variable, an NA value is generated.
*All* columns from both data tables are returned for these cases.

``` r
left_join(x = orders, y = customers, by = "id")
```

    ## # A tibble: 4 × 4
    ##   order    id date   name   
    ##   <dbl> <dbl> <chr>  <chr>  
    ## 1     1     4 Jan-01 Tukey  
    ## 2     2     8 Feb-01 Wickham
    ## 3     3    42 Apr-15 Cox    
    ## 4     4    50 Apr-17 <NA>

**Remark**

This new data frame now includes the `name` column. Since customer 50
does not appear in the **customers** data table, an `NA` (missing value)
is used for their name.

#### right_join

`right_join` returns all cases from the `y` data table, regardless of
whether there are matching values of the `by` variable(s) in `x`. All
columns from both data tables are returned for these cases. Note the
rows are ordered by the order of the right-hand (`customers`) data
table.

``` r
right_join(x = orders, y = customers, by = "id")
```

    ## # A tibble: 6 × 4
    ##   order    id date   name   
    ##   <dbl> <dbl> <chr>  <chr>  
    ## 1     1     4 Jan-01 Tukey  
    ## 2     2     8 Feb-01 Wickham
    ## 3     3    42 Apr-15 Cox    
    ## 4    NA    15 <NA>   Mason  
    ## 5    NA    16 <NA>   Jordan 
    ## 6    NA    23 <NA>   Patil

**Remark**

We have added the `order` and `date` columns to the **customers** data
table. Customers 15, 16, and 23 did not make purchases during this time
frame, so missing values (`NA`s) are used for their `order` and `date`
values.

#### full_join

`full_join` returns all rows and columns from both `x` and `y`.

``` r
full_join(x = orders, y = customers, by = "id")
```

    ## # A tibble: 7 × 4
    ##   order    id date   name   
    ##   <dbl> <dbl> <chr>  <chr>  
    ## 1     1     4 Jan-01 Tukey  
    ## 2     2     8 Feb-01 Wickham
    ## 3     3    42 Apr-15 Cox    
    ## 4     4    50 Apr-17 <NA>   
    ## 5    NA    15 <NA>   Mason  
    ## 6    NA    16 <NA>   Jordan 
    ## 7    NA    23 <NA>   Patil

**Remark**

We have fully merged the **orders** and **customers** data tables; thus,
we get all of the columns and all of the rows from both data tables.
`NA`s fill in the necessary values for customers not making purchases
and for orders without a customer record.

#### semi_join

`semi_join` returns all rows from the `x` data table where there are
matching values of the `by` variable(s) in `y`, and only the columns
from `x`.

``` r
semi_join(x = orders, y = customers, by = "id")
```

    ## # A tibble: 3 × 3
    ##   order    id date  
    ##   <dbl> <dbl> <chr> 
    ## 1     1     4 Jan-01
    ## 2     2     8 Feb-01
    ## 3     3    42 Apr-15

**Remark**

We lose the row for order 4 because customer 50 does not appear in the
**customers** data table.

#### inner_join vs. semi_join

Above, the `inner_join` and `semi_join` returned the same number of
rows, but this will not always be the case. For example, suppose that
customer 42 also placed an order on May-01 so that we have multiple
orders from the same customer.

``` r
extra_order <- data.frame(order = 5, id = 42, date = "May-01")
extra_order
```

    ##   order id   date
    ## 1     5 42 May-01

``` r
orders2 <- rbind(orders, extra_order)
orders2
```

    ## # A tibble: 5 × 3
    ##   order    id date  
    ##   <dbl> <dbl> <chr> 
    ## 1     1     4 Jan-01
    ## 2     2     8 Feb-01
    ## 3     3    42 Apr-15
    ## 4     4    50 Apr-17
    ## 5     5    42 May-01

``` r
inner_join(x = customers, y = orders2, by = "id")
```

    ## # A tibble: 4 × 4
    ##      id name    order date  
    ##   <dbl> <chr>   <dbl> <chr> 
    ## 1     4 Tukey       1 Jan-01
    ## 2     8 Wickham     2 Feb-01
    ## 3    42 Cox         3 Apr-15
    ## 4    42 Cox         5 May-01

``` r
semi_join(x = customers, y = orders2, by = "id")
```

    ## # A tibble: 3 × 2
    ##      id name   
    ##   <dbl> <chr>  
    ## 1     4 Tukey  
    ## 2     8 Wickham
    ## 3    42 Cox

**Remark**

The result of the `inner_join` includes two rows for customer 42 because
`inner_join` returns all of the columns from both data tables for `id`s
that match. The result of the `semi_join` only returns one row for each
customer because it only returns the rows from **customers** that have
matching `ids` in **orders2**.

#### anti_join

`anti_join` returns all rows from the `x` data table where there are
*not* matching values of the `by` variable(s) in `y`, and only the
columns from `x`.

``` r
anti_join(x = orders, y = customers, by = "id")
```

    ## # A tibble: 1 × 3
    ##   order    id date  
    ##   <dbl> <dbl> <chr> 
    ## 1     4    50 Apr-17

**Remark**

Order 4 is the only order from a customer without a record in the
**customers** data table, so it is the only row of **orders** returned.

``` r
anti_join(x = customers, y = orders, by = "id")
```

    ## # A tibble: 3 × 2
    ##      id name  
    ##   <dbl> <chr> 
    ## 1    15 Mason 
    ## 2    16 Jordan
    ## 3    23 Patil

**Remark**

Customers 15, 16, and 23 did not place orders during this time frame, so
their entries from the **customers** data table are returned.

### 4. Common complications

All of our examples have only used a single column to match the entries
between the data tables, and have also assumed that the columns will
have identical names. This will not always be the case. Below we detail
how to refine what variables you merge by.

- If you want to join by multiple variables, then you need to specify a
  vector of variable names: `by = c("var1", "var2", "var3")`. Here all
  three columns must match in both tables.

- If you want to use all variables that appear in both tables, then you
  can leave the `by` argument blank.

- A common situation is that the variable you wish to join by is not
  named identically in both tables. In this case, you specify
  `by = c("left_var" = "right_var")` to identify the linkage to be made.

Another issue that crops up occasionally is duplicate entries in the
variable(s) that you wish to merge by. We saw one example of this above
when there were two orders from the same customer. In that case the `id`
value was unique in the **customer** table, but not in the **orders**
table. The result of this join is quite logical, as seen above. If,
however, both tables contain duplicate entries in the variable(s) that
you wish to merge by, all possible combinations of these entries are
returned. A simple example for a `full_join` is shown below:

``` r
# Creating example data frames
table1 <- tibble(key = c("a", "a", "b", "b", "c"), var = 1:5)
table1
```

    ## # A tibble: 5 × 2
    ##   key     var
    ##   <chr> <int>
    ## 1 a         1
    ## 2 a         2
    ## 3 b         3
    ## 4 b         4
    ## 5 c         5

``` r
table2 <- tibble(key = c("a", "a", "b", "b", "c"), var = LETTERS[1:5])
table2
```

    ## # A tibble: 5 × 2
    ##   key   var  
    ##   <chr> <chr>
    ## 1 a     A    
    ## 2 a     B    
    ## 3 b     C    
    ## 4 b     D    
    ## 5 c     E

``` r
# A full join
full_join(x = table1, y = table2, by = "key")
```

    ## # A tibble: 9 × 3
    ##   key   var.x var.y
    ##   <chr> <int> <chr>
    ## 1 a         1 A    
    ## 2 a         1 B    
    ## 3 a         2 A    
    ## 4 a         2 B    
    ## 5 b         3 C    
    ## 6 b         3 D    
    ## 7 b         4 C    
    ## 8 b         4 D    
    ## 9 c         5 E

In this situation, the results for `left_join`, `right_join`, and
`full_join` will be identical.

### 5. On Your Own

The files `books.csv`, `authors.csv`, and `book_authors.csv` give
details about the planned summer reading of a statistics student.
`books.csv` provides details for each book (isbn, title, year, and
genre), `authors.csv` provides details about each author (authorid,
first name, last name, and nationality), and `book_authors.csv` provides
the author identification (`authorid`) for each isbn (books with
multiple authors will have multiple rows).

1.  Use `read_csv()` to read the three files into R, naming them
    **books**, **authors**, and **book_authors**.

2.  Use the appropriate `join` statement to add the ISBNs to the
    **authors** data table. Why does the resulting data frame have 31
    rows instead of 11?

3.  To eliminate the duplicate rows of your data frame from \#6 (which
    we’ll assume you named **df2**) run the following code (change the
    object names to align with your code as necessary):

        df2 <- unique(df2)

4.  Use the appropriate `join` statement to add the author information
    table from \#3 to the **books** data table.

5.  Are there any authors in the **authors** data table that do not
    correspond to books in the **books** data table? Use an appropriate
    join statement to do this.

6.  After reading *A Game of Thrones* the student decides to read the
    rest of the series over the summer. `books2.csv` contains the
    updated books on the student’s reading list. Read this file into R,
    naming it **books2**.

7.  Use the same join statement that you did in \#4, but using
    **books2** rather than **books**.

### 6. Additional Resources

- [RStudio’s data transformation cheat
  sheet](https://github.com/rstudio/cheatsheets/raw/master/data-transformation.pdf)
  provides a summary of commands used to combine data sets.

- [`dplyr` vignette on two table
  verbs](https://cran.r-project.org/web/packages/dplyr/vignettes/two-table.html)
  provides additional examples of merging data tables.

### 7. Acknowledgement

<div class="footnote">

Source: Tutorial adapted from [R Tutorials in Data
Science](https://github.com/ds4stats/r-tutorials).

</div>
