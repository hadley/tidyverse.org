---
title: rlang 0.3.0
date: 2018-10-29
slug: rlang-0-3-0
author: Lionel Henry
categories: [package]
tags:
  - rlang
  - r-lib
description: >
    API for working with tidy evaluation, base R types, and errors
photo:
  url: https://unsplash.com/photos/nI7knd5sQfo
  author: Brandon Siu
---








## Introduction

We're happy to announce that [rlang 0.3.0](https://cran.r-project.org/package=rlang) in now on CRAN! rlang is of most interest to package developers and R programmers, as it is intended for people developing data science tools rather than data scientists. rlang implements a consistent API for working with base types, hosts the tidy evaluation framework, and offers tools for error reporting. This release provides major improvements for each of those themes.

Consult the [changelog](https://rlang.r-lib.org/news/index.html#rlang-0-3-0) for the full list of changes, including many bug fixes. The rlang API is still maturing and a number of functions and arguments were deprecated or renamed. Check the [lifecycle section](https://rlang.r-lib.org/news/index.html#lifecycle) for a summary of the API changes.


## Tidy evaluation and tidy dots

Tidy evaluation is the framework that powers data-masking APIs like dplyr, tidyr, or ggplot2. Tidy dots is a related feature that allows you to use `!!!` in functions taking dots, among other things.


### Referring to columns with `.data`

The main user-facing change is that subsetting the `.data` pronoun with `[[` now behaves as if the index were implicitly unquoted. Concretely, this means that the index can no longer be confused with a data frame column. Subsetting `.data` is now always safe, even in functions:


```r
suppressPackageStartupMessages(
  library("dplyr")
)

df <- tibble(var = 1:4, g = c(1, 1, 2, 2))
var <- "g"

# `df` contains `var` but the column doesn't count!
df %>% group_by(.data[[var]])
```

<div class = "output"><pre class="knitr r">#&gt; <span style='color: #555555;'># A tibble: 4 x 2</span><span>
#&gt; </span><span style='color: #555555;'># Groups:   g [2]</span><span>
#&gt;     var     g
#&gt;   </span><span style='color: #555555;font-style: italic;'>&lt;int&gt;</span><span> </span><span style='color: #555555;font-style: italic;'>&lt;dbl&gt;</span><span>
#&gt; </span><span style='color: #555555;'>1</span><span>     1     1
#&gt; </span><span style='color: #555555;'>2</span><span>     2     1
#&gt; </span><span style='color: #555555;'>3</span><span>     3     2
#&gt; </span><span style='color: #555555;'>4</span><span>     4     2
</span></pre></div>


### New tidy dots options

Tidy dots refers to a set of features enabled in functions collecting dots. To enable tidy dots, use `list2()` instead of list:


```r
fn <- function(...) list2(...)
```

With tidy dots, users can splice in lists of arguments:


```r
x <- list(arg1 = "A", arg2 = "B")

fn(1, 2, !!!x, 3)
```

<div class = "output"><pre class="knitr r">#&gt; [[1]]
#&gt; [1] 1
#&gt; 
#&gt; [[2]]
#&gt; [1] 2
#&gt; 
#&gt; $arg1
#&gt; [1] "A"
#&gt; 
#&gt; $arg2
#&gt; [1] "B"
#&gt; 
#&gt; [[5]]
#&gt; [1] 3
</pre></div>

They can unquote names:


```r
nm <- "b"

fn(a = 1, !!nm := 2)
```

<div class = "output"><pre class="knitr r">#&gt; $a
#&gt; [1] 1
#&gt; 
#&gt; $b
#&gt; [1] 2
</pre></div>

And trailing empty arguments are always ignored to make copy-pasting easier:


```r
fn(
  foo = "foo",
  foo = "bar",
)
```

<div class = "output"><pre class="knitr r">#&gt; $foo
#&gt; [1] "foo"
#&gt; 
#&gt; $foo
#&gt; [1] "bar"
</pre></div>

While `list2()` hard-codes these features, `dots_list()` gains several options to control how to collect dots:

* `.preserve_empty` preserves empty arguments:

  
  ```r
  list3 <- function(...) dots_list(..., .preserve_empty = TRUE)
  
  list3(a = 1, b = , c = 2)
  ```
  
  <div class = "output"><pre class="knitr r">#&gt; $a
  #&gt; [1] 1
  #&gt; 
  #&gt; $b
  #&gt; 
  #&gt; 
  #&gt; $c
  #&gt; [1] 2
  </pre></div>

  We are using this option in `env_bind()` and `call_modify()` to allow assigning explicit missing values (see `?missing_arg()`):

  
  ```r
  call <- quote(mean())
  call_modify(call, ... = , trim = )
  ```
  
  <div class = "output"><pre class="knitr r">#&gt; mean(..., trim = )
  </pre></div>

* `.homonyms` controls whether to keep all arguments that have the same name (the default), only the first or last of these, or throw an error:

  
  ```r
  list3 <- function(...) dots_list(..., .homonyms = "last")
  
  list3(foo = 1, bar = 2, foo = 3, bar = 4, bar = 5)
  ```
  
  <div class = "output"><pre class="knitr r">#&gt; $foo
  #&gt; [1] 3
  #&gt; 
  #&gt; $bar
  #&gt; [1] 5
  </pre></div>

These options can be set in `enquos()` as well.


## Error reporting

`abort()` extends `base::stop()` to make it easy to create error objects with [custom class and metadata](https://adv-r.hadley.nz/conditions.html). With rlang 0.3.0, `abort()` automatically stores a backtrace in the error object and supports chaining errors.


### Backtraces

Storing a backtrace in rlang errors makes it possible to post-process the call tree that lead to an error and simplify it substantially. Let's define three functions calling each other, with `tryCatch()` and `evalq()` interspersed in order to create a complicated call tree:


```r
f <- function() tryCatch(g(), warning = identity) # Try g()
g <- function() evalq(h())                        # Eval h()
h <- function() abort("Oh no!")                   # And fail!
```

When a function signals an error with `abort()`, the user is invited to call `last_error()`:


```r
f()
```

<div class = "output"><pre class="knitr r">#&gt; Error: Oh no!
#&gt; <span style='color: #555555;'>Call `rlang::last_error()` to see a backtrace</span><span>
</span></pre></div>

Calling `last_error()` returns the last error object. The error prints with its backtrace:


```r
last_error()
```

<div class = "output"><pre class="knitr r">#&gt; <span style='font-weight: bold;'>&lt;error&gt;</span><span>
#&gt; message: </span><span style='font-style: italic;'>Oh no!</span><span>
#&gt; class:   `rlang_error`
#&gt; backtrace:
#&gt; </span><span style='color: #BB0000;'> ─global::f()</span><span>
#&gt; </span><span style='color: #BB0000;'> ─global::g()</span><span>
#&gt; </span><span style='color: #BB0000;'> ─global::h()</span><span>
#&gt; </span><span style='color: #555555;'>Call `summary(rlang::last_error())` to see the full backtrace</span><span>
</span></pre></div>

The backtrace is simple and to the point because it is printed in a simplified form by default. If you'd like to see the full story (or include the full backtrace in a bug report), call `summary()` on the error object:


```r
summary(last_error())
```

<div class = "output"><pre class="knitr r">#&gt; <span style='font-weight: bold;'>&lt;error&gt;</span><span>
#&gt; message: </span><span style='font-style: italic;'>Oh no!</span><span>
#&gt; class:   `rlang_error`
#&gt; fields:  `message`, `trace` and `parent`
#&gt; backtrace:
#&gt; </span><span style='color: #BB0000;'>█</span><span>
#&gt; </span><span style='color: #BB0000;'>└─global::f()</span><span>
#&gt; </span><span style='color: #BB0000;'>  ├─base::tryCatch(g(), warning = identity)</span><span>
#&gt; </span><span style='color: #BB0000;'>  │ └─base:::tryCatchList(expr, classes, parentenv, handlers)</span><span>
#&gt; </span><span style='color: #BB0000;'>  │   └─base:::tryCatchOne(expr, names, parentenv, handlers[[1L]])</span><span>
#&gt; </span><span style='color: #BB0000;'>  │     └─base:::doTryCatch(return(expr), name, parentenv, handler)</span><span>
#&gt; </span><span style='color: #BB0000;'>  └─global::g()</span><span>
#&gt; </span><span style='color: #BB0000;'>    ├─base::evalq(h())</span><span>
#&gt; </span><span style='color: #BB0000;'>    │ └─base::evalq(h())</span><span>
#&gt; </span><span style='color: #BB0000;'>    └─global::h()</span><span>
</span></pre></div>

Each call is prepended with a namespace prefix[^1] to reveal the flow of control across package contexts.

[^1]: Or `global::` if the function is defined in the global workspace.


### Chained errors

Chaining errors is relevant when you're calling low-level APIs such as web scraping, JSON parsing, etc. When these APIs encounter an error, they often fail with technical error messages. It is often a good idea to transform these developer-friendly error messages into something more meaningful and actionable for end users.

Several programming languages provide the ability of chaining errors for these situations. With chained errors, the low level and high level contexts are clearly separated in the error report. This makes the error more legible for the end user, without hiding the low level information that might be crucial for figuring out the problem.

Say we're writing a function `make_report()` to create an automated report and we're downloading a file as part of the process with `fetch_csv()`, which might be implemented in a package:


```r
fetch_csv <- function(url) {
  suppressWarnings(
    read.csv(url(url))
  )
}

prepare_data <- function(url) {
  data <- fetch_csv(url)
  tibble::as_tibble(data)
}

make_report <- function(url) {
  data <- prepare_data(url)

  # We're not going to get there because all our attempts to download
  # a file are going to fail!
  ...
}
```

This function might fail in `fetch_csv()` because of connection issues:


```r
make_report("https://rstats.edu/awesome-data.csv")
```

<div class = "output"><pre class="knitr r">#&gt; Error in open.connection(file, "rt"): cannot open the connection to 'https://rstats.edu/awesome-data.csv'
</pre></div>

Chaining errors makes it possible to transform this low-level API error into a high level error, without losing any debugging information. There are two steps involved in error chaining: catch low level errors, and rethrow them with a high level message. Catching can be done with `base::tryCatch()` or `rlang::with_handlers()`. Both these functions take an error handler: a function of one argument which is passed an error object when an error occurs.

To chain an error, simply call `abort()` in the error handler, with a high level error message and the original error passed as the `parent` argument. Here we're going to use `with_handlers()` because it supports the rlang syntax for lambda functions (also used in purrr), which makes it easy to write simple handlers:


```r
prepare_data <- function(url) {
  data <- with_handlers(
    error = ~ abort("Can't download file!", parent = .),
    fetch_csv(url)
  )
  tibble::as_tibble(data)
}

make_report("https://rstats.edu/awesome-data.csv")
```

<div class = "output"><pre class="knitr r">#&gt; Error: Can't download file!
#&gt; Parents:
#&gt;  ─cannot open the connection to 'https://rstats.edu/awesome-data.csv'
</pre></div>

The main error message is now the high level one. The low level message is still included in the output to avoid hiding precious debugging information. Errors can be chained multiple times and all the messages and all parent messages are included in the output. But note that only errors thrown with `abort()` contain a backtrace:


```r
last_error()
```

<div class = "output"><pre class="knitr r">#&gt; <span style='font-weight: bold;'>&lt;error&gt;</span><span>
#&gt; message: </span><span style='font-style: italic;'>Can't download file!</span><span>
#&gt; class:   `rlang_error`
#&gt; backtrace:
#&gt; </span><span style='color: #BB0000;'> ─global::make_report("https://rstats.edu/awesome-data.csv")</span><span>
#&gt; </span><span style='color: #BB0000;'> ─global::prepare_data(url)</span><span>
#&gt; </span><span style='font-weight: bold;'>&lt;error: parent&gt;</span><span>
#&gt; message: </span><span style='font-style: italic;'>cannot open the connection to 'https://rstats.edu/awesome-data.csv'</span><span>
#&gt; class:   `simpleError`
</span></pre></div>

For this reason, chaining errors is more effective with rlang errors than with errors thrown with `stop()` and the error report could be improved if `fetch_csv()` used `abort()` instead of `thrown()`. Fortunately it is easy to transform any error into an rlang error without changing any code!


### Promoting base errors to rlang errors

rlang provides `with_abort()` to run code with base errors automatically promoted to rlang errors. Let's wrap around `fetch_csv()` to run it in a `with_abort` context:


```r
my_fetch_csv <- function(url) {
  with_abort(fetch_csv(url))
}

prepare_data <- function(url) {
  data <- with_handlers(
    error = ~ abort("Can't download file!", parent = .),
    my_fetch_csv(url)
  )
  tibble::as_tibble(data)
}
```

Our own function calls `abort()` and the foreign functions are called within a `with_abort()`. Let's see how chained errors now look like:


```r
make_report("https://rstats.edu/awesome-data.csv")
```

<div class = "output"><pre class="knitr r">#&gt; Error: Can't download file!
#&gt; Parents:
#&gt;  ─cannot open the connection to 'https://rstats.edu/awesome-data.csv'
#&gt; <span style='color: #555555;'>Call `rlang::last_error()` to see a backtrace</span><span>
</span></pre></div>

The backtraces are automatically segmented between low level and high level contexts:


```r
last_error()
```

<div class = "output"><pre class="knitr r">#&gt; <span style='font-weight: bold;'>&lt;error&gt;</span><span>
#&gt; message: </span><span style='font-style: italic;'>Can't download file!</span><span>
#&gt; class:   `rlang_error`
#&gt; backtrace:
#&gt; </span><span style='color: #BB0000;'> ─global::make_report("https://rstats.edu/awesome-data.csv")</span><span>
#&gt; </span><span style='color: #BB0000;'> ─global::prepare_data(url)</span><span>
#&gt; </span><span style='font-weight: bold;'>&lt;error: parent&gt;</span><span>
#&gt; message: </span><span style='font-style: italic;'>cannot open the connection to 'https://rstats.edu/awesome-data.csv'</span><span>
#&gt; class:   `rlang_error`
#&gt; backtrace:
#&gt; </span><span style='color: #BB0000;'> ─global::my_fetch_csv(url)</span><span>
#&gt; </span><span style='color: #BB0000;'> ─global::fetch_csv(url)</span><span>
#&gt; </span><span style='color: #BB0000;'> ─utils::read.csv(url(url))</span><span>
#&gt; </span><span style='color: #BB0000;'> ─utils::read.table(...)</span><span>
#&gt; </span><span style='color: #BB0000;'> ─base::open.connection(file, "rt")</span><span>
#&gt; </span><span style='color: #555555;'>Call `summary(rlang::last_error())` to see the full backtrace</span><span>
</span></pre></div>

```r
summary(last_error())
```

<div class = "output"><pre class="knitr r">#&gt; <span style='font-weight: bold;'>&lt;error&gt;</span><span>
#&gt; message: </span><span style='font-style: italic;'>Can't download file!</span><span>
#&gt; class:   `rlang_error`
#&gt; fields:  `message`, `trace` and `parent`
#&gt; backtrace:
#&gt; </span><span style='color: #BB0000;'>█</span><span>
#&gt; </span><span style='color: #BB0000;'>└─global::make_report("https://rstats.edu/awesome-data.csv")</span><span>
#&gt; </span><span style='color: #BB0000;'>  └─global::prepare_data(url)</span><span>
#&gt; </span><span style='font-weight: bold;'>&lt;error: parent&gt;</span><span>
#&gt; message: </span><span style='font-style: italic;'>cannot open the connection to 'https://rstats.edu/awesome-data.csv'</span><span>
#&gt; class:   `rlang_error`
#&gt; fields:  `message`, `trace`, `parent` and `error`
#&gt; backtrace:
#&gt; </span><span style='color: #BB0000;'>█</span><span>
#&gt; </span><span style='color: #BB0000;'>└─global::my_fetch_csv(url)</span><span>
#&gt; </span><span style='color: #BB0000;'>  ├─rlang::with_abort(fetch_csv(url))</span><span>
#&gt; </span><span style='color: #BB0000;'>  │ └─base::withCallingHandlers(...)</span><span>
#&gt; </span><span style='color: #BB0000;'>  └─global::fetch_csv(url)</span><span>
#&gt; </span><span style='color: #BB0000;'>    ├─base::suppressWarnings(read.csv(url(url)))</span><span>
#&gt; </span><span style='color: #BB0000;'>    │ └─base::withCallingHandlers(expr, warning = function(w) invokeRestart("muffleWarning"))</span><span>
#&gt; </span><span style='color: #BB0000;'>    └─utils::read.csv(url(url))</span><span>
#&gt; </span><span style='color: #BB0000;'>      └─utils::read.table(...)</span><span>
#&gt; </span><span style='color: #BB0000;'>        ├─base::open(file, "rt")</span><span>
#&gt; </span><span style='color: #BB0000;'>        └─base::open.connection(file, "rt")</span><span>
</span></pre></div>

If you'd like to promote all errors to rlang errors at all time, you can try out this experimental option by adding this to your RProfile:


```r
if (requireNamespace("rlang", quietly = TRUE)) {
  options(error = quote(rlang:::add_backtrace()))
}
```


## Environments

The environment API gains two specialised print methods. `env_print()` prints information about the contents and the properties of environments. If you don't specify an environment, it prints the current environment by default, here the global environment:


```r
env_print()
```

<div class = "output"><pre class="knitr r">#&gt; <span style='font-weight: bold;'>&lt;environment: global&gt;</span><span>
#&gt; parent: &lt;environment: package:bindrcpp&gt;
#&gt; bindings:
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>fn: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>var: &lt;chr&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>x: &lt;list&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>fetch_csv: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>my_fetch_csv: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>list3: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>nm: &lt;chr&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>.Random.seed: &lt;int&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>call: &lt;language&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>f: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>g: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>h: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>make_report: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>df: &lt;tibble&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>colourise_chunk: &lt;fn&gt;
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>prepare_data: &lt;fn&gt;
</span></pre></div>

The global environment doesn't have any fancy features. Let's look at a package environment:


```r
env_print(pkg_env("rlang"))
```

<div class = "output"><pre class="knitr r">#&gt; <span style='font-weight: bold;'>&lt;environment: package:rlang&gt; [L]</span><span>
#&gt; parent: &lt;environment: package:stats&gt;
#&gt; bindings:
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>is_dbl_na: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>coerce_class: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>as_quosure: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>as_quosures: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>quo_get_env: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>return_to: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>env_binding_are_lazy: &lt;fn&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>quo_is_call: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>new_call: &lt;fn&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>is_scoped: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>set_names: &lt;fn&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>expr_deparse: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>`f_env&lt;-`: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>as_box_if: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>`%|%`: &lt;fn&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>lang_head: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>ns_env: &lt;fn&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>list_along: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>parse_quo: &lt;lazy&gt; [L]
#&gt;  </span><span style='color: #555555;font-weight: bold;'>* </span><span>lang_tail: &lt;lazy&gt; [L]
#&gt;    * ... with 438 more bindings
</span></pre></div>

This environment contains all functions exported by rlang. Its header includes the `[L]` tag to indicate that the environment is locked: you can't add or remove bindings from it. The same tag appears next to each binding to indicate that the bindings are locked and can't be changed to point to a different object. Finally, note how the type of many bindings is `<lazy>`. That's because packages are lazily loaded for performance reasons. Technically, the binding points to a *promise* that will eventually evaluate to the actual object, the first time it is accessed.

The second print method concerns lists of environments as returned by `search_envs()` and `env_parents()`. While `base::search()` returns the names of environments on the search path, `search_envs()` returns the corresponding list of environments:


```r
search_envs()
```

<div class = "output"><pre class="knitr r">#&gt;  [[1]] $ &lt;env: global&gt;
#&gt;  [[2]] $ &lt;env: package:bindrcpp&gt;
#&gt;  [[3]] $ &lt;env: package:dplyr&gt;
#&gt;  [[4]] $ &lt;env: package:rlang&gt;
#&gt;  [[5]] $ &lt;env: package:stats&gt;
#&gt;  [[6]] $ &lt;env: package:graphics&gt;
#&gt;  [[7]] $ &lt;env: package:grDevices&gt;
#&gt;  [[8]] $ &lt;env: package:utils&gt;
#&gt;  [[9]] $ &lt;env: package:datasets&gt;
#&gt; [[10]] $ &lt;env: package:methods&gt;
#&gt; [[11]] $ &lt;env: Autoloads&gt;
#&gt; [[12]] $ &lt;env: package:base&gt;
</pre></div>

`env_parents()` returns all parents of a given environment. For search environments, the last parent of the list is the empty environment:


```r
envs <- env_parents(pkg_env("utils"))
envs
```

<div class = "output"><pre class="knitr r">#&gt; [[1]] $ &lt;env: package:datasets&gt;
#&gt; [[2]] $ &lt;env: package:methods&gt;
#&gt; [[3]] $ &lt;env: Autoloads&gt;
#&gt; [[4]] $ &lt;env: package:base&gt;
#&gt; [[5]] $ &lt;env: empty&gt;
</pre></div>

For all other environments, the last parent is either the empty environment or the global environment. Most of the time the global env is part of the ancestry because package namespaces inherit from the search path:


```r
env_parents(ns_env("rlang"))
```

<div class = "output"><pre class="knitr r">#&gt; [[1]] $ &lt;env: imports:rlang&gt;
#&gt; [[2]] $ &lt;env: namespace:base&gt;
#&gt; [[3]] $ &lt;env: global&gt;
</pre></div>

It is possible to construct environments insulated from the search path. We'll use `env()` to create such an environment. Counting from rlang 0.3.0, you can pass a single unnamed environment to `env()` to specify a parent. The following creates a child of the base package:


```r
e <- env(base_env(), foo = "bar")
env_parents(e)
```

<div class = "output"><pre class="knitr r">#&gt; [[1]] $ &lt;env: package:base&gt;
#&gt; [[2]] $ &lt;env: empty&gt;
</pre></div>

Here is how to create a grandchild of the empty environment:


```r
e <- env(env(empty_env()))
env_parents(e)
```

<div class = "output"><pre class="knitr r">#&gt; [[1]]   &lt;env: 0x7fb6ec084238&gt;
#&gt; [[2]] $ &lt;env: empty&gt;
</pre></div>

We hope that these print methods make it easier to explore the structure and contents of R environments.


## Acknowledgements

Thanks to all contributors!

[&#xFF20;akbertram](https://github.com/akbertram), [&#xFF20;AndreMikulec](https://github.com/AndreMikulec), [&#xFF20;andresimi](https://github.com/andresimi), [&#xFF20;billdenney](https://github.com/billdenney), [&#xFF20;BillDunlap](https://github.com/BillDunlap), [&#xFF20;cfhammill](https://github.com/cfhammill), [&#xFF20;egnha](https://github.com/egnha), [&#xFF20;grayskripko](https://github.com/grayskripko), [&#xFF20;hadley](https://github.com/hadley), [&#xFF20;IndrajeetPatil](https://github.com/IndrajeetPatil), [&#xFF20;jimhester](https://github.com/jimhester), [&#xFF20;krlmlr](https://github.com/krlmlr), [&#xFF20;marinsokol5](https://github.com/marinsokol5), [&#xFF20;md0u80c9](https://github.com/md0u80c9), [&#xFF20;mikmart](https://github.com/mikmart), [&#xFF20;move[bot]](https://github.com/move[bot]), [&#xFF20;NikNakk](https://github.com/NikNakk), [&#xFF20;privefl](https://github.com/privefl), [&#xFF20;romainfrancois](https://github.com/romainfrancois), [&#xFF20;wibeasley](https://github.com/wibeasley), [&#xFF20;yutannihilation](https://github.com/yutannihilation), and [&#xFF20;zslajchrt](https://github.com/zslajchrt)
