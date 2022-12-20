---
title: Speedup WRF-Chem
date: 2018-12-02 20:58:10
tags: ["WRF-Chem"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2017-10/wrfchem_logo.png"
summary: "Some tips to speed up WRF-Chem"
---

## Background

This is my partial namelist of WRF-Chem:

```
&time_control
 run_days                           = 147,    
&domains
 time_step                          = 72,
 time_step_fract_num                = 0,
 time_step_fract_den                = 1,
 max_dom                            = 1,
 e_we                               = 430,
 e_sn                               = 345,
 e_vert                             = 40,
 dx                                 = 12000,
 dy                                 = 12000,
 p_top_requested                    = 10000,
```

I used `384` cores to run it and I found the speed of `calculation` and `writing` is **~5.45 seconds/step**, **~54.5 s/wrfout_file** and **~210 s/wrfrst_file**. The terrible one is the speed of **I/O  tasks**.

<!--more-->

## WRF I/O PERFORMANCE

Here's the part of [paper](https://cug.org/proceedings/cug2016_proceedings/includes/files/pap123s2-file1.pdf) *Improving I/O Performance of the Weather Research and Forecast (WRF) Model*.

> * __Serial NetCDF__: Default layer.
> * __Parallel NetCDF__: A good alternative layer that works well at lower core counts built on the Parallel NetCDF (PNetCDF) library, which supports parallel I/O.
> * __Quilt Servers__: A third technique for writes that uses I/O (or quilt) servers that deal exclusively with I/O, enabling the compute PEs to continue with their work without waiting for data to be written to disk before proceeding.
> * __Quilt Servers with PNetCDF__: An additional technique that combines the I/O server concept with PNetCDF to enable parallel asynchronous writes of WRF history and restart files. This technique proves to be highly advantageous on the Cray XC40 under certain circumstances.

It's better to read the paper first and then check this tutorial. This tutorial is arranged by the structure above.

### Parallel NetCDF

We need to edit the *namelist.input* to make sure PNetCDF works well:

```
 io_form_history                    = 11,
 io_form_restart                    = 11,
 io_form_input                      = 11,
```

This is the meaning of each one:

| Variable Names   | InputOption | Description     |
| ---------------- | ----------- | --------------- |
| io_form_history  | 2           | netCDF          |
|                  | 11          | parallel netCDF |
| io_form_restart  | 2           | netCDF          |
|                  | 11          | parallel netCDF |
| io_form_input    | 2           | netCDF          |
|                  | 11          | parallel netCDF |
| io_form_boundary | 2           | netCDF          |
|                  | 11          | parallel netCDF |

I don't know why InputOption = 11 isn't included in `io_form_restart` and` io_form_input` according to Uesr's Guide.

### Decomposition and Quilting

There're two types of MPI tasks: compute (client) and I/O (server).

**Compute tasks**

​	`Total number = nproc_x*nproc_y (number of processors along x and y axes for decomposion)`

> By default WRF will use the square root of the processors for values in nproc_x and nproc_y. If it is not possible, it will use some values that are close to each other.
>
> This is not correct as WRF responds better to a more rectangular decomposition (i.e. X<<Y). This leads to longer inner loops for better vector and register reuse, better cache blocking, and more efficient halo exchange communication pattern.

**I/O tasks**

> To set aside one or more ranks (known as quilt or I/O servers) to deal exclusively with the I/O, so that once the compute (client) ranks have sent their data to these I/O servers, they can continue with their work while the data is formatted and written to disk in the background (asynchronously).
>
> Whether or not this technique is appropriate depends on the amount of output time taken by PNetCDF and the number of compute ranks being used, since it can be inefficient to dedicate too high a proportion of ranks to I/O only.
>
> WRF attempts to match each I/O server with compute tasks in the east-west rows, and ideally (though this is not mandatory) nproc_y should be an exact multiple of nio_tasks_per_groups.

​	`Total number = nio_groups * nio_tasks_per_group`

### ~~Patch and tile~~

~~`numtiles` is the number of tiles per patch. The use of tiling has greatest effect on lower processors when the patches do not fit into cache.~~

## Tests

### Without PNetCDF

| Number of cores | MPI tasks | nproc_x * nproc_y | nio_groups * nio_tasks_per_group | Speed of calculation (s/step) | Speed of writing (s/file)          |
| --------------- | --------- | ----------------- | -------------------------------- | ----------------------------- | ---------------------------------- |
| 384             | 384       | 16*24             |                                  | 5.45                          | 54.5 (wrfout)<br />210 (wrfrst)    |
| 384             | 374       | 11*34             | 5*2                              | 5.5                           | 1.37 (wrfout)                      |
| 336             | 312       | 13*24             | 12*2                             | 6.6                           | 1.29 (wrfout)                      |
| 336             | 312       | 13*24             | 6*4                              | 6.6                           | 1.19(wrfout)<br />1.37(wrfrst)     |
| **384**         | **374**   | **11*34**         | **2*5**                          | **5.5**                       | **0.86(wrfout)<br />1.78(wrfrst)** |

`Number of cores = nproc_x * nproc_y + nio_groups * nio_tasks_per_group`

When using the second option, I got this error when using the first setting:

```
 FATAL CALLED FROM FILE:  <stdin>  LINE:     676
  Possible 32-bit overflow on output server. Try larger nio_tasks_per_group in n
 amelist.
```

According to this [pdf](https://cug.org/proceedings/cug2016_proceedings/includes/files/pap123s2-file1.pdf), Since the I/O servers gather data from many compute ranks, they require more memory than the compute ranks (generally a similar requirement to the serial rank 0 collector) and so cannot be fully packed onto nodes with large numbers of cores. 

I need a larger nio_tasks_per_group mentioned in that pdf. So, I tested option_3. However, it still failed.

After changing the collocation to `4*6`, it works. I guess WRF-Chem needs more groups than WRF to write wrfrst* files.

So, I chose **the last option** which meets two condition.

### With PNetCDF

stagged at `   med_initialdata_input: calling input_input`

| Number of cores | MPI tasks | nproc_x * nproc_y | nio_groups * nio_tasks_per_group | Speed of calculation (s/step) | Speed of writing (s/file) |
| --------------- | --------- | ----------------- | -------------------------------- | ----------------------------- | ------------------------- |
| 384             | 374       | 11*34             | 2*5                              | ??                            | ??                        |

## References

1. http://www.markomanolis.com/Research/Talks/2013_BSC-ES-Course.pdf
2. [WRF Quilting and Decomposition Notes](https://modelingguru.nasa.gov/docs/DOC-2560)
3. [Page 522 of High Performance Computing](https://books.google.com/books?id=2uDyCQAAQBAJ&pg=PA522&lpg=PA522&dq=how+to+use+quilting+in+WRF&source=bl&ots=ZwIrL4ZsS4&sig=j7AHQ13QxhimZhehRDy4DS99xK4&hl=en&sa=X&ved=2ahUKEwjC6dmSzoDfAhXGITQIHfrFBcU4HhDoATABegQIBRAB#v=onepage&q=how%20to%20use%20quilting%20in%20WRF&f=false)
4. [Scaling and performance of a high I/O WRF configuration](https://www.ecmwf.int/sites/default/files/elibrary/2014/13654-scaling-and-performance-high-io-wrf-configuration-three-different-parallel-supercomputers.pdf)
5. https://cug.org/proceedings/cug2016_proceedings/includes/files/pap123s2-file1.pdf