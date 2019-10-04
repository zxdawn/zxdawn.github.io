---
title: Initializing the WRF model with ERA5
date: 2018-04-20
tags: ["ERA5", "WRF-Chem"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2017-12/ECMWF.png"
summary: "Downloading ERA5 model level data by Python and initialize WRF model"
---

## Note

This is the tutorial of using WRF with **ERA5 model level** (138 vertical levels) data.

If you prefer **pressure level** (38 vertical levels), please check another [tutorial](https://dreambooker.site/2019/10/03/Initializing-the-WRF-model-with-ERA5-pressure-level/).

I recommend the pressure level data, because the download speed is much faster.

If you don’t have cdsapi, please check this official [tutorial](https://cds.climate.copernicus.eu/api-how-to).

## Summarization

1. Download
2. Preprocess
3. Automation
4. Geogrid
5. Ungrib
6. Produce additional intermediate files
7. Metgrid

<!--more-->

## Download

ERA5 data were downloaded as described in [How to download ERA5](https://confluence.ecmwf.int/display/CKB/How+to+download+ERA5).
Here I’m attaching scripts to download surface and model data:

if you want to use analysis and three-hour resolution, all you need to change are `data`, `area` and `target`. Otherwise, you should change as needed.

This is a little different from downloading ERA-Interim data. ERA-Interim includes Geopotential and Land-sea mask in invariant field. ERA5 includes them in surface dataset. So, we just need to download surface data and model level data.

### Surface data (GetERA5-sfc.py)

```
import cdsapi
c = cdsapi.Client()
c.retrieve('reanalysis-era5-complete',{
    'class':'ea',
    'date':'DATE1/to/DATE2',
    'area':'Nort/West/Sout/East',
    'expver':'1',
    'levtype':'sfc',
   'param':'msl/sp/skt/2t/10u/10v/2d/z/lsm/sst/ci/sd/stl1/stl2/stl3/stl4/swvl1/swvl2/swvl3/swvl4', 
    'stream':'oper',
    'time':'00:00:00/03:00:00/06:00:00/09:00:00/12:00:00/15:00:00/18:00:00/21:00:00',
    'type':'an',
    'grid':"0.25/0.25",
},'ERA5-DATE1-DATE2-sfc.grb')
```

### Model level data (GetERA5-ml.py)

```
import cdsapi
c = cdsapi.Client()
c.retrieve('reanalysis-era5-complete',{
    'class':'ea',
    'date':'DATE1/to/DATE2',
    'area':'Nort/West/Sout/East',
    'expver':'1',
    'levelist': '1/to/137',
    'levtype':'ml',
    'param':'129/130/131/132/133/152',
    'stream':'oper',
    'time':'00:00:00/03:00:00/06:00:00/09:00:00/12:00:00/15:00:00/18:00:00/21:00:00',
    'type':'an',
    'grid':"0.25/0.25",
},'ERA5-DATE1-DATE2-ml.grb')
```

## Preprocess

### Install  eccodes (recommended) or grib_api

Install [eccodes](https://software.ecmwf.int/wiki/display/ECC/ecCodes+Home) or [grib_api](https://software.ecmwf.int/wiki/display/GRIB/Home) according to ECMWF.

If you choose grib_api, you must install the new version of grib_api. Otherwise, you will get this error when using grib_set (e.g. grib_api Version 1.12.3):

```
GRIB_API ERROR   :  Key "numberOfVerticalCoordinateValues": Trying to encode value of 276 but the maximum allowable value is 255 (number of bits=8)

GRIB_API ERROR   :  unable to set NV=276 as long (Encoding invalid)
GRIB_API ERROR   :  grib_set_values[0] deletePV (1) failed: Key/value not found
GRIB_API ERROR   :  grib_set_values[1] edition (1) failed: Encoding invalid
```

### Preprocess surface data (optional)

```
echo 'write "[centre]_[dataDate]_[dataType]_[levelType]_[step].grib[edition]";' > split.rule  
grib_filter split.rule your/surface_data/name
```

### Preprocess model level data (necessary)

```
grib_set -s deletePV=1,edition=1 your/model_level_data/name your/model_level_data/name.grib1
grib_filter split.rule your/model_level_data/name.grib1
```

Finally, you will get these files (the structure of filename is `[centre]_[dataDate]_[dataType]_[levelType]_[step].grib[edition]`):

|-- ecmf_20150531_an_ml_0.grib1
|-- ecmf_20150531_an_sfc_0.grib1
|-- ecmf_20150601_an_ml_0.grib1
|-- ecmf_20150601_an_sfc_0.grib1

Link_grib.csh all these files in your preprocessing working directory.

## Automation script (Download + Preprocess)

You can automate the script rather than change elements one by one (as suggested by [Conor](http://conorsweeneyucd.blogspot.com/2015/01/download-era-interim-data.html)).

```
#!/bin/bash -l

CODEDIR=/nuist/u/home/yinyan/xin/scratch/data/ERA5/code
DATADIR=/nuist/u/home/yinyan/xin/scratch/data/ERA5/data

# Set your python environment
export PATH=~/xin/work/anaconda3/bin:$PATH
source activate root
cd $CODEDIR

DATE1=20170419
DATE2=20170420
Nort=90
West=0
Sout=-30
East=180
YY1=`echo $DATE1 | cut -c1-4`
MM1=`echo $DATE1 | cut -c5-6`
DD1=`echo $DATE1 | cut -c7-8`
YY2=`echo $DATE2 | cut -c1-4`
MM2=`echo $DATE2 | cut -c5-6`
DD2=`echo $DATE2 | cut -c7-8`

sed -e "s/DATE1/${DATE1}/g;s/DATE2/${DATE2}/g;s/Nort/${Nort}/g;s/West/${West}/g;s/Sout/${Sout}/g;s/East/${East}/g;" GetERA5-sfc.py > GetERA5-${DATE1}-${DATE2}-sfc.py

python GetERA5-${DATE1}-${DATE2}-sfc.py

sed -e "s/DATE1/${DATE1}/g;s/DATE2/${DATE2}/g;s/Nort/${Nort}/g;s/West/${West}/g;s/Sout/${Sout}/g;s/East/${East}/g;" GetERA5-ml.py > GetERA5-${DATE1}-${DATE2}-ml.py

python GetERA5-${DATE1}-${DATE2}-ml.py

mkdir -p ${DATADIR}/$YY1

mv ERA5-${DATE1}-${DATE2}-sfc.grb ERA5-${DATE1}-${DATE2}-ml.grb ${DATADIR}/$YY1/

cd ${DATADIR}/$YY1/

echo 'write "[centre]_[dataDate]_[dataType]_[levelType]_[step].grib[edition]";' > split.rule
grib_filter split.rule ERA5-${DATE1}-${DATE2}-sfc.grb
grib_set -s deletePV=1,edition=1 ERA5-${DATE1}-${DATE2}-ml.grb ERA5-${DATE1}-${DATE2}-ml.grib1
grib_filter split.rule ERA5-${DATE1}-${DATE2}-ml.grib1

# If you want to delete original files, you can uncomment the following line.
# rm *grb

exit 0
```

## Geogrid

run geogrid.exe as usual

## Ungrib

run ungrib.exe by using the following Vtable (as suggested by [valerio](https://valcap74.blogspot.com/2017/10/how-to-run-wrf-model-driven-by-era5-on.html?showComment=1524140953190#c5741939636532233503)):

```
GRIB | Level| Level| Level| metgrid | metgrid | metgrid                 |
Code | Code |  1 |  2 | Name   | Units  | Description               |
-----+------+------+------+----------+----------+------------------------------------------+  
 130 | 109 |  * |   | TT    | K    | Temperature               |
 131 | 109 |  * |   | UU    | m s-1  | U                    |
 132 | 109 |  * |   | VV    | m s-1  | V                    |
 133 | 109 |  * |   | SPECHUMD | kg kg-1 | Specific humidity            |
 152 | 109 |  * |   | LOGSFP  | Pa    | Log surface pressure           |
 157 | 109 |  * |   | RHUM   | %    | Relative humidity            |
 129 | 1  |  0 |   | SOILGEO | m2 s-2    |                     |
   | 1  |  0 |   | SOILHGT | m    | Terrain field of source analysis     |   
 165 | 1  |  0 |   | UU    | m s-1  | U                    | At 10 m   
 166 | 1  |  0 |   | VV    | m s-1  | V                    | At 10 m   
 167 | 1  |  0 |   | TT    | K    | Temperature               | At 2 m   
 168 | 1  |  0 |   | DEWPT  | K    |                     | At 2 m   
   | 1  |  0 |   | RH    | %    | Relative Humidity at 2 m         | At 2 m   
 172 | 1  |  0 |   | LANDSEA | 0/1 Flag | Land/Sea flag              |
 134 | 1  |  0 |   | PSFC   | Pa    | Surface Pressure             |
 134 | 109 |  1 |   | PSFCH  | Pa    |                     |
 151 | 1  |  0 |   | PMSL   | Pa    | Sea-level Pressure            |
 235 | 1  |  0 |   | SKINTEMP | K    | Sea-Surface Temperature         |
 31 | 1  |  0 |   | SEAICE  | 0/1 Flag | Sea-Ice-Flag               |
 34 | 1  |  0 |   | SST   | K    | Sea-Surface Temperature         |
 141 | 1  |  0 |   | SNOW_EC | m    |                     |
   | 1  |  0 |   | SNOW   | kg m-2  |Water Equivalent of Accumulated Snow Depth|  
 139 | 112 |  0 |  7 | ST000007 | K    | T of 0-7 cm ground layer         |
 170 | 112 |  7 | 28 | ST007028 | K    | T of 7-28 cm ground layer        |
 183 | 112 | 28 | 100 | ST028100 | K    | T of 28-100 cm ground layer       |
 236 | 112 | 100 | 255 | ST100255 | K    | T of 100-255 cm ground layer       |
 39 | 112 |  0 |  7 | SM000007 | fraction | Soil moisture of 0-7 cm ground layer   |   
 40 | 112 |  7 | 28 | SM007028 | fraction | Soil moisture of 7-28 cm ground layer  |   
 41 | 112 | 28 | 100 | SM028100 | fraction | Soil moisture of 28-100 cm ground layer |   
 42 | 112 | 100 | 255 | SM100255 | fraction | Soil moisture of 100-255 cm ground layer |   
-----+------+------+------+----------+----------+------------------------------------------+
```

## Produce additional intermediate files

Create the following ecmwf_coeffs table (named ecmwf_coeffs) and run */WPS/util/calc_ecmwf_p.exe:

```
0 0.000000 0.00000000  
1 2.000365 0.00000000  
2 3.102241 0.00000000  
3 4.666084 0.00000000  
4 6.827977 0.00000000  
5 9.746966 0.00000000  
6 13.605424 0.00000000  
7 18.608931 0.00000000  
8 24.985718 0.00000000  
9 32.985710 0.00000000  
10 42.879242 0.00000000  
11 54.955463 0.00000000  
12 69.520576 0.00000000  
13 86.895882 0.00000000  
14 107.415741 0.00000000  
15 131.425507 0.00000000  
16 159.279404 0.00000000  
17 191.338562 0.00000000  
18 227.968948 0.00000000  
19 269.539581 0.00000000  
20 316.420746 0.00000000  
21 368.982361 0.00000000  
22 427.592499 0.00000000  
23 492.616028 0.00000000  
24 564.413452 0.00000000  
25 643.339905 0.00000000  
26 729.744141 0.00000000  
27 823.967834 0.00000000  
28 926.344910 0.00000000  
29 1037.201172 0.00000000  
30 1156.853638 0.00000000  
31 1285.610352 0.00000000  
32 1423.770142 0.00000000  
33 1571.622925 0.00000000  
34 1729.448975 0.00000000  
35 1897.519287 0.00000000  
36 2076.095947 0.00000000  
37 2265.431641 0.00000000  
38 2465.770508 0.00000000  
39 2677.348145 0.00000000  
40 2900.391357 0.00000000  
41 3135.119385 0.00000000  
42 3381.743652 0.00000000  
43 3640.468262 0.00000000  
44 3911.490479 0.00000000  
45 4194.930664 0.00000000  
46 4490.817383 0.00000000  
47 4799.149414 0.00000000  
48 5119.895020 0.00000000  
49 5452.990723 0.00000000  
50 5798.344727 0.00000000  
51 6156.074219 0.00000000  
52 6526.946777 0.00000000  
53 6911.870605 0.00000000  
54 7311.869141 0.00000000  
55 7727.412109 0.00000700  
56 8159.354004 0.00002400  
57 8608.525391 0.00005900  
58 9076.400391 0.00011200  
59 9562.682617 0.00019900  
60 10065.978516 0.00034000  
61 10584.631836 0.00056200  
62 11116.662109 0.00089000  
63 11660.067383 0.00135300  
64 12211.547852 0.00199200  
65 12766.873047 0.00285700  
66 13324.668945 0.00397100  
67 13881.331055 0.00537800  
68 14432.139648 0.00713300  
69 14975.615234 0.00926100  
70 15508.256836 0.01180600  
71 16026.115234 0.01481600  
72 16527.322266 0.01831800  
73 17008.789062 0.02235500  
74 17467.613281 0.02696400  
75 17901.621094 0.03217600  
76 18308.433594 0.03802600  
77 18685.718750 0.04454800  
78 19031.289062 0.05177300  
79 19343.511719 0.05972800  
80 19620.042969 0.06844800  
81 19859.390625 0.07795800  
82 20059.931641 0.08828600  
83 20219.664062 0.09946200  
84 20337.863281 0.11150500  
85 20412.308594 0.12444800  
86 20442.078125 0.13831300  
87 20425.718750 0.15312500  
88 20361.816406 0.16891000  
89 20249.511719 0.18568900  
90 20087.085938 0.20349100  
91 19874.025391 0.22233300  
92 19608.572266 0.24224400  
93 19290.226562 0.26324200  
94 18917.460938 0.28535400  
95 18489.707031 0.30859800  
96 18006.925781 0.33293900  
97 17471.839844 0.35825400  
98 16888.687500 0.38436300  
99 16262.046875 0.41112500  
100 15596.695312 0.43839100  
101 14898.453125 0.46600300  
102 14173.324219 0.49380000  
103 13427.769531 0.52161900  
104 12668.257812 0.54930100  
105 11901.339844 0.57669200  
106 11133.304688 0.60364800  
107 10370.175781 0.63003600  
108 9617.515625 0.65573600  
109 8880.453125 0.68064300  
110 8163.375000 0.70466900  
111 7470.343750 0.72773900  
112 6804.421875 0.74979700  
113 6168.531250 0.77079800  
114 5564.382812 0.79071700  
115 4993.796875 0.80953600  
116 4457.375000 0.82725600  
117 3955.960938 0.84388100  
118 3489.234375 0.85943200  
119 3057.265625 0.87392900  
120 2659.140625 0.88740800  
121 2294.242188 0.89990000  
122 1961.500000 0.91144800  
123 1659.476562 0.92209600  
124 1387.546875 0.93188100  
125 1143.250000 0.94086000  
126 926.507812 0.94906400  
127 734.992188 0.95655000  
128 568.062500 0.96335200  
129 424.414062 0.96951300  
130 302.476562 0.97507800  
131 202.484375 0.98007200  
132 122.101562 0.98454200  
133 62.781250 0.98850000  
134 22.835938 0.99198400  
135 3.757813 0.99500300  
136 0.000000 0.99763000  
137 0.000000 1.00000000
```

## Metgrid

Edit namelist.wps like this:

```
&ungrib
out_format = 'WPS',
prefix = 'FILE',
/

&metgrid
fg_name = 'FILE','PRES'
io_form_metgrid = 2,
```

If you just run metgrid.exe, you'll get these warnings:

```
WARNING: Entry in METGRID.TBL not found for field ST100255. Default options will be used for this field!
WARNING: Entry in METGRID.TBL not found for field SM100255. Default options will be used for this field!
WARNING: Entry in METGRID.TBL not found for field LOGSFP. Default options will be used for this field!
```

You can edit METGRID.TBL to set interpolation method of `ST100255` `SM100255` and `LOGSFP`: 

```
========================================
name=ST
        z_dim_name=num_st_layers
        derived=yes
        ....
# ELSE IF
        fill_lev =   7 : ST000007(200100)
        fill_lev =  28 : ST007028(200100)
        fill_lev = 100 : ST028100(200100)
        fill_lev = 255 : ST100255(200100)
        ....
========================================
name=SM
        z_dim_name=num_sm_layers
        derived=yes
        ....
# ELSE IF
        fill_lev =   7 : SM000007(200100)
        fill_lev =  28 : SM007028(200100)
        fill_lev = 100 : SM028100(200100)
        fill_lev = 255 : SM100255(200100)
        ....
....
....
========================================
name=SM100255
        interp_option=sixteen_pt+four_pt+wt_average_4pt+wt_average_16pt+search
        masked=water
        interp_mask=LANDSEA(0)
        missing_value=-1.E30
        fill_missing=1.
        flag_in_output=FLAG_SM100255
========================================
....
....
========================================
name=ST100255
        interp_option=sixteen_pt+four_pt+wt_average_4pt+wt_average_16pt+search
        masked=water
        interp_mask=LANDSEA(0)
        missing_value=-1.E30
        fill_missing=285.
        flag_in_output=FLAG_ST100255
========================================
....
....
========================================
name=LOGSFP
        interp_option=four_pt+average_4pt
        fill_lev=200100:PSFC(200100.)
        flag_in_output=FLAG_LOGSFP
========================================
```

Run metgrid.exe with the modified METGRID.TBL.ARW table.

## Note of mozbc

~~If you're using MOZART in WRF-Chem, then you need to change LOGSFP field in met* files to PSFC:~~

```
$ for i in met*; do ncap2 -s 'PSFC=exp(LOGSFP)' "$i" "$i"_tmp; done
After checking tmp files:
$ rename nc_tmp nc *.nc_tmp
```

You should use surface pressure directly from surface level for MOZART in WRF-Chem.

## References

1. [How to run the WRF model using ERA5 (on model levels) as initial and boundary conditions](https://valcap74.blogspot.com/2017/10/how-to-run-wrf-model-driven-by-era5-on.html?showComment=1524140953190#c5741939636532233503)
2. [Grib to Netcdf conversion](https://software.ecmwf.int/wiki/display/OIFSUF/Grib+to+Netcdf+conversion)
3. [Download ERA-Interim data](http://conorsweeneyucd.blogspot.com/2015/01/download-era-interim-data.html)

## Version control

| Version | Action                                                       | Time       |
| ------- | ------------------------------------------------------------ | ---------- |
| 1.0     | Init                                                         | 2018-04-20 |
| 1.1     | ERA5 are accessed via the Climate Data Store (CDS) infrastructure. Update download scripts. | 2019-04-16 |
| 1.2     | Update Vtable                                                | 2019-08-22 |
| 1.3     | Add note of pressure level                                   | 2019-10-03 |
