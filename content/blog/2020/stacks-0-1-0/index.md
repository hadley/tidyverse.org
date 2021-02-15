---
output: hugodown::hugo_document

slug: stacks-0-1-0
title: stacks 0.1.0
date: 2020-11-30
author: Simon Couch and Max Kuhn
description: >
    Introducing ensemble learning to the tidymodels.

photo:
  url: https://unsplash.com/photos/3Xd5j9-drDA
  author: Derek Oyen

categories: [package] 
tags:
  - tidymodels
  - stacks
rmd_hash: 2a20527d4bfdf7f9

---

A few months ago, the tidymodels team coordinated a [community survey](https://connect.rstudioservices.com/tidymodels-priorities-survey/README.html) to get a sense for what users most wanted to see next in the [tidymodels](https://www.tidymodels.org/) package ecosystem. One resounding theme from responses was that tidymodels users wanted a framework for tidymodels-aligned model stacking.

![](https://education.rstudio.com/blog/2020/06/tidymodels-internship/priorities.png)

We're excited to announce the release of [stacks](https://stacks.tidymodels.org) 0.1.0, a package for model stacking that aligns with existing functionality in the tidymodels package ecosystem! Model stacking is an ensembling technique that combines the outputs of many models to generate a new model---referred to as an *ensemble* in this package---that generates predictions informed by each of its *members*.

You can install it from CRAN with:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/utils/install.packages.html'>install.packages</a></span>(<span class='s'>"stacks"</span>)
</code></pre>

</div>

To load the package:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/library.html'>library</a></span>(<span class='k'>stacks</span>)
</code></pre>

</div>

A Brief Overview
----------------

The process of a building a model stack goes something like this:

<div style="float:right;position: relative; top: -35px; width: 200px; padding-left: 30px; padding-bottom: 10px;">

![](https://github.com/tidymodels/stacks/raw/main/man/figures/logo.png)

</div>

1.  Define candidate ensemble members using functionality from [rsample](https://rsample.tidymodels.org/), [parsnip](https://parsnip.tidymodels.org/), [workflows](https://workflows.tidymodels.org/), [recipes](https://recipes.tidymodels.org/), and [tune](http://tune.tidymodels.org/)
2.  Initialize a `data_stack` object with [`stacks()`](https://stacks.tidymodels.org/reference/stacks.html)  
3.  Iteratively add candidate ensemble members to the `data_stack` with [`add_candidates()`](https://stacks.tidymodels.org/reference/add_candidates.html)  
4.  Evaluate how to combine their predictions with [`blend_predictions()`](https://stacks.tidymodels.org/reference/blend_predictions.html)  
5.  Fit candidate ensemble members with non-zero stacking coefficients with [`fit_members()`](https://stacks.tidymodels.org/reference/fit_members.html)  
6.  Predict on new data with [`predict()`](https://stacks.tidymodels.org/reference/predict.model_stack.html)

For an example of using stacks to build a stacked ensemble model, see the [package vignette](https://stacks.tidymodels.org/articles/basics.html).

Acknowledgements
----------------

Thank you to [Julie Jung](https://www.jungjulie.com/) for contributing the package's hex sticker as well as research data used in examples throughout the package. Read more about the example data [here](https://stacks.tidymodels.org/reference/tree_frogs.html).

This release was made possible in part by the RStudio internship program, which allowed me ([Simon Couch](https://twitter.com/simonpcouch)) to work on developing stacks full-time throughout July. I'm also grateful for support from Dr. Kelly McConville as well as the [Reed College Mathematics Department](https://www.reed.edu/math/), where this package will partially fulfill my undergraduate senior thesis requirement.

Also, we thank those who have tested and provided feedback on the developmental versions of the package over the last few months.

