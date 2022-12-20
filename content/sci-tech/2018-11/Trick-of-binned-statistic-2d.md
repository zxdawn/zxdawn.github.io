---
title: Trick of binned_statistic_2d
date: 2018-11-22 09:11:07
tags: ["Python"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Tricks of binned_statistic_2d in Python"
---

## Official example

```
>>> from scipy import stats
```

Calculate the counts with explicit bin-edges:

```
>>> x = [0.1, 0.1, 0.1, 0.6]
>>> y = [2.1, 2.6, 2.1, 2.1]
>>> binx = [0.0, 0.5, 1.0]
>>> biny = [2.0, 2.5, 3.0]
>>> ret = stats.binned_statistic_2d(x, y, None, 'count', bins=[binx,biny])
>>> ret.statistic
array([[ 2.,  1.],
       [ 1.,  0.]])
```

<!--more-->

## Reverse order

```
>>> x = [0.1, 0.1, 0.1, 0.6]
>>> y = [2.1, 2.6, 2.1, 2.1]
>>> binx = [1.0, 0.5, 0.0]
>>> biny = [2.0, 2.5, 3.0]
>>> ret = stats.binned_statistic_2d(x, y, None, 'count', bins=[binx,biny])
/home/xin/Software/anaconda3/lib/python3.6/site-packages/scipy/stats/_binned_statistic.py:536: RuntimeWarning: invalid value encountered in log10
  decimal = int(-np.log10(dedges[i].min())) + 6
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/xin/Software/anaconda3/lib/python3.6/site-packages/scipy/stats/_binned_statistic.py", line 344, in binned_statistic_2d
    expand_binnumbers=expand_binnumbers)
  File "/home/xin/Software/anaconda3/lib/python3.6/site-packages/scipy/stats/_binned_statistic.py", line 536, in binned_statistic_dd
    decimal = int(-np.log10(dedges[i].min())) + 6
ValueError: cannot convert float NaN to integer
```

Check line 536 and find that if reverse order is used,  `dedges[i] = np.diff(edges[i])` will be negative and `np.log10` won't work.

```
514     # Create edge arrays
515     for i in xrange(Ndim):
516         if np.isscalar(bins[i]):
517             nbin[i] = bins[i] + 2  # +2 for outlier bins
518             edges[i] = np.linspace(smin[i], smax[i], nbin[i] - 1)
519         else:
520             edges[i] = np.asarray(bins[i], float)
521             nbin[i] = len(edges[i]) + 1  # +1 for outlier bins
522         dedges[i] = np.diff(edges[i])
523 
524     nbin = np.asarray(nbin)
525 
526     # Compute the bin number each sample falls into, in each dimension
527     sampBin = {}
528     for i in xrange(Ndim):
529         sampBin[i] = np.digitize(sample[:, i], edges[i])
530 
531     # Using `digitize`, values that fall on an edge are put in the right bin.
532     # For the rightmost bin, we want values equal to the right
533     # edge to be counted in the last bin, and not as an outlier.
534     for i in xrange(Ndim):
535         # Find the rounding precision
536         decimal = int(-np.log10(dedges[i].min())) + 6
537         # Find which points are on the rightmost edge.
538         on_edge = np.where(np.around(sample[:, i], decimal) ==
539                            np.around(edges[i][-1], decimal))[0]
540         # Shift these points one bin to the left.
541         sampBin[i][on_edge] -= 1
```

