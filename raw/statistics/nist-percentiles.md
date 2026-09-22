# 7.2.6.2. Percentiles

> Source: https://www.itl.nist.gov/div898/handbook/prc/section2/prc262.htm
> Collected: 2026-09-21
> Published: Unknown

## Definitions of order statistics and ranks

For a series of measurements \(Y_1, \ldots, Y_N\), denote the data ordered in increasing order of magnitude by \(Y_{[1]}, \ldots, Y_{[N]}\). These ordered data are called order statistics. If \(Y_{[j]}\) is the order statistic that corresponds to the measurement \(Y_i\), then the rank for \(Y_i\) is \(j\).

## Definition of percentiles

Order statistics provide a way of estimating proportions of the data that should fall above and below a given value, called a percentile. The \(p\)th percentile is a value, \(Y_{(p)}\), such that at most \((100p)\)% of the measurements are less than this value and at most \(100(1-p)\)% are greater. The 50th percentile is called the median. Percentiles split a set of ordered data into hundredths. Deciles split ordered data into tenths.

For example, 70% of the data should fall below the 70th percentile. Given \(n\) points, the percentile corresponding to the \(i\)-th point is \(i/(n+1)\).

More typically we start with a desired percentile value and this percentile of interest may not correspond to a specific data point. In this case, interpolation between points is required. There is not a standard universally accepted way to perform this interpolation. All of the methods discussed here are used in practice.

## Estimation of percentiles

Percentiles can be estimated from \(N\) measurements as follows: for the \(p\)th percentile, set \(p(N+1)\) equal to \(k+d\), for \(k\) an integer and \(d\) a fraction greater than or equal to 0 and less than 1.

1. For \(0 < k < N\), \(Y_{(p)} = Y_{[k]} + d(Y_{[k+1]} - Y_{[k]})\).
2. For \(k=0\), \(Y_{(p)} = Y_{[1]}\).
3. For \(k \ge N\), \(Y_{(p)} = Y_{[N]}\).

## Note that there are other ways of calculating percentiles in common use

Hyndman and Fan (1996) evaluated nine methods for computing percentiles. Most statistical and spreadsheet software use one of the methods described in that article. The method described above corresponds to R6. R8 is the method advocated by Hyndman and Fan. Some software packages use R7; this is the method used by Excel and the default method for R. R6, R7, and R8 give fairly similar, but not exactly the same results, particularly for small samples.
