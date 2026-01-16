---
title: Upcoming breaking changes in Luminescence
date: '2026-01-16'
categories:
  - 'Luminescence'
author: Marco Colombo
description: 'Breaking changes in Luminescence with high impact to users'
---

The next release of `Luminescence` (which we expect towards the end of February)
will bring some small but potentially breaking changes to users. This post is
meant to give some advance warnings and provide solutions so that updating
the package version will not cause excessive pain.

## Changes in `recordType` from BIN/BINX files

In [issue 1275][iss1275] we are slightly changing what the `recordType` slot
stores in `RLum` objects generated from BIN/BINX files. To be clear what we
are referring to, consider this bit of code, which uses an example BIN file
provided with the `Luminescence` package:

```R
> library(Luminescence)
> data(ExampleData.BINfileData)
> object <- Risoe.BINfileData2RLum.Analysis(CWOSL.SAR.Data,
                                            pos = 1)
> object

 [RLum.Analysis-class]
	 originator: Risoe.BINfileData2RLum.Analysis()
	 protocol: unknown
	 additional info elements:  0
	 number of records: 30
	 .. : RLum.Data.Curve : 30
	 .. .. : #1 TL | #2 OSL | #3 TL | #4 OSL | #5 TL | #6 OSL | #7 TL
	 .. .. : #8 OSL | #9 TL | #10 OSL | #11 TL | #12 OSL | #13 TL | #14 OSL
	 .. .. : #15 TL | #16 OSL | #17 TL | #18 OSL | #19 TL | #20 OSL | #21 TL
	 .. .. : #22 OSL | #23 TL | #24 OSL | #25 TL | #26 OSL | #27 TL | #28 OSL
	 .. .. : #29 TL | #30 IRSL
```

The output above shows a summary of the content of the `RLum.Analysis` object
that was generated from a BIN file. The curves stored inside it show their
`recordType`, that is one of `TL`, `OSL` or `IRSL` in the example above.

The change we have introduced is changing those descriptions to also contain
the type of detector used, which for Risø instruments is a PMT. Therefore, the
output will now look like this:
```R
 [RLum.Analysis-class]
	 originator: Risoe.BINfileData2RLum.Analysis()
	 protocol: unknown
	 additional info elements:  0
	 number of records: 30
	 .. : RLum.Data.Curve : 30
	 .. .. : #1 TL (PMT) | #2 OSL (PMT) | #3 TL (PMT) | #4 OSL (PMT) | #5 TL (PMT) | #6 OSL (PMT) | #7 TL (PMT)
	 .. .. : #8 OSL (PMT) | #9 TL (PMT) | #10 OSL (PMT) | #11 TL (PMT) | #12 OSL (PMT) | #13 TL (PMT) | #14 OSL (PMT)
	 .. .. : #15 TL (PMT) | #16 OSL (PMT) | #17 TL (PMT) | #18 OSL (PMT) | #19 TL (PMT) | #20 OSL (PMT) | #21 TL (PMT)
	 .. .. : #22 OSL (PMT) | #23 TL (PMT) | #24 OSL (PMT) | #25 TL (PMT) | #26 OSL (PMT) | #27 TL (PMT) | #28 OSL (PMT)
	 .. .. : #29 TL (PMT) | #30 IRSL (PMT)
```

This was made for consistency with what the `recordType` slot stores for
objects generated from an XSYG file, which always store the type of detector
used (even if at times it is `NA` because the file doesn't explicitly specify
it).

While the package functions have been amended to support this change, users
may still be affected by it. For example, the following code meant to count
how many "OSL" curves are in the object will no longer work because `==` checks
for exact equality:

```R
> sum(sapply(object@records,
             function(x) x@recordType == "OSL"))

[1] 0
```

The correct approach is to check for matches within the string, which for
example can be accomplished using the `grepl()` function:
```R
> sum(sapply(object@records,
             function(x) grepl("OSL", x@recordType)))

[1] 14
```

Note that some `Luminescence` functions will take your input literally, and
therefore may not produce the result you expected if you forget to update your
input. For example, if you used `get_RLum()` (or `subset()`) to select only
some curves, this will not produce what you wanted:

```R
## same as `subset(object, recordType == "OSL")`
> get_RLum(object, subset = (recordType == "OSL"))

[get_RLum()] Error: 'subset' expression produced an empty selection, NULL returned
```

Other potentially affected functions are those that accept a `recordType`
argument (such as `trim_RLum.Data()`) or that delegate to `get_RLum()` to
perform a selection (such as `subset()` and `remove_RLum()`).

[iss1275]: https://github.com/R-Lum/Luminescence/issues/1275