---
layout: post
title:  "CuraEngine开源程序解读1——配置"
date:   2018-3-30 19:35:56
categories: jekyll update
---

3D打印行业近些年在国内发展迅猛，很多新的3D打印公司在国家政策的支持下纷纷建立起来。然而虽然3d打印机型号外形众多，绝大多数公司打印程序仍然采用的是Ultimaker公司提供在github上的一个开源项目——CuraEngine作为内核，
理解学习CuraEngine对于3D打印行业内部程序开发来说还是必不可少的一步。

### Cura  
Ultimaker公司将[Cura](https://github.com/Ultimaker/Cura)的源码提供在github上，Cura15.04版本之后Cura有着较大的改动，15.04版本的[Cura](https://github.com/daid/LegacyCura)同样也提供在了github之上。

Cura是一个采用C++引擎的利用python搭建起来的开源程序，上述链接中的程序实际上是一个pyhon GUI，内部模型的旋转切片gcode生成等都是由[CuraEngine](https://github.com/Ultimaker/CuraEngine)这个纯C++程序所提供的。

本身图形GUI不是我们研究的重点，所以将重点放在CuraEngine的解读之上。

### CuraEngine  
CuraEngine是一个用于生成3D打印gcode的C++控制台程序。相较于Skeinforge引擎，CuraEngine速度上要更为快速。在对其中切片所得到的二维图形进行处理时，CuraEngine采用了[Clipper](http://www.angusj.com/delphi/clipper.php)这个开源的免费库，该库基于Vatti的裁剪算法，对线和多边形进行裁剪，包括求交，合并，偏移等。除此之外，Cura与其后端和类似代码之间的通信，Ultimaker公司同样将开源代码[libArcus](https://github.com/Ultimaker/libArcus)提供在了github上。libArcus是基于Ptotocol Buffers库来进行通信的，所以配置libArcus的时候需要配置Protocol Buffers库，即[protobuf](https://github.com/google/protobuf)。

### 许可协议
CuraEngine根据AGPLv3许可证发布。许可证的条款可以在CuraEngine的LICENSE文件中找到,或者访问[GNU AFFERO通用公共许可证](http://www.gnu.org/licenses/agpl.html)。
如果使用CuraEngine来创建应用程序，需要分享你的源码。

### 安装配置
程序解读首先需要能够运行这个程序并且知道程序可以做到什么，按照官方所提供的安装配置方法来进行配置，这里采用Ubuntu16.04.3系统来进行编译。

安装可以分为三部分：编译protobuf，编译libArcus和编译CuraEngine

#### 编译protobuf
1.确保自己配置了`libtool`，若未安装，可以利用`apt`进行安装。  
````
sudo apt install libtool
````
2.从[protobuf](https://github.com/google/protobuf/releases/tag/v3.5.1)的发行版上下载protobuf(>3.0.0)。  
3.在protobuf的文件夹中运行`autogen.sh`文件
````
cd protobuf-×
./autogen.sh
````
如果安装过程中出现`autoreconf:not found`的错误提示，是由于系统中没有安装autoreconf工具，可以利用`apt`进行安装
````
sudo apt install autoreconf
````
4.运行configure：`./configure`  
5.`make`  
6.`sudo make install`  

#### 编译libArcus库
1.需要安装`CMake`，`python3-dev(3.4+)`和`python3-sip-dev(4.16+)`。  
2.git clone [libArcus](https://github.com/Ultimaker/libArcus)的仓库或者直接下载zip包。  
3.利用CMake对libArcus进行编译
````
mkdir build
cd build
cmake ..
make
sudo make install
````

#### 编译CuraEngine库
1.git clone [CuraEngine](https://github.com/Ultimaker/CuraEngine)的仓库或者直接下载zip包。  
2.利用CMake对CuraEngine进行编译
````
mkdir build
cd build
cmake ..
make
cmake . -G "CodeBlocks - Unix Makefiles"
````

### 运行CuraEngine
与Cura软件所不同的是，CuraEngine需要通过命令行的方式进行运行。运行CuraEngine需要一个JSON的配置文件，在Cura的repository中可以找到。自Cura2.1版本后，json文件结构发生了变化。
CuraEngine切片的用法在CuraEngine Help中提供如下：  
````
CuraEngine slice [-v] [-p] [-j <settings.json>] [-s <settingkey>=<value>] [-g] [-e<extruder_nr>] [-o <output.gcode>] [-l <model.stl>] [--next]
````  
- -v：增加详细级别（这里可以输出log消息）
- -m\<thread_count\>:设置所需要的线程数
- -p：记录log信息
- -j：加载settings.def.json文件以注册所有设置及其默认值
- -s \<setting\>=\<value\>：将最后提供的对象的某个setting值更改为value
- -l \<model_file\>：加载STL文件
- -g：将设置焦点切换到当前网格组，适用于单次打印
- -e\<extruder_nr\>：根据给定值转移设置焦点
- --next：为先前提供的网格组生成gcode，并将其附加到其他模型的gcode中
- -o \<output_file\>：选择gcode写入的文件

由于文件一直在更新，官方文件所提供的例子无法直接运行，这里提供一个例子：  
进入CuraEngine所在文件夹
````
./build/CuraEngine slice -v -j ../Cura/resources/definitions/zone3d_printer.def.json -o "./output/test.gcode" -e0 -s infill_line_distance=1 -e0 -l "./build/testcub1.STL"
````

运行结果如图所示：  
![example](https://github.com/conceptclear/conceptclear.github.io/blob/master/images/CuraEngine_set/example_CuraEngine_setting.png "Example")

利用Cura打开生成的gcode文件如图：  
![gcode](https://github.com/conceptclear/conceptclear.github.io/blob/master/images/CuraEngine_set/example_CuraEngine_gcode_testcub.png "gcode")

### Reference
[1]https://github.com/Ultimaker/CuraEngine  
[2]https://github.com/Ultimaker/Cura  
[3]https://github.com/Ultimaker/Uranium  
[4]https://github.com/Ultimaker/libArcus  
[5]CuraEngine Help

<div id="container"></div>
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
<script>
var gitment = new Gitment({
  id: 'CuraEngine_set.href', // 可选。默认为 location.href
  owner: 'conceptclear',
  repo: 'githubpages-comments',
  oauth: {
    client_id: '6a29f84533d3ebc673da',
    client_secret: 'b1537face0afad64fafa7e6fd7169df85b9d9eb2',
  },
})
gitment.render('container')
</script>
