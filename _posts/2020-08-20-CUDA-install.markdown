---
layout: post
title:  "ubuntu裸机安装特定版本cuda"
date:   2020-08-20 02:35:56
category: GPU
---

由于linux自己有一个第三方图形驱动nouveau，CUDA安装经常出现问题，这里记录一下裸机如何安装特定版本的CUDA。

## 安装依赖库
`gcc`和`make`是安装cuda所必需的库，首先安装一下

```
sudo apt install gcc
sudo apt install make
```

## 禁用nouveau
nouveau是一个旨在为nvidia的GPU建立高质量的，免费自由的开源驱动项目，但是有了这个之后安装显卡驱动和cuda都会出问题，所以要先禁用。        
打开blacklist
```
sudo gedit /etc/modprobe.d/blacklist.conf
```
将nouveau添加进去
```
blacklist nouveau
```
重新生成 kernel initramfs
```
sudo update-initramfs -u
```
然后重启
```
reboot
```

## 安装显卡驱动
禁用nouveau重启之后，显示会出现一定的问题，分辨率很低，这只要安装完NVIDIA的驱动之后就可以解决。显卡驱动可以和cuda一起由cuda的安装包安装，但是这种方法容易出现问题，这里采用的是单独安装显卡之后再安装cuda的方法。显卡驱动可以从NVIDIA的官网下载，最新的显卡驱动安装的时候似乎不需要关掉图形界面了，比较老的显卡驱动版本还需要关闭图形界面来进行安装。               
直接下载驱动之后，利用terminal安装
```
sudo sh NVIDIA-Linux-x86_64-x.run
```
一路回车就行，安装完驱动之后显示并不会立刻变正常，可以重启使之生效。

## 安装CUDA
安装完成显卡驱动之后，就可以接着安装CUDA了，这里因为安装的是制定版本的CUDA，它安装的时候里面是附带了一个NVIDIA的显卡驱动的，一般来说会比最新的驱动版本要低，因为在之前已经安装完成，在安装的时候直接选择不装驱动而直接装其他模块就行。
```
sudo sh cuda_x.run
```
安装完成之后，需要将cuda的添加到环境变量中，
```
sudo gedit ~/.bashrc
```
添加两句话进文件中
```
export PATH=/usr/local/cuda-10.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64$LD_LIBRARY_PATH
```
终端运行
```
source ~/.bashrc
```
检查一下
```
nvcc --version
```
显示出CUDA的安装版本就安装成功了。
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Fri_Feb__8_19:08:17_PST_2019
Cuda compilation tools, release 10.1, V10.1.105
```

## Reference
[1] https://blog.csdn.net/wanzhen4330/article/details/81699769