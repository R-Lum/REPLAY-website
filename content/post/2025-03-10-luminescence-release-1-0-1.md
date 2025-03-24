---
title: Luminescence Release 1.0.1
date: '2025-03-10'
categories:
  - 'Luminescence'
author: Marco Colombo
description: 'Luminescence version 1.0.1'
---

With the release of [version 1.0][v100] still very fresh, we are announcing
the release of **version 1.0.1**, the first maintenance release of the
[Luminescence R package][lumi]. This is a minor release meant to address one
important regression, but it also brings a few additional robustness and
correctness improvements. In total we addressed 13 issues in 53 commits.

<!--more-->

### The main regression

Just days after the release of `Luminescence` 1.0 we discovered an important
[regression in analyse_FadingMeasurement()][iss589], which caused the plot
of the luminescence curves to be completely garbled. This was caused by an
innocuous-looking change in `plot_RLum.Analysis()`, the function that
actually performs the plotting.

The bug was actually quite subtle: it turns out that setting the `mfrow`
graphical parameter (which controls how multiple plots are displayed on a
single page) resets all other parameters to their default values, even in
the case when `mfrow` is set to its current value. This can be seen in
action in this snippet of code:

```R
## show the default parameters
> par()[c("cex", "mfrow")]
$cex
[1] 1

$mfrow
[1] 1 1

## change the cex parameter
> par(cex = 2)
> par()["cex"]
[1] 2

## setting mfrow resets cex to its default
> par(mfrow = c(1, 1))
> par()["cex"]
[1] 1
```

This reinforces our drive to expand the set of graphical snapshot tests, which
would have caught immediately this regression.

### Checking the graphical device size

In its default setting, `analyse_pIRIRSequence()` attempts to display all
plots on a single page, which can result in "figure margins too large" errors
unless the device size is large enough.

In version 1.0 we introduced a check on the device size, so that we could
return a message suggesting the users to plot to a PDF file with at least
18 inches per side. However, in [issue 593][iss593] we realised that the
user experience could be further improved.

First, the implementation was not rigorous enough and didn't prevent the user
from setting a size such as 18 x 12 inches, which would once again lead to
those undesired "figure margins too large" errors.

While investigating that, we realised that when we set a given device size
and then query R about it, R could return an value affected by some floating
point error, although this was not immediately visible. This snippet of code
shows this behaviour:

```R
## open the pdf device of a given size
> pdf("out.pdf", width = 18, height = 18)

## query the device size
> dev.size <- grDevices::dev.size("in")
> dev.size
[1] 18 18

## this was unexpected
> dev.size < 18
[1] FALSE  TRUE

## printing with higher precision shows the issue
> sprintf("%18.15f %18.15f", dev.size[1], dev.size[2])
[1] "18.000000000000000 17.999999999999996"
```

After having accounted also for this quirk, we realised that we could allow
the PDF file to be as small as 16x16 inches instead of 18x18, although we
still encourage to use 18x18 as it in our tests it produces better looking
plots.

In any case, one can use the `plot_singlePanels = TRUE` argument so that each
plot is printed independently of all others, and thus avoid using a PDF file.

### Other changes

This release brings a number of crash fixes in case of misspecified inputs,
some of which come from our current focus of bringing the [RLumShiny package][rshiny]
up to date with the current versions of `Shiny` and `Luminescence`.

The most notable changes are the following:

* `analyse_FadingMeasurement()` now checks that there is at least an
irradiation step available if the input is an object generated from an XSYG
file.

* `calc_FadingCorr()` at times produced [implausibly large error estimates][iss597]:
this would happen only sporadically, and was presumably due to some numerical
issue in `uniroot()`. We solved it by discarding error estimates that were
several times away from the interquartile range computed over all Monte Carlo
estimates. We also made this function more parsimonious in terms of memory
allocations, which brought a small performance improvement.

* `sort_RLum()` is now more resilient when it's called on an empty object (which
may happen when processing automatically a list of objects generated elsewhere)
and when a user attempts to use more than one field for sorting. This is not yet
fully supported, but at least we have now fixed the most obvious crashes. We
have plans to make this function more powerful, allowing it to sort on multiple
fields. This is being tracked in [issue 605][iss605].

Full details of all changes can be seen on the [release 1.0.1 page][v101].

As always, bug reports are very welcome, in particular if you spot other
regressions or encounter performance problems. And if you'd like to
contribute new functionalities or bug fixes directly, check out our
[guidelines for contributors][contr].

[lumi]:   https://r-lum.github.io/Luminescence/
[rshiny]: https://tzerk.github.io/RLumShiny/
[v100]:   https://github.com/R-Lum/Luminescence/releases/tag/v1.0.0
[v101]:   https://github.com/R-Lum/Luminescence/releases/tag/v1.0.1
[iss589]: https://github.com/R-Lum/Luminescence/issues/589
[iss593]: https://github.com/R-Lum/Luminescence/issues/593
[iss597]: https://github.com/R-Lum/Luminescence/issues/597
[iss605]: https://github.com/R-Lum/Luminescence/issues/605
[contr]:  https://github.com/R-Lum/Luminescence/blob/master/CONTRIBUTING.md
