[![Netlify Status](https://api.netlify.com/api/v1/badges/90b72bec-4010-40b3-bce3-2d84c3fd417f/deploy-status)](https://app.netlify.com/sites/tidyverse-org/deploys)

# tidyverse.org

This repo is the source of <https://tidyverse.org>, and this readme tells you 
how it all works. 

* If you spot any small problems with the website, please feel empowered to fix 
  them directly with a PR. 
  
* If you see any larger problems, an issue is probably better: that way we can 
  discuss the problem before you commit any time to it.

* If you'd like to contribute a blog post, please chat with one of us first.
  Then read the [contributing guide](CONTRIBUTING.md).

This repo (and resulting website) is licensed as [CC BY-SA](LICENSE.md).

## Structure

The source of the website is a collection of `.md` and `.Rmd` files stored in 
[`content/`](content/), which are rendered for the site with 
[blogdown](https://bookdown.org/yihui/blogdown). 

* `content/*/index.md`: these files generate the top-level pages on the site:
  packages, learn, help, and contribute. 
  
* `content/blog/`: these files are the tidyverse blog. New blog entries
  should be given name `year-month-slug/index.Rmarkdown`. This date isn't actually used when generating the output: you'll need to set up the yaml metadata. More on that below.
    + For `*.markdown` posts, no `*.html` file should be committed. If you generate one locally during development, delete it once it's no longer useful to you. Keep it out of this repo.
    
    + For `*.Rmarkdown` posts, an `*.markdown` file _should_ be committed. It will be generated when you run `blogdown::serve_site()` (see recommended workflow below).      
    + If your post includes emoji, use the `.Rmarkdown` format, and incorporate emoji using the [emo](https://github.com/hadley/emo) package.  

* `data/events.yaml`: this yaml file contains information about upcoming 
  events. The site automatically filters out events that have happened,
  sorts by date, and then shows at most two events.

* 

## Previewing changes

### Use `blogdown::serve_site()`

To build the site locally, you'll need to install blogdown, and then install 
hugo, the music behind the magic of blogdown:

```R
install.packages("blogdown")
blogdown::install_hugo()
```

Then run

```R
blogdown::serve_site()
```

This will open a preview of the site in your web browser, and it will 
automatically update whenever you modify one of the input files. For `.Rmarkdown`, 
this will generate an `.markdown` file, which you should commit and push to GitHub.

#### Troubleshooting

If blogdown attempts to re-render posts (potentially on a massive scale), you need to make all the derived files look more recently modified than their respective source files. This affects (`.Rmarkdown`, `.markdown`) and (`.Rmd`, `.html`) file pairs. Do something like this:

```R
library(fs)

md <- dir_ls("content", recurse = TRUE, glob = "*.markdown")
file_touch(md)

html <- dir_ls("content", recurse = TRUE, glob = "*.html")
file_touch(html)
```

For other problems, consider that you need to update blogdown or to run `blogdown::update_hugo()` (perhaps in an R session launched with `sudo`).

#### Other methods of local preview

You should really preview the site using `blogdown::serve_site()`. But if, accidentally or intentionally, you knit or preview the content using another method (e.g. click the **Preview** button in RStudio for `.[R]md`), make sure you don't commit an `.html` file from an **`.md`** file.

### In PRs

The tidyverse site is automatically published with 
[netlify](http://netlify.com/). One big advantage of netlify is that every PR 
automatically gets a live preview. Once the PR is merged, that preview becomes 
the live site.
