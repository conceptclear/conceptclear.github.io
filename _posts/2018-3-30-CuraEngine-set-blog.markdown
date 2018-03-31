---
layout: post
title:  "CuraEngine开源程序解读1——配置"
date:   2018-3-30 19:35:56
categories: jekyll update
---

3D打印行业近些年在国内发展迅猛，很多新的3D打印公司在国家政策的支持下纷纷建立起来。然而虽然3d打印机型号外形众多，绝大多数公司打印程序仍然采用的是Cura公司提供在github上的一个开源项目——CuraEngine作为内核，
理解学习CuraEngine对于3D打印行业内部程序开发来说还是必不可少的一步。

### Cura  
Cura公司将[Cura](https://github.com/Ultimaker/Cura)的源码提供在github上，Cura15.04版本之后Cura有着较大的改动，15.04版本的[Cura](https://github.com/daid/LegacyCura)同样也提供在了github之上。

Cura是一个采用C++引擎的利用python搭建起来的开源程序，上述链接中的程序实际上是一个pyhon GUI，内部模型的旋转切片gcode生成等都是由[CuraEngine](https://github.com/Ultimaker/CuraEngine)这个纯C++程序所提供的。

本身图形GUI不是我们研究的重点，所以将重点放在CuraEngine的解读之上。

### CuraEngine  
emmmm
