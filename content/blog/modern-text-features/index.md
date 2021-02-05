---
output: hugodown::hugo_document

slug: modern-text-features
title: Modern Text Features in R
date: 2021-02-06
author: Thomas Lin Pedersen
description: >
  ragg has taken a major leap forward in text rendering capabilities with the
  latest releases of systemfonts, textshaping, and ragg itself. This post will
  go into detail with what is now possible and how it compares to the build in 
  devices.

photo:
  url: https://unsplash.com/photos/bMybTSV7RFY
  author: Natalia Y

# one of: "deep-dive", "learn", "package", "programming", or "other"
categories: [deep-dive] 
tags: []
rmd_hash: c239d02c76ac3d3b

---

I'm extremely pleased to present some new functionality when it comes to text rendering and font support in R. This is the culmination of work that started during the development of the ragg package where I was first exposed to the intricacies of text rendering. The new features presented herein spans the systemfonts, textshaping, and ragg packages, but from a user point of view everything will be available simply by using the graphic devices in ragg.

ragg is now in a state where it "just works", and works comparably across all major operating systems. This is a first for a raster device in R and is the result of several years of dedicated work. I couldn't be happier with this release!

The features that will be discussed in the following are:

1.  Support for non-Latin scripts including Right-to-Left (RtL) scripts as well as a mix between script directions (bidirectional text)
2.  Support for OpenType features such as ligatures, glyph substitutions, etc.
3.  Support for color fonts
4.  Support for font fallback

The tl;dr of it is that all area mentioned above now has full support in ragg out of the box, but I'd invite you to read on to learn how it works, how to control it, and what it all means for you as a user.

### Graphical tl;dr;

With the new version of ragg you'll be able to render plots such as this and expect it to simply work:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='http://ggplot2.tidyverse.org'>ggplot2</a></span><span class='o'>)</span>
<span class='nv'>city_names</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span>
  <span class='s'>"Tokyo (東京)"</span>,
  <span class='s'>"Yokohama (横浜)"</span>,
  <span class='s'>"Osaka (大阪市)"</span>,
  <span class='s'>"Nagoya (名古屋市)"</span>,
  <span class='s'>"Sapporo (札幌市)"</span>,
  <span class='s'>"Kobe (神戸市)"</span>,
  <span class='s'>"Kyoto (京都市)"</span>,
  <span class='s'>"Fukuoka (福岡市)"</span>,
  <span class='s'>"Kawasaki (川崎市)"</span>,
  <span class='s'>"Saitama (さいたま市)"</span>
<span class='o'>)</span>
<span class='nv'>main_cities</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://rdrr.io/r/base/data.frame.html'>data.frame</a></span><span class='o'>(</span>
  name <span class='o'>=</span> <span class='nv'>city_names</span>,
  lat <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='m'>35.690</span>, <span class='m'>35.444</span>, <span class='m'>34.694</span>, <span class='m'>35.183</span>, <span class='m'>43.067</span>, 
          <span class='m'>34.69</span>, <span class='m'>35.012</span>, <span class='m'>33.583</span>, <span class='m'>35.517</span>, <span class='m'>35.861</span><span class='o'>)</span>,
  lon <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='m'>139.692</span>, <span class='m'>139.638</span>, <span class='m'>135.502</span>, <span class='m'>136.9</span>, <span class='m'>141.35</span>, 
          <span class='m'>135.196</span>, <span class='m'>135.768</span>, <span class='m'>130.4</span>, <span class='m'>139.7</span>, <span class='m'>139.646</span><span class='o'>)</span>
<span class='o'>)</span>
<span class='nv'>japan</span> <span class='o'>&lt;-</span> <span class='nf'>rnaturalearth</span><span class='nf'>::</span><span class='nf'><a href='https://rdrr.io/pkg/rnaturalearth/man/ne_countries.html'>ne_countries</a></span><span class='o'>(</span>
  scale <span class='o'>=</span> <span class='m'>10</span>, 
  country <span class='o'>=</span> <span class='s'>"Japan"</span>, 
  returnclass <span class='o'>=</span> <span class='s'>"sf"</span>
<span class='o'>)</span>
<span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggplot.html'>ggplot</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggsf.html'>geom_sf</a></span><span class='o'>(</span>
    data <span class='o'>=</span> <span class='nv'>japan</span>, 
    fill <span class='o'>=</span> <span class='s'>"forestgreen"</span>, 
    colour <span class='o'>=</span> <span class='s'>"grey10"</span>, 
    size <span class='o'>=</span> <span class='m'>0.2</span>
  <span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'>ggrepel</span><span class='nf'>::</span><span class='nf'><a href='https://rdrr.io/pkg/ggrepel/man/geom_text_repel.html'>geom_label_repel</a></span><span class='o'>(</span>
    <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span><span class='nv'>lon</span>, <span class='nv'>lat</span>, label <span class='o'>=</span> <span class='nv'>name</span><span class='o'>)</span>, 
    data <span class='o'>=</span> <span class='nv'>main_cities</span>,
    fill <span class='o'>=</span> <span class='s'>"#FFFFFF88"</span>,
    box.padding <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/grid/unit.html'>unit</a></span><span class='o'>(</span><span class='m'>5</span>, <span class='s'>"mm"</span><span class='o'>)</span>
  <span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_point.html'>geom_point</a></span><span class='o'>(</span><span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span><span class='nv'>lon</span>, <span class='nv'>lat</span><span class='o'>)</span>, <span class='nv'>main_cities</span><span class='o'>)</span> <span class='o'>+</span>
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/labs.html'>ggtitle</a></span><span class='o'>(</span>
    <span class='s'>"Location of largest cities in Japan (日本) 🇯🇵"</span>
  <span class='o'>)</span> <span class='o'>+</span>
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggtheme.html'>theme_void</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/theme.html'>theme</a></span><span class='o'>(</span>panel.background <span class='o'>=</span> <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/element.html'>element_rect</a></span><span class='o'>(</span><span class='s'>"steelblue"</span><span class='o'>)</span>,
        plot.title <span class='o'>=</span> <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/element.html'>element_text</a></span><span class='o'>(</span>margin <span class='o'>=</span> <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/element.html'>margin</a></span><span class='o'>(</span><span class='m'>5</span>, <span class='m'>0</span>, <span class='m'>5</span>, <span class='m'>0</span><span class='o'>)</span><span class='o'>)</span><span class='o'>)</span>

</code></pre>
<img src="figs/unnamed-chunk-1-1.png" width="700px" style="display: block; margin: auto;" />

</div>

Pay note to the effortless mix of text in English and Japanese, along with the inclusion of emojis in regular text strings. If this has piqued your interest, read on!

Advanced script support
-----------------------

English, being the lingua franca of programming, has generally dominated everything related to text within programming, ranging from encoding to rendering. Because of this, the Latin script, which is used in most of the western world, has been the best (or often only) supported script in many text rendering pipelines. This has also been true in the R world where the build-in graphic devices has struggled to display scripts that differed from the standard Latin layout (the Cairo devices on Linux being the exception as we will see). This is not a jab at the provided graphic devices. ragg has handled text rendering in the exact same way up until now and the work that has gone into changing it has not been trivial. Technicalities aside, it is about time (overdue, actually) that the graphic system in R begins to support the display of non-Latin script and becomes more inclusive in terms of which languages can be used. It is thus with great joy that I can finally support it in ragg.

### Right-to-Left scripts

To start off we will look at a sample of different scripts that poses a problem due to their direction.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>arabic_text</span> <span class='o'>&lt;-</span> <span class='s'>"هذا مكتوب باللغة العربية"</span>
<span class='nv'>hebrew_text</span> <span class='o'>&lt;-</span> <span class='s'>"זה כתוב בעברית"</span>
<span class='nv'>sindhi_text</span> <span class='o'>&lt;-</span> <span class='s'>"هي سنڌيءَ ۾ لکيو ويو آهي"</span>

<span class='nv'>p</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggplot.html'>ggplot</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_text.html'>geom_text</a></span><span class='o'>(</span>
    <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span>, y <span class='o'>=</span> <span class='m'>3</span><span class='o'>:</span><span class='m'>1</span>, label <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='nv'>arabic_text</span>, <span class='nv'>hebrew_text</span>, <span class='nv'>sindhi_text</span><span class='o'>)</span><span class='o'>)</span>, 
    family <span class='o'>=</span> <span class='s'>"Arial"</span>
  <span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/expand_limits.html'>expand_limits</a></span><span class='o'>(</span>y <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='m'>0</span>, <span class='m'>4</span><span class='o'>)</span><span class='o'>)</span>

<span class='nf'>preview_devices</span><span class='o'>(</span><span class='nv'>p</span>, <span class='s'>"rtl_example"</span><span class='o'>)</span>
</code></pre>

</div>

<div class="highlight">

<img src="figs/rtl_example_macOS_ragg.png" width="33%" style="display: inline;"><img src="figs/rtl_example_Windows_ragg.png" width="33%" style="display: inline;"><img src="figs/rtl_example_Linux_ragg.png" width="33%" style="display: inline;"><img src="figs/rtl_example_macOS_cairo.png" width="33%" style="display: inline;"><img src="figs/rtl_example_Windows_cairo.png" width="33%" style="display: inline;"><img src="figs/rtl_example_Linux_cairo.png" width="33%" style="display: inline;"><img src="figs/rtl_example_macOS_quartz.png" width="33%" style="display: inline;"><img src="figs/rtl_example_Windows_windows.png" width="33%" style="display: inline;">

</div>

Now, if you are not trained in reading any of the three languages above it may be hard to see what is right and what is wrong. You may, however, look at how the text in the code is rendered in the browser and compare that to the device rendering. If you do that you can see that the Hebrew script is rendered in the wrong direction for all the non-ragg devices (except Cairo on Linux). For the other scripts it is less obvious that the direction is wrong since the text glyphs are fundamentally different due to ligature substitutions. Still, by comparing to the browser rendering you can see that the same devices failing on the Hebrew script fail here as well.

A point to make is that all of this "just works" in ragg. How is that? Shouldn't we have to indicate which kind of script we want to use? This is not necessary due to the genius of the Unicode standard which relates characters to specific scripts. The script, and by extension the layout, can thus simply be deduced from the provided string without needing to specify any additional information.

The Cairo device on Linux handles this task well, as we have noted above. How come this works, but only on one OS (Cairo on macOS and Windows performs as bad as the other native devices)? Cairo is a fundamental library of many Linux distributions and is usually build on top of the Pango library which handles text layouting on Linux. It thus have access to OS level text rendering when used on Linux, but not on the other major platforms. The text rendering quality of the Cairo device on Linux is something that will be consistent through these examples and while it is very good, it is not fair to ask users to switch operating system to get modern font support. Even if you are already using Linux you'd have to accept that your output is not necessarily reproducible on other systems if you using the default device.

### Bidirectional text

In the case of a mix of scripts, most importantly a mix of scripts with different direction, the string needs to be split based on the bidirectional algorithm (also defined in the Unicode standard) and each script run should then be laid out individually and combined in the end.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>bidi_text</span> <span class='o'>&lt;-</span> <span class='s'>"The Hebrew (עִברִית) script\nis right-to-left"</span>

<span class='nv'>p</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggplot.html'>ggplot</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_text.html'>geom_text</a></span><span class='o'>(</span>
    <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span>, y <span class='o'>=</span> <span class='m'>0</span>, label <span class='o'>=</span> <span class='nv'>bidi_text</span><span class='o'>)</span>, 
    family <span class='o'>=</span> <span class='s'>"Arial"</span>
  <span class='o'>)</span>

<span class='nf'>preview_devices</span><span class='o'>(</span><span class='nv'>p</span>, <span class='s'>"bidi_example"</span><span class='o'>)</span>
</code></pre>

</div>

<div class="highlight">

<img src="figs/bidi_example_macOS_ragg.png" width="33%" style="display: inline;"><img src="figs/bidi_example_Windows_ragg.png" width="33%" style="display: inline;"><img src="figs/bidi_example_Linux_ragg.png" width="33%" style="display: inline;"><img src="figs/bidi_example_macOS_cairo.png" width="33%" style="display: inline;"><img src="figs/bidi_example_Windows_cairo.png" width="33%" style="display: inline;"><img src="figs/bidi_example_Linux_cairo.png" width="33%" style="display: inline;"><img src="figs/bidi_example_macOS_quartz.png" width="33%" style="display: inline;"><img src="figs/bidi_example_Windows_windows.png" width="33%" style="display: inline;">

</div>

It comes as no surprise that the devices that struggle with RtL scripts also fail when these are mixed with Left-to-Right (LtR), so the point here is mainly that ragg (and Cairo on Linux) supports this additional complication with no additional work on the user.

### Scripts with special consideration

While some scripts are simple in the sense that only the text direction needs to be reversed (e.g. Hebrew), others are more demanding of the layout algorithm. Arabic, for example, is not only a RtL script but also relies heavily on ligatures (substitution of multiple glyphs with a single one) and position adjustments to achieve the correct look of the text. Such information is not encoded in the text string but are instead rules encoded in the font used to render it. Correctly laying out a string will thus require both figuring out the script to use, as well as converting the characters in the string to the correct set of glyphs to use based on substitution tables found in the font file. This is not straightforward, but is being handled in ragg (and Cairo on Linux) as they both build upon the HarfBuzz library to lay out text.

Advanced font feature support
-----------------------------

As noted above, part of supporting some scripts is to have support for ligatures. While ligatures is a requirement for the correct rendering of some scripts it is also an optional feature of fonts in general to support different text variations. More generally, the OpenType font format describes a long range of features, many optional, that defines specific glyph substitutions (both one-to-one and many-to-one) or position adjustments that can be turned on or off and will affect the look of the final rendered text. Some of these features are turned on automatically for specific scripts (e.g. required ligatures for Arabic), while others are left for the user to turn on at their discretion (e.g. tabular numerics). As part of the work to add support for non-Latin scripts the infrastructure to support all OpenType features was build. This, of course, requires that the font in use supports the requested feature.

Some fonts uses ligatures as a main part of their appeal, and these will now work as advertised with ragg:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>code</span> <span class='o'>&lt;-</span> <span class='s'>"x &lt;- y != z"</span>
<span class='nv'>logo</span> <span class='o'>&lt;-</span> <span class='s'>"twitter"</span>
<span class='nv'>p</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggplot.html'>ggplot</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_text.html'>geom_text</a></span><span class='o'>(</span>
    <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span>, y <span class='o'>=</span> <span class='m'>2</span>, label <span class='o'>=</span> <span class='nv'>code</span><span class='o'>)</span>, 
    family <span class='o'>=</span> <span class='s'>"Fira Code"</span>
  <span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_text.html'>geom_text</a></span><span class='o'>(</span>
    <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span>, y <span class='o'>=</span> <span class='m'>1</span>, label <span class='o'>=</span> <span class='nv'>logo</span><span class='o'>)</span>, 
    family <span class='o'>=</span> <span class='s'>"Font Awesome 5 brands"</span>
  <span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/expand_limits.html'>expand_limits</a></span><span class='o'>(</span>y <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='m'>0</span>, <span class='m'>3</span><span class='o'>)</span><span class='o'>)</span>

<span class='nf'>preview_devices</span><span class='o'>(</span><span class='nv'>p</span>, <span class='s'>"def_features"</span><span class='o'>)</span>
</code></pre>

</div>

<div class="highlight">

<img src="figs/def_features_macOS_ragg.png" width="33%" style="display: inline;"><img src="figs/def_features_Windows_ragg.png" width="33%" style="display: inline;"><img src="figs/def_features_Linux_ragg.png" width="33%" style="display: inline;"><img src="figs/def_features_macOS_cairo.png" width="33%" style="display: inline;"><img src="figs/def_features_Windows_cairo.png" width="33%" style="display: inline;"><img src="figs/def_features_Linux_cairo.png" width="33%" style="display: inline;"><img src="figs/def_features_macOS_quartz.png" width="33%" style="display: inline;"><img src="figs/def_features_Windows_windows.png" width="33%" style="display: inline;">

</div>

But what about non-default features? The capabilities of the graphic engine in R presents a problem here. There is very little information that the user is able to sent along with the text to be plotted, apart from location and font (**bold** and *italic* on/off is the extend of it). So, having a device with support for advanced OpenType features in and off itself is nearly useless as there is no way to specify in your plot code that you want to turn a feature on or off.

In order to get around this without dropping support for the standard ways one puts text in plots with R, systemfonts now allows you to register font features along with a font under a different name. The font registration mechanism was previously mostly used for giving access to fonts that were not installed on the system (but e.g. provided by a package), but its use has now expanded and a [`register_variant()`](https://rdrr.io/pkg/systemfonts/man/register_variant.html) function has been added to quickly create a new version of an existing font:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='kr'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='o'>(</span><span class='nv'><a href='https://github.com/r-lib/systemfonts'>systemfonts</a></span><span class='o'>)</span>
<span class='nf'><a href='https://rdrr.io/pkg/systemfonts/man/register_variant.html'>register_variant</a></span><span class='o'>(</span>
  name <span class='o'>=</span> <span class='s'>"Montserrat Extreme"</span>, 
  family <span class='o'>=</span> <span class='s'>"Montserrat"</span>, 
  weight <span class='o'>=</span> <span class='s'>"semibold"</span>,
  features <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/pkg/systemfonts/man/font_feature.html'>font_feature</a></span><span class='o'>(</span>ligatures <span class='o'>=</span> <span class='s'>"discretionary"</span>, letters <span class='o'>=</span> <span class='s'>"stylistic"</span><span class='o'>)</span>
<span class='o'>)</span>
</code></pre>

</div>

The code above creates a new font based on Montserrat using a light weight and turning on standard ligatures and stylistic letter substitution. Now, in your text plotting code all you have to do is specify `"Montserrat Extreme"` as the font family and the features and weights will be used. It should be noted that there is no point in comparing with other devices here, since none of the others are build on top of systemfonts and will thus not have accessed to the registered font:

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggplot.html'>ggplot</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_text.html'>geom_text</a></span><span class='o'>(</span>
    <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span>, y <span class='o'>=</span> <span class='m'>1</span>, label <span class='o'>=</span> <span class='s'>"This text should definitely differ"</span><span class='o'>)</span>,
    family <span class='o'>=</span> <span class='s'>"Montserrat"</span>
  <span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_text.html'>geom_text</a></span><span class='o'>(</span>
    <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span>, y <span class='o'>=</span> <span class='m'>0</span>, label <span class='o'>=</span> <span class='s'>"This text should definitely differ"</span><span class='o'>)</span>,
    family <span class='o'>=</span> <span class='s'>"Montserrat Extreme"</span>
  <span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/expand_limits.html'>expand_limits</a></span><span class='o'>(</span>y <span class='o'>=</span> <span class='nf'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='o'>(</span><span class='o'>-</span><span class='m'>1</span>, <span class='m'>2</span><span class='o'>)</span><span class='o'>)</span>

</code></pre>
<img src="figs/unnamed-chunk-10-1.png" width="700px" style="display: block; margin: auto;" />

</div>

We can see that by using this font registration we gain access to weights other than normal and bold, but also to glyph substitutions such as the "Th" ligature, and the stylistic variations seen with the "t", "f", "l", and "e" glyphs.

While a lot of the optional OpenType features are mainly of interest to achieve a specific stylistic look of the rendered text, some have more importance for data visualizations, such as those related to how numbers are displayed. It is both possible to force even-width numbers, as well as correct display of fractional numbers using OpenType as long as the font supports it, so it is definitely something to look into when you want to add that final polish to your visualization.

Colour fonts
------------

A recent (in font technology terms) development is the availability of color fonts, i.e. fonts where the glyphs have designated colors. This development is largely driven by the ubiquity of emojis in modern text, and while it may seem that emojis have been around forever, it is recent enough that the world has yet to converge to a single standard for color fonts. The system emoji font on macOS, Windows, and Linux all uses different font technologies for storing the color glyphs, ranging from storing a single bitmap, to storing each glyph as an SVG. This, unsurprisingly, complicates things. To add insult to injury, emojis often gets rendered slightly larger than the surrounding text and with a slightly lowered baseline in a very OS-specific way (this does not apply to all color fonts; only emojis).

Why am I telling you this? Well, honestly it is mostly to make you appreciate the labor that went into the fact that color fonts (and by extension, emojis) now just works.

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>emojis</span> <span class='o'>&lt;-</span> <span class='s'>"👩🏾‍💻🔥📊"</span>

<span class='nv'>p</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggplot.html'>ggplot</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_text.html'>geom_label</a></span><span class='o'>(</span>
    <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span>, y <span class='o'>=</span> <span class='m'>0</span>, label <span class='o'>=</span> <span class='nv'>emojis</span><span class='o'>)</span>, 
    family <span class='o'>=</span> <span class='s'>"Apple Color Emoji"</span>
  <span class='o'>)</span>

<span class='nf'>preview_devices</span><span class='o'>(</span><span class='nv'>p</span>, <span class='s'>"emoji"</span><span class='o'>)</span>
</code></pre>

</div>

<div class="highlight">

<img src="figs/emoji_macOS_ragg.png" width="33%" style="display: inline;"><img src="figs/emoji_Windows_ragg.png" width="33%" style="display: inline;"><img src="figs/emoji_Linux_ragg.png" width="33%" style="display: inline;"><img src="figs/emoji_macOS_cairo.png" width="33%" style="display: inline;"><img src="figs/emoji_Windows_cairo.png" width="33%" style="display: inline;"><img src="figs/emoji_Linux_cairo.png" width="33%" style="display: inline;"><img src="figs/emoji_macOS_quartz.png" width="33%" style="display: inline;"><img src="figs/emoji_Windows_windows.png" width="33%" style="display: inline;">

</div>

As one can see the failures range from not being able to render anything, to rendering in monochrome. Further, it appears as if the devices have trouble figuring out the dimensions of the glyphs. One additional wrinkle is that while Cairo on macOS capable of rendering in monochrome, it fails to get the correct emoji. This is because emojis relies heavily on ligatures, and the "dark-skinned woman at a computer" emoji is actually a ligature of the "woman", "dark skin" and "computer" emojis.

Font fallback
-------------

In all of the above examples we have been very mindful in setting the font-face to a font that contains all the glyphs we need. This is not always practical, especially when one wants to mix emojis and regular text such as it is done normally. It is also an absolute requirement when mixing Latin and CJK (Chinese, Japanese, and Korean) text as it is unfeasible to include all CJK glyphs in a single font. However, we are used to things just working at the system level. No matter the font it seems that a glyph is always displayed in e.g. browsers and text editors. This is because the OS is employing font fallback, which is the act of figuring out an alternative font to use when a glyph is not present in the chosen font. Wouldn't it be great if we could have that in a graphic device? Well, now we have!

<div class="highlight">

<pre class='chroma'><code class='language-r' data-lang='r'><span class='nv'>fallback_text</span> <span class='o'>&lt;-</span> <span class='s'>"This is English, この文は日本語です 🚀"</span>

<span class='nv'>p</span> <span class='o'>&lt;-</span> <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/ggplot.html'>ggplot</a></span><span class='o'>(</span><span class='o'>)</span> <span class='o'>+</span> 
  <span class='nf'><a href='https://ggplot2.tidyverse.org/reference/geom_text.html'>geom_text</a></span><span class='o'>(</span><span class='nf'><a href='https://ggplot2.tidyverse.org/reference/aes.html'>aes</a></span><span class='o'>(</span>x <span class='o'>=</span> <span class='m'>0</span>, y <span class='o'>=</span> <span class='m'>0</span>, label <span class='o'>=</span> <span class='nv'>fallback_text</span><span class='o'>)</span>, size <span class='o'>=</span> <span class='m'>2.5</span><span class='o'>)</span>

<span class='nf'>preview_devices</span><span class='o'>(</span><span class='nv'>p</span>, <span class='s'>"fallback"</span><span class='o'>)</span>
</code></pre>

</div>

<div class="highlight">

<img src="figs/fallback_macOS_ragg.png" width="33%" style="display: inline;"><img src="figs/fallback_Windows_ragg.png" width="33%" style="display: inline;"><img src="figs/fallback_Linux_ragg.png" width="33%" style="display: inline;"><img src="figs/fallback_macOS_cairo.png" width="33%" style="display: inline;"><img src="figs/fallback_Windows_cairo.png" width="33%" style="display: inline;"><img src="figs/fallback_Linux_cairo.png" width="33%" style="display: inline;"><img src="figs/fallback_macOS_quartz.png" width="33%" style="display: inline;"><img src="figs/fallback_Windows_windows.png" width="33%" style="display: inline;">

</div>

The bottom line is that with ragg, you now don't need to think about missing glyphs in any font you choose (unless you request a character that is not covered by any font on your system).

Where's the catch
-----------------

Most of what we have shown today simply works automagically and may (depending on your prior frustrations with script support in R) seem too good to be true. Is there any catch? Not really. systemfonts, textshaping, and ragg tries to be as smart as possible about text shaping and only take additional action if required. Further everything is heavily cached. Any hit on performance is thus negligible.

There is something missing though, which we haven't touched upon. Not all scripts are LtR or RtL. A few, especially Asian scripts, are top-to-bottom. Top-to-bottom scripts are sadly not yet supported. This is not due to any limitation in the underlying shaping technology, but due to limitations in the R graphic engine, which assumes horizontal text in key places of the API. This means that until the graphic engine is updated it is outside the grasp of graphic devices to support vertical text. Hopefully, this is an area that will improve in the future.

Wrapping up
-----------

I hope you'll appreciate the new features being described here. I'd like to thank everyone who have helped validate the text rendering on Twitter. A special thank goes out to Behdad Esfahbod (<a href="http://behdad.org" class="uri">http://behdad.org</a>) for his work on HarfBuzz, Fribidi, and almost everything else underlying modern font rendering. He has been especially gracious in his help and support.

