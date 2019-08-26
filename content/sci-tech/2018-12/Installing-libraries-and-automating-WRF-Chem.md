---
title: Installing libraries and automating WRF-Chem
date: 2018-12-13
tags: ["WRF-Chem"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2017-10/wrfchem_logo.png"
summary: "Some tips to install libraries and automate WRF-Chem"
---

## Backgroud

NETCDF version (`netcdf/4.5.0/02-CF-17-par`) on [TH-2](https://en.wikipedia.org/wiki/Tianhe-2) seems something wrong. This results in error when  **NETCDF4** of WRF-Chem enabled (NETCDF4=1). So, I decide to compile `NETCDF` on my own.

## Prerequisite

According to NetCDF's [instruction ](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html), we need [zlib](http://www.zlib.net/) and [hdf5](https://www.hdfgroup.org/downloads/) first. For zlib, I use the version `zlib/1.2.11-icc17` originally compiled by TH-2.

### HDF5

The newest version is **1.10.4**.

```
$ ./configure --prefix=/HOME/nuist_chenq_2/WORKSPACE/xin/software/hdf5-1.10.4 --enable-fortran --enable-static-exec
```

But, I got this error:

<!--more-->

```
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
: command not found/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 3: 
: command not found/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 5: 
: command not found/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 8: 
: command not found/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 13: 
: command not found/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 18: 
: command not found/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 21: 
: command not found/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 26: 
'HOME/nuist_chenq_2/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 32: syntax error near unexpected token `in
'HOME/nuist_chenq_2/WORKSPACE/xin/software/sources/hdf5-1.10.4/bin/missing: line 32: `case $1 in
configure: WARNING: 'missing' script is too old or missing
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether make supports nested variables... (cached) yes
checking whether to enable maintainer-specific portions of Makefiles... no
configure: error: cannot run /bin/sh bin/config.sub
```

It seems the newest version of hdf5 1.10 is not compatiable with the system.

I returned back to hdf5 1.8.21 and it works fine.

```
$ ./configure --prefix=/HOME/nuist_chenq_2/WORKSPACE/xin/software/hdf5-1.10.4 --enable-fortran --enable-static-exec
$ make -j 8 install &> make.log&
```

Edit `~/.bashrc` for installation of `NetCDF-c and NetCDF-fortran`:

```
module load intel-compilers/2017_update4 MPI/Intel/MPICH/3.2-icc2017-dyn zlib/1.2.11-icc17 jasper/1.900.1/01-CF-15-libpng
export CC=icc
export FC=ifort

# hdf5 and netcdf
software=/HOME/nuist_chenq_2/WORKSPACE/xin/software
export HDF5=${software}/hdf5-1.8.21
export NETCDF=${software}/netcdf-4.6.2
export PATH="${HDF5}/bin:${NETCDF}/bin:${PATH}"
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH=}:${HDF5}/lib:${NETCDF}/lib
export LIBRARY_PATH=${LIBRARY_PATH}:${NETCDF}/lib

# WRF-Chem
((EM_CORE=WRF_EM_CORE=WRF_CHEM=1))
((NMM_CORE=WRF_NMM_CORE=0))
export EM_CORE WRF_EM_CORE WRF_CHEM
export NMM_CORE WRF_NMM_CORE

export NETCDF4=1
export WRFIO_NCD_LARGE_FILE_SUPPORT=1
export PATH="${software}/byacc-20180609:${software}/flex-2.5.3/bin:$PATH"
export YACC="${software}/byacc-20180609/yacc -d"
export FLEX=${software}/flex-2.5.3/bin/flex
export FLEX_LIB_DIR=${software}/flex-2.5.3/lib
```

### NetCDF-c

```
$ CPPFLAGS=-I${HDF5}/include LDFLAGS=-L${HDF5}/lib ./configure --prefix=${NETCDF}
$ make -j 8 install &> make.log&
```

### NetCDF-fortran

```
$ CPPFLAGS=-I${NETCDF}/include LDFLAGS=-L${NETCDF}/lib ./configure --prefix=${NETCDF}
$ make -j 8 install &> make.log&
```

## WRF-Chem

### Configure and compile

Auto-script see [AutoWRFChem-R2SMH](https://github.com/zxdawn/AutoWRFChem-R2SMH).

```
$ ./autowrfchem config
Use KPP (kinetic preprocessor)? Default is Y. [y/n]: y

24.  Linux x86_64 i486 i586 i686, Xeon (SNB with AVX mods) ifort compiler with icc  (dmpar)
19.  Linux x86_64, Intel compiler    (dmpar)
Choose the meterology type:
  1: NARR
  2: Other
Enter choice (1-2):1
What would you like to do?
   1: Create or modify namelists
   2: Show help
   3: Quit
Enter 1-3: 1
What namelist would you like to load?
A empty answer will cancel.
   1: Load standard templates
   2: Load existing namelists
   3: Modify the current namelists
   4: Quit
Enter 1-4: 1
Choose what to modify or do:
   1: Start/end date
   2: Domain (common opts only)
   3: Meterology
   4: Chemistry
   5: Other WRF options
   6: Other WPS options
   7: Check for NEI compatibility
   8: Select MOZBC file
   9: Display WRF options
   10: Display WPS options
   11: Save and exit
Enter 1-11: 11
Save the namelists?
A empty answer will cancel.
   1: Write namelist regularly (namelist files and pickle)
   2: Write just the namelist files (not pickle - i.e. temporary namelists)
   3: Save namelists to NAMELISTS folder for later
   4: Do not save
Enter 1-4: 1
Goodbye
 Configuration complete! 

*****************************************************************************
Configuration files generated. Run autowrfchem with 'compile' to begin compilation
*****************************************************************************
```

**Note**: Please make sure the absolute path of `WRFV3` is shorter than 40. Otherwise, the compilation of kpp will fail (Line 105 - 107):

```
   if (  `echo $WRFC_ROOT | awk '{print ( length ( $1 ) > 40 ) }' `) then
   echo WARNING: If kpp fails here the path to WRF-Chem might be too long for yacc ...
   endif
```

Edit `configure.wrf` according to this [suggestion](https://software.intel.com/en-us/articles/wrf-conus12km-on-intel-xeon-phi-coprocessors-and-intel-xeon-processors) and [suggestion_2](http://www.nscc-gz.cn/newsdetail.html?6137):

```
1. Remove -DUSE_NETCDF4_FEATURES and replace –O3 with –O2.
2. Replace !DEC$ vector always with !DEC$ SIMD on line 7578 in the dyn_em/module_advect_em.F source file.

To speed up compile times, set the environment variable J to "-j 8" or whatever number of parallel make tasks you wish to use.
```

However, I got an error when running wrf.exe by this configuration:

```
  photolysis_driver: called for domain            1
 Forced exit from Rosenbrock due to the following error:
 No of steps exceeds maximum bound
 T=   72.0000000000000      and H=   72.0000000000000
 Forced exit from Rosenbrock due to the following error:
 No of steps exceeds maximum bound
 T=   72.0000000000000      and H=   72.0000000000000
 Forced exit from Rosenbrock due to the following error:
 No of steps exceeds maximum bound
 T=   72.0000000000000      and H=   72.0000000000000
```

After talking on the [WRF-Chem forum](http://forum.mmm.ucar.edu/phpBB3/viewtopic.php?f=83&t=502&sid=396fc1435fec08cb4a0fc7393d230b65#p1576), I find it's caused by `Optimization_level`, especially `floating point math`. Please check that and the [test part](https://dreambooker.site/2018/12/07/Installing-libraries-and-automating-WRF-Chem/#Test) of this article.

Edit the path of NetCDF in `MEGAN/src/make_util` and `MOZBC/src/make_mozbc`:

```
1. setenv FC ifort

2. if( $OP_SYS == "Linux" ) then
    if( $found_ncf_lib ) then
      setenv NETCDF_DIR $tst_dir
    else
      setenv NETCDF_DIR /HOME/nuist_chenq_2/WORKSPACE/xin/software/netcdf-4.6.2
    endif
```

Then `./autowrfchem compile >& compile.log &` and check the logs in `AutoWRFChem-R2SMH/BUILD/COMPLOGS`.

If you have problem with compiling KPP, please check [KPP_yacc_flex_problems.pdf ](https://ruc.noaa.gov/wrf/wrf-chem/KPP_yacc_flex_problems.pdf) written by Anton Kulchitsky.

### Namelists

Link data of `MEGAN, MOZBC,NARR` to correct path according to script.

Copy my namelists to `AutoWRFChem-R2SMH/BUILD/CONFIG/NAMELISTS`. Then, use the `autowrfchem` script to generate namelists again.

```
$ ./autowrfchem config namelist

What would you like to do?
   1: Create or modify namelists
   2: Show help
   3: Quit
Enter 1-3: 1
What namelist would you like to load?
A empty answer will cancel.
   1: Load standard templates
   2: Load existing namelists
   3: Modify the current namelists
   4: Quit
Enter 1-4: 2
Select the namelist files to use. If there is a discrepancy in the domain, the WPS file
takes precedence.
Choose the WRF namelist: 
A empty answer will cancel.
   1: Standard template
   2: namelist.input
   3: namelist.input.LNOx_US
..........
```

## Job

Because all processes of preparing data for WRF-Chem is included in `autowrfchem`, I just need to edit some for `mpirun` which is `yhrun` on TH-2.

### prepinpt

Here're scripts in `AutoWRFChem-R2SMH/BUILD/PREPINPUT`:

```
bestemissyear.py  mozbc-chemopt-list.txt  postcheck.py  prepmegan  prepnei_convert       prepwps    splitwps
metgriblist.py    mozcheck.py             precheck      prepmozbc  prepnei_intermediate  pyncdf.py
```

Since login nodes of TH-2 are unstable and slow, I need to submit jobs to multiple computing nodes.

I want to use **24 (24\*1) cores** for preparing all input data.

Because `autowrfchem` doesn't have the function of setting cores, all `./real.exe` in scripts should be replaced by `yhrun -n 24 -N 1 -p work ./real.exe`.

It's easier to use `sed`. I find the solution from [question_1](https://unix.stackexchange.com/questions/112023/how-can-i-replace-a-string-in-a-files) and [question_2](https://stackoverflow.com/questions/19151954/how-to-use-variables-in-a-command-in-sed).

```
$ pattern='yhrun -n 24 -N 1 -p work'
$ sed -i -- "s/.\/real.exe/${pattern} .\/real.exe/g; s/.\/geogrid.exe/${pattern} .\/geogrid.exe/g; s/.\/metgrid.exe/${pattern} .\/metgrid.exe/g" prep*
```

Then, create `prep.sh` in `AutoWRFChem-R2SMH` directory:

```
#!/bin/bash
export PATH=~/WORKSPACE/xin/anaconda3/bin:$PATH
source activate base
date
source readlink.sh
./autowrfchem prepinpt > 2014_prep.log
date
```

`readlink.sh` is the script to source `readlink`:

```
#!/bin/sh
  export OSCPP=/WORK/app/osenv/ln1
  export PATH=${PATH}:${OSCPP}/usr/local/mpi3-dynamic/bin:${OSCPP}/usr/lib64/qt-3.3/bin:${OSCPP}/usr/kerberos/sbin:${OSCPP}/usr/kerberos/bin:${OSCPP}/usr/local/bin:${OSCPP}/bin:${OSCPP}/usr/bin:${OSCPP}/usr/local/sbin:${OSCPP}/sbin:${OSCPP}/usr/libexec/gcc/x86_64-redhat-linux/3.4.6
  export LD_LIBRARY_PATH=/lib64:/usr/lib64:${LD_LIBRARY_PATH}:${OSCPP}/usr/local/mpi3-dynamic/lib:${OSCPP}/usr/lib64:${OSCPP}/usr/lib64/mpich2/lib:${OSCPP}/usr/lib64/mysql:${OSCPP}/lib64:${OSCPP}/intel64:${OSCPP}/usr/libexec/gcc/x86_64-redhat-linux/3.4.6:${OSCPP}/usr/local/cuda/lib:${OSCPP}/usr/local/cuda/lib64
  export GNUPLOT_PS_DIR=${OSCPP}/usr/share/gnuplot/4.2/PostScript
```

Now, I can submit the job:

```
$ yhbatch -n 24 -N 1 -p work ./prep.sh
```

### Switch from Anaconda to Miniconda

Because the `autowrfchem` script needs python which has netCDF4 package installed, firstly I install `anaconda` on my own.

As TH-2 doesn't have Internet, [crftime](https://pypi.org/project/cftime/) and [netCDF4](https://pypi.org/project/netCDF4/) should be uploaded and install from source code.

**Note:** Please add `CC=gcc` before `python setup.py install`, otherwise you will get error like this:

```
>>> from netCDF4 import Dataset as ncdat
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/HOME/nuist_chenq_2/WORKSPACE/xin/anaconda3/lib/python3.7/site-packages/netCDF4-1.4.2-py3.7-linux-x86_64.egg/netCDF4/__init__.py", line 3, in <module>
    from ._netCDF4 import *
  File "netCDF4/_netCDF4.pyx", line 1111, in init netCDF4._netCDF4
  File "/HOME/nuist_chenq_2/WORKSPACE/xin/anaconda3/lib/python3.7/site-packages/cftime-1.0.3.4-py3.7-linux-x86_64.egg/cftime/__init__.py", line 1, in <module>
    from ._cftime import utime, JulianDayFromDate, DateFromJulianDay
ImportError: /HOME/nuist_chenq_2/WORKSPACE/xin/anaconda3/lib/python3.7/site-packages/cftime-1.0.3.4-py3.7-linux-x86_64.egg/cftime/_cftime.cpython-37m-x86_64-linux-gnu.so: undefined symbol: __intel_sse2_strchr
```

The solution is found [here](https://stackoverflow.com/questions/49524083/importing-any-module-in-cython-file-gives-undefined-symbol-error?noredirect=1&lq=1):

> The problem was that I used the Intel compiler without running the Intel Python distribution, or otherwise providing the Intel runtime libraries.

However, the `hdf5` installed by anaconda conflicts with `HDF5 1.8.21` when using `netCDF4`:

```
Warning! ***HDF5 library version mismatched error***
The HDF5 header files used to compile this application do not match
the version used by the HDF5 library to which this application is linked.
Data corruption or segmentation faults may occur if the application continues.
This can happen when an application was compiled by one version of HDF5 but
linked with a different version of static or shared HDF5 library.
You should recompile the application or check your shared library related
settings such as 'LD_LIBRARY_PATH'.
You can, at your own risk, disable this warning by setting the environment
variable 'HDF5_DISABLE_VERSION_CHECK' to a value of '1'.
Setting it to 2 or higher will suppress the warning messages totally.
Headers are 1.8.21, library is 1.10.2
```

Since TH-2 has no Internet, I can't create a new environment!

So, I decided to install `miniconda` which doesn't have `hdf5` package and use `HDF5 1.8.21` directly.

After the installation of `miniconda`, `numpy, Cython, cftime and netCDF4` were uploaded and installed.

Now, all functions of `autowrfchem` works fine!

### Run

The script `AutoWRFChem-R2SMH/BUILD/runwrf` reads variable `ntasks` for running wrf.exe by `usempi`.

However, the submitting script only has `cores` and `nodes`.

I need to convert `ntasks` to `cores` and `nodes` around line 156:

```
elif $usempi; then
    nodes=$(expr $ntasks / 24 ) 
    runcmd="yhrun -n $ntasks -N $nodes -p work ./wrf.exe"
```

Then, create `run.sh` for running wrf.exe:

```
#!/bin/bash
export PATH=~/WORKSPACE/xin/miniconda3/bin:$PATH
source activate base
date
source readlink.sh
./autowrfchem run --ntasks=96 > run.log
date
```

Submit job:

```
$ yhbatch -n 96 -N 4 -p work ./run.sh
```

## Test

Here's the test of speed using different option (Just use less cores to test :)): 

> Option\_19: Linux x86_64 i486 i586 i686, ifort compiler with icc (dmpar)
> Option\_24: Linux x86_64 i486 i586 i686, Xeon (SNB with AVX mods) ifort compiler with icc (dmpar)

| Platform | Optimization_level | fp-model | Speed (seconds/time step) | Cores |
| -------- | ------------------ | -------- | ------------------------- | ----- |
| 19       | O2                 | precise  | ~27                       | 96    |
| 24       | O1                 | fast=1   | ~22                       | 96    |
| 24       | O2                 | precise  | ~23.5                     | 96    |

If you want to speed up reading/writing, please check my another [article](https://dreambooker.site/2018/12/02/Speedup-WRF-Chem/) and use `quilting`.

So, it's better to use the last one.

