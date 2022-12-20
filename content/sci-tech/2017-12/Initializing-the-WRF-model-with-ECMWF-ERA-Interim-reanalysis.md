---
title: Initializing the WRF model with ECMWF ERA-Interim reanalysis
date: 2017-12-20
tags: ["WRF-Chem","ERA"]
categories: ["sci-tech"]
draft: false,
toc: true
img: "/images/sci-tech/2017-12/ECMWF.png"
summary: "Download ERA-Interim data by Python and initialize WRF model."
---

## Summarization

> To summarise, you need three separate data files from the reanalysis data:
>
> 1. Pressure level data (or model levels)
>2. Surface variable data
> 3. Fixed data.

<!--more-->

## Downloading the data via Python script

You’ll be required to select which surface varibles you want to download, as well as which pressure levels, too. The land-sea mask is just a single file. According to [夏朗的芒果](http://bbs.06climate.com/forum.php?mod=viewthread&tid=30962&extra=&page=1), we just need to download UNGRIB - Required Fields

> The details of the [Python API are here](https://software.ecmwf.int/wiki/display/WEBAPI/Access+ECMWF+Public+Datasets), it’s well explained so I will only summarise here what you need to do:
>
> 1. Register on the ECMWF site
> 2. Download an access key to placed on the computer or system you are downloading to.
> 3. Install the ecmwfapi Python module.
> 4. Either write your own python script using the API documentation above, or have the ECMWF web interface generate one for you. (Click the ‘view MARS request’ after making your selections, and then copy the Python script that is displayed. An example python script looks like this:

### pl.py

```
from ecmwfapi import ECMWFDataServer
server = ECMWFDataServer()

server.retrieve({
          'class'   : "ei",
          'dataset' : "interim",
          'expver'  : "1",
          'step'    : "0",
          'stream'  : "oper",
          'number'  : "all",
          'levtype' : "pl",
          'levelist': "10/20/30/50/70/100/125/150/200/250/300/350/400/450/500/550/600/650/700/750/800/850/900/925/950/975/1000",
          'date'    : "20150806/to/20150809",
          'time'    : "00/06/12/18",
          'origin'  : "all",
          'type'    : "an",
          'param'   : "129.128/133.128/131.128/132.128/130.128/157.128",
          'grid'    : "0.75/0.75",
          'target'  : "ERA-Int_pl_20150806_20150809_0.75.grib"
          })
```

### sfc.py

```
from ecmwfapi import ECMWFDataServer
server = ECMWFDataServer()

server.retrieve({
          "class": "ei",
          'dataset' : "interim",
          "expver": "1",
          'step'    : "0",
          'stream'  : "oper",
          'class'   : "ei",
          'number'  : "all",
          'levtype' : "sfc",
          'date'    : "20150806/to/20150809",
          'time'    : "00/06/12/18",
          'type'    : "an",
          'param'   : "134.128/151.128/235.128/167.128/165.128/166.128/168.128/34.128/31.128/141.128/139.128/170.128/183.128/236.128/39.128/40.128/41.128/42.128/33.128",
          'grid'    : "0.75/0.75",
          'target'  : "ERA-Int_sfc_20150806_20150809_0.75.grib"
          })
```

### fix.py

```
from ecmwfapi import ECMWFDataServer
server = ECMWFDataServer()
server.retrieve({
    "class": "ei",
    "dataset": "interim",
    "date": "1989-01-01",
    "expver": "1",
    "grid": "0.75/0.75",
    "levtype": "sfc",
    "param": "129.128/172.128",
    "step": "0",
    "stream": "oper",
    "time": "12:00:00",
    "type": "an",
    "target": "ERA_inv.grib",
})
```

ECMWF provide the data in two different file formats, GRIB (gridded-binary) and netCDF (.nc files). WPS comes with the ungribber tool (ungrib.exe) so I’ve gone for the grib data format here. (Selected by default in the Python download script).

## Ungribbing the data

Now we need to ‘ungrib’ data to convert into the WPS intermediate file format, before running metgrid. This has to be done in two stages - one for the surface and pressure level data, and one for the land-sea mask. This is because the land-sea mask has a ‘start date’ of 1989-01-01 if you try to run ungrib with the date of your case study, ungrib will fail, complaining that the dates specified could not be found. 

Link the correct Vtable to the file name “Vtable” in the directory. The Vtable can be found in`WPS/ungrib/Variable_Tables/Vtable.ECMWF`

1. Link GRIB files to the correct file names in the run directory. 

   ```
   ./link_grib.csh ERA-Int_pl_20150806_20150809_0.75.grib
   ```

   

2. Ungribbing the pressure level data.In the namelist.wps file, I set the the `&ungrib` section to the following:

   ```
   out_format = 'WPS',
   prefix = 'PL'
   ```
   ```
   ./ungrib.exe
   ```

3. Repeat again for the surface data:

   ```
   ./link_grib.csh ERA-Int_sfc_20150806_20150809_0.75.grib
   ```

   ```
   out_format = 'WPS',
   prefix = 'SFC'
   ```

   ```
   ./ungrib.exe
   ```

4. Repeat again for the  fixed data:

   ```
   start_date = '1989-01-01_12:00:00','1989-01-01_12:00:00',
   end_date   = '1989-01-01_12:00:00','1989-01-01_12:00:00',
   .
   .
   .
   out_format = 'WPS',
   prefix = 'FIX'
   ```

   ```
   ./ungrib.exe
   ```

5. You should have these files now:

   ```
   FIX:1989-01-01_12  PL:2015-08-07_12  PL:2015-08-08_06   SFC:2015-08-07_06  SFC:2015-08-08_00
   PL:2015-08-07_00   PL:2015-08-07_18  PL:2015-08-08_12   SFC:2015-08-07_12  SFC:2015-08-08_06
   PL:2015-08-07_06   PL:2015-08-08_00  SFC:2015-08-07_00  SFC:2015-08-07_18  SFC:2015-08-08_12
   ```

## Metgrid

If you are using the sea surface temparatures field from the ECMWF data, you’ll need to make a few changes to the `METGRID.TBL` file for it to correctly interpolate and mask the sea surface temparatures around land. In the `METGRID.TBL` file (located in `WPS/metgrid/`), change the entry of the SST field to the following:

```
SST
  interp_option=sixteen_pt+four_pt+wt_average_4pt+wt_average_16pt+search
  missing_value=-1.E30
  masked=land
  interp_mask=LANDMASK(1)
  fill_missing=0.
  flag_in_output=FLAG_SST 
```

The changes are to make the interp_mask use the LANDMASK mask instead of LANDSEA (the default), and to change the interpolation option slightly. Without the changes, [Declan](http://dvalts.io/modelling/data/2016/12/23/ECMWF-data-in-WRF.html) found that for his inner domain the sea surface temperatures were incorrectly masked, and had been interpolated over land as well. Although metgrid.exe did not complain when run, the met files generated caused an error when real.exe was run - generating an error message saying:

```
-------------- FATAL CALLED ---------------
FATAL CALLED FROM FILE:  <stdin>  LINE:    2970
mismatch_landmask_ivgtyp
-------------------------------------------
```

The changes to METGRID.TBL should remedy this error message.

Before running metgrid, there is one last change to make to the namelist.wps file:

```
&share
 start_date = '2015-08-07_00:00:00','2015-08-07_00:00:00',
 end_date   = '2015-08-08_12:00:00','2015-08-08_12:00:00',
&metgrid
 fg_name = 'PL','SFC',
 constants_name = 'FIX:1989-01-01_12',
 io_form_metgrid = 2,
```

```
./metgrid.exe
```

> Declan:
>
> Hopefully it will produce all the input met files needed, and correctly interpolated. It’s a good idea before running real.exe and wrf.exe to check that the fields look reasonable. In particular, check the SST field if you are using sea-surface temperatures. Check the nested domains as well - I found that my SST field had been incorrectly interpolated in the inner domain, which required the change to the METGRID.TBL file above. Ncview is a useful utility for checking the met files generated by metgrid.

## References

1. [Initialising the WRF model with ECMWF ERA-20C reanalysis data](http://dvalts.io/modelling/data/2016/12/23/ECMWF-data-in-WRF.html)
2. [06climate](http://bbs.06climate.com/forum.php?mod=viewthread&tid=30962&extra=&page=1)