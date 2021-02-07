---
title: "Swap on Raspbian"
date: 2021-02-07T15:55:12+08:00
draft: false
tags: ["Linux"]
---

最近在 Raspberry Pi 上編譯 ARM 版本的 Docker 時遇到的

```
gcc: cannot allocate memory
```

這台樹莓派只有 1G RAM，快絕望的時候突然想到是不是可以朝 Swap 的方向去嘗試，Google 了一下果然有相關的討論。

# dphys-swapfile
Raspbian 可以修改 `/etc/dphys-swapfile` 來控制Swap

```
CONF_SWAPSIZE=2000
```

```
$ sudo service dphys-swapfile restart
```

Swap的讀寫非常頻繁，容易耗損 SD Card 這類讀寫次數有限的裝置，把 Swap File 放在 External USB Stick 可以避免這問題，但讀寫速度是大缺點。

```
CONF_SWAPFILE=/mnt/USB/swap
```

# mkswap

另一台沒有dphys-swapfile可以用mkswap

```shell
# Create swap file
# 1024 * 512M = 524288 block size
dd if=/dev/zero of=/tmp/testswap bs=1024 count=524288 

# Adjust permission
chown root:root /tmp/testswap
chmod 0600 /tmp/testswap

# Set up swap area
mkswap /tmp/testswap

# Enable swap
swapon /tmp/testswap

# Check swap status
swapon -s

# Disable swap
swapoff 
```

開機自動掛載，在 `/etc/fstab` 加上
```shell
/tmp/testswap none swap sw 0 0
```

網路上看到有些人在Create swap file用的是`fallocate`，但這其實不對，在SWAPON(8)有說明
> Commands like cp(1) or truncate(1)  create  files  with  holes.   These
> files will be rejected by swapon.

> Preallocated  files created by fallocate(1) may be interpreted as files
> with holes too depending of the filesystem.   Preallocated  swap  files
> are supported on XFS since Linux 4.18.

> The  most  portable  solution to create a swap file is to use dd(1) and 
> /dev/zero.







