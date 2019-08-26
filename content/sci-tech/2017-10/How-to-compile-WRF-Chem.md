---
title: How to compile WRF-Chem
date: 2017-10-02
tags: ["WRF-Chem", "Linux"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2017-10/wrfchem_logo.png"
---
**This is the basic one about WRF-Chem. You can check [this one](https://dreambooker.site/2018/12/07/Installing-libraries-and-automating-WRF-Chem/) for more information.**

# Requirements

## flex(2.5.3)

<!--more-->

```
$ cd flex_directory
$ wget http://www.ncl.ucar.edu/Download/files/flex.tar.gz
$ tar -xzf flex.tar.gz
$ cd flex-2.5.3
$ ./configure --prefix=flex_directory
$ make
$ make install
```

## byacc

```
$ wget https://invisible-island.net/datafiles/release/byacc.tar.gz
$ tar -xzvf byacc.tar.gz
$ ./configure --prefix=<byacc_dir>
$ make
$ make install
```

# Environment Variables

```
$ vi ~/.bashrc
```

```
#for chem

export WRF_CHEM=1
export WRF_KPP=1
export PATH=<yacc_dir>:$PATH
export PATH=<flex_dir>/bin:$PATH
export YACC='<yacc_dir>/yacc -d'
export FLEX=<flex_dir>/bin/flex
export FLEX_LIB_DIR=<flex_dir>/lib
source ~/.bash_profile
```
# WRF-Chem

## Fix bug

```
In line 82 of chem/KPP/kpp/kpp-2.1/src/Makefile, change 
	@$(YACC) scan.y
to 
	@$(YACC) -d scan.y
Deleting -ll link:
Open file chem/KPP/kpp/kpp-2.1/src/Makefile
delete all (there are two) entrances of -ll there.
```

There're two issues in module_lightning_driver in the phys/ directory (as mentioned in [wrf-chem lightning forum](https://groups.google.com/a/ucar.edu/forum/?hl=en#!forum/wrf-chem-lightning)):

> 1. in subroutine lightning_driver, the following line (in my version on line 310): 
>
> IF ( MOD((itimestep * dt - lightning_start_seconds), lightning_dt ) .ne. 0 ) RETURN
> will exit the subroutine if lightning_start_seconds is a multiple of lightning_dt (in the default case where lightning_dt is set to the model time step).
>
> 2. line 474: 
>
> WRITE(message, * ) ' lightning_driver: using user-prescribed IC:CG ratio = ', iccg_prescribed_num, iccg_prescribed_den 
> results in an error because only one variable can be added to the string. Changing this to: 
> WRITE(message, * ) ' lightning_driver: using user-prescribed IC:CG ratio = ', iccg_prescribed_num / iccg_prescribed_den 
> resolves the issue. 

## Compile

```
$ cd WRFV3/
$ ./configure
(dmpar)   INTEL (ifort/icc)
basic
$ vi configure.wrf
$ ./compile em_real >& compile.log &
```

# WPS

```
$ ./configure
intel (dmpar)
./compile > compile.log &
```

# PS:

## My environment variables:

### hpc nuist

```
module unload intel
module load intel/16.0.1 hdf5 netcdf jasper zlib libpng grib_api ncview ncl openmpi

#for chem
export WRF_CHEM=1
export WRF_KPP=1
export PATH=my_work_path/model/yacc/byacc-20170709:$PATH
export PATH=my_work_path/model/flex/flex-2.5.3/bin:$PATH
export YACC='my_work_path/model/yacc/byacc-20170709/yacc -d'
export FLEX=my_work_path/model/flex/flex-2.5.3/bin/flex
export FLEX_LIB_DIR=my_work_path/model/flex/flex-2.5.3/lib

# added by Anaconda3 4.4.0 installer
#export PATH="my_home_path/anaconda3/bin:$PATH"

# hdfview
#export PATH=my_work_path/softwares/hdfview/HDFView-3.0.0-Linux/HDFView/3.0.0:$PATH
```
### dqwl
```
## intel
export OPENMPLIB=/public/software/compiler/composer_xe_2013_sp1.0.080/compiler/lib/intel64

## netcdf and hdf5
software=/public/home/zhangxin/softwares
export HDF5=${software}/hdf5-1.8.21
export NETCDF=${software}/netcdf-4.6.2
export PATH="${HDF5}/bin:${NETCDF}/bin:${PATH}"
export LD_LIBRARY_PATH="${HDF5}/lib:${NETCDF}/lib:${LD_LIBRARY_PATH}"

## jasper
export JASPER=/public/software/mathlib/jasper
export JASPERINC=${JASPER}/include
export JASPERLIB=${JASPER}/lib

export PATH=${JASPER}/bin:$PATH
export LD_LIBRARY_PATH=${JASPER}/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=${JASPER}/lib:$LIBRARY_PATH

## WRF-Chem
((EM_CORE=WRF_EM_CORE=WRF_CHEM=WRF_KPP=1))
((NMM_CORE=WRF_NMM_CORE=0))
export EM_CORE WRF_EM_CORE WRF_CHEM WRF_KPP
export NMM_CORE WRF_NMM_CORE

export WRFIO_NCD_LARGE_FILE_SUPPORT=1

export PATH=${software}/byacc-20170709/bin:$PATH
export PATH=${software}/flex-2.5.3/bin:$PATH
export YACC="${software}/byacc-20170709/yacc -d"
export FLEX=${software}/flex-2.5.3/bin/flex
export FLEX_LIB_DIR=${software}/flex-2.5.3/lib


#hdfview
export PATH=${software}/HDFView/3.0.0:$PATH
export PATH=${software}/HDFView/3.0.0/slf4j-1.7.25:$PATH

#ncview
export PATH=/public/software/mathlib/ncview-2.1.5/bin:$PATH
```