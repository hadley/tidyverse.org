---
output: hugodown::hugo_document

slug: here-1-0-0
title: here 1.0.0 and rprojroot 2.0.2
date: 2020-11-15
author: Kirill Müller
description: >
    here offers a simple way to find your files in a project-oriented workflow.
    Under the hood, rprojroot implements the logic.
    Version 1.0.0 introduces a new way to declare the project root, and brings a few other features.

photo:
  url: https://unsplash.com/photos/C2zhShTnl5I
  author: Nick Fewings

# one of: "deep-dive", "learn", "package", "programming", or "other"
categories: [package] 
tags: []
rmd_hash: 941faf1943715603

---

<!--
TODO:
* [x] Pick category and tags (see existing with `post_tags()`)
* [x] Find photo & update yaml metadata
* [x] Create `thumbnail-sq.jpg`; height and width should be equal
* [x] Create `thumbnail-wd.jpg`; width should be >5x height
* [x] `hugodown::use_tidy_thumbnail()`
* [ ] Add intro sentence
* [ ] `use_tidy_thanks()`
-->

We're chuffed to announce the release of [here](https://here.r-lib.org/) 1.0.0 and [rprojroot](https://rprojroot.r-lib.org/) 2.0.2. here offers a simple way to find your files in a [project-oriented workflow](https://rstats.wtf/project-oriented-workflow.html). Under the hood, rprojroot implements the logic. Version 1.0.0, the first update since the original CRAN release, introduces a new way to declare the project root. This release also includes an overhaul of the documentation and added safety for path construction.

You can install it from CRAN with:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://rdrr.io/r/utils/install.packages.html'>install.packages</a></span><span class='o'>(</span><span class='s'>"here"</span><span class='o'>)</span>
</code></pre>

</div>

This blog post shows the two most important user-facing changes: declaring the project root in your scripts and reports, and mixing project-relative and absolute paths. You can see a full list of changes in the release notes for [here](https://here.r-lib.org/news/index.html) and [rprojroot](https://rprojroot.r-lib.org/news/index.html).

Here I am!
----------

The source of this blog post lives in the `index.Rmd` file in the `content/blog/here-1-0-0` subdirectory of the [tidyverse.org](https://github.com/tidyverse/tidyverse.org) repository. It declares its own location:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>here</span><span class='nf'>::</span><span class='nf'><a href='https://here.r-lib.org//reference/i_am.html'>i_am</a></span><span class='o'>(</span><span class='s'>"content/blog/here-1-0-0/index.Rmd"</span><span class='o'>)</span>

<span class='c'>#&gt; here() starts at /home/kirill/git/R/tidyverse.org</span>
</code></pre>

</div>

A message describes the location of the project root on my machine. It is probably different on your machine, the purpose of the here package is to help with that situation.

After establishing the project root, the [`here()`](https://here.r-lib.org//reference/here.html) function helps navigate the project.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://here.r-lib.org/'>here</a></span><span class='o'>)</span>
<span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://github.com/r-lib/conflicted'>conflicted</a></span><span class='o'>)</span>

<span class='nf'><a href='https://rdrr.io/r/base/list.files.html'>dir</a></span><span class='o'>(</span><span class='nf'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='o'>(</span><span class='s'>"content"</span>, <span class='s'>"blog"</span>, <span class='s'>"here-1-0-0"</span><span class='o'>)</span><span class='o'>)</span>

<span class='c'>#&gt; [1] "index.md"         "index.Rmd"        "thumbnail-sq.jpg" "thumbnail-wd.jpg"</span>

<span class='nf'><a href='https://rdrr.io/r/base/list.files.html'>dir</a></span><span class='o'>(</span><span class='nf'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='o'>(</span><span class='s'>"content/blog/here-1-0-0"</span><span class='o'>)</span><span class='o'>)</span>

<span class='c'>#&gt; [1] "index.md"         "index.Rmd"        "thumbnail-sq.jpg" "thumbnail-wd.jpg"</span>

<span class='nf'><a href='https://rdrr.io/r/base/strwrap.html'>strwrap</a></span><span class='o'>(</span><span class='nf'><a href='https://rdrr.io/r/base/readLines.html'>readLines</a></span><span class='o'>(</span><span class='nf'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='o'>(</span><span class='s'>"CODE_OF_CONDUCT.md"</span><span class='o'>)</span>, <span class='m'>5</span><span class='o'>)</span><span class='o'>)</span>

<span class='c'>#&gt;  [1] "# Contributor Covenant Code of Conduct"                                </span>
<span class='c'>#&gt;  [2] ""                                                                      </span>
<span class='c'>#&gt;  [3] "## Our Pledge"                                                         </span>
<span class='c'>#&gt;  [4] ""                                                                      </span>
<span class='c'>#&gt;  [5] "In the interest of fostering an open and welcoming environment, we as" </span>
<span class='c'>#&gt;  [6] "contributors and maintainers pledge to making participation in our"    </span>
<span class='c'>#&gt;  [7] "project and our community a harassment-free experience for everyone,"  </span>
<span class='c'>#&gt;  [8] "regardless of age, body size, disability, ethnicity, gender identity"  </span>
<span class='c'>#&gt;  [9] "and expression, level of experience, nationality, personal appearance,"</span>
<span class='c'>#&gt; [10] "race, religion, or sexual identity and orientation."</span>
</code></pre>

</div>

You can still load[^1] the here package without calling [`here::i_am()`](https://here.r-lib.org//reference/i_am.html). The new approach resolves ambiguity and also protects against running the file in the wrong project or outside of a project:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'>here</span><span class='nf'>::</span><span class='nf'><a href='https://here.r-lib.org//reference/i_am.html'>i_am</a></span><span class='o'>(</span><span class='s'>"foo/bar.R"</span><span class='o'>)</span>

<span class='c'>#&gt; Error: Could not find associated project in working directory or any parent directory.</span>
<span class='c'>#&gt; - Path in project: foo/bar.R</span>
<span class='c'>#&gt; - Current working directory: /home/kirill/git/R/tidyverse.org/content/blog/here-1-0-0</span>
<span class='c'>#&gt; Please open the project associated with this file and try again.</span>
</code></pre>

</div>

Read more at [`vignette("here", package = "here")`](https://here.r-lib.org/articles/here.html).

Absolute vs. relative paths
---------------------------

The result of [`here()`](https://here.r-lib.org//reference/here.html) is always a character vector with absolute paths. Prior to rprojroot 2.0.2, passing absolute paths to [`here()`](https://here.r-lib.org//reference/here.html) resulted in garbage. The update modifies this behavior: if the first argument to [`here()`](https://here.r-lib.org//reference/here.html) is an absolute path, the project root is ignored and [`here()`](https://here.r-lib.org//reference/here.html) works like [`file.path()`](https://rdrr.io/r/base/file.path.html):

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>blog_path</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='o'>(</span><span class='s'>"content/blog/here-1-0-0"</span><span class='o'>)</span>
<span class='nv'>blog_path</span>

<span class='c'>#&gt; [1] "/home/kirill/git/R/tidyverse.org/content/blog/here-1-0-0"</span>

<span class='nf'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='o'>(</span><span class='s'>"content/blog/here-1-0-0"</span>, <span class='s'>"index.Rmd"</span><span class='o'>)</span>

<span class='c'>#&gt; [1] "/home/kirill/git/R/tidyverse.org/content/blog/here-1-0-0/index.Rmd"</span>

<span class='nf'><a href='https://here.r-lib.org//reference/here.html'>here</a></span><span class='o'>(</span><span class='nv'>blog_path</span>, <span class='s'>"index.Rmd"</span><span class='o'>)</span>

<span class='c'>#&gt; [1] "/home/kirill/git/R/tidyverse.org/content/blog/here-1-0-0/index.Rmd"</span>
</code></pre>

</div>

This allows mixing project-relative and absolute paths without compromising on safety.

Acknowledgements
----------------

We would like to thank the contributors who have helped with this release with pull requests and discussion.

### here

[@ABcreation98](https://github.com/ABcreation98), [@Alanocallaghan](https://github.com/Alanocallaghan), [@antass](https://github.com/antass), [@batpigandme](https://github.com/batpigandme), [@boshek](https://github.com/boshek), [@cderv](https://github.com/cderv), [@chris-prener](https://github.com/chris-prener), [@cloversleaves](https://github.com/cloversleaves), [@cportner](https://github.com/cportner), [@czeildi](https://github.com/czeildi), [@ghost](https://github.com/ghost), [@hadley](https://github.com/hadley), [@ijlyttle](https://github.com/ijlyttle), [@JamesCuster](https://github.com/JamesCuster), [@jasonpott](https://github.com/jasonpott), [@jennybc](https://github.com/jennybc), [@karldw](https://github.com/karldw), [@kpjonsson](https://github.com/kpjonsson), [@lgaborini](https://github.com/lgaborini), [@LTzavella](https://github.com/LTzavella), [@moodymudskipper](https://github.com/moodymudskipper), [@NoushinN](https://github.com/NoushinN), [@nzgwynn](https://github.com/nzgwynn), [@pjrdata](https://github.com/pjrdata), [@prosoitos](https://github.com/prosoitos), [@rajanand](https://github.com/rajanand), [@robertamezquita](https://github.com/robertamezquita), [@Sebaristoteles](https://github.com/Sebaristoteles), [@sharlagelfand](https://github.com/sharlagelfand), [@smach](https://github.com/smach), [@solarchemist](https://github.com/solarchemist), [@StevenHibble](https://github.com/StevenHibble), [@StevenMMortimer](https://github.com/StevenMMortimer), [@swayson](https://github.com/swayson), and [@yogat3ch](https://github.com/yogat3ch).

### rprojroot

[@BarkleyBG](https://github.com/BarkleyBG), [@batpigandme](https://github.com/batpigandme), [@ctbrown](https://github.com/ctbrown), [@florisvdh](https://github.com/florisvdh), [@hadley](https://github.com/hadley), [@hansvancalster](https://github.com/hansvancalster), [@jennybc](https://github.com/jennybc), [@jonathan-g](https://github.com/jonathan-g), [@jthurner](https://github.com/jthurner), [@kslays](https://github.com/kslays), [@moodymudskipper](https://github.com/moodymudskipper), [@uribo](https://github.com/uribo), and [@yonicd](https://github.com/yonicd).

[^1]: attach with [`library()`](https://rdrr.io/r/base/library.html)

