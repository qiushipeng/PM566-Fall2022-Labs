Assignment 04 - HPC and SQL
================
Qiushi Peng
2022-11-15

## HPC

#### Problem 1: Make sure your code is nice

Rewrite the following R functions to make them faster. It is OK (and
recommended) to take a look at Stackoverflow and Google

``` r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}

fun1alt <- function(mat) {
  # YOUR CODE HERE
  rowSums(mat)
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}

fun2alt <- function(mat) {
  # YOUR CODE HERE
  t(apply(dat, MARGIN = 1, FUN = function(x) cumsum(x)))
}


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat) , unit = "milliseconds", check = "equivalent"
)
```

    ## Unit: milliseconds
    ##          expr      min       lq     mean   median       uq      max neval
    ##     fun1(dat) 4.261709 4.484293 4.578102 4.590521 4.653251 4.842584   100
    ##  fun1alt(dat) 2.499043 2.507480 2.536296 2.513647 2.531751 3.867500   100

``` r
# Test for the second
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat) , unit = "milliseconds", check = "equivalent"
)
```

    ## Unit: milliseconds
    ##          expr      min       lq     mean   median       uq      max neval
    ##     fun2(dat) 4.328334 4.377834 4.489937 4.401021 4.424043 12.75400   100
    ##  fun2alt(dat) 2.846209 3.032083 3.247060 3.199438 3.264480 10.30104   100

The last argument, check = “equivalent”, is included to make sure that
the functions return the same result.

#### Problem 2: Make things run faster with parallel computing

The following function allows simulating PI

``` r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

    ## [1] 3.132

In order to get accurate estimates, we can run this function multiple
times, with the following code:

``` r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##  15.594   0.151  15.818

Rewrite the previous code using parLapply() to make it run faster. Make
sure you set the seed using clusterSetRNGStream():

``` r
library(parallel)
cl <- makePSOCKcluster(4) 
clusterSetRNGStream(cl, 1231) # YOUR CODE HERE
system.time({
  clusterExport(cl, "pi") # YOUR CODE HERE
  ans <- unlist(parLapply(cl, 1:4000, sim_pi, n = 10000)) # YOUR CODE HERE
  print(mean(ans))
  stopCluster(cl) # YOUR CODE HERE
})
```

    ## [1] 3.141578

    ##    user  system elapsed 
    ##   0.010   0.001   5.068

## SQL

Setup a temporary database by running the following chunk

``` r
# install.packages(c("RSQLite", "DBI"))

library(RSQLite)
library(DBI)

# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```

When you write a new chunk, remember to replace the r with sql,
connection=con. Some of these questions will reqruire you to use an
inner join. Read more about them here
<https://www.w3schools.com/sql/sql_join_inner.asp>

#### Question 1

How many many movies is there avaliable in each rating category.

``` sql
SELECT rating, COUNT(*) AS count
FROM film
GROUP BY rating
```

| rating | count |
|:-------|------:|
| G      |   180 |
| NC-17  |   210 |
| PG     |   194 |
| PG-13  |   223 |
| R      |   195 |

5 records

#### Question 2

What is the average replacement cost and rental rate for each rating
category.

``` sql
SELECT rating, 
  AVG(replacement_cost) AS avg_replacement_cost,
  AVG(rental_rate) AS avg_rental_rate,
  COUNT(*) AS count
FROM film
GROUP BY rating
```

| rating | avg_replacement_cost | avg_rental_rate | count |
|:-------|---------------------:|----------------:|------:|
| G      |             20.12333 |        2.912222 |   180 |
| NC-17  |             20.13762 |        2.970952 |   210 |
| PG     |             18.95907 |        3.051856 |   194 |
| PG-13  |             20.40256 |        3.034843 |   223 |
| R      |             20.23103 |        2.938718 |   195 |

5 records

#### Question 3

Use table film_category together with film to find the how many films
there are with each category ID

``` sql
SELECT category_id, COUNT(*) AS count
FROM film_category AS a INNER JOIN film AS b
  ON a.film_id = b.film_id
GROUP BY category_id
```

| category_id | count |
|:------------|------:|
| 1           |    64 |
| 2           |    66 |
| 3           |    60 |
| 4           |    57 |
| 5           |    58 |
| 6           |    68 |
| 7           |    62 |
| 8           |    69 |
| 9           |    73 |
| 10          |    61 |

Displaying records 1 - 10

#### Question 4

Incorporate table category into the answer to the previous question to
find the name of the most popular category.

``` sql
SELECT name, COUNT(*) AS count
FROM film_category AS a LEFT JOIN category AS b
ON a.category_id = b.category_id
GROUP BY name
ORDER BY count DESC
```

| name        | count |
|:------------|------:|
| Sports      |    74 |
| Foreign     |    73 |
| Family      |    69 |
| Documentary |    68 |
| Animation   |    66 |
| Action      |    64 |
| New         |    63 |
| Drama       |    62 |
| Sci-Fi      |    61 |
| Games       |    61 |

Displaying records 1 - 10
