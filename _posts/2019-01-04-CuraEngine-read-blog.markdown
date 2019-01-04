---
layout: post
title:  "CuraEngine开源程序————工具类"
date:   2019-01-04 10:35:56
category: 3dprint
---

CuraEngine作为一个不断在维护的开源软件，在版本更新的过程中代码经常会有一些变化，这里尽力按照类别来对其进行解读，也方便代码更新之后进行修改。
源码中将一部分文件归类于utils文件夹中，这部分代码主要起到的是基础工具类的作用，这里首先对这部分代码进行分析解读。
由于CuraEngine主要重用了clipperlib库中提供的IntPoint的定义，所以顶点坐标数据都是以整数点的形式保存在内存之中，单位为um（微米）。

## Coord_t
Coord_t中没有定义一个新的类，主要是一些坐标转换的宏定义，其中包括：           
- 对Clipper库中的cInt类型给出了一个别名coord_t;           
- 宏定义INT2MM(n),INT2MM2(n),MM2INT(n),MM2_2INT(n),MM3_2INT(n),主要实现int类型的坐标（微米）同毫米之间的转换；             
- 宏定义INT2MICRON(n),MICRON2INT(n)，实现int类型坐标同微米之间的转换。              

## Point3
Point3中定义了Point3类，用于存储coord_t类型的三维坐标值。          
### Point3类
#### 成员变量
存储三维的常量整数点坐标，coord_t x,y,z。               
#### 成员函数
- **运算符重载**。重载了几个常用的运算符，包括同Point3类的+,-,\*,/,+=,-=,\*=,/=,==,!=以及和模板类型数据的\*,/,\*=,/=；          
- **coord_t max() const**。用于检测坐标值的最大值；          
- **bool testLength(coord_t len) const**。用于检测坐标点距离原点长度是否满足要求；        
- **coord_t vSize2() const**。用于返回三维整数点距离原点距离的平方；          
- **coord_t vSize() const**。用于返回三维整数点距离原点的距离；          
- **double vSizeMM() const**。用于返回三维点距离原点的距离（mm）；          
- **coord_t dot(const Point3& p) const**。用于返回两个三维向量的点积；                    
### 全局变量
定义了一个简单占位符坐标点no_point3（brief Placeholder coordinate point (3D)），作用不明。该坐标点是由三个int32_t类型所能提供的最小值构成的点。           

## IntPoint
IntPoint中包含了CuraEngine用来尽可能快的使用的二位整数点类IntPoint，二维点矩阵变换类PointMatrix和三维点矩阵变换类Point3Matrix，并对clipperlib中的IntPoint类赋予了Point的别名。                
### IntPoint类
#### 成员变量
存储二维坐标变量，int X, Y。          
#### 成员函数
- **Point p()**。用于返回Point类型的x，y坐标；          

### Point类
对clipper中的IntPoint类定义了一个别名，即为这里的Point类。            
#### 相关内联函数
- **运算符重载**。重载了几个常用的运算符，包括和Point类的+,-,+=,-=,/以及和模板类型的\*,/；          
- **Point normal(const Point& p0, coord_t len)**。用于返回len长度的Point向量；          
- **bool shorterThen(const Point& p0, int32_t len)**。用于检测坐标点距离原点长度是否满足要求；        
- **coord_t vSize2(const Point& p0)**。用于返回二维整数点距离原点距离的平方；          
- **float vSize2f(const Point& p0)**。用于返回二维浮点距离原点距离的平方；          
- **coord_t vSize(const Point& p0)**。用于返回二维整数点距离原点的距离；          
- **double vSizeMM(const Point& p0)**。用于返回二维点距离原点的距离（mm）；          
- **coord_t dot(const Point& p0, const Point& p1)**。用于返回两个二维向量的点积；          
- **Point turn90CCW(const Point& p0)**。用于返回p0向量逆时针旋转90°；          
- **Point rotate(const Point& p0, double angle)**。用于返回p0向量逆时针旋转angle(弧度)；          
- **int angle(const Point& p)**。用于返回p0向量与x轴的夹角（角度）；          

### PointMatrix类           
PointMatrix类是用来对二维点进行旋转变换的矩阵类。              
#### 成员变量           
定义了double类型的大小为4的一维数组matrix[4]。          
#### 成员函数
- **构造函数**。不带参数的构造函数默认matrix数组为{1,0,0,1}；可以用弧度制和Point类型的向量值来构造变换矩阵。
- **Point apply(const Point p) const**。Point p逆时针旋转。                 
- **Point unapply(const Point p) const**。Point p顺时针旋转。                 

### Point3Matrix类        
Point3Matrix类是用来对三维点进行旋转变换的矩阵类。              
#### 成员变量         
定义了double类型的大小为9的一维数组matrix[9]。           
#### 成员函数          
- **构造函数**。不带参数的构造函数默认matrix数组为{1,0,0,0,1,0,0,0,1}；可以用PointMatrix类型的向量值来构造缺少第三个方向值的变换矩阵。                   
- **运算符重载**。重载了与Point3类型和Point类型的+,+=,-,-=。             
- **Point3 apply(const Point3 p) const**。Point3 p逆时针旋转。                 
- **Point apply(const Point p) const**。Point p逆时针旋转（只在xy平面投影）。           
- **static Point3Matrix translate(const Point p)**。构造平移Point p向量的矩阵。           
- **Point3Matrix compose(const Point3Matrix& b)**。实现矩阵相乘。           

## floatpoint
在模型加载期间会使用到三维浮点向量，这里定义了以毫米为单位的浮点类FPoint3和其旋转变换的矩阵类FMatrix3x3。             
### FPoint3类
#### 成员变量
存储三维的常量浮点坐标，float x,y,z。               
#### 成员函数
- **构造函数**。不带参数的构造函数不会给成员变量赋值或者初始化，可以直接用float类型参数给成员变量初始化，也可以通过Point3类型的变量p通过坐标变换初始化。           
- **运算符重载**。重载了几个常用的运算符，包括同FPoint3类的+,-,+=,-=,==和!=，以及同float类型的\*,/,\*=；          
- **float max() const**。用于检测坐标值的最大值；          
- **bool testLength(float len) const**。用于检测坐标点距离原点长度是否满足要求；        
- **float vSize2() const**。用于返回三维浮数点距离原点距离的平方；          
- **float vSize() const**。用于返回三维浮数点距离原点的距离；          
- **FPoint3 normalized() const**。用于对浮点向量进行归一化；          
- **FPoint3 cross(const FPoint3& p) const**。用于实现两个向量叉乘；                    
- **Point3 toPoint3**。用于实现三维浮点坐标转换为整数点坐标；                      

### FMatrix3x3类           
FMatrix3x3类是用来对三维浮点数点进行旋转变换的矩阵类。              
#### 成员变量           
定义了double类型的总大小为3*3的二维数组m[3][3]。              
#### 成员函数
- **构造函数**。不带参数的构造函数默认m[3][3]数组为{1.0,0.0,0.0,0.0,1.0,0.0,0.0,0.0,1.0}。                    
- **Point3 apply(const FPoint3& p) const**。Point p逆时针旋转并将旋转后的向量转为Point3类型。                                

## Date
Date中定义了用来表示当前年月日的一个简单类Date。                 
### Date类
#### 成员变量
存储年月日的三个int类型量，int year,month,day。                       
#### 成员函数
- **构造函数**。带参数的构造函数直接初始化年月日三个量，不带参数的构造函数为私有成员函数，默认全部初始化为-1。          
- **static Date getDate()**。通过宏__DATE__获取编译时的时间。            
- **std::string toStringDashed()**。将Date类中的数据转换为yyyy-mm-dd格式输出。         

## algorithm
algorithm文件主要是对C++的algorithm库的拓展，定义了一个用于返回vector向量排序索引的函数 **order**。                         

## math
math文件主要是对C++的cmath库的拓展。
### 宏定义
C++11开始不再对M_PI有宏定义，这里定义M_PI。                 
### 全局函数
- **static const float sqrt2**。全局常量，值为 $$\sqrt{2}$$。            
- **inline T square(const T& a)**。计算模板类型的平方。                    
- **inline unsigned int round_divide(unsigned int dividend, unsigned int divisor)**。计算dividend被divisor平分后最接近的的组数。                  
- **inline unsigned int round_up_divide(unsigned int dividend, unsigned int divisor)**。计算平分后的组数，向上取整。                 

## gettime
gettime中定义了一个TimeKeeper类用于获取程序运行时间。                     
### TimeKeeper类
TimeKeeper类主要是用于获取程序段的运行时间。                        
#### 成员函数
double类型的startTime。          
#### 成员函数
- **构造函数**。调用restart函数。              
- **double restart()**。返回当前时间与之前调用构造函数时的时间之差。        
### 静态全局函数
- **static inline double getTime()**。根据编译平台不同，采用不同的函数，返回当前时间，单位为s。                

## NoCopy
C++默认构造的类是具有拷贝构造函数的，但是有一些类在程序中不应该被拷贝，所以通过建立NoCopy类实现拷贝构造函数及=运算符重载的私有化。当需要定义不能被拷贝的类时，只需要继承NoCopy类即可以实现。

## macros
只定义了一个用于抑制编译器对未使用参数警告的宏。

## polygon
polygon中包含了几个用于保存二维多边形轮廓的类，构造了一个list容器保存point并定义了一个别名ListPolygon，构造了一个vector容器保存ListPolygon并定义别名为ListPolygons。

### ConstPolygonRef
构造了一个存储多边形轮廓的类，外轮廓定义为逆时针方向，内轮廓为顺时针方向，左边为x负方向，下为y负方向。            
#### 成员变量
定义了一个指向Path类型的*path指针，path类型定义在ClipperLib文件中，实际上是`std::vector< IntPoint >`的别名。        
#### 成员函数
- **构造函数**。传递一个const ClipperLib::Path类型的polygon地址；        
- **禁用==运算符**。多边形比较非常复杂，这里直接禁用了；        
- **禁用=运算符**。无法分配const对象；        
- **size_t size() const**。用于返回多边形顶点数；        
- **bool empty() const**。用于判断多边形是否为空。没有顶点时返回true，有顶点返回false；                            
- **const Point& operator[] (unsigned int index) const**。用于返回index索引点；                            
- **重载*运算符**。返回path的指针；          
- **ClipperLib::Path::const_iterator begin() const**。返回path向量头指针；            
- **ClipperLib::Path::const_iterator end() const**。返回path向量尾指针；            
- **ClipperLib::Path::const_reference back() const**。返回path向量尾元素的引用；            
- **const void* data() const**。返回指向path向量头元素的指针；          
- **bool orientation() const**。判断多边形方向，若为逆时针方向则返回true；          
