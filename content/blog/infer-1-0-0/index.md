---
output: hugodown::hugo_document

slug: infer-1-0-0
title: infer 1.0.0
date: 2021-07-19
author: Simon Couch
description: >
    The first major release of infer, a package implementing a unified approach
    to statistical inference, is now on CRAN.

photo:
  url: https://simonpcouch.com
  author: Simon Couch

categories: [package] 
tags: [tidymodels]
rmd_hash: db97483b2a155b89

---

<!--
TODO:
* [x] Look over / edit the post's title in the yaml
* [x] Edit (or delete) the description; note this appears in the Twitter card
* [x] Pick category and tags (see existing with [`hugodown::tidy_show_meta()`](https://rdrr.io/pkg/hugodown/man/use_tidy_post.html))
* [x] Find photo & update yaml metadata
* [x] Create `thumbnail-sq.jpg`; height and width should be equal
* [x] Create `thumbnail-wd.jpg`; width should be >5x height
* [x] [`hugodown::use_tidy_thumbnails()`](https://rdrr.io/pkg/hugodown/man/use_tidy_post.html)
* [x] Add intro sentence, e.g. the standard tagline for the package
* [x] [`usethis::use_tidy_thanks()`](https://usethis.r-lib.org/reference/use_tidy_thanks.html)
-->

We're super excited announce the release of [infer](https://infer.tidymodels.org/) 1.0.0! infer is a package for statistical inference that implements an expressive statistical grammar that adheres to the tidyverse design framework. Rather than providing methods for specific statistical tests, this package consolidates the principles that are shared among common hypothesis tests and confidence intervals into a set of four main verbs (functions), supplemented with many utilities to visualize and extract value from their outputs.

You can install it from CRAN with:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/utils/install.packages.html'>install.packages</a></span><span class='o'>(</span><span class='s'>"infer"</span><span class='o'>)</span></code></pre>

</div>

This release includes a number of major changes and new features. Namely:

-   Support for multiple regression
-   Alignment of theory-based methods
-   Improvements to behavorial consistency of [`calculate()`](https://infer.tidymodels.org/reference/calculate.html)

However, the infer package has been on CRAN since 2017, and we haven't written about the package on the tidyverse blog before. Thus, I'll start out by demonstrating the basics of the package. After, I'll highlight some of the more neat features introduced in this version of the package. You can find a full list of changes in version 1.0.0 of the package in the [release notes](https://github.com/tidymodels/infer/releases/tag/v1.0.0).

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://github.com/tidymodels/infer'>infer</a></span><span class='o'>)</span></code></pre>

</div>

## Getting to Know infer

Regardless of the hypothesis test in question, an analyst asks the same kind of question when conducting statistical inference: is the effect/difference in the observed data real, or due to random chance? To answer this question, the analyst begins by assuming that the effect in the observed data was simply due to random chance, and calls this assumption the *null hypothesis*. (In reality, they might not believe in the null hypothesis at all---the null hypothesis is in opposition to the *alternate hypothesis*, which supposes that the effect present in the observed data is actually due to the fact that "something is going on.") The analyst then calculates a *test statistic* from the data that describes the observed effect. They can use this test statistic to calculate a *p-value* via juxtaposition with a *null distribution*, giving the probability that the observed data could come about if the null hypothesis were true. If this probability is below some pre-defined *significance level* $\alpha$, then the analyst can reject the null hypothesis.

The workflow of this package is designed around this idea. Starting out with some dataset,

-   [`specify()`](https://infer.tidymodels.org/reference/specify.html) allows the analyst to specify the variable, or relationship between variables, that they are interested in.
-   [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html) allows the analyst to declare the null hypothesis.
-   [`generate()`](https://infer.tidymodels.org/reference/generate.html) allows the analyst to generate data reflecting the null hypothesis or using the bootstrap.
-   [`calculate()`](https://infer.tidymodels.org/reference/calculate.html) allows the analyst to calculate summary statistics, either from
    -   the observed data, to form the observed test statistic.
    -   data [`generate()`](https://infer.tidymodels.org/reference/generate.html)d to reflect the null hypothesis, to form a randomization-based null distribution of test statistics.

As such, the ultimate output of an infer pipeline using these four functions is generally an *observed statistic* or *null distribution* of test statistics. These four functions are thus supplemented with several utilities to visualize and extract value from their outputs.

-   [`visualize()`](https://infer.tidymodels.org/reference/visualize.html) plots the null distribution of test statistics.
    -   [`shade_p_value()`](https://infer.tidymodels.org/reference/shade_p_value.html) situates the observed statistic in the null distribution, shading the region as or more extreme.
-   [`get_p_value()`](https://infer.tidymodels.org/reference/get_p_value.html) calculates a p-value via the juxtaposition of the test statistic and null distribution.

The workflow outlined above can also be used for constructing confidence intervals via bootstrapping with the omission of the [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html) step in the pipeline. The resulting bootstrap distribution can then be visualized with [`visualize()`](https://infer.tidymodels.org/reference/visualize.html), the confidence interval region can be situated in the bootstrap distribution with [`shade_confidence_interval()`](https://infer.tidymodels.org/reference/shade_confidence_interval.html), and the bounds of the confidence interval can be calculated with [`get_confidence_interval()`](https://infer.tidymodels.org/reference/get_confidence_interval.html).

To demonstrate, we'll walk through a typical infer pipeline step-by-step. Throughout this post, we make use of `gss`, a dataset supplied by infer containing a sample of 500 observations of 11 variables from the *General Social Survey*.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># load in the dataset</span>
<span class='nf'><a href='https://rdrr.io/r/utils/data.html'>data</a></span><span class='o'>(</span><span class='nv'>gss</span><span class='o'>)</span>

<span class='c'># take a look at its structure</span>
<span class='nf'><a href='https://rdrr.io/r/utils/str.html'>str</a></span><span class='o'>(</span><span class='nv'>gss</span><span class='o'>)</span>
<span class='c'>#&gt; tibble [500 × 11] (S3: tbl_df/tbl/data.frame)</span>
<span class='c'>#&gt;  $ year   : num [1:500] 2014 1994 1998 1996 1994 ...</span>
<span class='c'>#&gt;  $ age    : num [1:500] 36 34 24 42 31 32 48 36 30 33 ...</span>
<span class='c'>#&gt;  $ sex    : Factor w/ 2 levels "male","female": 1 2 1 1 1 2 2 2 2 2 ...</span>
<span class='c'>#&gt;  $ college: Factor w/ 2 levels "no degree","degree": 2 1 2 1 2 1 1 2 2 1 ...</span>
<span class='c'>#&gt;  $ partyid: Factor w/ 5 levels "dem","ind","rep",..: 2 3 2 2 3 3 1 2 3 1 ...</span>
<span class='c'>#&gt;  $ hompop : num [1:500] 3 4 1 4 2 4 2 1 5 2 ...</span>
<span class='c'>#&gt;  $ hours  : num [1:500] 50 31 40 40 40 53 32 20 40 40 ...</span>
<span class='c'>#&gt;  $ income : Ord.factor w/ 12 levels "lt $1000"&lt;"$1000 to 2999"&lt;..: 12 11 12 12 12 12 12 12 12 10 ...</span>
<span class='c'>#&gt;  $ class  : Factor w/ 6 levels "lower class",..: 3 2 2 2 3 3 2 3 3 2 ...</span>
<span class='c'>#&gt;  $ finrela: Factor w/ 6 levels "far below average",..: 2 2 2 4 4 3 2 4 3 1 ...</span>
<span class='c'>#&gt;  $ weight : num [1:500] 0.896 1.083 0.55 1.086 1.083 ...</span></code></pre>

</div>

Each row is an individual survey response, containing some basic demographic information on the respondent as well as some additional variables. See [`?gss`](https://infer.tidymodels.org/reference/gss.html) for more information on the variables included and their source.

### specify(): Specifying Response (and Explanatory) Variables

The [`specify()`](https://infer.tidymodels.org/reference/specify.html) function can be used to specify which of the variables in the dataset you're interested in. If you're only interested in, say, the `age` of the respondents, you might write:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>age</span><span class='o'>)</span>
<span class='c'>#&gt; Response: age (numeric)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 500 x 1</span></span>
<span class='c'>#&gt;      age</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>    36</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>    34</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>    24</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>    42</span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>    31</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>    32</span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>    48</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>    36</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>    30</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>    33</span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 490 more rows</span></span></code></pre>

</div>

On the front-end, the output of [`specify()`](https://infer.tidymodels.org/reference/specify.html) just looks like it selects off the columns in the dataframe that you've specified. Checking the class of this object, though:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>age</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://rdrr.io/r/base/class.html'>class</a></span><span class='o'>(</span><span class='o'>)</span>
<span class='c'>#&gt; [1] "infer"      "tbl_df"     "tbl"        "data.frame"</span></code></pre>

</div>

We can see that the `infer` class has been appended on top of the dataframe classes--this new class stores some extra metadata.

If you're interested in two variables--`age` and `partyid`, for example--you can [`specify()`](https://infer.tidymodels.org/reference/specify.html) their relationship in one of two (equivalent) ways:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># as a formula</span>
<span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span><span class='nv'>age</span> <span class='o'>~</span> <span class='nv'>partyid</span><span class='o'>)</span>
<span class='c'>#&gt; Response: age (numeric)</span>
<span class='c'>#&gt; Explanatory: partyid (factor)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 500 x 2</span></span>
<span class='c'>#&gt;      age partyid</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span>  </span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>    36 ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>    34 rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>    24 ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>    42 ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>    31 rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>    32 rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>    48 dem    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>    36 ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>    30 rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>    33 dem    </span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 490 more rows</span></span>

<span class='c'># with the named arguments</span>
<span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>age</span>, explanatory <span class='o'>=</span> <span class='nv'>partyid</span><span class='o'>)</span>
<span class='c'>#&gt; Response: age (numeric)</span>
<span class='c'>#&gt; Explanatory: partyid (factor)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 500 x 2</span></span>
<span class='c'>#&gt;      age partyid</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span>  </span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>    36 ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>    34 rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>    24 ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>    42 ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>    31 rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>    32 rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>    48 dem    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>    36 ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>    30 rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>    33 dem    </span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 490 more rows</span></span></code></pre>

</div>

### hypothesize(): Declaring the Null Hypothesis

The next step in an infer pipeline is often to declare a null hypothesis using [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html). The first step is to supply one of "independence" or "point" to the `null` argument. If your null hypothesis assumes independence between two variables, then this is all you need to supply to [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html):

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span><span class='nv'>college</span> <span class='o'>~</span> <span class='nv'>partyid</span>, success <span class='o'>=</span> <span class='s'>"degree"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"independence"</span><span class='o'>)</span>
<span class='c'>#&gt; Response: college (factor)</span>
<span class='c'>#&gt; Explanatory: partyid (factor)</span>
<span class='c'>#&gt; Null Hypothesis: independence</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 500 x 2</span></span>
<span class='c'>#&gt;    college   partyid</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span>  </span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span> degree    ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span> no degree rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span> degree    ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span> no degree ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span> degree    rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span> no degree rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span> no degree dem    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span> degree    ind    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span> degree    rep    </span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span> no degree dem    </span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 490 more rows</span></span></code></pre>

</div>

If you're doing inference on a point estimate, you will also need to provide one of `p` (the true proportion of successes, between 0 and 1), `mu` (the true mean), `med` (the true median), or `sigma` (the true standard deviation). For instance, if the null hypothesis is that the mean number of hours worked per week in our population is 40, we would write:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"point"</span>, mu <span class='o'>=</span> <span class='m'>40</span><span class='o'>)</span>
<span class='c'>#&gt; Response: hours (numeric)</span>
<span class='c'>#&gt; Null Hypothesis: point</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 500 x 1</span></span>
<span class='c'>#&gt;    hours</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>    50</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>    31</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>    40</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>    40</span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>    40</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>    53</span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>    32</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>    20</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>    40</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>    40</span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 490 more rows</span></span></code></pre>

</div>

Again, from the front-end, the dataframe outputted from [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html) looks almost exactly the same as it did when it came out of [`specify()`](https://infer.tidymodels.org/reference/specify.html), but infer now "knows" your null hypothesis.

### generate(): Generating the Null Distribution

Once we've asserted our null hypothesis using [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html), we can construct a null distribution based on this hypothesis. We can do this using one of several methods, supplied in the `type` argument:

-   `bootstrap`: A bootstrap sample will be drawn for each replicate, where a sample of size equal to the input sample size is drawn (with replacement) from the input sample data.  
-   `permute`: For each replicate, each input value will be randomly reassigned (without replacement) to a new output value in the sample.  
-   `draw`: A value will be sampled from a theoretical distribution with parameters specified in [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html) for each replicate. (This option is currently only applicable for testing point estimates.)

Continuing on with our example above, about the average number of hours worked a week, we might write:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"point"</span>, mu <span class='o'>=</span> <span class='m'>40</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/generate.html'>generate</a></span><span class='o'>(</span>reps <span class='o'>=</span> <span class='m'>1000</span>, type <span class='o'>=</span> <span class='s'>"bootstrap"</span><span class='o'>)</span>
<span class='c'>#&gt; Response: hours (numeric)</span>
<span class='c'>#&gt; Null Hypothesis: point</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 500,000 x 2</span></span>
<span class='c'>#&gt; <span style='color: #555555;'># Groups:   replicate [1,000]</span></span>
<span class='c'>#&gt;    replicate hours</span>
<span class='c'>#&gt;        <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>         1  38.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>         1  28.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>         1  39.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>         1  43.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>         1  38.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>         1  42.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>         1  58.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>         1  33.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>         1  38.6</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>         1  38.6</span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 499,990 more rows</span></span></code></pre>

</div>

In the above example, we take 1000 bootstrap samples to form our null distribution.

To generate a null distribution for the independence of two variables, we could randomly reshuffle the pairings of explanatory and response variables to break any existing association. For instance, to generate 1000 replicates that can be used to create a null distribution under the assumption that political party affiliation is not affected by age:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span><span class='nv'>partyid</span> <span class='o'>~</span> <span class='nv'>age</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"independence"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/generate.html'>generate</a></span><span class='o'>(</span>reps <span class='o'>=</span> <span class='m'>1000</span>, type <span class='o'>=</span> <span class='s'>"permute"</span><span class='o'>)</span>
<span class='c'>#&gt; Response: partyid (factor)</span>
<span class='c'>#&gt; Explanatory: age (numeric)</span>
<span class='c'>#&gt; Null Hypothesis: independence</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 500,000 x 3</span></span>
<span class='c'>#&gt; <span style='color: #555555;'># Groups:   replicate [1,000]</span></span>
<span class='c'>#&gt;    partyid   age replicate</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;fct&gt;</span>   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span> ind        36         1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span> ind        34         1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span> ind        24         1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span> ind        42         1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span> dem        31         1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span> dem        32         1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span> dem        48         1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span> rep        36         1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span> rep        30         1</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span> ind        33         1</span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 499,990 more rows</span></span></code></pre>

</div>

### calculate(): Calculating Summary Statistics

[`calculate()`](https://infer.tidymodels.org/reference/calculate.html) calculates summary statistics from the output of infer core functions. The function, for one, takes in a `stat` argument, which is currently one of `"mean"`, `"median"`, `"sum"`, `"sd"`, `"prop"`, `"count"`, `"diff in means"`, `"diff in medians"`, `"diff in props"`, `"Chisq"`, `"F"`, `"t"`, `"z"`, `"slope"`, or `"correlation"`. For example, continuing our example above to calculate the null distribution of mean hours worked per week:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"point"</span>, mu <span class='o'>=</span> <span class='m'>40</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/generate.html'>generate</a></span><span class='o'>(</span>reps <span class='o'>=</span> <span class='m'>1000</span>, type <span class='o'>=</span> <span class='s'>"bootstrap"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"mean"</span><span class='o'>)</span>
<span class='c'>#&gt; Response: hours (numeric)</span>
<span class='c'>#&gt; Null Hypothesis: point</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1,000 x 2</span></span>
<span class='c'>#&gt;    replicate  stat</span>
<span class='c'>#&gt;        <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>         1  40.3</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>         2  40.1</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>         3  40.5</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>         4  40.3</span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>         5  40.9</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>         6  39.5</span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>         7  40.6</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>         8  40.5</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>         9  40.8</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>        10  39.0</span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 990 more rows</span></span></code></pre>

</div>

The output of [`calculate()`](https://infer.tidymodels.org/reference/calculate.html) here shows us the sample statistic (in this case, the mean) for each of our 1000 replicates. To calculate the mean from the observed data, just omit the [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html) and [`generate()`](https://infer.tidymodels.org/reference/generate.html) steps.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"mean"</span><span class='o'>)</span>
<span class='c'>#&gt; Response: hours (numeric)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 1</span></span>
<span class='c'>#&gt;    stat</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>  41.4</span></code></pre>

</div>

### Other Utilities

infer offers several utilities to extract meaning from summary statistics and distributions---the package provides functions to visualize where a statistic is relative to a distribution (with [`visualize()`](https://infer.tidymodels.org/reference/visualize.html)), calculate p-values (with [`get_p_value()`](https://infer.tidymodels.org/reference/get_p_value.html)), and calculate confidence intervals (with [`get_confidence_interval()`](https://infer.tidymodels.org/reference/get_confidence_interval.html)).

To illustrate, we'll go back to the example of determining whether the mean number of hours worked per week is 40 hours.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># find the point estimate</span>
<span class='nv'>point_estimate</span> <span class='o'>&lt;-</span> <span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"mean"</span><span class='o'>)</span>

<span class='c'># generate a distribution of means</span>
<span class='nv'>dist</span> <span class='o'>&lt;-</span> <span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"point"</span>, mu <span class='o'>=</span> <span class='m'>40</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/generate.html'>generate</a></span><span class='o'>(</span>reps <span class='o'>=</span> <span class='m'>1000</span>, type <span class='o'>=</span> <span class='s'>"bootstrap"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"mean"</span><span class='o'>)</span></code></pre>

</div>

Our point estimate 41.382 seems *pretty* close to 40, but a little bit different. We might wonder if this difference is just due to random chance, or if the mean number of hours worked per week in the population really isn't 40.

We could initially just visualize the distribution.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>dist</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/visualize.html'>visualize</a></span><span class='o'>(</span><span class='o'>)</span>
</code></pre>
<img src="figs/visualize-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Where does our sample's observed statistic lie on this distribution? We can use the `obs_stat` argument to specify this.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>dist</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/visualize.html'>visualize</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/shade_p_value.html'>shade_p_value</a></span><span class='o'>(</span>obs_stat <span class='o'>=</span> <span class='nv'>point_estimate</span>, direction <span class='o'>=</span> <span class='s'>"two-sided"</span><span class='o'>)</span>
</code></pre>
<img src="figs/visualize2-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Notice that infer has also shaded the regions of the null distribution that are as (or more) extreme than our observed statistic. The red bar looks like it's slightly far out on the right tail of the null distribution, so observing a sample mean of 41.382 hours would be somewhat unlikely if the mean was actually 40 hours. How unlikely, though?

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>p_value</span> <span class='o'>&lt;-</span> <span class='nv'>dist</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/get_p_value.html'>get_p_value</a></span><span class='o'>(</span>obs_stat <span class='o'>=</span> <span class='nv'>point_estimate</span>, direction <span class='o'>=</span> <span class='s'>"two-sided"</span><span class='o'>)</span>

<span class='nv'>p_value</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 1</span></span>
<span class='c'>#&gt;   p_value</span>
<span class='c'>#&gt;     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>   0.038</span></code></pre>

</div>

It looks like the p-value is 0.038, which is pretty small---if the true mean number of hours worked per week was actually 40, the probability of our sample mean being this far (1.382 hours) from 40 would be 0.038. This may or may not be statistically significantly different, depending on the significance level $\alpha$ you decided on *before* you ran this analysis. If you had set $\alpha = .05$, then this difference would be statistically significant, but if you had set $\alpha = .01$, then it would not be.

To get a confidence interval around our estimate, we can write:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>dist</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/get_confidence_interval.html'>get_confidence_interval</a></span><span class='o'>(</span>
    point_estimate <span class='o'>=</span> <span class='nv'>point_estimate</span>,
    level <span class='o'>=</span> <span class='m'>.95</span>,
    type <span class='o'>=</span> <span class='s'>"se"</span>
  <span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 2</span></span>
<span class='c'>#&gt;   lower_ci upper_ci</span>
<span class='c'>#&gt;      <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>     40.1     42.7</span></code></pre>

</div>

As you can see, 40 hours per week is not contained in this interval, which aligns with our previous conclusion that this finding is significant at the confidence level $\alpha = .05$.

## What's New?

There are a number of improvements and new features in this release that resolve longstanding gaps in the package's functionality. We'll highlight three:

-   Support for multiple regression
-   Alignment of theory-based methods
-   Behavioral consistency of [`calculate()`](https://infer.tidymodels.org/reference/calculate.html)

### Support for multiple regression

The 2016 "Guidelines for Assessment and Instruction in Statistics Education" \[1\] state that, in introductory statistics courses, "\[s\]tudents should gain experience with how statistical models, including multivariable models, are used." In line with this recommendation, we introduce support for randomization-based inference with multiple explanatory variables via a new `fit.infer` core verb.

If passed an `infer` object, the method will parse a formula out of the `formula` or `response` and `explanatory` arguments, and pass both it and `data` to a [`stats::glm`](https://rdrr.io/r/stats/glm.html) call.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span><span class='nv'>hours</span> <span class='o'>~</span> <span class='nv'>age</span> <span class='o'>+</span> <span class='nv'>college</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://generics.r-lib.org/reference/fit.html'>fit</a></span><span class='o'>(</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 3 x 2</span></span>
<span class='c'>#&gt;   term          estimate</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>            <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span> intercept     40.6    </span>
<span class='c'>#&gt; <span style='color: #555555;'>2</span> age            0.005<span style='text-decoration: underline;'>96</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>3</span> collegedegree  1.53</span></code></pre>

</div>

Note that the function returns the model coefficients as `estimate` rather than their associated $t$-statistics as `stat`.

If passed a [`generate()`](https://infer.tidymodels.org/reference/generate.html)d object, the model will be fitted to each replicate.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span><span class='nv'>hours</span> <span class='o'>~</span> <span class='nv'>age</span> <span class='o'>+</span> <span class='nv'>college</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"independence"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/generate.html'>generate</a></span><span class='o'>(</span>reps <span class='o'>=</span> <span class='m'>100</span>, type <span class='o'>=</span> <span class='s'>"permute"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://generics.r-lib.org/reference/fit.html'>fit</a></span><span class='o'>(</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 300 x 3</span></span>
<span class='c'>#&gt; <span style='color: #555555;'># Groups:   replicate [100]</span></span>
<span class='c'>#&gt;    replicate term          estimate</span>
<span class='c'>#&gt;        <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>            <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>         1 intercept     40.9    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>         1 age            0.010<span style='text-decoration: underline;'>0</span> </span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>         1 collegedegree  0.198  </span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>         2 intercept     35.9    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>         2 age            0.133  </span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>         2 collegedegree  0.347  </span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>         3 intercept     41.5    </span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>         3 age            0.006<span style='text-decoration: underline;'>71</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>         3 collegedegree -<span style='color: #BB0000;'>1.04</span>   </span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>         4 intercept     41.3    </span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 290 more rows</span></span></code></pre>

</div>

If `type = "permute"`, a set of unquoted column names in the data to permute (independently of each other) can be passed via the `cols` argument to `generate`. It defaults to only the response variable.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span><span class='nv'>hours</span> <span class='o'>~</span> <span class='nv'>age</span> <span class='o'>+</span> <span class='nv'>college</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"independence"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/generate.html'>generate</a></span><span class='o'>(</span>reps <span class='o'>=</span> <span class='m'>100</span>, type <span class='o'>=</span> <span class='s'>"permute"</span>, cols <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='nv'>age</span>, <span class='nv'>college</span><span class='o'>)</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://generics.r-lib.org/reference/fit.html'>fit</a></span><span class='o'>(</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 300 x 3</span></span>
<span class='c'>#&gt; <span style='color: #555555;'># Groups:   replicate [100]</span></span>
<span class='c'>#&gt;    replicate term          estimate</span>
<span class='c'>#&gt;        <span style='color: #555555; font-style: italic;'>&lt;int&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>            <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>         1 intercept      43.4   </span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>         1 age            -<span style='color: #BB0000;'>0.023</span><span style='color: #BB0000; text-decoration: underline;'>8</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>         1 collegedegree  -<span style='color: #BB0000;'>2.95</span>  </span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>         2 intercept      38.7   </span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>         2 age             0.071<span style='text-decoration: underline;'>5</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>         2 collegedegree  -<span style='color: #BB0000;'>0.556</span> </span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>         3 intercept      42.5   </span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>         3 age            -<span style='color: #BB0000;'>0.029</span><span style='color: #BB0000; text-decoration: underline;'>6</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>         3 collegedegree   0.321 </span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>         4 intercept      41.0   </span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 290 more rows</span></span></code></pre>

</div>

This feature allows for more detailed exploration of the effect of disrupting the correlation structure among explanatory variables on outputted model coefficients.

Each of the auxillary functions [`get_p_value()`](https://infer.tidymodels.org/reference/get_p_value.html), [`get_confidence_interval()`](https://infer.tidymodels.org/reference/get_confidence_interval.html), [`visualize()`](https://infer.tidymodels.org/reference/visualize.html), [`shade_p_value()`](https://infer.tidymodels.org/reference/shade_p_value.html), and [`shade_confidence_interval()`](https://infer.tidymodels.org/reference/shade_confidence_interval.html) have methods to handle [`fit()`](https://generics.r-lib.org/reference/fit.html) output! See their help-files for example usage.

### Alignment of theory-based methods

While infer is primarily a package for randomization-based statistical inference, the package has partially supported theory-based methods in a number of ways over the years. This release introduces a principled, opinionated, and consistent interface for theory-based methods for statistical inference. The new interface is based on a new verb, [`assume()`](https://infer.tidymodels.org/reference/assume.html), that returns a distribution that, once created, interfaces in the same way that simulation-based distributions do.

To demonstrate, we'll return to the example of inference on a mean using infer's `gss` dataset. Supposed that we believe the true mean number of hours worked by Americans in the past week is 40.

First, calculating the observed $t$-statistic:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>obs_stat</span> <span class='o'>&lt;-</span> <span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"point"</span>, mu <span class='o'>=</span> <span class='m'>40</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"t"</span><span class='o'>)</span>

<span class='nv'>obs_stat</span>
<span class='c'>#&gt; Response: hours (numeric)</span>
<span class='c'>#&gt; Null Hypothesis: point</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 1</span></span>
<span class='c'>#&gt;    stat</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>  2.09</span></code></pre>

</div>

The code to define the null distribution is very similar to that required to calculate a theorized observed statistic, switching out [`calculate()`](https://infer.tidymodels.org/reference/calculate.html) for [`assume()`](https://infer.tidymodels.org/reference/assume.html) and replacing arguments as needed.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>dist</span> <span class='o'>&lt;-</span> <span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"point"</span>, mu <span class='o'>=</span> <span class='m'>40</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/assume.html'>assume</a></span><span class='o'>(</span>distribution <span class='o'>=</span> <span class='s'>"t"</span>, df <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/nrow.html'>nrow</a></span><span class='o'>(</span><span class='nv'>gss</span><span class='o'>)</span> <span class='o'>-</span> <span class='m'>1</span><span class='o'>)</span>

<span class='nv'>dist</span> 
<span class='c'>#&gt; A T distribution with 499 degrees of freedom.</span></code></pre>

</div>

This null distribution can now be interfaced with in the same way as a simulation-based null distribution elsewhere in the package. For example, calculating a p-value by juxtaposing the observed statistic and null distribution:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://infer.tidymodels.org/reference/get_p_value.html'>get_p_value</a></span><span class='o'>(</span><span class='nv'>dist</span>, <span class='nv'>obs_stat</span>, direction <span class='o'>=</span> <span class='s'>"both"</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 1</span></span>
<span class='c'>#&gt;   p_value</span>
<span class='c'>#&gt;     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>  0.037<span style='text-decoration: underline;'>6</span></span></code></pre>

</div>

...or juxtaposing the two visually:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://infer.tidymodels.org/reference/visualize.html'>visualize</a></span><span class='o'>(</span><span class='nv'>dist</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://infer.tidymodels.org/reference/shade_p_value.html'>shade_p_value</a></span><span class='o'>(</span><span class='nv'>obs_stat</span>, direction <span class='o'>=</span> <span class='s'>"both"</span><span class='o'>)</span>
</code></pre>
<img src="figs/unnamed-chunk-8-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Confidence intervals lie on the scale of the observed data rather than the standardized scale of the theoretical distributions. Calculating a mean rather than the standardized $t$-statistic:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>obs_mean</span> <span class='o'>&lt;-</span> <span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"mean"</span><span class='o'>)</span>

<span class='nv'>obs_mean</span>
<span class='c'>#&gt; Response: hours (numeric)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 1</span></span>
<span class='c'>#&gt;    stat</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>  41.4</span></code></pre>

</div>

The distribution here just defines the spread for the standard error calculation.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>ci</span> <span class='o'>&lt;-</span> 
  <span class='nf'><a href='https://infer.tidymodels.org/reference/get_confidence_interval.html'>get_confidence_interval</a></span><span class='o'>(</span>
    <span class='nv'>dist</span>,
    level <span class='o'>=</span> <span class='m'>.95</span>,
    point_estimate <span class='o'>=</span> <span class='nv'>obs_mean</span>
  <span class='o'>)</span>

<span class='nv'>ci</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 2</span></span>
<span class='c'>#&gt;   lower_ci upper_ci</span>
<span class='c'>#&gt;      <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>     40.1     42.7</span></code></pre>

</div>

Visualizing the confidence interval results in the theoretical distribution being recentered and rescaled to align with the scale of the observed data:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://infer.tidymodels.org/reference/visualize.html'>visualize</a></span><span class='o'>(</span><span class='nv'>dist</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://infer.tidymodels.org/reference/shade_confidence_interval.html'>shade_confidence_interval</a></span><span class='o'>(</span><span class='nv'>ci</span><span class='o'>)</span>
</code></pre>
<img src="figs/unnamed-chunk-11-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Previous methods for interfacing with theoretical distributions are superseded---they will continue to be supported, though documentation will forefront the [`assume()`](https://infer.tidymodels.org/reference/assume.html) interface.

### Behavioral consistency

Another major change to the package in this release is a set of standards for behavorial consistency of [`calculate()`](https://infer.tidymodels.org/reference/calculate.html). Namely, the package will now

-   supply a consistent error when the supplied `stat` argument isn't well-defined for the variables [`specify()`](https://infer.tidymodels.org/reference/specify.html)d

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"diff in means"</span><span class='o'>)</span>
<span class='c'>#&gt; Error: A difference in means is not well-defined for a numeric response variable (hours) and no explanatory variable.</span></code></pre>

</div>

or

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span><span class='nv'>college</span> <span class='o'>~</span> <span class='nv'>partyid</span>, success <span class='o'>=</span> <span class='s'>"degree"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"diff in props"</span><span class='o'>)</span>
<span class='c'>#&gt; Dropping unused factor levels DK from the supplied explanatory variable 'partyid'.</span>
<span class='c'>#&gt; Error: A difference in proportions is not well-defined for a dichotomous categorical response variable (college) and a multinomial categorical explanatory variable (partyid).</span></code></pre>

</div>

-   supply a consistent message when the user supplies unneeded information via [`hypothesize()`](https://infer.tidymodels.org/reference/hypothesize.html) to [`calculate()`](https://infer.tidymodels.org/reference/calculate.html) an observed statistic

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># supply mu = 40 when it's not needed</span>
<span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>hours</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/hypothesize.html'>hypothesize</a></span><span class='o'>(</span>null <span class='o'>=</span> <span class='s'>"point"</span>, mu <span class='o'>=</span> <span class='m'>40</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"mean"</span><span class='o'>)</span>
<span class='c'>#&gt; Message: The point null hypothesis `mu = 40` does not inform calculation of the observed statistic (a mean) and will be ignored.</span>
<span class='c'>#&gt; Response: hours (numeric)</span>
<span class='c'>#&gt; Null Hypothesis: point</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 1</span></span>
<span class='c'>#&gt;    stat</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>  41.4</span></code></pre>

</div>

and

-   supply a consistent warning and assume a reasonable null value when the user does not supply sufficient information to calculate an observed statistic

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># don't hypothesize `p` when it's needed</span>
<span class='nv'>gss</span> <span class='o'>%&gt;%</span>
    <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>sex</span>, success <span class='o'>=</span> <span class='s'>"female"</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
    <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"z"</span><span class='o'>)</span>
<span class='c'>#&gt; Warning: A z statistic requires a null hypothesis to calculate the observed statistic. </span>
<span class='c'>#&gt; Output assumes the following null value: `p = .5`.</span>
<span class='c'>#&gt; Response: sex (factor)</span>
<span class='c'>#&gt; Null Hypothesis: point</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 1</span></span>
<span class='c'>#&gt;    stat</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span> -<span style='color: #BB0000;'>1.16</span></span></code></pre>

</div>

or

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='c'># don't hypothesize `p` when it's needed</span>
<span class='nv'>gss</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/specify.html'>specify</a></span><span class='o'>(</span>response <span class='o'>=</span> <span class='nv'>partyid</span><span class='o'>)</span> <span class='o'>%&gt;%</span>
  <span class='nf'><a href='https://infer.tidymodels.org/reference/calculate.html'>calculate</a></span><span class='o'>(</span>stat <span class='o'>=</span> <span class='s'>"Chisq"</span><span class='o'>)</span>
<span class='c'>#&gt; Dropping unused factor levels DK from the supplied response variable 'partyid'.</span>
<span class='c'>#&gt; Warning: A chi-square statistic requires a null hypothesis to calculate the observed statistic. </span>
<span class='c'>#&gt; Output assumes the following null values: `p = c(dem = 0.2, ind = 0.2, rep = 0.2, other = 0.2, DK = 0.2)`.</span>
<span class='c'>#&gt; Response: partyid (factor)</span>
<span class='c'>#&gt; Null Hypothesis: point</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 1</span></span>
<span class='c'>#&gt;    stat</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>  334.</span></code></pre>

</div>

We don't anticipate that any of these changes are "breaking" in the sense that code that previously worked will continue to, though it may now message or warn in a way that it did not used to or error with a different (and hopefully more informative) message.

## Acknowledgements

This release was made possible with financial support from RStudio and the Reed College Mathematics Department. Thanks to [@aarora79](https://github.com/aarora79), [@acpguedes](https://github.com/acpguedes), [@AlbertRapp](https://github.com/AlbertRapp), [@alexpghayes](https://github.com/alexpghayes), [@aloy](https://github.com/aloy), [@AmeliaMN](https://github.com/AmeliaMN), [@andrewpbray](https://github.com/andrewpbray), [@apreshill](https://github.com/apreshill), [@atheobold](https://github.com/atheobold), [@beanumber](https://github.com/beanumber), [@bigdataman2015](https://github.com/bigdataman2015), [@bragks](https://github.com/bragks), [@brendanhcullen](https://github.com/brendanhcullen), [@CarlssonLeo](https://github.com/CarlssonLeo), [@ChalkboardSonata](https://github.com/ChalkboardSonata), [@chriscardillo](https://github.com/chriscardillo), [@clauswilke](https://github.com/clauswilke), [@congdanh8391](https://github.com/congdanh8391), [@corinne-riddell](https://github.com/corinne-riddell), [@cristianvaldez](https://github.com/cristianvaldez), [@daranzolin](https://github.com/daranzolin), [@davidbaniadam](https://github.com/davidbaniadam), [@davidhodge931](https://github.com/davidhodge931), [@doug-friedman](https://github.com/doug-friedman), [@dshelldhillon](https://github.com/dshelldhillon), [@dsolito](https://github.com/dsolito), [@echasnovski](https://github.com/echasnovski), [@EllaKaye](https://github.com/EllaKaye), [@enricochavez](https://github.com/enricochavez), [@gdbassett](https://github.com/gdbassett), [@ghost](https://github.com/ghost), [@GitHunter0](https://github.com/GitHunter0), [@hardin47](https://github.com/hardin47), [@hfrick](https://github.com/hfrick), [@higgi13425](https://github.com/higgi13425), [@instantkaffee](https://github.com/instantkaffee), [@ismayc](https://github.com/ismayc), [@jbourak](https://github.com/jbourak), [@jcvall](https://github.com/jcvall), [@jimrothstein](https://github.com/jimrothstein), [@kennethban](https://github.com/kennethban), [@m-berkes](https://github.com/m-berkes), [@mikelove](https://github.com/mikelove), [@mine-cetinkaya-rundel](https://github.com/mine-cetinkaya-rundel), [@Minhasshazu](https://github.com/Minhasshazu), [@msberends](https://github.com/msberends), [@mt-edwards](https://github.com/mt-edwards), [@muschellij2](https://github.com/muschellij2), [@nfultz](https://github.com/nfultz), [@nicholasjhorton](https://github.com/nicholasjhorton), [@PirateGrunt](https://github.com/PirateGrunt), [@PsychlytxTD](https://github.com/PsychlytxTD), [@richierocks](https://github.com/richierocks), [@romainfrancois](https://github.com/romainfrancois), [@rpruim](https://github.com/rpruim), [@rudeboybert](https://github.com/rudeboybert), [@rundel](https://github.com/rundel), [@sastoudt](https://github.com/sastoudt), [@sbibauw](https://github.com/sbibauw), [@sckott](https://github.com/sckott), [@THargreaves](https://github.com/THargreaves), [@topepo](https://github.com/topepo), [@torockel](https://github.com/torockel), [@ttimbers](https://github.com/ttimbers), [@vikram-rawat](https://github.com/vikram-rawat), [@vladimirvrabely](https://github.com/vladimirvrabely), and [@xiaochi-liu](https://github.com/xiaochi-liu) for their contributions to the package.

\[1\]: GAISE College Report ASA Revision Committee, "Guidelines for Assessment and Instruction in Statistics Education College Report 2016," <http://www.amstat.org/education/gaise>.

