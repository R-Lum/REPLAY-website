---
title: RLumShiny Release 0.2.4
date: '2025-03-24'
categories:
  - 'RLumShiny'
description: 'RLumShiny version 0.2.4'
---

One of the [goals of the REPLAY project][about] is to facilitate interfacing
with the `Luminescence` package. This is a rather ambitious goal, but luckily
there is already something available to make the use of `Luminescence` more
direct and approachable.

The [RLumShiny package][rlshiny] integrates a bunch of `Luminescence`
functions into a web-based app built on top of the [shiny framework][shiny].
This allows the users to work interactively with their data, letting them
explore the available options and see how they affect the plots and results.
This provides an easy entry point to discover what the package provides while
performing an analysis of interest, or perhaps just playing around with sample
data.

A nice feature of `RLumShiny` is that once an analysis has been performed,
the corresponding R code can be visualised and saved locally. This makes the
analyses reproducible and much faster if a batch of similar analyses need
to be performed.

The [work we have been carrying out on the Luminescence package][v100] had
however made `RLumShiny` unusable, as `Luminescence` is now much stricter
in validating its inputs. Besides that, we realised that also changes that
had occurred to the `shiny` package had broken some functionality.

Therefore, we took some time to bring the package up to date, and the new
[RLumShiny 0.2.4 release][rls024] is now fully compatible with
[Luminescence 1.0.1][v101].

While this is essentially a maintenance release, we expect to have another
release relatively soon, as we add a few more apps to cover additional
`Luminescence` functions. The future for `RLumShiny` is indeed quite bright!

[about]:   {{< ref "about" >}}
[lumi]:    https://r-lum.github.io/Luminescence/
[rlshiny]: https://tzerk.github.io/RLumShiny/
[shiny]:   https://shiny.posit.co/
[rls024]:  https://github.com/tzerk/RLumShiny/releases/tag/v0.2.4
[v100]:    {{< ref "post/2025-02-24-the-road-to-luminescence-1-0/" >}}
[v101]:    {{< ref "post/2025-03-10-luminescence-release-1-0-1/" >}}
