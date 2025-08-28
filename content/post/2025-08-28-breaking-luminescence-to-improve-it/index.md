---
title: Breaking Luminescence to improve it
date: '2025-08-28'
categories:
  - 'Luminescence'
author: Marco Colombo
description: 'CBTF version 0.5.0'
---

One of the challenges I faced as I started working on the
[Luminescence R package][lumi] was how to navigate the very large set of
functions that the package provides (by the last count, there are 155
functions that are exposed to the user, plus several internal helpers).

As I fumbled my way through it, I started noticing that when a function failed
on a certain input, similar failures would also occur elsewhere.
However, finding manually which other functions were affected was frustrating
and time-consuming. Wouldn't it be great if we could (quickly and with limited
manual intervention) go through all the functions in the package and being
told which are the ones that need to be fixed?

This got me to think about **fuzz testing**, and how that approach could be
adapted for my purposes.

<!--more-->

### Fuzz testing

[Fuzz testing][fuzz] is a software testing technique that generates random,
unexpected or malformed data and feeds them into a program. By observing
how the software behaves under such conditions, this approach may uncover
potential weaknesses, vulnerabilities and security flaws in applications that
could be exploited by malicious actors.

As such, it's adopted in systems where maintaining the integrity of the
software and of the machine running it is of crucial importance, such as in
the development of operating systems, hardware drivers, language compilers,
browsers, cryptographic, network-facing and security-related software.

Clearly, `Luminescence` doesn't fall into that category of software. However,
the underlying idea is still valid: in our setting, fuzz testing can be used
to identify functions that lack sufficient argument validation and to uncover
sets of inputs that may cause issues within the function body.

A major problem with using a fuzz-testing approach is dealing with false
positives. Reporting all errors produced is not particularly helpful, as
some errors are expected or they have already been handled, and therefore
don't need to be looked at again. How do we know if an error reported is
something we should be worrying about?

### How `Luminescence` reports errors

The functions in `Luminescence`are instrumented with a simple but effective
error report interface. Whenever a user-facing function throws an error that
the developer had expected (and handled), it also reports the function's name
in the error message. For example:

```R
read_BIN2R("wrong.bin")
```
```
## Error: [read_BIN2R()] File 'wrong.bin' does not exist
```

Consider instead this other failure (already fixed in version 1.0.0):
```R
read_BIN2R(character(0))
```
```
## Error in if (grepl(pattern = "https?\\:\\/\\/", x = url, perl = TRUE)) { :
##   argument is of length zero
```

Clearly, this error wasn't handled by the function, but it comes straight
from the R core engine. Therefore, it should be reported as a true positive,
as it signals insufficient argument validation. Effectively, the code assumed
that the user would never provide a zero-length file name. While this is
probably true in the majority of cases, it may still occur accidentally or
as part of a more complex script.

It makes for a better user experience if even these corner cases were
gracefully handled by the package. In contrast to the one above, the current
error message to shows immediately the cause of the problem, making it easier
to find a solution:
```
## Error: [read_BIN2R()] 'file' cannot be an empty character
```

The way errors and warnings are reported in `Luminescence` provides an
effective way of removing the vast majority of false positives from the
output. This mechanism allows us to assume that, if a function reports an
error message containing its own name, the developers have already considered
that possibility and wrote the code defensively against that type of failure.
Such errors can be considered *handled* and can be suppressed from the
output. This has the big advantage of leaving a minimal number of cases to be
investigated after fuzzing.

### Caught by the Fuzz!

The [initial implementation][v010] of our fuzz-testing approach was really
straightforward and fit in about 50 lines of R code. Indeed, it is simply a
matter of calling each function in the package with a given input (one that
had already shown to be problematic somewhere) and record any error or warning
produced. Those that appear to be unhandled errors are reported so that they
can be inspected and fixed.

From these humble beginnings, the code slowly developed into a fully-fledged
R package, [CBTF (Caught by the Fuzz!)][cbtf], which as of two weeks ago has
finally reached [CRAN][cran].

[![CBTF-logo](cbtf-logo.png#center "Thanks to Supergrass for the inspiration for the package name!")][song]

The `CBTF` package was developed with the expressed purpose of testing and
improving robustness of `Luminescence` (although nothing prevents it from being
used for other packages too).

The core functionality of the package is in the `fuzz()` function, which calls
each provided function with a certain input and records the output produced.
The `get_exported_functions()` helper is used to find all user-facing functions
from a package, which are those that will be fuzzed.

This is what happened last November when I fuzzed all functions using a data
frame containing a zero value in the first column as input:
```R
library(CBTF)
funs <- get_exported_functions("Luminescence")
what <- list(df_with_zero = data.frame(ED = 0:5,
                                       ED_Err = 1))
fuzz(funs, what, ignore_warnings = TRUE)
```
```
ℹ Fuzzing 154 functions on 1 input
✔ Test input: df_with_zero  [5.1s]
✖  🚨   CAUGHT BY THE FUZZ!   🚨

── Test input: df_with_zero
      analyse_IRSAR.RF  FAIL  no applicable method for `@` applied to an object of class "integer"
   calc_CobbleDoseRate  FAIL  missing value where TRUE/FALSE needed
calc_OSLLxTxDecomposed  FAIL  'names' attribute [4] must be the same length as the vector [2]
      fit_OSLLifeTimes  FAIL  NaN value of objective function!  Perhaps adjust the bounds.
             get_Quote  FAIL  the condition has length > 1
        github_commits  FAIL  [github_branches()] 'user' should be of class 'character'
       plot_RadialPlot  FAIL  'from' must be a finite number

[ FAIL 7 | WARN 0 | SKIP 2 | OK 145 ]
```

These pointed to a number of unhandled error conditions that were
[eventually solved][pr819], so that the current output is the following:
```
ℹ Fuzzing 155 functions on 1 input
✔ Test input: df_with_zero  [5.1s]
✔  🏃 You didn't get caught by the fuzz!

 [ FAIL 0 | WARN 0 | SKIP 2 | OK 153 ]
```

We began using `CBTF` shortly before the release of
[`Luminescence` version 1.0.0][v100]. At that point, we had 272 calls to our
validation functions; for version 1.0.0 we increased them to 459, and
further to 582 for version 1.1.0. These numbers do not count any
additional checks and validations that were done using standard R
constructs, nor fixes to bugs revealed by `CBTF` that were unrelated
to input value validation. All applications of `CBTF` to `Luminescence` are
documented in a [tracking bug][is439] which, as of today, lists 35 issues,
for a total of 212 problems identified and fixed.

We will continue using this approach to further improve the stability and
user-friendliness of `Luminescence`, and hope that similar techniques will
be used more and more in the R ecosystem at large.

[lumi]:  https://r-lum.github.io/Luminescence/
[fuzz]:  https://en.wikipedia.org/wiki/Fuzzing
[v010]:  https://github.com/mcol/caught-by-the-fuzz/blob/v0.1.0/R/fuzz.R
[cbtf]:  https://mcol.github.io/caught-by-the-fuzz/
[cran]:  https://cran.r-project.org/package=CBTF
[song]:  https://www.youtube.com/watch?v=uJ-mpul94eo
[pr819]: https://github.com/R-Lum/Luminescence/pull/819
[v100]:  {{< ref "post/2025-02-24-the-road-to-luminescence-1-0/" >}}
[is439]: https://github.com/R-Lum/Luminescence/issues/439
