---
title: Satpy Basic (FY4A)
date: 2020-02-06
tags: ["Satpy", "Satellite","Python"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-02/pytroll_light_small.png"
summary: "Using Satpy to read and plot FY4A data"
---

## 1. Support Files

FY4A AGRI data in NetCDF format. Both Full DISK and regional images are supported.

Example filenames:

**Full DISK:**

    FY4A-_AGRI--_N_DISK_1047E_L1-_FDI-_MULT_NOM_20190807060000_20190807061459_4000M_V0001.HDF

**REGC:**

    FY4A-_AGRI--_N_REGC_1047E_L1-_FDI-_MULT_NOM_20190807045334_20190807045750_1000M_V0001.HDF

*Full disk scans are identified by DISK , regional scans by REGC.*

*Data Links:*

    Real Time Data Service (30 days) and Introduction Files:
        https://fy4.nsmc.org.cn/data/en/data/realtime.html
    History data (2018-03-12 -- ):
        http://satellite.nsmc.org.cn/PortalSite/Data/Satellite.aspx
    FY4A official weather application platform:
        http://rsapp.nsmc.org.cn/geofy/

## 2. Calibration

You have three options:

1. The raw detector counts (All channels)

2. Reflectance (C01 - C06)

3. Radiance and Brightness Temperature (C07 - C14)

## 3. Examples

### Installation

```
conda install -c conda-forge satpy
```

### Loading data


```python
import os, glob
from satpy.scene import Scene
from satpy.utils import debug_on
from trollimage.colormap import greys, spectral
```


```python
# load FY4A filenames
filenames = glob.glob('/xin/data/FY4A/20190807/FY4A-_AGRI*4000M_V0001.HDF')

# create the scene object
scn = Scene(filenames, reader='agri_l1')

# check available channels
scn.available_dataset_names()
```

*Output*:


    ['C01',
     'C02',
     'C03',
     'C04',
     'C05',
     'C06',
     'C07',
     'C08',
     'C09',
     'C10',
     'C11',
     'C12',
     'C13',
     'C14',
     'satellite_azimuth_angle',
     'satellite_zenith_angle',
     'solar_azimuth_angle',
     'solar_glint_angle',
     'solar_zenith_angle']

Loading `ir_channel` data:


```python
# take the ir channel as example
ir_channel = 'C12'
scn.load([ir_channel], generate=False, calibration='brightness_temperature')
```

### Full Disk Image


```python
# display in notebook
scn.show(ir_channel)
```

![agri_C12](/images/sci-tech/2020-02/agri_C12.png)


```python
# save to file
# scn.save_dataset(ir_channel, filename='{sensor}_{name}.png')
```

### True color Image


```python
# get a list of all available composites for the current scene
scn.available_composite_names()
```

*Output*:

    ['ash',
     'dust',
     'fog',
     'green',
     'green_snow',
     'ir108_3d',
     'ir_cloud_day',
     'natural_color',
     'natural_color_sun',
     'night_background',
     'night_background_hires',
     'overview',
     'overview_sun',
     'true_color']

Loading `true_color`:


```python
#Beware that this step might need much memory available on the processing machine (depending on the number of cpu cores)

composite = 'true_color'
scn.load([composite])
scn.show(composite)
# scn.save_dataset(composite, filename='{sensor}_{name}.png')
```

    Required file type 'agri_l1_4000m_geo' not found or loaded for 'satellite_zenith_angle'
    Required file type 'agri_l1_4000m_geo' not found or loaded for 'satellite_azimuth_angle'
    Required file type 'agri_l1_4000m_geo' not found or loaded for 'solar_azimuth_angle'
    Required file type 'agri_l1_4000m_geo' not found or loaded for 'solar_zenith_angle'

![agri_true_color](/images/sci-tech/2020-02/agri_true_color.png)

### Region of Interest

I take the typhoon **LEKIMA** as an example.

We can define a `map-projection` and a sub `area`, and project the data on this area.

`Pyresample` can be used to define the area easily.

This definition can also be put in the `area.yaml` configuration file.


```python
from pyresample import get_area_def

area_id = 'lekima'

x_size = 549
y_size = 499
area_extent = (-1098006.560556, -967317.140452, 1098006.560556, 1026777.426728)
projection = '+proj=laea +lat_0=19.0 +lon_0=128.0 +ellps=WGS84'
description = "Typhoon Lekima"
proj_id = 'laea_128.0_19.0'

areadef = get_area_def(area_id, description, proj_id, projection,x_size, y_size, area_extent)
```

You can generate the area easily by [coord2area_def.py](https://github.com/pytroll/satpy/blob/master/utils/coord2area_def.py)

Here's the output of `python coord2area_def.py lekima_4km laea 10 28 118 138 4`:

```
lekima_4km:
  description: lekima_4km
  projection:
    proj: laea
    ellps: WGS84
    lat_0: 19.0
    lon_0: 128.0
  shape:
    height: 499
    width: 549
  area_extent:
    lower_left_xy: [-1098006.560556, -967317.140452]
    upper_right_xy: [1098006.560556, 1026777.426728]
```

Now, you can add the configuration to `$PPP_CONFIG_DIR/areas.yaml` and use it directly


```python
# If you have added it to areas.yaml, you can use the name directly:
os.environ['PPP_CONFIG_DIR'] = '/yin_raid/xin/satpy_config/'
lekima_scene = scn.resample('lekima_4km')

# Otherwise, you need to use the areadef defined above:
# lekima_scene = scn.resample(areadef)
```


```python
lekima_scene.show(composite)
# lekima_scene.save_dataset(composite, filename='{sensor}_{name}_resampled.png')
```

![agri_true_color_resampled](/images/sci-tech/2020-02/agri_true_color_resampled.png)

If you want to generate pictures with **specific colormap** like the figure below,

please check another notebook about `enhancement`.

![agri_C12_resampled_colorize](/images/sci-tech/2020-02/agri_C12_resampled_colorize.png)


## References

1. Notebooks: https://github.com/zxdawn/FY-4/tree/master/satpy/examples
2. Satpy examples: https://satpy.readthedocs.io/en/latest/examples.html
3. Presample: http://pyresample.readthedocs.org/
4. Pytroll: http://pytroll.github.io/

## Version control

| Version | Action | Time       |
| ------- | ------ | ---------- |
| 1.0     | Init   | 2020-02-06 |
