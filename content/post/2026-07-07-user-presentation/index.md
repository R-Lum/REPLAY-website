---
title: Presentation about 'CBTF' at useR! 2026
date: '2026-07-07'
categories:
  - 'Luminescence'
author: Marco Colombo
description: 'useR! presentation'
---

Today I am presenting the [CBTF (Caught by the Fuzz!)][cbtf] package at the
[useR! 2026 conference][user] in Warsaw.

I have already [written about CBTF in the past][post]. The package has grown
a lot in the meantime, adding new functionality, better user experience and
lighting-fast performance.

<!--more-->

[![CBTF-logo](cbtf-logo.png#center "Thanks to Supergrass for the inspiration for the package name!")][song]

The slides for the talk are available [here](useR-slides.pdf). Unfortunately,
given time constraints, a number of interesting and fascinating issues needed
to be cut from the presentation. Here I'll summarise the changes that have
occurred in the `CBTF` package over the last year.

## Support for argument lists

The original version of `CBTF` was only able to fuzz the first function
argument. This was already enough to identify a huge number of cases in
which `Luminescence` would crash upon a misspecified input.

However, for the package to be fully functional, it was a must to support
fuzzing any argument list. This meant adding support for the `args` argument,
so that each of its element could be modified in turn by each of the input
values (specified in the `what` argument). It became immediately obvious that
support for named arguments was essential. This would allow us to fuzz
arguments out of order and also to reach arguments that are passed via `...`.
So, for example, if we set `args = list(x = 11, y = 22, 33)` and
`what = list(NA, "")`, the following inputs would be generated and tested:
- `list(x = NA, y = 22, 33)`
- `list(x = "", y = 22, 33)`
- `list(x = 11, y = NA, 33)`
- `list(x = 11, y = "", 33)`
- `list(x = 11, y = 22, NA)`
- `list(x = 11, y = 22, "")`

I then realised that in a number of cases it may be convenient to fix some
arguments to a given value, so that they are never fuzzed. This is helpful
when we are already satisfied that a specific argument is fully validated,
or in any other cases when we want to reduce the computation time, or simply
to reduce the number of failures reported. Fixed arguments can be specified
by prepending a `..` to the argument name. Continuing the previous example,
if we set `args = list(..x = 11, y = 22, 33)`, the following inputs would be
tested:
- `list(x = 11, y = NA, 33)`
- `list(x = 11, y = "", 33)`
- `list(x = 11, y = 22, NA)`
- `list(x = 11, y = 22, "")`

## Parallelisation and timeouts

When several function arguments are being fuzzed with a large number of inputs,
the number of tests to be performed can grow very quickly, especially if the
number of functions to fuzz is sizeable. For example, in `Luminescence` we
have about 140 fuzzable functions: fuzzing 5 arguments with 50 inputs brings
already 35,000 function calls.

Given that each function call is independent of all the others, the code is
trivially parallelisable. However, being able to parallelise effectively the
execution of functions may not be enough. If a given input passes validation
without triggering an error, the function may run to completion, which in some
cases can take several seconds or longer. Another example of problematic
behaviour occurs when the fuzzed function calls `readline()`: in this case,
the function waits for user input, which may interrupt the fuzzing process
indefinitely. Therefore, there must be a way to set a maximum time after which
execution is interrupted.

An initial attempt to solve the timeout issue was performed with `callr`. This
allows to run execution in a background process, which can be killed after a
certain amount of time if execution is taking too long. However, after a
process was killed, a new background process had to be started before the
rest of the fuzzing could occur. This was not particularly performant: running
the `CBTF` test suite (without self-fuzzing) would take 20 seconds with `callr`
instead of the usual 2s; fuzzing `Luminescence` took 346s instead of the usual
215s (but this latter figure could be reached only if functions that called
`readline()` were skipped). Overall this was perhaps acceptable given the
importance of supporting timeouts.

It turns out that the naive `callr` implementation was never committed, as in
the meantime I discovered the `[mirai][mirai]` package, which seemed able to
address both the parallelisation and the timeout issue. `mirai` provides a way
to run code asynchronously on persistent background processes (called `daemons`
in its terminology). `mirai` is very efficient in spawning processes and
dealing with the communication between host process and daemons, and also
supports interrupting a task without killing (and having to restart) the
background process.

An initial parallelisation was relatively straightforward. `mirai` could
very quickly set up thousands of futures (that is, asynchronous tasks, each
corresponding to a function call) and take care of starting a new task
whenever a daemon became free. Unfortunately, `mirai` starts evaluating the
timeout when a future  is launched, and not when the task is effectively
started by the daemon. This meant that the default timeout approach offered
by `mirai` could not be used, as it ended up timing out all but the first few
function calls.

I ended up writing a queuing system so that a future would be launched only
when a background process became available. Having taken care of when each
task was launched meant that I could let `mirai` handle the timeouts. With
that, the `CBTF` test suite could be completed in 5s and `Luminescence` could
be fuzzed in 30s when using 2 daemons.

The initial implementation was further refined, and now it's possible to
run over 90,000 tests in less than 20s when using 6 daemons.

## Future changes

It is my hope that future `mirai` versions will be able to handle timeouts in
a better way, so that I could vastly simplify the `CBTF` implementation by
removing the queuing system. If that were to happen, it's very likely that the
performance would further improve, as process handling is far better done in
`C` than in R.

Something that I'm still wondering about is how to structure the object
returned by the `fuzz()` function. The current version is functional, but
feels rigid and at times quite messy. It was designed to work well when
fuzzing multiple functions, but handling the object feels harder than
necessary if only one function is fuzzed.

It would also be nice to provide faster ways to specify the whitelist
patterns. Currently, these must be specified by hand either when calling
`fuzz()` or later when calling `whitelist(). Allowing to have a file with
whitelist patterns may speed up users working on a specific package; an
other option is to add patterns from a result object, perhaps by specifying
the input index that produced a given error to be whitelisted.

The way the results are reported could also be improved, for example by
allowing to group results according to the error message received.


[cbtf]: https://mcol.github.io/caught-by-the-fuzz/
[user]: https://user2026.r-project.org/
[song]: https://www.youtube.com/watch?v=uJ-mpul94eo
[post]: {{< ref "post/2025-08-28-fuzzing-luminescence-to-improve-it/" >}}
[lumi]: https://r-lum.github.io/Luminescence/
[mirai]:https://mirai.r-lib.org/
