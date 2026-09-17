---
title: Luminescence Release 1.3.1
date: '2026-09-18'
categories:
  - 'Luminescence'
author: Marco Colombo
description: 'Luminescence version 1.3.1'
---

Less than two months since our [previous release][v130], we are very proud to
announce the release of **version 1.3.1**. This is a minor release that brings
a few regression fixes and cosmetic improvements.

For a change of pace, it's nice, every now and then, to have a release that
doesn't try to do too much or that brings breaking changes. Given the summer
time and the slightly delayed release of 1.3.0, this release cycle has been
short and sweet. Despite all that, in total we addressed 55 issues in 168
commits.

<!--more-->

## Other changes

The work on the testing infrastructure has progressed quite nicely. This time,
we have concentrated on adding output snapshot tests, that is specific tests
that check that the results tables that we display on the terminal do not
change in unexpected ways.

The addition of this new family of tests has been quite painless, as we
piggy-backed on the existing snapshot test setup. Specifically, whenever we
store a numerical snapshot, we can now ask to store an output snapshot as
well. This means that, for the most part, we simply needed to add
`expect_snapshot_output = TRUE` to the desired tests, and the rest would be
taken care by our internal helper function. Overall, this brought the number
of output tests from 12 to 85. This work is documented in [issue 1638][i1638].

Overall, the number of tests has increased from 4130 to 4293, despite the
replacement of a bunch of handwritten tests with fewer (and more thorough)
numerical snapshot tests.

[lumi]:   https://r-lum.github.io/Luminescence/
[v130]:   {{< ref "post/2026-07-22-luminescence-release-1-3-0/" >}}
[ghub]:   https://github.com/R-Lum/Luminescence/issues/
[i1638]:  https://github.com/R-Lum/Luminescence/issues/1638
