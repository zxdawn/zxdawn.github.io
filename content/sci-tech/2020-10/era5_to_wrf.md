---
title: Using the ERA5 O3 data in the WRF initial condition
date: 2020-10-26
tags: ["wrf-chem","era5"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-10/era5_overview_2.png"
summary: "How to edit the WRF IC"
---

Because of the COVID-19, [CAM-chem](https://www.acom.ucar.edu/cam-chem/cam-chem.shtml) output files aren't available after June 2019.

I compared the [WACCM](https://www.acom.ucar.edu/waccm/download.shtml) output file and [ERA5 reanalysis](https://cds.climate.copernicus.eu/cdsapp#!/dataset/reanalysis-era5-pressure-levels?tab=overview) pressure level data and found that the upper tropospheric ozone is better for the ERA5 data.

As a result, I decided to directly replace the O3 variable in `wrfinput*` files by that of the ERA5 data.

Here're the steps:


```python
import numpy as np
import xarray as xr
import proplot as plot
from netCDF4 import Dataset
from wrf import getvar, ll_to_xy
import scipy.interpolate as interp
```

## Read data


```python
# read wrfinput file
ncfile = Dataset('./wrfinput/20200901/mozbc/wrfinput_d01', 'r+') # for modification
ds_wrf = xr.open_dataset('./wrfinput/20200901/mozbc/wrfinput_d01') # for reading data
# calculate the pressure levels (hPa)
p_wrf = (ds_wrf['P']+ds_wrf['PB']).isel(Time=0)/1e2
p_wrf.attrs['units'] = 'hPa'
# convert to ppbv
o3 = ds_wrf['o3']*1e3

# read era5 data
ds_era5 = xr.open_dataset('./era5/era5_20200901.nc')
era5_o3 = ds_era5['o3']*28.9644/47.9982*1e9
```

## Correct the ERA5 data manually (optional)

I correct the ERA5 data based on the ozonesonde data:


```python
# correct the ERA5 data manually
era5_o3 = xr.where((era5_o3.level<=500) & (era5_o3.level>=100) & (era5_o3-20>10), era5_o3-20, era5_o3)
era5_o3 = xr.where((era5_o3.level<=700) & (era5_o3.level>500) & (era5_o3-10>10), era5_o3-10, era5_o3)
```

## Interpolation

### Interpolating era5 data to wrf grids (lon/lat)


```python
era5_o3_interp = era5_o3.interp(longitude=o3.XLONG.isel(Time=0), latitude=o3.XLAT.isel(Time=0))

## expand 1d to 3d
p_era5 = era5_o3_interp.level.expand_dims(['south_north', 'west_east'])
p_era5 = np.repeat(p_era5, repeats=o3.sizes['south_north'], axis=0)
p_era5 = np.repeat(p_era5, repeats=o3.sizes['west_east'], axis=1)
p_era5.attrs['units'] = 'hPa'
```

Check the result of interpolation:


```python
f, axs = plot.subplots(ncols=2, share=0)

ax = axs[0]
m = ax.pcolormesh(era5_o3.longitude,
                  era5_o3.latitude,
                  era5_o3.isel(time=0, level=-1),
                  cmap='viridis')
ax.colorbar(m, label='O$_3$ (ppbv)', loc='r')
ax.format(title='Original')

ax = axs[1]
m = ax.pcolormesh(era5_o3_interp.longitude,
                  era5_o3_interp.latitude,
                  era5_o3_interp.isel(time=0, level=-1),
                  cmap='viridis')
ax.format(title='Interpolated')
ax.colorbar(m, label='O$_3$ (ppbv)', loc='r')

axs.format(xlim=(o3.XLONG.min(), o3.XLONG.max()),
           ylim=(o3.XLAT.min(), o3.XLAT.max()),
           xlabel='longitude',
           ylabel='latitude',)
```


    
![overview](/images/sci-tech/2020-10/era5_overview_1.png)
    


### Interpolating to WRF pressure levels

Define the interpolation function used by `xr.apply_ufunc`:


```python
def interp1d_np(data, x, xi):
    from scipy import interpolate
    f = interpolate.interp1d(x, data, fill_value='extrapolate')
    return f(xi)
```

Display the structue of each DataArray:


```python
display(era5_o3, p_era5, p_wrf)
```


<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray (level: 37, time: 72, latitude: 201, longitude: 281)&gt;
array([[[[2436.0215  , 2437.7124  , 2438.7883  , ..., 2450.778   ,
          2450.9316  , 2451.239   ],
         [2441.5552  , 2442.631   , 2443.246   , ..., 2456.158   ,
          2456.4653  , 2456.7725  ],
         [2446.0127  , 2447.0886  , 2447.0886  , ..., 2461.845   ,
          2462.1526  , 2462.4602  ],
         ...,
         [3067.625   , 3057.7874  , 3049.026   , ..., 3244.2405  ,
          3248.083   , 3255.9224  ],
         [3072.2366  , 3061.4768  , 3051.7925  , ..., 3257.9207  ,
          3262.9932  , 3269.449   ],
         [3075.7715  , 3064.397   , 3053.176   , ..., 3273.599   ,
          3278.3643  , 3284.8203  ]],

        [[2427.26    , 2429.2583  , 2431.2563  , ..., 2447.8574  ,
          2447.8574  , 2448.0112  ],
         [2431.8713  , 2433.716   , 2435.5603  , ..., 2451.3926  ,
          2451.3926  , 2451.3926  ],
         [2436.9438  , 2438.9421  , 2440.6328  , ..., 2454.7744  ,
          2454.467   , 2454.3132  ],
...
         [  26.428476,   26.428476,   26.428476, ...,   26.890043,
            26.890043,   26.736372],
         [  26.428476,   26.428476,   26.274803, ...,   27.043716,
            27.043716,   27.043716],
         [  26.428476,   26.428476,   26.274803, ...,   27.19739 ,
            27.19739 ,   27.19739 ]],

        [[  35.34426 ,   35.34426 ,   35.34426 , ...,   44.720516,
            44.720516,   44.720516],
         [  35.49793 ,   35.651604,   35.651604, ...,   44.720516,
            44.720516,   44.566837],
         [  35.80528 ,   35.95895 ,   35.80528 , ...,   44.720516,
            44.566837,   44.25949 ],
         ...,
         [  26.428476,   26.428476,   26.428476, ...,   26.736372,
            26.582697,   26.428476],
         [  26.428476,   26.428476,   26.274803, ...,   26.890043,
            26.890043,   26.890043],
         [  26.428476,   26.428476,   26.274803, ...,   27.043716,
            27.043716,   27.043716]]]], dtype=float32)
Coordinates:
  * level      (level) int32 1 2 3 5 7 10 20 30 ... 850 875 900 925 950 975 1000
  * longitude  (longitude) float32 90.0 90.25 90.5 90.75 ... 159.5 159.75 160.0
  * latitude   (latitude) float32 60.0 59.75 59.5 59.25 ... 10.5 10.25 10.0
  * time       (time) datetime64[ns] 2020-08-01 ... 2020-09-01T23:00:00</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'></div><ul class='xr-dim-list'><li><span class='xr-has-index'>level</span>: 37</li><li><span class='xr-has-index'>time</span>: 72</li><li><span class='xr-has-index'>latitude</span>: 201</li><li><span class='xr-has-index'>longitude</span>: 281</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-720b4fc7-4be9-4553-b11f-df593c18099b' class='xr-array-in' type='checkbox' checked><label for='section-720b4fc7-4be9-4553-b11f-df593c18099b' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>2436.0215 2437.7124 2438.7883 ... 27.043716 27.043716 27.043716</span></div><div class='xr-array-data'><pre>array([[[[2436.0215  , 2437.7124  , 2438.7883  , ..., 2450.778   ,
          2450.9316  , 2451.239   ],
         [2441.5552  , 2442.631   , 2443.246   , ..., 2456.158   ,
          2456.4653  , 2456.7725  ],
         [2446.0127  , 2447.0886  , 2447.0886  , ..., 2461.845   ,
          2462.1526  , 2462.4602  ],
         ...,
         [3067.625   , 3057.7874  , 3049.026   , ..., 3244.2405  ,
          3248.083   , 3255.9224  ],
         [3072.2366  , 3061.4768  , 3051.7925  , ..., 3257.9207  ,
          3262.9932  , 3269.449   ],
         [3075.7715  , 3064.397   , 3053.176   , ..., 3273.599   ,
          3278.3643  , 3284.8203  ]],

        [[2427.26    , 2429.2583  , 2431.2563  , ..., 2447.8574  ,
          2447.8574  , 2448.0112  ],
         [2431.8713  , 2433.716   , 2435.5603  , ..., 2451.3926  ,
          2451.3926  , 2451.3926  ],
         [2436.9438  , 2438.9421  , 2440.6328  , ..., 2454.7744  ,
          2454.467   , 2454.3132  ],
...
         [  26.428476,   26.428476,   26.428476, ...,   26.890043,
            26.890043,   26.736372],
         [  26.428476,   26.428476,   26.274803, ...,   27.043716,
            27.043716,   27.043716],
         [  26.428476,   26.428476,   26.274803, ...,   27.19739 ,
            27.19739 ,   27.19739 ]],

        [[  35.34426 ,   35.34426 ,   35.34426 , ...,   44.720516,
            44.720516,   44.720516],
         [  35.49793 ,   35.651604,   35.651604, ...,   44.720516,
            44.720516,   44.566837],
         [  35.80528 ,   35.95895 ,   35.80528 , ...,   44.720516,
            44.566837,   44.25949 ],
         ...,
         [  26.428476,   26.428476,   26.428476, ...,   26.736372,
            26.582697,   26.428476],
         [  26.428476,   26.428476,   26.274803, ...,   26.890043,
            26.890043,   26.890043],
         [  26.428476,   26.428476,   26.274803, ...,   27.043716,
            27.043716,   27.043716]]]], dtype=float32)</pre></div></div></li><li class='xr-section-item'><input id='section-9cb4bc94-0f16-4f68-bfa9-441289071662' class='xr-section-summary-in' type='checkbox'  checked><label for='section-9cb4bc94-0f16-4f68-bfa9-441289071662' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>level</span></div><div class='xr-var-dims'>(level)</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>1 2 3 5 7 ... 900 925 950 975 1000</div><input id='attrs-b5ab0ded-6f6f-427f-8174-9f358060f7e4' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-b5ab0ded-6f6f-427f-8174-9f358060f7e4' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-393e4d38-b5ed-4ea5-8734-8d7f161f7958' class='xr-var-data-in' type='checkbox'><label for='data-393e4d38-b5ed-4ea5-8734-8d7f161f7958' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>millibars</dd><dt><span>long_name :</span></dt><dd>pressure_level</dd></dl></div><div class='xr-var-data'><pre>array([   1,    2,    3,    5,    7,   10,   20,   30,   50,   70,  100,  125,
        150,  175,  200,  225,  250,  300,  350,  400,  450,  500,  550,  600,
        650,  700,  750,  775,  800,  825,  850,  875,  900,  925,  950,  975,
       1000], dtype=int32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>longitude</span></div><div class='xr-var-dims'>(longitude)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>90.0 90.25 90.5 ... 159.75 160.0</div><input id='attrs-377b82e6-07d0-4a36-8395-c01eee3ce5f4' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-377b82e6-07d0-4a36-8395-c01eee3ce5f4' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-98b65de9-f196-4165-932a-4c6279ae6beb' class='xr-var-data-in' type='checkbox'><label for='data-98b65de9-f196-4165-932a-4c6279ae6beb' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>degrees_east</dd><dt><span>long_name :</span></dt><dd>longitude</dd></dl></div><div class='xr-var-data'><pre>array([ 90.  ,  90.25,  90.5 , ..., 159.5 , 159.75, 160.  ], dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>latitude</span></div><div class='xr-var-dims'>(latitude)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>60.0 59.75 59.5 ... 10.5 10.25 10.0</div><input id='attrs-68618649-6582-4e64-8b36-0c9864fcfae3' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-68618649-6582-4e64-8b36-0c9864fcfae3' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f9bfacc1-e012-4126-9b4a-09b89847ecea' class='xr-var-data-in' type='checkbox'><label for='data-f9bfacc1-e012-4126-9b4a-09b89847ecea' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>degrees_north</dd><dt><span>long_name :</span></dt><dd>latitude</dd></dl></div><div class='xr-var-data'><pre>array([60.  , 59.75, 59.5 , ..., 10.5 , 10.25, 10.  ], dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>2020-08-01 ... 2020-09-01T23:00:00</div><input id='attrs-433e7c6c-75c5-4c78-816f-6b3f3ec9ff1a' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-433e7c6c-75c5-4c78-816f-6b3f3ec9ff1a' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-150c09d5-77da-48f2-bff4-fe0d1a07cae9' class='xr-var-data-in' type='checkbox'><label for='data-150c09d5-77da-48f2-bff4-fe0d1a07cae9' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>time</dd></dl></div><div class='xr-var-data'><pre>array([&#x27;2020-08-01T00:00:00.000000000&#x27;, &#x27;2020-08-01T01:00:00.000000000&#x27;,
       &#x27;2020-08-01T02:00:00.000000000&#x27;, &#x27;2020-08-01T03:00:00.000000000&#x27;,
       &#x27;2020-08-01T04:00:00.000000000&#x27;, &#x27;2020-08-01T05:00:00.000000000&#x27;,
       &#x27;2020-08-01T06:00:00.000000000&#x27;, &#x27;2020-08-01T07:00:00.000000000&#x27;,
       &#x27;2020-08-01T08:00:00.000000000&#x27;, &#x27;2020-08-01T09:00:00.000000000&#x27;,
       &#x27;2020-08-01T10:00:00.000000000&#x27;, &#x27;2020-08-01T11:00:00.000000000&#x27;,
       &#x27;2020-08-01T12:00:00.000000000&#x27;, &#x27;2020-08-01T13:00:00.000000000&#x27;,
       &#x27;2020-08-01T14:00:00.000000000&#x27;, &#x27;2020-08-01T15:00:00.000000000&#x27;,
       &#x27;2020-08-01T16:00:00.000000000&#x27;, &#x27;2020-08-01T17:00:00.000000000&#x27;,
       &#x27;2020-08-01T18:00:00.000000000&#x27;, &#x27;2020-08-01T19:00:00.000000000&#x27;,
       &#x27;2020-08-01T20:00:00.000000000&#x27;, &#x27;2020-08-01T21:00:00.000000000&#x27;,
       &#x27;2020-08-01T22:00:00.000000000&#x27;, &#x27;2020-08-01T23:00:00.000000000&#x27;,
       &#x27;2020-08-31T00:00:00.000000000&#x27;, &#x27;2020-08-31T01:00:00.000000000&#x27;,
       &#x27;2020-08-31T02:00:00.000000000&#x27;, &#x27;2020-08-31T03:00:00.000000000&#x27;,
       &#x27;2020-08-31T04:00:00.000000000&#x27;, &#x27;2020-08-31T05:00:00.000000000&#x27;,
       &#x27;2020-08-31T06:00:00.000000000&#x27;, &#x27;2020-08-31T07:00:00.000000000&#x27;,
       &#x27;2020-08-31T08:00:00.000000000&#x27;, &#x27;2020-08-31T09:00:00.000000000&#x27;,
       &#x27;2020-08-31T10:00:00.000000000&#x27;, &#x27;2020-08-31T11:00:00.000000000&#x27;,
       &#x27;2020-08-31T12:00:00.000000000&#x27;, &#x27;2020-08-31T13:00:00.000000000&#x27;,
       &#x27;2020-08-31T14:00:00.000000000&#x27;, &#x27;2020-08-31T15:00:00.000000000&#x27;,
       &#x27;2020-08-31T16:00:00.000000000&#x27;, &#x27;2020-08-31T17:00:00.000000000&#x27;,
       &#x27;2020-08-31T18:00:00.000000000&#x27;, &#x27;2020-08-31T19:00:00.000000000&#x27;,
       &#x27;2020-08-31T20:00:00.000000000&#x27;, &#x27;2020-08-31T21:00:00.000000000&#x27;,
       &#x27;2020-08-31T22:00:00.000000000&#x27;, &#x27;2020-08-31T23:00:00.000000000&#x27;,
       &#x27;2020-09-01T00:00:00.000000000&#x27;, &#x27;2020-09-01T01:00:00.000000000&#x27;,
       &#x27;2020-09-01T02:00:00.000000000&#x27;, &#x27;2020-09-01T03:00:00.000000000&#x27;,
       &#x27;2020-09-01T04:00:00.000000000&#x27;, &#x27;2020-09-01T05:00:00.000000000&#x27;,
       &#x27;2020-09-01T06:00:00.000000000&#x27;, &#x27;2020-09-01T07:00:00.000000000&#x27;,
       &#x27;2020-09-01T08:00:00.000000000&#x27;, &#x27;2020-09-01T09:00:00.000000000&#x27;,
       &#x27;2020-09-01T10:00:00.000000000&#x27;, &#x27;2020-09-01T11:00:00.000000000&#x27;,
       &#x27;2020-09-01T12:00:00.000000000&#x27;, &#x27;2020-09-01T13:00:00.000000000&#x27;,
       &#x27;2020-09-01T14:00:00.000000000&#x27;, &#x27;2020-09-01T15:00:00.000000000&#x27;,
       &#x27;2020-09-01T16:00:00.000000000&#x27;, &#x27;2020-09-01T17:00:00.000000000&#x27;,
       &#x27;2020-09-01T18:00:00.000000000&#x27;, &#x27;2020-09-01T19:00:00.000000000&#x27;,
       &#x27;2020-09-01T20:00:00.000000000&#x27;, &#x27;2020-09-01T21:00:00.000000000&#x27;,
       &#x27;2020-09-01T22:00:00.000000000&#x27;, &#x27;2020-09-01T23:00:00.000000000&#x27;],
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-16b1a21e-f7c1-45d3-bbcc-39ebdd29a03f' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-16b1a21e-f7c1-45d3-bbcc-39ebdd29a03f' class='xr-section-summary'  title='Expand/collapse section'>Attributes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'></dl></div></li></ul></div></div>



<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;level&#x27; (south_north: 119, west_east: 119, level: 37)&gt;
array([[[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]],

       [[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]],

       [[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
...
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]],

       [[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]],

       [[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]]], dtype=int32)
Coordinates:
  * level    (level) int32 1 2 3 5 7 10 20 30 ... 850 875 900 925 950 975 1000
Dimensions without coordinates: south_north, west_east
Attributes:
    units:    hPa</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'level'</div><ul class='xr-dim-list'><li><span>south_north</span>: 119</li><li><span>west_east</span>: 119</li><li><span class='xr-has-index'>level</span>: 37</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-bc5ccd20-6320-411f-99e6-c1b2a90db2bb' class='xr-array-in' type='checkbox' checked><label for='section-bc5ccd20-6320-411f-99e6-c1b2a90db2bb' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>1 2 3 5 7 10 20 30 50 70 ... 775 800 825 850 875 900 925 950 975 1000</span></div><div class='xr-array-data'><pre>array([[[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]],

       [[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]],

       [[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
...
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]],

       [[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]],

       [[   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        ...,
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000],
        [   1,    2,    3, ...,  950,  975, 1000]]], dtype=int32)</pre></div></div></li><li class='xr-section-item'><input id='section-16efa49a-06a2-417b-bd6d-191e23a67e60' class='xr-section-summary-in' type='checkbox'  checked><label for='section-16efa49a-06a2-417b-bd6d-191e23a67e60' class='xr-section-summary' >Coordinates: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>level</span></div><div class='xr-var-dims'>(level)</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>1 2 3 5 7 ... 900 925 950 975 1000</div><input id='attrs-7e1cd1d0-3f1a-47d4-8945-084a561cc686' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-7e1cd1d0-3f1a-47d4-8945-084a561cc686' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-65d9d8aa-0e9d-4e23-bfb9-0b93ca4242b1' class='xr-var-data-in' type='checkbox'><label for='data-65d9d8aa-0e9d-4e23-bfb9-0b93ca4242b1' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>millibars</dd><dt><span>long_name :</span></dt><dd>pressure_level</dd></dl></div><div class='xr-var-data'><pre>array([   1,    2,    3,    5,    7,   10,   20,   30,   50,   70,  100,  125,
        150,  175,  200,  225,  250,  300,  350,  400,  450,  500,  550,  600,
        650,  700,  750,  775,  800,  825,  850,  875,  900,  925,  950,  975,
       1000], dtype=int32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-bfcf0acb-8de9-4a8d-bfc5-e48d51cd8e6d' class='xr-section-summary-in' type='checkbox'  checked><label for='section-bfcf0acb-8de9-4a8d-bfc5-e48d51cd8e6d' class='xr-section-summary' >Attributes: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>hPa</dd></dl></div></li></ul></div></div>



<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray (bottom_top: 74, south_north: 119, west_east: 119)&gt;
array([[[ 977.6088   ,  973.8247   ,  980.0365   , ..., 1005.25305  ,
         1005.3736   , 1005.49805  ],
        [ 972.2685   ,  972.64575  ,  980.2555   , ..., 1005.1616   ,
         1005.28357  , 1005.40784  ],
        [ 973.7658   ,  979.50134  ,  983.94495  , ..., 1005.05743  ,
         1005.1929   , 1005.32166  ],
        ...,
        [ 910.8054   ,  898.6346   ,  894.9356   , ..., 1013.1167   ,
         1012.9816   , 1012.9492   ],
        [ 903.1422   ,  885.46564  ,  880.11725  , ..., 1013.20825  ,
         1013.1019   , 1012.9691   ],
        [ 868.00275  ,  854.2058   ,  859.6906   , ..., 1009.3077   ,
         1013.17224  , 1013.0759   ]],

       [[ 970.978    ,  967.2171   ,  973.3924   , ...,  998.4201   ,
          998.54095  ,  998.6655   ],
        [ 965.6708   ,  966.0465   ,  973.6117   , ...,  998.33044  ,
          998.4526   ,  998.5758   ],
        [ 967.15784  ,  972.86304  ,  977.2757   , ...,  998.2269   ,
          998.3619   ,  998.4895   ],
...
        [  11.009878 ,   10.996195 ,   10.992074 , ...,   11.122887 ,
           11.122656 ,   11.1225195],
        [  11.0012455,   10.981459 ,   10.975507 , ...,   11.123066 ,
           11.122817 ,   11.1225195],
        [  10.961985 ,   10.946582 ,   10.95274  , ...,   11.118411 ,
           11.122959 ,   11.122741 ]],

       [[  10.351611 ,   10.350232 ,   10.352553 , ...,   10.362022 ,
           10.362007 ,   10.361997 ],
        [  10.349756 ,   10.349896 ,   10.352734 , ...,   10.361921 ,
           10.362008 ,   10.362052 ],
        [  10.350339 ,   10.352377 ,   10.3541   , ...,   10.361861 ,
           10.361966 ,   10.361929 ],
        ...,
        [  10.328756 ,   10.324332 ,   10.322899 , ...,   10.3655405,
           10.365465 ,   10.36542  ],
        [  10.325963 ,   10.319494 ,   10.317566 , ...,   10.3655205,
           10.3654995,   10.365434 ],
        [  10.313118 ,   10.308083 ,   10.31015  , ...,   10.364057 ,
           10.365552 ,   10.365443 ]]], dtype=float32)
Coordinates:
    XLAT     (south_north, west_east) float32 16.568985 16.612839 ... 45.305634
    XLONG    (south_north, west_east) float32 104.098694 104.3277 ... 138.15619
Dimensions without coordinates: bottom_top, south_north, west_east
Attributes:
    units:    hPa</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'></div><ul class='xr-dim-list'><li><span>bottom_top</span>: 74</li><li><span>south_north</span>: 119</li><li><span>west_east</span>: 119</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-c339c545-37c4-490d-b7ad-065a079dde04' class='xr-array-in' type='checkbox' checked><label for='section-c339c545-37c4-490d-b7ad-065a079dde04' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>977.6088 973.8247 980.0365 985.46826 ... 10.364057 10.365552 10.365443</span></div><div class='xr-array-data'><pre>array([[[ 977.6088   ,  973.8247   ,  980.0365   , ..., 1005.25305  ,
         1005.3736   , 1005.49805  ],
        [ 972.2685   ,  972.64575  ,  980.2555   , ..., 1005.1616   ,
         1005.28357  , 1005.40784  ],
        [ 973.7658   ,  979.50134  ,  983.94495  , ..., 1005.05743  ,
         1005.1929   , 1005.32166  ],
        ...,
        [ 910.8054   ,  898.6346   ,  894.9356   , ..., 1013.1167   ,
         1012.9816   , 1012.9492   ],
        [ 903.1422   ,  885.46564  ,  880.11725  , ..., 1013.20825  ,
         1013.1019   , 1012.9691   ],
        [ 868.00275  ,  854.2058   ,  859.6906   , ..., 1009.3077   ,
         1013.17224  , 1013.0759   ]],

       [[ 970.978    ,  967.2171   ,  973.3924   , ...,  998.4201   ,
          998.54095  ,  998.6655   ],
        [ 965.6708   ,  966.0465   ,  973.6117   , ...,  998.33044  ,
          998.4526   ,  998.5758   ],
        [ 967.15784  ,  972.86304  ,  977.2757   , ...,  998.2269   ,
          998.3619   ,  998.4895   ],
...
        [  11.009878 ,   10.996195 ,   10.992074 , ...,   11.122887 ,
           11.122656 ,   11.1225195],
        [  11.0012455,   10.981459 ,   10.975507 , ...,   11.123066 ,
           11.122817 ,   11.1225195],
        [  10.961985 ,   10.946582 ,   10.95274  , ...,   11.118411 ,
           11.122959 ,   11.122741 ]],

       [[  10.351611 ,   10.350232 ,   10.352553 , ...,   10.362022 ,
           10.362007 ,   10.361997 ],
        [  10.349756 ,   10.349896 ,   10.352734 , ...,   10.361921 ,
           10.362008 ,   10.362052 ],
        [  10.350339 ,   10.352377 ,   10.3541   , ...,   10.361861 ,
           10.361966 ,   10.361929 ],
        ...,
        [  10.328756 ,   10.324332 ,   10.322899 , ...,   10.3655405,
           10.365465 ,   10.36542  ],
        [  10.325963 ,   10.319494 ,   10.317566 , ...,   10.3655205,
           10.3654995,   10.365434 ],
        [  10.313118 ,   10.308083 ,   10.31015  , ...,   10.364057 ,
           10.365552 ,   10.365443 ]]], dtype=float32)</pre></div></div></li><li class='xr-section-item'><input id='section-a664d938-3432-42fd-8bf2-823f1a8377a7' class='xr-section-summary-in' type='checkbox'  checked><label for='section-a664d938-3432-42fd-8bf2-823f1a8377a7' class='xr-section-summary' >Coordinates: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>XLAT</span></div><div class='xr-var-dims'>(south_north, west_east)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>16.568985 16.612839 ... 45.305634</div><input id='attrs-e54793ab-c887-4620-9aec-bb6e3688e464' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-e54793ab-c887-4620-9aec-bb6e3688e464' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-a9af8f7e-6c04-4d19-93fd-ffe96b2516ba' class='xr-var-data-in' type='checkbox'><label for='data-a9af8f7e-6c04-4d19-93fd-ffe96b2516ba' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>FieldType :</span></dt><dd>104</dd><dt><span>MemoryOrder :</span></dt><dd>XY </dd><dt><span>description :</span></dt><dd>LATITUDE, SOUTH IS NEGATIVE</dd><dt><span>units :</span></dt><dd>degree_north</dd><dt><span>stagger :</span></dt><dd></dd></dl></div><div class='xr-var-data'><pre>array([[16.568985, 16.612839, 16.656075, ..., 17.305672, 17.273308,
        17.240318],
       [16.788559, 16.832611, 16.87603 , ..., 17.52864 , 17.496117,
        17.462982],
       [17.008484, 17.052727, 17.09636 , ..., 17.751953, 17.719307,
        17.68602 ],
       ...,
       [43.68952 , 43.762844, 43.835167, ..., 44.925404, 44.870953,
        44.815437],
       [43.929745, 44.003387, 44.076023, ..., 45.171005, 45.116314,
        45.06056 ],
       [44.16991 , 44.243866, 44.316814, ..., 45.416573, 45.361637,
        45.305634]], dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>XLONG</span></div><div class='xr-var-dims'>(south_north, west_east)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>104.098694 104.3277 ... 138.15619</div><input id='attrs-2a3d3d69-5a30-49d5-a5dc-190213a85314' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-2a3d3d69-5a30-49d5-a5dc-190213a85314' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-ce91913e-049a-4d85-a202-b83b6e5ae5ad' class='xr-var-data-in' type='checkbox'><label for='data-ce91913e-049a-4d85-a202-b83b6e5ae5ad' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>FieldType :</span></dt><dd>104</dd><dt><span>MemoryOrder :</span></dt><dd>XY </dd><dt><span>description :</span></dt><dd>LONGITUDE, WEST IS NEGATIVE</dd><dt><span>units :</span></dt><dd>degree_east</dd><dt><span>stagger :</span></dt><dd></dd></dl></div><div class='xr-var-data'><pre>array([[104.098694, 104.3277  , 104.55701 , ..., 131.43512 , 131.66837 ,
        131.90143 ],
       [104.05249 , 104.282135, 104.512054, ..., 131.46875 , 131.7027  ,
        131.9364  ],
       [104.00604 , 104.23633 , 104.46686 , ..., 131.50262 , 131.73718 ,
        131.97156 ],
       ...,
       [ 96.13068 ,  96.46338 ,  96.796875, ..., 137.30432 , 137.65088 ,
        137.9968  ],
       [ 96.02817 ,  96.36212 ,  96.6969  , ..., 137.3808  , 137.72882 ,
        138.07614 ],
       [ 95.924805,  96.26001 ,  96.59607 , ..., 137.45795 , 137.80743 ,
        138.15619 ]], dtype=float32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-6df035ab-53c1-4bf2-9954-8badf25b640a' class='xr-section-summary-in' type='checkbox'  checked><label for='section-6df035ab-53c1-4bf2-9954-8badf25b640a' class='xr-section-summary' >Attributes: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>hPa</dd></dl></div></li></ul></div></div>


It's time to run the interpolation:


```python
# Explanations are given in https://github.com/pydata/xarray/issues/3931
interped = xr.apply_ufunc(
    interp1d_np,
    era5_o3_interp,
    p_era5,
    p_wrf,
    input_core_dims=[['level'], ['level'], ["bottom_top"]],
    output_core_dims=[['bottom_top']],
    exclude_dims=set(("bottom_top",)),
    vectorize=True,
)
```

Reorgnizie interpolated data to WRF variable dimension:


```python
# sort the dims and delete useless coordinates
interped_o3 = interped.sel(time='2020-09-01 00:00').transpose('bottom_top', ...).expand_dims('Time').drop_vars(['time', 'longitude', 'latitude'])
display(interped_o3)
```


<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt, dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray (Time: 1, bottom_top: 74, south_north: 119, west_east: 119)&gt;
array([[[[  26.85544708,   26.89004326,   26.89004326, ...,
            27.85048673,   27.96454546,   27.97777195],
         [  26.79858192,   26.89004517,   26.90632874, ...,
            27.86999883,   28.03319995,   28.00973581],
         [  26.89004326,   26.91651859,   26.96341212, ...,
            27.75184335,   27.91883112,   28.01447156],
         ...,
         [  42.88559052,   42.90171678,   42.84873043, ...,
            43.76431619,   42.66415577,   42.48096888],
         [  43.05763403,   43.23619572,   42.85495261, ...,
            43.87452885,   42.7925106 ,   42.38859107],
         [  43.60155035,   43.34088363,   43.06596098, ...,
            44.04397503,   43.13010603,   42.49644742]],

        [[  26.86850858,   26.89004326,   26.89004326, ...,
            27.74865668,   27.90227851,   27.96283885],
         [  26.82568012,   26.89004517,   26.89004326, ...,
            27.74462751,   27.90841153,   27.95417291],
         [  26.89004326,   26.89004326,   26.90871055, ...,
            27.62621128,   27.79286227,   27.9352251 ],
...
         [7541.28326442, 7527.26233612, 7509.01665573, ...,
          7506.93546421, 7541.55757499, 7586.09060089],
         [7511.94835776, 7497.8423389 , 7482.09586978, ...,
          7479.4326681 , 7493.47807596, 7517.35630734],
         [7479.79887861, 7466.09814386, 7455.23409676, ...,
          7466.90025385, 7466.7015107 , 7469.38325193]],

        [[9694.82835713, 9693.7357371 , 9689.58353861, ...,
          9585.6528903 , 9590.35306113, 9595.75774179],
         [9686.9817295 , 9685.91018815, 9682.07826455, ...,
          9576.20928858, 9580.64839163, 9585.35674978],
         [9678.86203706, 9677.71921539, 9674.5757891 , ...,
          9566.05929144, 9569.49277165, 9574.19528187],
         ...,
         [7659.76051491, 7642.8204401 , 7622.65272498, ...,
          7648.34201255, 7683.81940009, 7730.1049812 ],
         [7626.72798026, 7609.10646871, 7591.36561556, ...,
          7622.32897339, 7635.5818719 , 7659.58415364],
         [7585.96803727, 7569.48853948, 7558.64050131, ...,
          7612.8093532 , 7611.19161405, 7612.15335775]]]])
Coordinates:
    XLAT     (south_north, west_east) float32 16.568985 16.612839 ... 45.305634
    XLONG    (south_north, west_east) float32 104.098694 104.3277 ... 138.15619
Dimensions without coordinates: Time, bottom_top, south_north, west_east</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'></div><ul class='xr-dim-list'><li><span>Time</span>: 1</li><li><span>bottom_top</span>: 74</li><li><span>south_north</span>: 119</li><li><span>west_east</span>: 119</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-e96934e6-6098-484e-977e-d73018305195' class='xr-array-in' type='checkbox' checked><label for='section-e96934e6-6098-484e-977e-d73018305195' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>26.86 26.89 26.89 26.89 ... 7.62e+03 7.613e+03 7.611e+03 7.612e+03</span></div><div class='xr-array-data'><pre>array([[[[  26.85544708,   26.89004326,   26.89004326, ...,
            27.85048673,   27.96454546,   27.97777195],
         [  26.79858192,   26.89004517,   26.90632874, ...,
            27.86999883,   28.03319995,   28.00973581],
         [  26.89004326,   26.91651859,   26.96341212, ...,
            27.75184335,   27.91883112,   28.01447156],
         ...,
         [  42.88559052,   42.90171678,   42.84873043, ...,
            43.76431619,   42.66415577,   42.48096888],
         [  43.05763403,   43.23619572,   42.85495261, ...,
            43.87452885,   42.7925106 ,   42.38859107],
         [  43.60155035,   43.34088363,   43.06596098, ...,
            44.04397503,   43.13010603,   42.49644742]],

        [[  26.86850858,   26.89004326,   26.89004326, ...,
            27.74865668,   27.90227851,   27.96283885],
         [  26.82568012,   26.89004517,   26.89004326, ...,
            27.74462751,   27.90841153,   27.95417291],
         [  26.89004326,   26.89004326,   26.90871055, ...,
            27.62621128,   27.79286227,   27.9352251 ],
...
         [7541.28326442, 7527.26233612, 7509.01665573, ...,
          7506.93546421, 7541.55757499, 7586.09060089],
         [7511.94835776, 7497.8423389 , 7482.09586978, ...,
          7479.4326681 , 7493.47807596, 7517.35630734],
         [7479.79887861, 7466.09814386, 7455.23409676, ...,
          7466.90025385, 7466.7015107 , 7469.38325193]],

        [[9694.82835713, 9693.7357371 , 9689.58353861, ...,
          9585.6528903 , 9590.35306113, 9595.75774179],
         [9686.9817295 , 9685.91018815, 9682.07826455, ...,
          9576.20928858, 9580.64839163, 9585.35674978],
         [9678.86203706, 9677.71921539, 9674.5757891 , ...,
          9566.05929144, 9569.49277165, 9574.19528187],
         ...,
         [7659.76051491, 7642.8204401 , 7622.65272498, ...,
          7648.34201255, 7683.81940009, 7730.1049812 ],
         [7626.72798026, 7609.10646871, 7591.36561556, ...,
          7622.32897339, 7635.5818719 , 7659.58415364],
         [7585.96803727, 7569.48853948, 7558.64050131, ...,
          7612.8093532 , 7611.19161405, 7612.15335775]]]])</pre></div></div></li><li class='xr-section-item'><input id='section-1266f87e-7b2b-45be-8fed-921e34b29f0a' class='xr-section-summary-in' type='checkbox'  checked><label for='section-1266f87e-7b2b-45be-8fed-921e34b29f0a' class='xr-section-summary' >Coordinates: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>XLAT</span></div><div class='xr-var-dims'>(south_north, west_east)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>16.568985 16.612839 ... 45.305634</div><input id='attrs-7fad8ee5-c1cd-47a4-b296-ba9b80abd28d' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-7fad8ee5-c1cd-47a4-b296-ba9b80abd28d' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-24ce1929-2633-4261-b3b0-b84c44a1c644' class='xr-var-data-in' type='checkbox'><label for='data-24ce1929-2633-4261-b3b0-b84c44a1c644' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>FieldType :</span></dt><dd>104</dd><dt><span>MemoryOrder :</span></dt><dd>XY </dd><dt><span>description :</span></dt><dd>LATITUDE, SOUTH IS NEGATIVE</dd><dt><span>units :</span></dt><dd>degree_north</dd><dt><span>stagger :</span></dt><dd></dd></dl></div><div class='xr-var-data'><pre>array([[16.568985, 16.612839, 16.656075, ..., 17.305672, 17.273308,
        17.240318],
       [16.788559, 16.832611, 16.87603 , ..., 17.52864 , 17.496117,
        17.462982],
       [17.008484, 17.052727, 17.09636 , ..., 17.751953, 17.719307,
        17.68602 ],
       ...,
       [43.68952 , 43.762844, 43.835167, ..., 44.925404, 44.870953,
        44.815437],
       [43.929745, 44.003387, 44.076023, ..., 45.171005, 45.116314,
        45.06056 ],
       [44.16991 , 44.243866, 44.316814, ..., 45.416573, 45.361637,
        45.305634]], dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>XLONG</span></div><div class='xr-var-dims'>(south_north, west_east)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>104.098694 104.3277 ... 138.15619</div><input id='attrs-df8341b6-3ca2-46cc-90e4-4b11684c41a6' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-df8341b6-3ca2-46cc-90e4-4b11684c41a6' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f6c1bb7b-75da-4b6f-8f26-c54f346300d0' class='xr-var-data-in' type='checkbox'><label for='data-f6c1bb7b-75da-4b6f-8f26-c54f346300d0' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>FieldType :</span></dt><dd>104</dd><dt><span>MemoryOrder :</span></dt><dd>XY </dd><dt><span>description :</span></dt><dd>LONGITUDE, WEST IS NEGATIVE</dd><dt><span>units :</span></dt><dd>degree_east</dd><dt><span>stagger :</span></dt><dd></dd></dl></div><div class='xr-var-data'><pre>array([[104.098694, 104.3277  , 104.55701 , ..., 131.43512 , 131.66837 ,
        131.90143 ],
       [104.05249 , 104.282135, 104.512054, ..., 131.46875 , 131.7027  ,
        131.9364  ],
       [104.00604 , 104.23633 , 104.46686 , ..., 131.50262 , 131.73718 ,
        131.97156 ],
       ...,
       [ 96.13068 ,  96.46338 ,  96.796875, ..., 137.30432 , 137.65088 ,
        137.9968  ],
       [ 96.02817 ,  96.36212 ,  96.6969  , ..., 137.3808  , 137.72882 ,
        138.07614 ],
       [ 95.924805,  96.26001 ,  96.59607 , ..., 137.45795 , 137.80743 ,
        138.15619 ]], dtype=float32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-14d88b3a-a8b4-4d4b-9abd-e5286c63fd95' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-14d88b3a-a8b4-4d4b-9abd-e5286c63fd95' class='xr-section-summary'  title='Expand/collapse section'>Attributes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'></dl></div></li></ul></div></div>


Check the surface O3 again:


```python
f, axs = plot.subplots(ncols=2, share=0)

ax = axs[0]
m = ax.pcolormesh(era5_o3.longitude,
                  era5_o3.latitude,
                  era5_o3.isel(time=0, level=-1),
                  cmap='viridis',
                  vmin=20,
                  vmax=65)
ax.colorbar(m, label='O$_3$ (ppbv)', loc='r')
ax.format(title='Original')

ax = axs[1]
m = ax.pcolormesh(interped_o3.XLONG,
                  interped_o3.XLAT,
                  interped_o3.isel(Time=0, bottom_top=0),
                  cmap='viridis',
                  vmin=20,
                  vmax=65)
ax.format(title='Interpolated')
ax.colorbar(m, label='O$_3$ (ppbv)', loc='r')

axs.format(xlim=(o3.XLONG.min(), o3.XLONG.max()),
           ylim=(o3.XLAT.min(), o3.XLAT.max()),
           xlabel='longitude',
           ylabel='latitude',)
```


    
![png](/images/sci-tech/2020-10/era5_overview_2.png)

    
The interpolated result is different from the previous one, because the lowest level of ERA5 data is 1000 hPa while the WRF uses the lowest height for the defined eta level.

We can get the index of specific location and use it to pick the interpolated O3 profile.


```python
# convert to index in wrf grid
x_y = ll_to_xy(ncfile, 32, 119)
x, y = x_y[0].values, x_y[1].values
```

## Save and export data

Original surface ozone:


```python
ds_wrf['o3'].isel(Time=0, bottom_top=0).plot(vmin=20*1e-3, vmax=65*1e-3)
```



    
![waccm](/images/sci-tech/2020-10/waccm.png)
    


Replace the era5 surface ozone:


```python
ds_wrf['o3'] = interped_o3*1e-3 # back to ppmv
ds_wrf['o3'].isel(Time=0, bottom_top=0).plot(vmin=20*1e-3, vmax=65*1e-3)
```

    
![era5](/images/sci-tech/2020-10/era5.png)
    


Because the coordinates in the attributes of variables are important for WRF, we should use `netCDF4` instead of `xarray` to change the O3 value.

To make sure the structure and attributes of the new wrfinput is correct, it's better to compare the head info using `ncdump -h`.


```python
# ds_wrf.to_netcdf(path='./wrfinput/20200901/era5/wrfinput_d01',
#                  mode='w',
#                  engine='netcdf4')

# back up the original wrfinput file, please change the path by yourself
!cp ./wrfinput/20200901/mozbc/wrfinput_d01 ./wrfinput/20200901/mozbc/wrfinput_d01_bak

ncfile['o3'][:] = interped_o3*1e-3 # back to ppmv
ncfile.close()

# move the edited one to anywhere you like
!mv ./wrfinput/20200901/mozbc/wrfinput_d01 ./wrfinput/20200901/era5/wrfinput_d01
```

## Comparison of O3 profiles


```python
# check the O3 profile at one pixel
f, axs = plot.subplots(axwidth=4)

# o3_profile = era5_o3.sel(longitude=119, latitude=32, time='2020-09-01 00:00')
# legend_1 = axs.plot(o3_profile, o3_profile.level, label='Original')

interped_o3_profile = interped_o3.sel(west_east=x, south_north=y).isel(Time=0)
p_profile = p_wrf.sel(west_east=x, south_north=y)
legend_1 = axs.plot(interped_o3_profile,
                    p_profile,
                    label='interpolated ERA5 O3 profile',
                    linewidth=3)

wrf_o3_profile = o3.sel(west_east=x, south_north=y).isel(Time=0)
legend_2 = axs.plot(wrf_o3_profile, p_profile, label='orginal wrfinput')

new_wrf_o3_profile = ds_wrf['o3'].sel(west_east=x, south_north=y).isel(Time=0)*1e3 # ppbv
legend_3 = axs.plot(new_wrf_o3_profile,
                    p_profile,
                    label='updated wrfinput',
                    linewidth=1,
                    color='yellow5')

axs.legend([legend_1, legend_2, legend_3], loc='r', ncols=1)
axs.format(xlim=(0, 200),
           ylim=(1000, 100),
           xlabel='Pressure (hPa)',
           ylabel='O$_3$ (ppbv)',)
```


    
![png](/images/sci-tech/2020-10/profiles.png)
    

## Version control

| Version | Action | Time       |
| ------- | ------ | ---------- |
| 1.0     | Init   | 2020-10-27 |