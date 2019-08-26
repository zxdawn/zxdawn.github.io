---
title: Trick of pcolormesh
date: 2018-10-31 10:40:24
tags: ["Python"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Tricks of pcolormesh"
img: "/images/sci-tech/2018-10/pcolormesh.png"
---

## Reference

http://alex.seeholzer.de/2014/05/fun-with-matplotlib-pcolormesh-getting-data-to-display-in-the-right-position/

## Code

<!--more-->

```
import pylab as pl
import numpy as np

# values x and y give values at z
xmin = 1; xmax = 4; dx = 1
ymin = 1; ymax = 3; dy = .5
x,y = np.meshgrid(np.arange(xmin,xmax,dx),np.arange(ymin,ymax,dy))
z = x*y

# transform x and y to boundaries of x and y
x2,y2 = np.meshgrid(np.arange(xmin,xmax+dx,dx)-dx/2.,np.arange(ymin,ymax+dy,dy)-dy/2.)

# pcolormesh without x and y just uses indexing as labels
pl.subplot(121)
pl.pcolormesh(z)
pl.title("Wrong ticks")

# pcolormesh with x and y values gives a wrong plot, x and y are treated as boundaries
pl.subplot(122)
pl.title("Wrong: x,y as values")
pl.pcolormesh(x,y,z)

pl.figure()
# using the boundaries gives correct plot
pl.subplot(121)
pl.title("Right: x,y as boundaries")
pl.pcolormesh(x2,y2,z)
pl.axis([x2.min(),x2.max(),y2.min(),y2.max()])

# using the boundaries gives correct plot
pl.subplot(122)
pl.title("Correct ticks")
pl.pcolormesh(x2,y2,z)
pl.axis([x2.min(),x2.max(),y2.min(),y2.max()])
pl.xticks(np.arange(xmin,xmax,dx))
pl.yticks(np.arange(ymin,ymax,dy))

pl.show()
```

![](https://i0.wp.com/alex.seeholzer.de/wp-content/uploads/2014/05/download1.png?w=375)

![](https://i1.wp.com/alex.seeholzer.de/wp-content/uploads/2014/05/download-2.png?w=367)

## Conclusion

When I store .nc files, I need to save lon_center and lat_center.

Then, I can create correct ticks when plotting:

```
lat,lon = np.meshgrid(np.append(lat_center-resolution/2,lat_center[-1]+resolution/2),np.append(lon_center-resolution/2,lon_center[-1]+resolution/2))
```

