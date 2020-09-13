---
title: conflicted 1.0.0
author: Hadley Wickham
date: '2018-09-25'
slug: conflicted-1-0-0
categories:
  - package
photo:
  url: https://unsplash.com/photos/DCtwjzQ9uVE
  author: CloudVisual
tags: []
---




We're deligned to announce the release of  [conflicted](https://github.com/r-lib/conflicted#conflicted) 1.0.0 The goal of conflicted is to provide an alternative way of resolving conflicts caused by ambiguous function names. R handles ambiguity by reporting conflicts when you load a package, but otherwise lets the most recently loaded package win. This can make it hard to detect conflicts, because it's easy to miss the messages since you often load packages at the top of the script, and you don't see a problem until much later. conflicted takes a different approach to resolving ambiguity, instead making every conflict an error and forcing you to explicitly choose which function to use.


```r
library(conflicted)
library(dplyr)

filter(mtcars, am & cyl == 8)
#> Error: [conflicted] `filter` found in 2 packages.
#> Either pick the one you want with `::` 
#> * dplyr::filter
#> * stats::filter
#> Or declare a preference with `conflict_prefer()`
#> * conflict_prefer("filter", "dplyr")
#> * conflict_prefer("filter", "stats")
```

Install conflicted by running:


```r
install.packages("conflicted")
```

The code above shows one of the new features in conflicted 1.0.0: there's now an official way to prefer one function over another for an entire session: `conflict_prefer()`. 


```r
conflict_prefer("filter", "dplyr")
#> [conflicted] Will prefer dplyr::filter over any other package
filter(mtcars, am & cyl == 8)
#>    mpg cyl disp  hp drat   wt qsec vs am gear carb
#> 1 15.8   8  351 264 4.22 3.17 14.5  0  1    5    4
#> 2 15.0   8  301 335 3.54 3.57 14.6  0  1    5    8
```

conflicted has another new function that I'll use to illustrate the changes to conflict resolution: `conflict_scout()`. This function makes it easy to find all the potential conflicts across a set of packages. Let's start by taking a look at the conflicts between the dplyr and stats packages:


```r
conflict_scout(c("dplyr", "stats"))
#> 2 conflicts:
#> * `filter`: [dplyr]
#> * `lag`   : dplyr, stats
```

It shows that `filter()` and `lag()` both have conflicts. What about conflicts between dplyr and the base package?


```r
conflict_scout(c("dplyr", "base"))
#> 4 conflicts:
#> * `intersect`: [dplyr]
#> * `setdiff`  : [dplyr]
#> * `setequal` : [dplyr]
#> * `union`    : [dplyr]
```

You'll notice that there are four conflicting functions, but there's only one package listed. That's because the dplyr functions all obey the "superset" principle: they do everything that the existing base function does, and a little bit more. Here dplyr makes `intersect()`, `setdiff()`, `setequal()` and `union()` into generics so that they can work differently for vectors and data frames. These functions expand the possible set of valid inputs, so 

conflicts now generally expects packages that override functions in base packages to obey the "superset principle", i.e. that `foo::bar(...)` must return the same value of `base::bar(...)` whenever the input is not an error. In other words, if you override a base function you can only extend the API, not change or reduce it. We can see this principle in operation with the  which provides S4 versions of base functions:

The lubridate package has a similar set of conflicts with the base package:


```r
conflict_scout(c("lubridate", "base"))
#> 5 conflicts:
#> * `as.difftime`: [lubridate]
#> * `date`       : [lubridate]
#> * `intersect`  : [lubridate]
#> * `setdiff`    : [lubridate]
#> * `union`      : [lubridate]
```

Unfortunately the definitions that dplyr and lubridate use are not the same so if you load both you'll get some conflicts that you need to resolve. We're planning on fixing this problem with the [generics](https://github.com/r-lib/generics), which we hope to release later this year (generics is inspired by [BiocGenerics](https://bioconductor.org/packages/release/bioc/html/BiocGenerics.html) which does a similar job for S4 generics).


```r
conflict_scout(c("dplyr", "lubridate", "base"))
#> 6 conflicts:
#> * `as.difftime`: [lubridate]
#> * `date`       : [lubridate]
#> * `intersect`  : dplyr, lubridate
#> * `setdiff`    : dplyr, lubridate
#> * `setequal`   : [dplyr]
#> * `union`      : dplyr, lubridate
```

If the arguments of the two functions are not compatible (i.e. the function in the package doesn't include all arguments of the base package), conflicts can tell it doesn't follow the superset principle. Additionally, `dplyr::lag()` fails to follow the superset principle, and is marked as a special case (`lag()` is an acknowledged mistake, unfortunately fixing it now would cause just as many problems).

Another important case where conflicted automatically declares a winner is functions that have been deprecated because they have been moved between packages. For example, in the development version of [devtools](http://devtools.r-lib.org) many functions have been deprecated in favour of the specialised [usethis](http://usethis.r-lib.org) package. Because the old devtools functions call (e.g.) `.Deprecated("usethis::use_appveyor")`, conflicted automatically resolves the conflicts in favour of usethis:


```r
head(conflict_scout(c("devtools", "usethis")))
#> 27 conflicts:
#> * `use_appveyor`       : [usethis]
#> * `use_build_ignore`   : [usethis]
#> * `use_code_of_conduct`: [usethis]
#> * `use_coverage`       : [usethis]
#> * `use_cran_badge`     : [usethis]
#> * `use_cran_comments`  : [usethis]
#> ...
```

## Future plans

I have been automatically loading `conflicted()` in my .Rprofile for several months, and I have found it to catch many problems without creating a large additional burden. For this reason, I'm considering automatically loading it as part of a future version of the tidyverse package. I'd really appreciate it if you'd also try out the conflicted package and let me know how it works for you. It's my belief that it will cause a small amount of short term pain in return for a large benefit in the long run.


