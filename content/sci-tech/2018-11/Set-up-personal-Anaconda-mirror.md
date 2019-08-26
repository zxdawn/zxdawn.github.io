---
title: Set up personal Anaconda mirror
date: 2018-11-16 10:13:08
tags: ["Python"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2018-11/anaconda.png"
summary: "Set up anaconda mirror on hpc without internet"
---

## Predicaments

If you encounter any problems as follows, you should try to set up a personal Python(Anaconda) mirror:

1. Deploy a python environment on servers that have no access to the Internet;
2. Deploy a python environment which can be used directly by other users on the server;
3. Internet speed or connection to cloud mirror is slow.


## Steps

### Installation

Just install Anaconda according to this [guide](https://conda.io/docs/user-guide/install/index.html).

Of course, you can download Anaconda installer archive from [here](https://repo.continuum.io/pkgs/).

### Download(mirror)

You can use `wget` to mirror the packages from [official Anaconda websites](https://repo.continuum.io/pkgs/) or other mirror websites ([TUNA](https://mirrors.tuna.tsinghua.edu.cn/anaconda/) and [USTC](https://mirrors.ustc.edu.cn/anaconda)).

If your server is located in China, I recommend the later one.

I'll take the later one as example.

<!--more-->

Create your directory for saving packages:

```
$ mkdir anaconda
```

Create two files for downloading packages:

```
$ cd anaconda
$ touch download.sh links.txt
$ chmod +x download.sh
```

Save all urls you wanted to `links.txt`:

```
https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/linux-64/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/noarch/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/noarch/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro/linux-64/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro/noarch/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/linux-64/
https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/noarch/
```

Then, edit the `download.sh` file to download all files in each directory:

```
#!/bin/bash
for link in `cat links.txt`; do
    wget -m -np -nH --cut-dirs=1 -e robots=off -R "index.html*" $link
done
```

Explanations of options:

```
-m,  --mirror
    Turn on options suitable for mirroring.  This option turns on
    recursion and time-stamping, sets infinite recursion depth and
    keeps FTP directory listings.  It is currently equivalent to 
    -r -N -l inf --no-remove-listing.
-np, --no-parent
    don't ascend to the parent directory.
-nH, --no-host-directories
    Disable generation of host-prefixed directories.
--cut-dirs=number
    Ignore number directory components. This is useful for getting a fine-grained control over     the directory where recursive retrieval will be saved.
-e robots=off
    ignore robots.txt.
-R,  --reject=LIST
    comma-separated list of rejected extensions.
```

*PS : `conda-forge` is much larger than others.* Of course, you can just download specific version and packages:

```
# Reject other versions except Python2.7
# Beware, not to use something like "only py27" packages.
# Indeed, many packages in the repo #don't have version string in their name;
# and you'll miss them as dependency.
# All package information is written in a repodata.json and repodata.json.bz2.
$ wget -m -np -nH --cut-dirs=2 -e robots=off -R --regex-type pcre --reject-regex '(.*py26.*)|(.*py3[3456].*)' https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/linux-64/
```

**Don't forget to download `noarch`, otherwise `conda` won't work correctly.**

Anyway, if your `links.txt` and `download.sh` are as same sa mine, you will get this tree of directory:

```
└── anaconda
    ├── cloud
    │   └── conda-forge
    │       ├── linux-64
    │       └── noarch
    └── pkgs
        ├── free
        │   ├── linux-64
        │   └── noarch
        ├── main
        │   ├── linux-64
        │   └── noarch
        └── pro
            ├── linux-64
            └── noarch
```

### Settings

It's more convenient to  save all packages in one directory and just using `channel_alias` for channels.

Let's move `cloud/conda-forge` to `anaconda/pkgs`.

```
└── anaconda
    └── pkgs
        ├── conda-forge
        │   ├── linux-64
        │   └── noarch
        ├── free
        │   ├── linux-64
        │   └── noarch
        ├── main
        │   ├── linux-64
        │   └── noarch
        └── pro
            ├── linux-64
            └── noarch
```

Now, you can modify the channel_alias and add channels:

```
$ conda config --set channel_alias file://Path_to_your_channel/anaconda/pkgs
$ conda config --add channels free
$ conda config --add channels pro
$ conda config --add channels main
$ conda config --add channels conda-forge
$ conda config --remove channels defaults
$ conda config --set offline True
```

Then, check `~/.condarc` :

```
channels:
  - conda-forge
  - main
  - pro
  - free
channel_alias: file://Path_to_your_channel/anaconda/pkgs
```

If you prefer command line, you can just use `conda config --get` :

```
$ conda config --get
--set channel_alias file://Path_to_your_channel/anaconda/pkgs
--add channels 'free'   # lowest priority
--add channels 'pro'
--add channels 'main'
--add channels 'conda-forge'   # highest priority
--set offline True
```

If you make a mistake during adding channel, you can remove it:

```
$ conda config --remove channels <your_mistake>
```

>  By default, conda now prefers packages from a higher priority channel over any version from a lower priority channel. Therefore, you can now safely put channels at the bottom of your channel list to provide additional packages that are not in the default channels, and still be confident that these channels will not override the core package set.

So, if you want to set the order to free &rarr; main &rarr; conda-forge, you can rearrange them by terminal or edit the `.condarc` file directly.

Anyway, you will get this `.condarc` file:

```
allow_other_channels : false
channel_alias: file://datadir2/anaconda/pkgs
channels:
  - free
  - main
  - pro 
  - conda-forge
offline: true
```

---

**Content between these dividing lines are wrong according to this [answer](https://stackoverflow.com/a/37543307/7347925).**

Let's create a new `env` named `test` to prepare for next step (Aggregation).

```
$ conda create -n test python=3.7
$ source activate test
$ conda install conda-build
```

### ~~Aggregation~~

> **What about the 'anaconda' channel on anaconda.org at https://anaconda.org/anaconda?** 
> The Anaconda channel on anaconda.org is an aggregated mirror of the packages in https://repo.continuum.io/pkgs/main, https://repo.continuum.io/pkgs/free, and https://repo.continuum.io/pkgs/pro. If you are using conda's 'defaults' channel, you do not need the 'anaconda' channel.

Although you've downloaded packages, but if you want to install as usual like `conda install -c anaconda netcdf4`, you have to aggregate `/main`, `/free` and `/pro`.

So, you should create a directory named `anaconda` under `anaconda_mirror`.

Then, link all files under each `linux-64` and `noarch` of `/main`, `/free` and `/pro` to `linux-64` and `noarch` under `anaconda` directory.

Now, you can use `conda-build` to generates a file `repodata.json`.

```
$ conda index Path_to_your_channel/anaconda/pkgs/anaconda/linux-64/ Path_to_your_channel/anaconda/pkgs/anaconda/noarch/
```

---

### Update/Install

Now, you can use `conda update/install` :

```
$ conda install <package>
$ conda install --channel conda-forge <package>
```

Don't use `anaconda` channel anymore, because we just have `free`, `main`, `pro` and `conda-forge` channels.

For example, if you want to install `netcdf4` from `anaconda` channel, don't specify channel name:

```
$ conda install netcdf4
```

If you have many packaged to install, you can create `requirements.txt` and install all of them by one command.

```
$ while read requirement; do conda install --yes $requirement; done < requirements.txt 2>error.log
```

Here's one example of `requirements.txt` :

```
netcdf4
h5py
matplotlib
statsmodels
```

## Advanced

If you're the administrator of the server, you can use system `.condarc` to set a environment for all users.

> You can make conda and any number of packages available to a group of 1 or more users, while preventing these users from installing unwanted packages with conda:
>
> 1. Install conda and the allowed packages, if any, in a location that is under administrator control and accessible to users.
> 2. Create a [.condarc system configuration file](https://conda.io/docs/user-guide/configuration/use-condarc.html) in the root directory of the installation. This system-level configuration file will override any user-level configuration files installed by the user.
>
> Each user accesses the central conda installation, which reads settings from the user `.condarc`configuration file located in their home directory. The path to the user file is the same as the root environment prefix displayed by `conda info`, as shown in [User configuration file](https://conda.io/docs/user-guide/configuration/admin-multi-user-install.html#admin-inst-user) below. The user`.condarc` file is limited by the system `.condarc` file.
>
> System configuration settings are commonly used in a system `.condarc` file but may also be used in a user `.condarc` file. All user configuration settings may also be used in a system `.condarc` file.

You can just check the official [document](https://conda.io/docs/user-guide/configuration/admin-multi-user-install.html) about administrator-controlled installation.

Here's the example of my system `.condarc` :

```
allow_other_channels : false
channel_alias: file://Path_to_your_channel/anaconda/pkgs
channels:
  - free
  - main
  - pro
  - conda-forge
offline: true
```

## References

1. https://stackoverflow.com/questions/37391824/simply-use-python-anaconda-without-internet-connection
2. http://www.meteoboy.com/conda-without-internet.html#
3. https://conda.io/docs/user-guide/configuration/use-condarc.html
4. https://conda.io/docs/user-guide/configuration/admin-multi-user-install.html
5. https://conda.io/docs/user-guide/tasks/create-custom-channels.html