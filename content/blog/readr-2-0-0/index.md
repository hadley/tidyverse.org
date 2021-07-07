---
output: hugodown::hugo_document

slug: readr-2-0-0
title: readr 2.0.0
date: 2021-07-06
author: Jim Hester
description: |
  This major release of readr includes a new multi-threaded parsing engine powered by vroom and a 
  number of user interface improvements.
photo:
  url: https://unsplash.com/photos/XOW1WqrWNKg
  author: Anastasia Zhenina
categories:
  - package
rmd_hash: 168617e3ba38b4ee

---

We're thrilled to announce the release of [readr](https://readr.tidyverse.org/) 2.0.0!

The readr package makes it easy to get rectangular data out of comma separated (csv), tab separated (tsv) or fixed width files (fwf) and into R. It is designed to flexibly parse many types of data found in the wild, while still cleanly failing when data unexpectedly changes.

The easiest way to install the latest version from CRAN is to install the whole tidyverse.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/utils/install.packages.html'>install.packages</a></span><span class='o'>(</span><span class='s'>"tidyverse"</span><span class='o'>)</span></code></pre>

</div>

Alternatively, install just readr from CRAN:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/utils/install.packages.html'>install.packages</a></span><span class='o'>(</span><span class='s'>"readr"</span><span class='o'>)</span></code></pre>

</div>

This blog post will show off the most important changes to the package.

You can see a full list of changes in the [readr release notes](https://github.com/r-lib/readr/releases) and [vroom release notes](https://github.com/r-lib/vroom/releases).

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://readr.tidyverse.org'>readr</a></span><span class='o'>)</span></code></pre>

</div>

## readr edition two

Readr 2.0.0 is a major release of readr and introduces a new second edition parsing and writing engine implemented via the [vroom](https://vroom.r-lib.org/) package.

This engine takes advantage of lazy reading, multi-threading and performance characteristics of modern SSD drives to significantly improve the performance of reading and writing compared to the first edition engine.

We will continue to support the first edition for a number of releases, but eventually this support will be first deprecated and then removed.

You can use the [`with_edition()`](https://readr.tidyverse.org/reference/with_edition.html) or [`local_edition()`](https://readr.tidyverse.org/reference/with_edition.html) functions to temporarily change the edition of readr for a section of code.

e.g.

-   [`with_edition(1, read_csv("my_file.csv"))`](https://readr.tidyverse.org/reference/with_edition.html) will read `my_file.csv` with the first edition of readr.

-   [`readr::local_edition(1)`](https://readr.tidyverse.org/reference/with_edition.html) placed at the top of your function or script will use the first edition for the rest of the function or script.

## Reading multiple files at once

Edition two has built-in support for reading sets of files with the same columns into one output table in a single command. Just pass the filenames to be read in the same vector to the reading function.

First we generate some files to read by splitting the nycflights dataset by airline.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='http://github.com/hadley/nycflights13'>nycflights13</a></span><span class='o'>)</span>
<span class='nf'>purrr</span><span class='nf'>::</span><span class='nf'><a href='https://purrr.tidyverse.org/reference/imap.html'>iwalk</a></span><span class='o'>(</span>
  <span class='nf'><a href='https://rdrr.io/r/base/split.html'>split</a></span><span class='o'>(</span><span class='nv'>flights</span>, <span class='nv'>flights</span><span class='o'>$</span><span class='nv'>carrier</span><span class='o'>)</span>,
  <span class='o'>~</span> <span class='o'>&#123;</span> <span class='nv'>.x</span><span class='o'>$</span><span class='nv'>carrier</span><span class='o'>[[</span><span class='m'>1</span><span class='o'>]</span><span class='o'>]</span>; <span class='nf'>vroom</span><span class='nf'>::</span><span class='nf'><a href='https://vroom.r-lib.org/reference/vroom_write.html'>vroom_write</a></span><span class='o'>(</span><span class='nv'>.x</span>, <span class='nf'>glue</span><span class='nf'>::</span><span class='nf'><a href='https://glue.tidyverse.org/reference/glue.html'>glue</a></span><span class='o'>(</span><span class='s'>"flights_&#123;.y&#125;.tsv"</span><span class='o'>)</span>, delim <span class='o'>=</span> <span class='s'>"\t"</span><span class='o'>)</span> <span class='o'>&#125;</span>
<span class='o'>)</span></code></pre>

</div>

Then we can efficiently read them into one tibble by passing the filenames directly to readr.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>files</span> <span class='o'>&lt;-</span> <span class='nf'>fs</span><span class='nf'>::</span><span class='nf'><a href='http://fs.r-lib.org/reference/dir_ls.html'>dir_ls</a></span><span class='o'>(</span>glob <span class='o'>=</span> <span class='s'>"flights*tsv"</span><span class='o'>)</span>
<span class='nv'>files</span>
<span class='c'>#&gt; flights_9E.tsv flights_AA.tsv flights_AS.tsv flights_B6.tsv flights_DL.tsv </span>
<span class='c'>#&gt; flights_EV.tsv flights_F9.tsv flights_FL.tsv flights_HA.tsv flights_MQ.tsv </span>
<span class='c'>#&gt; flights_OO.tsv flights_UA.tsv flights_US.tsv flights_VX.tsv flights_WN.tsv </span>
<span class='c'>#&gt; flights_YV.tsv</span>
<span class='nf'>readr</span><span class='nf'>::</span><span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_tsv</a></span><span class='o'>(</span><span class='nv'>files</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Rows: </span><span style='color: #0000BB;'>336776</span> <span style='font-weight: bold;'>Columns: </span><span style='color: #0000BB;'>19</span></span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> "\t"</span>
<span class='c'>#&gt; <span style='color: #BB0000;'>chr</span>   (4): carrier, tailnum, origin, dest</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span>  (14): year, month, day, dep_time, sched_dep_time, dep_delay, arr_time, ...</span>
<span class='c'>#&gt; <span style='color: #0000BB;'>dttm</span>  (1): time_hour</span>
<span class='c'>#&gt; </span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Use <span style='color: #000000; background-color: #BBBBBB;'>`spec()`</span> to retrieve the full column specification for this data.</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Specify the column types or set <span style='color: #000000; background-color: #BBBBBB;'>`show_col_types = FALSE`</span> to quiet this message.</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 336,776 x 19</span></span>
<span class='c'>#&gt;     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>          <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>          <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>  <span style='text-decoration: underline;'>2</span>013     1     1      810            810         0     <span style='text-decoration: underline;'>1</span>048           <span style='text-decoration: underline;'>1</span>037</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>451           <span style='text-decoration: underline;'>1</span>500        -<span style='color: #BB0000;'>9</span>     <span style='text-decoration: underline;'>1</span>634           <span style='text-decoration: underline;'>1</span>636</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>452           <span style='text-decoration: underline;'>1</span>455        -<span style='color: #BB0000;'>3</span>     <span style='text-decoration: underline;'>1</span>637           <span style='text-decoration: underline;'>1</span>639</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>454           <span style='text-decoration: underline;'>1</span>500        -<span style='color: #BB0000;'>6</span>     <span style='text-decoration: underline;'>1</span>635           <span style='text-decoration: underline;'>1</span>636</span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>507           <span style='text-decoration: underline;'>1</span>515        -<span style='color: #BB0000;'>8</span>     <span style='text-decoration: underline;'>1</span>651           <span style='text-decoration: underline;'>1</span>656</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>530           <span style='text-decoration: underline;'>1</span>530         0     <span style='text-decoration: underline;'>1</span>650           <span style='text-decoration: underline;'>1</span>655</span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>546           <span style='text-decoration: underline;'>1</span>540         6     <span style='text-decoration: underline;'>1</span>753           <span style='text-decoration: underline;'>1</span>748</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>550           <span style='text-decoration: underline;'>1</span>550         0     <span style='text-decoration: underline;'>1</span>844           <span style='text-decoration: underline;'>1</span>831</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>552           <span style='text-decoration: underline;'>1</span>600        -<span style='color: #BB0000;'>8</span>     <span style='text-decoration: underline;'>1</span>749           <span style='text-decoration: underline;'>1</span>757</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>  <span style='text-decoration: underline;'>2</span>013     1     1     <span style='text-decoration: underline;'>1</span>554           <span style='text-decoration: underline;'>1</span>600        -<span style='color: #BB0000;'>6</span>     <span style='text-decoration: underline;'>1</span>701           <span style='text-decoration: underline;'>1</span>734</span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 336,766 more rows, and 11 more variables: arr_delay &lt;dbl&gt;,</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>#   carrier &lt;chr&gt;, flight &lt;dbl&gt;, tailnum &lt;chr&gt;, origin &lt;chr&gt;, dest &lt;chr&gt;,</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>#   air_time &lt;dbl&gt;, distance &lt;dbl&gt;, hour &lt;dbl&gt;, minute &lt;dbl&gt;, time_hour &lt;dttm&gt;</span></span></code></pre>

</div>

If the filenames contain data, such as the date when the sample was collected, use `id` argument to include the paths as a column in the data. You will likely have to post-process the paths to keep only the relevant portion for your use case.

## Delimiter guessing

Edition two supports automatic guessing of delimiters. Because of this you can now use [`read_delim()`](https://readr.tidyverse.org/reference/read_delim.html) without specifying a `delim` argument in many cases.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>x</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_delim</a></span><span class='o'>(</span><span class='nf'><a href='https://readr.tidyverse.org/reference/readr_example.html'>readr_example</a></span><span class='o'>(</span><span class='s'>"mtcars.csv"</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Rows: </span><span style='color: #0000BB;'>32</span> <span style='font-weight: bold;'>Columns: </span><span style='color: #0000BB;'>11</span></span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> ","</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span> (11): mpg, cyl, disp, hp, drat, wt, qsec, vs, am, gear, carb</span>
<span class='c'>#&gt; </span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Use <span style='color: #000000; background-color: #BBBBBB;'>`spec()`</span> to retrieve the full column specification for this data.</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Specify the column types or set <span style='color: #000000; background-color: #BBBBBB;'>`show_col_types = FALSE`</span> to quiet this message.</span></code></pre>

</div>

## New column specification output

On Feburary 11, 2021 we conducted a [survey on twitter](https://twitter.com/jimhester_/status/1359969288501739528) asking for the community's opinion on the column specification output. We recieved over 750 responses to the survey! It revealed some useful information.

-   3/4 of respondents found printing the column specifications helpful.
-   2/3 of respondents preferred the edition output vs legacy output.
-   Only 1/5 of respondents correctly knew how to supress printing of the column specifications.

Based on these results we have added two new ways to more easily suppress the column specification printing.

-   Use [`read_csv(show_col_types = FALSE)`](https://readr.tidyverse.org/reference/read_delim.html) to disable printing for a single function call.
-   Use [`options(readr.show_types = FALSE)`](https://rdrr.io/r/base/options.html) to disable printing for the entire session.

We will also continue to print the column specifications with the new output.

Note you can still obtain the old output style by printing the column specification object directly.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://readr.tidyverse.org/reference/spec.html'>spec</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span>
<span class='c'>#&gt; cols(</span>
<span class='c'>#&gt;   mpg = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   cyl = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   disp = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   hp = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   drat = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   wt = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   qsec = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   vs = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   am = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   gear = <span style='color: #00BB00;'>col_double()</span>,</span>
<span class='c'>#&gt;   carb = <span style='color: #00BB00;'>col_double()</span></span>
<span class='c'>#&gt; )</span></code></pre>

</div>

Or the new style by calling [`summary()`](https://rdrr.io/r/base/summary.html) on the specficiation object.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/summary.html'>summary</a></span><span class='o'>(</span><span class='nf'><a href='https://readr.tidyverse.org/reference/spec.html'>spec</a></span><span class='o'>(</span><span class='nv'>x</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> ","</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span> (11): mpg, cyl, disp, hp, drat, wt, qsec, vs, am, gear, carb</span></code></pre>

</div>

## Column selection

Edition two introduces a new argument, `col_select`, which makes selecting columns to keep (or omit) more straightforward than before.

`col_select` uses the same interface as [`dplyr::select()`](https://dplyr.tidyverse.org/reference/select.html), so you can perform very flexible selection operations.

Select with the column names directly.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>data</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_tsv</a></span><span class='o'>(</span><span class='s'>"flights_AA.tsv"</span>, col_select <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='nv'>year</span>, <span class='nv'>flight</span>, <span class='nv'>tailnum</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Rows: </span><span style='color: #0000BB;'>32729</span> <span style='font-weight: bold;'>Columns: </span><span style='color: #0000BB;'>3</span></span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> "\t"</span>
<span class='c'>#&gt; <span style='color: #BB0000;'>chr</span> (1): tailnum</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span> (2): year, flight</span>
<span class='c'>#&gt; </span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Use <span style='color: #000000; background-color: #BBBBBB;'>`spec()`</span> to retrieve the full column specification for this data.</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Specify the column types or set <span style='color: #000000; background-color: #BBBBBB;'>`show_col_types = FALSE`</span> to quiet this message.</span></code></pre>

</div>

Or by numeric column.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>data</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_tsv</a></span><span class='o'>(</span><span class='s'>"flights_AA.tsv"</span>, col_select <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='m'>1</span>, <span class='m'>2</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Rows: </span><span style='color: #0000BB;'>32729</span> <span style='font-weight: bold;'>Columns: </span><span style='color: #0000BB;'>2</span></span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> "\t"</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span> (2): year, month</span>
<span class='c'>#&gt; </span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Use <span style='color: #000000; background-color: #BBBBBB;'>`spec()`</span> to retrieve the full column specification for this data.</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Specify the column types or set <span style='color: #000000; background-color: #BBBBBB;'>`show_col_types = FALSE`</span> to quiet this message.</span></code></pre>

</div>

Drop columns by name by prefixing them with [`-`](https://rdrr.io/r/base/Arithmetic.html).

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>data</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_tsv</a></span><span class='o'>(</span><span class='s'>"flights_AA.tsv"</span>, col_select <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='o'>-</span><span class='nv'>dep_time</span>, <span class='o'>-</span><span class='nv'>air_time</span><span class='o'>:</span><span class='o'>-</span><span class='nv'>time_hour</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Rows: </span><span style='color: #0000BB;'>32729</span> <span style='font-weight: bold;'>Columns: </span><span style='color: #0000BB;'>13</span></span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> "\t"</span>
<span class='c'>#&gt; <span style='color: #BB0000;'>chr</span> (4): carrier, tailnum, origin, dest</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span> (9): year, month, day, sched_dep_time, dep_delay, arr_time, sched_arr_ti...</span>
<span class='c'>#&gt; </span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Use <span style='color: #000000; background-color: #BBBBBB;'>`spec()`</span> to retrieve the full column specification for this data.</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Specify the column types or set <span style='color: #000000; background-color: #BBBBBB;'>`show_col_types = FALSE`</span> to quiet this message.</span></code></pre>

</div>

Use the selection helpers such as `ends_with()`.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>data</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_tsv</a></span><span class='o'>(</span><span class='s'>"flights_AA.tsv"</span>, col_select <span class='o'>=</span> <span class='nf'>ends_with</span><span class='o'>(</span><span class='s'>"time"</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Rows: </span><span style='color: #0000BB;'>32729</span> <span style='font-weight: bold;'>Columns: </span><span style='color: #0000BB;'>5</span></span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> "\t"</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span> (5): dep_time, sched_dep_time, arr_time, sched_arr_time, air_time</span>
<span class='c'>#&gt; </span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Use <span style='color: #000000; background-color: #BBBBBB;'>`spec()`</span> to retrieve the full column specification for this data.</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Specify the column types or set <span style='color: #000000; background-color: #BBBBBB;'>`show_col_types = FALSE`</span> to quiet this message.</span></code></pre>

</div>

Or even rename columns by using a named list.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>data</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_tsv</a></span><span class='o'>(</span><span class='s'>"flights_AA.tsv"</span>, col_select <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/list.html'>list</a></span><span class='o'>(</span>plane <span class='o'>=</span> <span class='nv'>tailnum</span>, <span class='nf'>everything</span><span class='o'>(</span><span class='o'>)</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Rows: </span><span style='color: #0000BB;'>32729</span> <span style='font-weight: bold;'>Columns: </span><span style='color: #0000BB;'>19</span></span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> "\t"</span>
<span class='c'>#&gt; <span style='color: #BB0000;'>chr</span>   (4): carrier, tailnum, origin, dest</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span>  (14): year, month, day, dep_time, sched_dep_time, dep_delay, arr_time, ...</span>
<span class='c'>#&gt; <span style='color: #0000BB;'>dttm</span>  (1): time_hour</span>
<span class='c'>#&gt; </span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Use <span style='color: #000000; background-color: #BBBBBB;'>`spec()`</span> to retrieve the full column specification for this data.</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Specify the column types or set <span style='color: #000000; background-color: #BBBBBB;'>`show_col_types = FALSE`</span> to quiet this message.</span>
<span class='nv'>data</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 32,729 x 19</span></span>
<span class='c'>#&gt;    plane   year month   day dep_time sched_dep_time dep_delay arr_time</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;chr&gt;</span>  <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>          <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span> N619AA  <span style='text-decoration: underline;'>2</span>013     1     1      542            540         2      923</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span> N3ALAA  <span style='text-decoration: underline;'>2</span>013     1     1      558            600        -<span style='color: #BB0000;'>2</span>      753</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span> N3DUAA  <span style='text-decoration: underline;'>2</span>013     1     1      559            600        -<span style='color: #BB0000;'>1</span>      941</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span> N633AA  <span style='text-decoration: underline;'>2</span>013     1     1      606            610        -<span style='color: #BB0000;'>4</span>      858</span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span> N3EMAA  <span style='text-decoration: underline;'>2</span>013     1     1      623            610        13      920</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span> N3BAAA  <span style='text-decoration: underline;'>2</span>013     1     1      628            630        -<span style='color: #BB0000;'>2</span>     <span style='text-decoration: underline;'>1</span>137</span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span> N3CYAA  <span style='text-decoration: underline;'>2</span>013     1     1      629            630        -<span style='color: #BB0000;'>1</span>      824</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span> N3GKAA  <span style='text-decoration: underline;'>2</span>013     1     1      635            635         0     <span style='text-decoration: underline;'>1</span>028</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span> N4WNAA  <span style='text-decoration: underline;'>2</span>013     1     1      656            700        -<span style='color: #BB0000;'>4</span>      854</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span> N5FMAA  <span style='text-decoration: underline;'>2</span>013     1     1      656            659        -<span style='color: #BB0000;'>3</span>      949</span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 32,719 more rows, and 11 more variables: sched_arr_time &lt;dbl&gt;,</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>#   arr_delay &lt;dbl&gt;, carrier &lt;chr&gt;, flight &lt;dbl&gt;, origin &lt;chr&gt;, dest &lt;chr&gt;,</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>#   air_time &lt;dbl&gt;, distance &lt;dbl&gt;, hour &lt;dbl&gt;, minute &lt;dbl&gt;, time_hour &lt;dttm&gt;</span></span></code></pre>

</div>

## Name repair

Often the names of columns in the original dataset are not ideal to work with. Edition two uses the same [name_repair](https://www.tidyverse.org/articles/2019/01/tibble-2.0.1/#name-repair) argument as in the tibble package, so you can use one of the default name repair strategies or provide a custom function. One useful approach is to use the [janitor::make_clean_names()](http://sfirke.github.io/janitor/).

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_tsv</a></span><span class='o'>(</span><span class='s'>"flights_AA.tsv"</span>, name_repair <span class='o'>=</span> <span class='nf'>janitor</span><span class='nf'>::</span><span class='nv'><a href='https://rdrr.io/pkg/janitor/man/make_clean_names.html'>make_clean_names</a></span><span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 32,729 x 19</span></span>
<span class='c'>#&gt;     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>          <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>     <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>          <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>  <span style='text-decoration: underline;'>2</span>013     1     1      542            540         2      923            850</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>  <span style='text-decoration: underline;'>2</span>013     1     1      558            600        -<span style='color: #BB0000;'>2</span>      753            745</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>  <span style='text-decoration: underline;'>2</span>013     1     1      559            600        -<span style='color: #BB0000;'>1</span>      941            910</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>  <span style='text-decoration: underline;'>2</span>013     1     1      606            610        -<span style='color: #BB0000;'>4</span>      858            910</span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>  <span style='text-decoration: underline;'>2</span>013     1     1      623            610        13      920            915</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>  <span style='text-decoration: underline;'>2</span>013     1     1      628            630        -<span style='color: #BB0000;'>2</span>     <span style='text-decoration: underline;'>1</span>137           <span style='text-decoration: underline;'>1</span>140</span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>  <span style='text-decoration: underline;'>2</span>013     1     1      629            630        -<span style='color: #BB0000;'>1</span>      824            810</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>  <span style='text-decoration: underline;'>2</span>013     1     1      635            635         0     <span style='text-decoration: underline;'>1</span>028            940</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>  <span style='text-decoration: underline;'>2</span>013     1     1      656            700        -<span style='color: #BB0000;'>4</span>      854            850</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>  <span style='text-decoration: underline;'>2</span>013     1     1      656            659        -<span style='color: #BB0000;'>3</span>      949            959</span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 32,719 more rows, and 11 more variables: arr_delay &lt;dbl&gt;,</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>#   carrier &lt;chr&gt;, flight &lt;dbl&gt;, tailnum &lt;chr&gt;, origin &lt;chr&gt;, dest &lt;chr&gt;,</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>#   air_time &lt;dbl&gt;, distance &lt;dbl&gt;, hour &lt;dbl&gt;, minute &lt;dbl&gt;, time_hour &lt;dttm&gt;</span></span>

<span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_tsv</a></span><span class='o'>(</span><span class='s'>"flights_AA.tsv"</span>, name_repair <span class='o'>=</span> <span class='o'>~</span> <span class='nf'>janitor</span><span class='nf'>::</span><span class='nf'><a href='https://rdrr.io/pkg/janitor/man/make_clean_names.html'>make_clean_names</a></span><span class='o'>(</span><span class='nv'>.</span>, case <span class='o'>=</span> <span class='s'>"lower_camel"</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 32,729 x 19</span></span>
<span class='c'>#&gt;     year month   day depTime schedDepTime depDelay arrTime schedArrTime arrDelay</span>
<span class='c'>#&gt;    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>        <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>        <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span>    <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 1</span>  <span style='text-decoration: underline;'>2</span>013     1     1     542          540        2     923          850       33</span>
<span class='c'>#&gt; <span style='color: #555555;'> 2</span>  <span style='text-decoration: underline;'>2</span>013     1     1     558          600       -<span style='color: #BB0000;'>2</span>     753          745        8</span>
<span class='c'>#&gt; <span style='color: #555555;'> 3</span>  <span style='text-decoration: underline;'>2</span>013     1     1     559          600       -<span style='color: #BB0000;'>1</span>     941          910       31</span>
<span class='c'>#&gt; <span style='color: #555555;'> 4</span>  <span style='text-decoration: underline;'>2</span>013     1     1     606          610       -<span style='color: #BB0000;'>4</span>     858          910      -<span style='color: #BB0000;'>12</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 5</span>  <span style='text-decoration: underline;'>2</span>013     1     1     623          610       13     920          915        5</span>
<span class='c'>#&gt; <span style='color: #555555;'> 6</span>  <span style='text-decoration: underline;'>2</span>013     1     1     628          630       -<span style='color: #BB0000;'>2</span>    <span style='text-decoration: underline;'>1</span>137         <span style='text-decoration: underline;'>1</span>140       -<span style='color: #BB0000;'>3</span></span>
<span class='c'>#&gt; <span style='color: #555555;'> 7</span>  <span style='text-decoration: underline;'>2</span>013     1     1     629          630       -<span style='color: #BB0000;'>1</span>     824          810       14</span>
<span class='c'>#&gt; <span style='color: #555555;'> 8</span>  <span style='text-decoration: underline;'>2</span>013     1     1     635          635        0    <span style='text-decoration: underline;'>1</span>028          940       48</span>
<span class='c'>#&gt; <span style='color: #555555;'> 9</span>  <span style='text-decoration: underline;'>2</span>013     1     1     656          700       -<span style='color: #BB0000;'>4</span>     854          850        4</span>
<span class='c'>#&gt; <span style='color: #555555;'>10</span>  <span style='text-decoration: underline;'>2</span>013     1     1     656          659       -<span style='color: #BB0000;'>3</span>     949          959      -<span style='color: #BB0000;'>10</span></span>
<span class='c'>#&gt; <span style='color: #555555;'># … with 32,719 more rows, and 10 more variables: carrier &lt;chr&gt;, flight &lt;dbl&gt;,</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>#   tailnum &lt;chr&gt;, origin &lt;chr&gt;, dest &lt;chr&gt;, airTime &lt;dbl&gt;, distance &lt;dbl&gt;,</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>#   hour &lt;dbl&gt;, minute &lt;dbl&gt;, timeHour &lt;dttm&gt;</span></span></code></pre>

</div>

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/base/unlink.html'>unlink</a></span><span class='o'>(</span><span class='nv'>files</span><span class='o'>)</span></code></pre>

</div>

## UTF-16 and UTF-32 support

Edition two now has much better support for UTF-16 and UTF-32 multi-byte unicode encodings. When files with these encodings are read they are automatically converted to UTF-8 internally in an efficeint streaming fashion.

## Lazy reading

Edition two uses lazy reading by default. When you first call a `read_*()` function the delimiters and newlines throughout the entire file are found, but the data is not actually read until it is used in your program. This can provide substantial speed improvements for reading character data. It is particularly useful during interactive exploration of only a subset of a full dataset.

However this also means that problematic values are not necessarily seen immediately, only when they are actually read. Because of this a warning will be issued the first time a problem is encountered, which may happen after initial reading.

Run [`problems()`](https://readr.tidyverse.org/reference/problems.html) on your dataset to read the entire dataset and return all of the problems found. Run [`problems(lazy = TRUE)`](https://readr.tidyverse.org/reference/problems.html) if you only want to retrieve the problems found so far.

Deleting files after reading is also impacted by laziness. On Windows open files cannot be deleted as long as a process has the file open. Because readr keeps a file open when reading lazily this means you cannot read, then immediately delete the file. readr will in most cases close the file once it has been completely read. However, if you know you want to be able to delete the file after reading it is best to pass `lazy = FALSE` when reading the file.

## Control over quoting and escaping when writing

You can now explicitly control how fields are quoted and escaped when writing with the `quote` and `escape` arguments to `write_*()` functions.

`quote` has three options.

1.  'needed' - which will quote fields only when needed.
2.  'all' - which will always quote all fields.
3.  'none' - which will never quote any fields.

`escape` also has three options, to control how quote characters are escaped.

1.  'double' - which will use double quotes to escape quotes.
2.  'backslash' - which will use a backslash to escape quotes.
3.  'none' - which will not do anything to escape quotes.

We hope these options will give people the flexibility they need when writing files using readr.

## Literal data

In edition one the reading functions treated any input with a newline in it or vectors of length \> 1 as literal data. In edition two vectors of length \> 1 are nowassumed to correspond to multiple files. Because of this we now have a more explicit way to represent literal data, by putting [`I()`](https://rdrr.io/r/base/AsIs.html) around the input.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>readr</span><span class='nf'>::</span><span class='nf'><a href='https://readr.tidyverse.org/reference/read_delim.html'>read_csv</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/AsIs.html'>I</a></span><span class='o'>(</span><span class='s'>"a,b\n1,2"</span><span class='o'>)</span><span class='o'>)</span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Rows: </span><span style='color: #0000BB;'>1</span> <span style='font-weight: bold;'>Columns: </span><span style='color: #0000BB;'>2</span></span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>──</span> <span style='font-weight: bold;'>Column specification</span> <span style='color: #00BBBB;'>────────────────────────────────────────────────────────</span></span>
<span class='c'>#&gt; <span style='font-weight: bold;'>Delimiter:</span> ","</span>
<span class='c'>#&gt; <span style='color: #00BB00;'>dbl</span> (2): a, b</span>
<span class='c'>#&gt; </span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Use <span style='color: #000000; background-color: #BBBBBB;'>`spec()`</span> to retrieve the full column specification for this data.</span>
<span class='c'>#&gt; <span style='color: #00BBBB;'>ℹ</span> Specify the column types or set <span style='color: #000000; background-color: #BBBBBB;'>`show_col_types = FALSE`</span> to quiet this message.</span>
<span class='c'>#&gt; <span style='color: #555555;'># A tibble: 1 x 2</span></span>
<span class='c'>#&gt;       a     b</span>
<span class='c'>#&gt;   <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span> <span style='color: #555555; font-style: italic;'>&lt;dbl&gt;</span></span>
<span class='c'>#&gt; <span style='color: #555555;'>1</span>     1     2</span></code></pre>

</div>

## Lighter installation requirements

readr now should be much easier to install. Previous versions of readr used the Boost C++ library to do some of the numeric parsing. While these are well written, robust libraries, the BH package which contains them has a large number of files (1500+) which can take a long time to install. In addition the code within these headers is complicated and can take a large amount of memory (2+ Gb) to compile, which made it challenging to compile readr from source in some cases.

readr no longer depends on Boost or the BH package, so should compile more quickly in most cases.

## Deprecated functions and features

-   [`melt_csv()`](https://readr.tidyverse.org/reference/melt_delim.html), [`melt_delim()`](https://readr.tidyverse.org/reference/melt_delim.html), [`melt_tsv()`](https://readr.tidyverse.org/reference/melt_delim.html) and [`melt_fwf()`](https://readr.tidyverse.org/reference/melt_fwf.html) have been deprecated. These functions rely on the first edition parsing code and would be challenging to update to the new parser. When the first edition parsing code is eventually removed from readr they will be split off into a new package.

-   [`read_table2()`](https://readr.tidyverse.org/reference/read_table2.html) has been renamed to [`read_table()`](https://readr.tidyverse.org/reference/read_table.html), as most users expect [`read_table()`](https://readr.tidyverse.org/reference/read_table.html) to work like [`utils::read.table()`](https://rdrr.io/r/utils/read.table.html). If you want the previous strict behavior of the [`read_table()`](https://readr.tidyverse.org/reference/read_table.html) you can use [`read_fwf()`](https://readr.tidyverse.org/reference/read_fwf.html) with [`fwf_empty()`](https://readr.tidyverse.org/reference/read_fwf.html) directly (\#717).

-   Normalizing newlines in files with just carriage returns `\r` is no longer supported. The last major OS to use only CR as the newline was 'classic' Mac OS, which had its final release in 2001.

## License changes

We are systematically re-licensing tidyverse and r-lib packages to use the MIT license, to make our package licenses as clear and permissive as possible.

To this end the readr and vroom packages are now released under the MIT license.

## Acknowledgements

A big thanks to everyone who helped make this release possible by testing the development vearsions, asking questions, providing reprexes, writing code and more! [@Aariq](https://github.com/Aariq), [@adamroyjones](https://github.com/adamroyjones), [@antoine-sachet](https://github.com/antoine-sachet), [@basille](https://github.com/basille), [@batpigandme](https://github.com/batpigandme), [@benjaminhlina](https://github.com/benjaminhlina), [@bigey](https://github.com/bigey), [@billdenney](https://github.com/billdenney), [@binkleym](https://github.com/binkleym), [@BrianOB](https://github.com/BrianOB), [@cboettig](https://github.com/cboettig), [@CTMCBP](https://github.com/CTMCBP), [@Dana996](https://github.com/Dana996), [@DarwinAwardWinner](https://github.com/DarwinAwardWinner), [@deeenes](https://github.com/deeenes), [@dernst](https://github.com/dernst), [@dicorynia](https://github.com/dicorynia), [@estroger34](https://github.com/estroger34), [@FixTestRepeat](https://github.com/FixTestRepeat), [@GegznaV](https://github.com/GegznaV), [@giocomai](https://github.com/giocomai), [@GiuliaPais](https://github.com/GiuliaPais), [@hadley](https://github.com/hadley), [@HedvigS](https://github.com/HedvigS), [@HenrikBengtsson](https://github.com/HenrikBengtsson), [@hidekoji](https://github.com/hidekoji), [@hongooi73](https://github.com/hongooi73), [@hsbadr](https://github.com/hsbadr), [@idshklein](https://github.com/idshklein), [@jasyael](https://github.com/jasyael), [@JeremyPasco](https://github.com/JeremyPasco), [@jimhester](https://github.com/jimhester), [@jonasfoe](https://github.com/jonasfoe), [@jzadra](https://github.com/jzadra), [@KasperThystrup](https://github.com/KasperThystrup), [@keesdeschepper](https://github.com/keesdeschepper), [@kingcrimsontianyu](https://github.com/kingcrimsontianyu), [@KnutEBakke](https://github.com/KnutEBakke), [@krlmlr](https://github.com/krlmlr), [@larnsce](https://github.com/larnsce), [@ldecicco-USGS](https://github.com/ldecicco-USGS), [@M3IT](https://github.com/M3IT), [@maelle](https://github.com/maelle), [@martinmodrak](https://github.com/martinmodrak), [@meowcat](https://github.com/meowcat), [@messersc](https://github.com/messersc), [@mewu3](https://github.com/mewu3), [@mgperry](https://github.com/mgperry), [@michaelquinn32](https://github.com/michaelquinn32), [@MikeJohnPage](https://github.com/MikeJohnPage), [@mine-cetinkaya-rundel](https://github.com/mine-cetinkaya-rundel), [@msberends](https://github.com/msberends), [@nbenn](https://github.com/nbenn), [@niheaven](https://github.com/niheaven), [@peranti](https://github.com/peranti), [@petrbouchal](https://github.com/petrbouchal), [@pfh](https://github.com/pfh), [@pgramme](https://github.com/pgramme), [@Raesu](https://github.com/Raesu), [@rmcd1024](https://github.com/rmcd1024), [@rmvpaeme](https://github.com/rmvpaeme), [@sebneus](https://github.com/sebneus), [@seth127](https://github.com/seth127), [@Shians](https://github.com/Shians), [@sonicdoe](https://github.com/sonicdoe), [@svraka](https://github.com/svraka), [@timothy-barry](https://github.com/timothy-barry), [@tmalsburg](https://github.com/tmalsburg), [@vankesteren](https://github.com/vankesteren), [@xuqingyu](https://github.com/xuqingyu), and [@yutannihilation](https://github.com/yutannihilation).

