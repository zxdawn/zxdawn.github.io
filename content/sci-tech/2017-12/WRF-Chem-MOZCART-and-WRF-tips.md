---
title: WRF-Chem (MOZCART) and WRF_tips
date: 2017-12-10
tags: ["WRF-Chem"]
categories: ["sci-tech"]
draft: false,
toc: true
img: "/images/sci-tech/2017-12/MOZART.png"
summary: "Notes of MOZCART and some tips"
---
## WPS

### namelist

#### i_parent_start and j_parent_start

<!--more-->

As a rough approximation, keep about 1/3 of the coarse-grid domain surrounding each side of the nest. 

Your nest domain dimensions will be (ending index - starting index)*3+1

#### e_we and e_sn 

For nested domains, e_we and e_sn must be **one greater than an interger multiple of the nest's parent_grid_ratio**. This is to ensure that a nest always starts and ends on a parent grid point. (i.e., **e_we=n*parent_grid_ratio+1** for some integer n).

It is recommended to have domains no smaller than about **100x100**. Note that there will be about 10 grid points (minimum of 5) on each side, in the boundary zone. If domains are too small, the solution will be determined by forcing data. Child domains should contain similar number of grid cells as the parent.

#### geog_data_res

Possible available resolutions are 10m (~19km), 5m (~9km), 2m (~4km), and 30s (~0.9km).

When there are multiple resolutions available for geographical input data, choose a resolution that is slightly better than your grid resolution. For example, if your grid resolution is 9km, using 2 minute static data would be ideal.

#### dx and dy 

It is recommended that dx and dy have the same value for Mercator, Polar, and Lambert projections.

#### ref_lat and ref_lon 

it's recommended that these are set to the **center of the coarse domain**

#### stand_lon

If this longitude is set to the same value as ref_lon, your coarsest domain will be centered.

#### geog_data_path

`/nuist/p/public/data/geog`

#### &ungrib

```
out_format = 'WPS',
prefix = '............/WRF3.9.1/WPS/ungrib_output/FILE',   !set output_dir
```

#### &metgrid

```
fg_name = ........../WRF3.9.1/WPS/ungrib_output/FILE'
io_form_metgrid = 2,
opt_output_from_metgrid_path = '.............../WRF3.9.1/WPS/metgrid_output'   !set output_dir
```

## Run

1. Run `geogrid.exe`

2. `ln –s ungrib/Variable_Tables/Vtable.GFS Vtable`

   Link GRIB files to the correct file names in the run directory:

   ​    `link_grib.csh path_of_your_data`

   Run `ungrib.exe`

3. Run `metgrid.exe`

## WRF-Chem

Here are my steps to generate emissions:

### Real.exe 

Run real.exe (chem_opt=0) to generate `wrfinput_d0*` and `wrfbdy_d01`;

### FINN

Generate **FINN** fire-emissions following the `README.WRF.fire`;

#### Structure of directory

```
data_files                 fire_emis.inp        fire_emis.tgz    README.WRF.fire  test
FINNv1.5_2016.MOZ4.tar.gz  fire_emis_input.tar  README.GLB.fire  src
```

#### Files in 'data_files' directory

```
GLOBAL_FINNv15_2016_MOZ4_05022017.txt

shrub_from_img.nc

tropfor_from_img.nc

wrfinput_d0*
grass_from_img.nc

tempfor_from_img.nc
```
#### test.inp (in 'src' directory)

```
&control
domains        = 3,
fire_directory = '/public/home/zhangxin/models/data/FINN/data_files/',
fire_filename  = 'GLOBAL_FINNv15_2016_MOZ4_05022017.txt',
wrf_directory  = '/public/home/zhangxin/models/data/FINN/data_files/',
start_date     = '2015-08-07',
end_date       = '2015-08-08',
diag_level     = 400,

wrf2fire_map = 'co -> CO', 'no -> NO', 'so2 -> SO2', 'bigalk -> BIGALK',
               'bigene -> BIGENE', 'c2h4 -> C2H4', 'c2h5oh -> C2H5OH',
                'c2h6 -> C2H6', 'c3h8 -> C3H8','c3h6 -> C3H6','ch2o -> CH2O', 'ch3cho -> CH3CHO',
                'ch3coch3 -> CH3COCH3','ch3oh -> CH3OH','mek -> MEK','toluene -> TOLUENE',
                'nh3 -> NH3','no2 -> NO2','open -> BIGALD','c10h16 -> C10H16',
                'gly -> CH3COCHO','acetol -> HYAC','isop -> ISOP','macr -> MACR'
                'mvk -> MVK', 'ch3cooh -> CH3COOH','cres -> CRESOL','glyald -> GLYALD','mgly -> CH3COCHO',
                'oc -> 0.24*PM25 + 0.3*PM10;aerosol', 'bc -> 0.01*PM25 + 0.08*PM10;aerosol',
                'sulf -> -0.01*PM25 + 0.02*PM10;aerosol',
                'pm25 -> 0.36*PM25;aerosol','pm10 -> -0.61*PM25 + 0.61*PM10;aerosol'
/

```

### MEGAN

Generate **MEGAN** biogenic-emissions following `README.bio_emiss` and no error in the log;

#### megan_bio_emiss.inp

```
&control
domains = 3,
start_lai_mnth = 7,
end_lai_mnth   = 8,
wrf_dir   = '/nuist/u/home/xinzhang1215/work/model/history',

/
```

###Anthropogenic Emission

Generate MEIC anthropogenic-emissions （like NEI anthropogenic-emissions) successfully;

### wesely and exo_coldens

Generate **wesely and exo_coldens** data files for MOZART successfully;

#### exo_coldens.inp

```
&control
domains = 3,
wrf_dir = '/nuist/u/home/xinzhang1215/work/model/history',
/
```

#### wesely.inp

```
&control
domains = 3,
wrf_dir = '/nuist/u/home/xinzhang1215/work/model/history',
/
```

### MOZBC

1. Link all these emission files to the 'em_real' directory
2. Change chem_opt=112 and run real.exe again.
3. Create lateral boundary and initial conditions from a global chemistry model NCAR/ACOM. Download required files of **mozbc** from https://www2.acom.ucar.edu/wrf-chem/wrf-chem-tools-community.

**Note: mozart data must be 1-2 days before and after simulation date. For example, you simulation date is 2015-08-07~2015-08-08. Then you should download 2015-08-05~2015-08-10 mozart data**

#### MOZCART.inp

```
&control

do_bc     = .true.
do_ic     = .true.
domain    = 3 !change for each domain and run n times
dir_wrf   = '/nuist/u/home/xinzhang1215/work/model/history/' !where I put copy of wrfbdy_d01 and wrfinput*
dir_moz = '/nuist/u/home/xinzhang1215/work/data/mozbc/'
fn_moz  = 'h0001.nc' !name of your mozart data
def_missing_var = .true.
spc_map = 'o3 -> O3', 'no -> NO',
          'no2 -> NO2', 'no3 -> NO3', 'nh3 -> NH3', 'hno3 -> HNO3', 'hno4 -> HO2NO2',
          'n2o5 -> N2O5', 'ho -> OH', 'ho2 -> HO2', 'h2o2 -> H2O2',
          'ch4 -> CH4', 'co -> CO', 'ch3o2 -> CH3O2', 'ch3ooh -> CH3OOH',
          'hcho -> CH2O', 'ch3oh -> CH3OH', 'c2h4 -> C2H4',
          'ald -> CH3CHO', 'ch3cooh -> CH3COOH', 'acet -> CH3COCH3', 'mgly -> CH3COCHO',
          'paa -> CH3COOOH', 'gly -> GLYOXAL',
          'pan -> PAN', 'mpan -> MPAN', 'macr -> MACR',
          'mvk -> MVK', 'c2h6 -> C2H6', 'c3h6 -> C3H6', 'c3h8 -> C3H8',
          'c2h5oh -> C2H5OH', 'etooh -> C2H5OOH', 'c10h16 -> C10H16',
          'onit -> ONIT', 'onitr -> ONITR', 'isopr -> ISOP',
          'isopn -> ISOPNO3', 'acetol -> HYAC', 'glyald -> GLYALD',
          'hydrald -> HYDRALD', 'mek -> MEK',
          'bigene -> BIGENE', 'open -> BIGALD', 'bigalk -> BIGALK',
          'tol -> TOLUENE',
          'cres -> CRESOL', 'dms -> DMS', 'so2 -> SO2',
          'BC1 -> .4143*CB1;1.e9', 'BC2 -> .4143*CB2;1.e9',
          'OC1 -> .4143*OC1;1.e9', 'OC2 -> .4143*OC2;1.e9',
          'SEAS_1 -> 2.*SA1;1.e9', 'SEAS_2 -> 2.*SA2;1.e9',
          'SEAS_3 -> 2.*SA3;1.e9', 'SEAS_4 -> 2.*SA4;1.e9'
          'DUST_1 -> 1.1738*[DUST1];1.e9', 'DUST_2 -> .939*[DUST2];1.e9',
          'DUST_3 -> .2348*[DUST2]+.939*[DUST3];1.e9',
          'DUST_4 -> .2348*[DUST3]+.5869*[DUST4];1.e9', 'DUST_5 -> .5869*[DUST4];1.e9'
/
```
4. copy modfied `wrfinput*` and `wrfbdy_d01` to `em_real` directory
5. Run wrf.exe