---
output: hugodown::hugo_document

slug: tune-0-1-2
title: tune 0.1.2
date: 2020-11-23
author: Max Kuhn
description: >
    A new version of the tune package contains numerous new features. 

photo:
  url: https://unsplash.com/photos/75UqJ3X4VwA
  author: Leo Wieling

# one of: "deep-dive", "learn", "package", "programming", or "other"
categories: [package] 
tags: [tidymodels,workflows,tune]
---

<!--
TODO:
* [ ] Pick category and tags (see existing with `post_tags()`)
* [ ] Find photo & update yaml metadata
* [ ] Create `thumbnail-sq.jpg`; height and width should be equal
* [ ] Create `thumbnail-wd.jpg`; width should be >5x height
* [ ] `hugodown::use_tidy_thumbnail()`
* [ ] Add intro sentence
* [ ] `use_tidy_thanks()`
-->

We're pleased to announce the release of version 0.1.2 of the [tune](https://tune.tidymodels.org/) package. tune is a tidy interface for optimizing model tuning parameters. 

You can install it from CRAN with:


```r
install.packages("tune")
```

There is a lot to discuss! So much that this is the first of three blog posts. Here, we'll show off most of the new features. The two other blog posts will talk about how to benefit from sparse matrices with tidymodels and improvements to parallel processing. 

## Pick a class level

Deciding how to define the event of interest for two-class classification models is a major pain. Sometimes the second level of the factor is assumed to be the event of interest, but this is a vestigial notion almost entirely driven by how things were in The Old Days when outcome classes were encoded as zero and one. Thankfully, we've evolved significantly since those days. tidymodels assumes that the first factor level is the event as a default. 

However, we want to accommodate multiple preferences. Previously, there was a global option that you could set to decide whether the first or second factor level is the event. We have come to realize that this was not the best idea from a technical standpoint. The new approach uses control arguments to the tune functions to make this specification. For example, `control_grid(event_level = "second")` would change the default when using `tune_grid()`. 

## Adding variables 

There is a [new variable specification interface](https://workflows.tidymodels.org/reference/add_variables.html) in the workflows package called `add_variables()`. This can be a good approach to use if you are not interested in using a recipe or formula to declare which columns are outcomes or predictors. You can now use this interface with the tune package. 

## Gaussian process options

For Bayesian optimization, you can now pass options to `GPfit::GP_fit()` through `tune_bayes()`. If you are a "Go Matérn covariance function or go home" person, this is a nice addition. 

## Augmenting `tune` objects

There is now an [`augment()`](https://broom.tidymodels.org/articles/broom.html) method for `tune_*` objects. This method does not have a data argument and returns the _out-of-sample_ predictions for the object, different from other `augment()` methods you may have used. For objects produced by `last_fit()`, the function returns the test set results. 

## Acknowledgements

Thanks to everyone who contributed code or filed issues since the last version: [&#x0040;AndrewKostandy](https://github.com/AndrewKostandy), [&#x0040;bloomingfield](https://github.com/bloomingfield), [&#x0040;cespeleta](https://github.com/cespeleta), [&#x0040;cimentadaj](https://github.com/cimentadaj), [&#x0040;DavisVaughan](https://github.com/DavisVaughan), [&#x0040;dmalkr](https://github.com/dmalkr), [&#x0040;EmilHvitfeldt](https://github.com/EmilHvitfeldt), [&#x0040;hnagaty](https://github.com/hnagaty), [&#x0040;jcpsantiago](https://github.com/jcpsantiago), [&#x0040;juliasilge](https://github.com/juliasilge), [&#x0040;kbzsl](https://github.com/kbzsl), [&#x0040;kelseygonzalez](https://github.com/kelseygonzalez), [&#x0040;matthewrspiegel](https://github.com/matthewrspiegel), [&#x0040;mdneuzerling](https://github.com/mdneuzerling), [&#x0040;MxNl](https://github.com/MxNl), [&#x0040;SeeNewt](https://github.com/SeeNewt), [&#x0040;simonschoe](https://github.com/simonschoe), [&#x0040;Steviey](https://github.com/Steviey), [&#x0040;topepo](https://github.com/topepo), [&#x0040;trevorcampbell](https://github.com/trevorcampbell), and [&#x0040;UnclAlDeveloper](https://github.com/UnclAlDeveloper)
