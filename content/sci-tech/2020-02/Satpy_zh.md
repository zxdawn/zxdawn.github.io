---
title: Satpy中英双语简介
date: 2020-02-24
tags: ["Satpy", "Satellite","Python"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-02/slack_logo.png"
summary: "Satpy中英双语手册"
---

## 概览

Satpy is designed to provide easy access to common operations for processing meteorological remote sensing data.

Satpy旨在为处理气象卫星数据提供一个便捷的工具。

Any details needed to perform these operations are configured internally to Satpy meaning users should not have to worry about *how* something is done, only ask for what they want.

Satpy已经在模块内部配置好了所需信息，用户无需考虑每步的具体操作，**只需提供自己想获得的内容即可**。

Most of the features provided by Satpy can be configured by keyword arguments (see the API Documentation or other specific section for more details).

用户可以通过`关键字参数`来使用大部分的功能（详见[API文档](https://satpy.readthedocs.io/en/latest/api/satpy.html)和其他[官方文档](https://satpy.readthedocs.io/en/latest)）。

For more complex customizations or added features Satpy uses a set of configuration files that can be modified by the user.

如果需要自己定制或者添加其他功能特性，用户可以通过修改Satpy的`配置文档`来轻松实现。

The various components and concepts of Satpy are described below. The Quickstart guide also provides simple example code for the available features of Satpy.

下面主要介绍Satpy的常用模块和概念，更多示例请访问英文版[速成教程](https://satpy.readthedocs.io/en/latest/quickstart.html)。

## Slack群组

Satpy在[*朝曦dawn*](https://dreambooker.site/)和[*Martin Raspaud*](https://github.com/mraspaud)的建议下，建立了Slack的`PyTroll中文版`板块：

![Slack](/images/sci-tech/2020-02/slack_zh.png)

邀请链接：https://pytrollslackin.herokuapp.com/
Slack Chat链接：https://pytroll.slack.com/

## Scene

Satpy provides most of its functionality through the [`Scene`](https://satpy.readthedocs.io/en/latest/api/satpy.html#satpy.scene.Scene) class.

Satpy主要通过`Scene`来实现大部分功能。

This acts as a container for the datasets being operated on and provides methods for acting on those datasets.

`Scene`其实是一个针对数据集的容器，并且提供了处理数据集的很多方法。

It attempts to reduce the amount of low-level knowledge needed by the user while still providing a pythonic interface to the functionality underneath.

`Scene`既可以降低用户的使用门槛，又可以为底层的函数提供接口。

A Scene object represents a single geographic region of data, typically at a single continuous time range.

一个单独的`Scene`对象代表了一块单独的区域（通常是在某个连续时间段内）。

It is possible to combine Scenes to form a Scene with multiple regions or multiple time observations, but it is not guaranteed that all functionality works in these situations.

用户可以把多个（时间或者空间）`Scene`组合成一个`Scene`，但是合成之后部分功能可能无法适用。

## DataArrays

Satpy’s lower-level container for data is the [`xarray.DataArray`](https://xarray.pydata.org/en/stable/generated/xarray.DataArray.html#xarray.DataArray). For historical reasons DataArrays are often referred to as “Datasets” in Satpy.

Satpy针对数据的低级容器是`xarray.DataArray`，在Satpy里称为`Datasets`。

These objects act similar to normal numpy arrays, but add additional metadata and attributes for describing the data.

这些对象不仅类似于numpy数组，而且还保留了数据的元数据和属性。

Metadata is stored in a `.attrs` dictionary and named dimensions can be accessed in a `.dims` attribute, along with other attributes.

`.attrs`的字典包含了元数据信息，而`.dims`则储存了维度和其他属性。

In most use cases these objects can be operated on like normal NumPy arrays with special care taken to make sure the metadata dictionary contains expected values. See the XArray documentation for more info on handling `xarray.DataArray` objects.

大多数情况下，用户可以像操作Numpy数组一样，对这些对象进行操作，但得留意元数据是否正常。

更多信息参考`xarray.DataArray`的说明。

Additionally, Satpy uses a special form of DataArrays where data is stored in [`dask.array.Array`](https://docs.dask.org/en/latest/array-api.html#dask.array.Array) objects which allows Satpy to perform multi-threaded lazy operations vastly improving the performance of processing. For help on developing with dask and xarray see [Migrating to xarray and dask](https://satpy.readthedocs.io/en/latest/dev_guide/xarray_migration.html) or the documentation for the specific project.

此外，Satpy通过`dask.array.Array`对象实现多线程计算，从而提高处理性能。

更多信息参考dask和xarray的使用说明。

To uniquely identify `DataArray` objects Satpy uses DatasetID. A `DatasetID` consists of various pieces of available metadata. This usually includes name and wavelength as identifying metadata, but also includes resolution, calibration, polarization, and additional modifiers to further distinguish one dataset from another.

为了区别`DataArray`，Satpy提供了类似于身份证的`DatasetID`。

`DatasetID`由多种元数据构成，如：名字，波长，分辨率，校准方式，极化方式等等。

> Warning
>
> XArray includes other object types called “Datasets”. These are different from the “Datasets” mentioned in Satpy.

> 注：
>
> XArray 里的“Datasets”不同于Satpy里的“Datasets”

## Reading

One of the biggest advantages of using Satpy is the large number of input file formats that it can read. It encapsulates this functionality into individual [Readers](https://satpy.readthedocs.io/en/latest/readers.html).

使用Satpy的一大好处就是可以读取多种数据格式。这一功能封装在`Readers`里。

Satpy Readers handle all of the complexity of reading whatever format they represent.

无论数据的格式读起来多么复杂，Satpy都可以处理。

Meteorological Satellite file formats can be extremely complex and formats are rarely reused across satellites or instruments.

而气象卫星的文件格式往往都晦涩难懂，不同卫星或仪器之间格式也很少通用。

No matter the format, Satpy’s Reader interface is meant to provide a consistent data loading interface while still providing flexibility to add new complex file formats.

`Reader`就是在这种情形下，为用户提供一个固定的读取界面，同时也能让用户灵活地添加新的文件格式。

## Compositing

Many users of satellite imagery combine multiple sensor channels to bring out certain features of the data. This includes using one dataset to enhance another, combining 3 or more datasets in to an RGB image, or any other combination of datasets.

很多用户想将多通道合成为具有一定特性的数据，比如用一个数据来优化另一个，将3个或者更多数据合成为一张RGB真彩色图，或者任何其他特殊的数据集。

 Satpy comes with a lot of common composite combinations built-in and allows the user to request them like any other dataset.

Satpy提供了很多通用的`composite`组合供用户使用。

Satpy also makes it possible to create your own custom composites and have Satpy treat them like any other dataset. See [Composites](https://satpy.readthedocs.io/en/latest/composites.html) for more information.

当然，用户也可以自己定制`composites`让Satpy处理。

更多信息，参考Composites。

## Resampling

Satellite imagery data comes in two forms when it comes to geolocation, native satellite swath coordinates and uniform gridded projection coordinates.

卫星图像数据的坐标系有两种：原始卫星幅宽坐标系、均匀网格投影坐标

It is also common to see the channels from a single sensor in multiple resolutions, making it complicated to combine or compare the datasets.

此外，一个传感器的通道也有多种分辨率，这使得融合和对比资料比较困难。

Many use cases of satellite data require the data to be in a certain projection other than the native projection or to have output imagery cover a specific area of interest.

许多用户需要将数据进行投影变换或者生成特定区域的图像。

Satpy makes it easy to resample datasets to allow for users to combine them or grid them to these projections or areas of interest.

Satpy可以让用户轻松地把数据集重采样，合并，或者投影。

Satpy uses the PyTroll pyresample package to provide nearest neighbor, bilinear, or elliptical weighted averaging resampling methods. See [Resampling](https://satpy.readthedocs.io/en/latest/resample.html) for more information.

Satpy重采样使用的是`pyresample`包，提供了邻近，双线性和椭圆加权平均重采样方法。

## Enhancements

When making images from satellite data the data has to be manipulated to be compatible with the output image format and still look good to the human eye.

当用卫星数据出图时，我们还要考虑是否符合审美需求。

Satpy calls this functionality “enhancing” the data, also commonly called scaling or stretching the data.

Satpy称之为`enhancing`（美化）数据，也叫做缩放或拉伸数据。

This process can become complicated not just because of how subjective the quality of an image can be, but also because of historical expectations of forecasters and other users for how the data should look.

这一过程因数据质量的主观性，预报员或其他用户的不同需求，而变得复杂。

Satpy tries to hide the complexity of all the possible enhancement methods from the user and just provide the best looking image by default.

Satpy将所有可能的`enhancement`方法隐藏起来，仅仅为用户提供默认最美观的图像。

Satpy still makes it possible to customize these procedures, but in most cases it shouldn’t be necessary. See the documentation on [Writers](https://satpy.readthedocs.io/en/latest/writers.html) for more information on what’s possible for output formats and enhancing images.

Satpy当然也可以让用户定制属于自己的美化方案。

更多信息，参考Writers。

## Writing

Satpy is designed to make data loading, manipulating, and analysis easy.

Satpy旨在让加载，处理，分析数据整个流程变得简单易懂。

However, the best way to get satellite imagery data out to as many users as possible is to make it easy to save it in multiple formats.

然而，要将卫星图像提供给多用户，最好的方法还是提供多种存储格式。

Satpy allows users to save data in image formats like PNG or GeoTIFF as well as data file formats like NetCDF.

Satpy可以将图像存储为PNG或GeoTIFF格式，也可以存储为NetCDF格式数据。

Each format’s complexity is hidden behind the interface of individual Writer objects and includes keyword arguments for accessing specific format features like compression and output data type. See the [Writers](https://satpy.readthedocs.io/en/latest/writers.html) documentation for the available writers and how to use them.

每种格式的复杂性都在每个`Writer`的底层，并且提供了关键字参数来指定压缩和输出格式。

更多信息，参考Writers。

## References

1. Overview: https://satpy.readthedocs.io/en/latest
2. Satpy examples: https://satpy.readthedocs.io/en/latest/examples.html
3. Presample: http://pyresample.readthedocs.org/
4. Pytroll: http://pytroll.github.io/

## Version control

| Version | Action | Time       |
| ------- | ------ | ---------- |
| 1.0     | Init   | 2020-02-24 |
