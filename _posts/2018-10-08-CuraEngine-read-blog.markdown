---
layout: post
title:  "CuraEngine开源程序解读1"
date:   2018-10-08 10:35:56
category: 3dprint
---

CuraEngine这个开源程序用到的东西太多，想要整理出来不是特别容易，这里先按照我读程序的顺序来进行程序解读，如果之后有时间可以从新整理一下。

## intpoint
intpoint这个头文件主要包含了curaengine用来尽可能快的使用的整数点类，使用整数点主要是为了避免浮点运算中的舍入误差，并且由于需要用到clipperlib，而在ClipperLib中也是采用的整数点类来进行运算的，为了保持一致性，从而建立了这样一个整数点类。       
intpoint类中调用了clipperlib库主要是为了重用clipperlib库中提供的IntPoint的定义，在ClipperLib中，IntPoint是一个用来代表Clipper库中相关点的整形点数据结构，定义如下：
```
struct IntPoint {
  cInt X;
  cInt Y;
#ifdef use_xyz
  cInt Z;
  IntPoint(cInt x = 0, cInt y = 0, cInt z = 0): X(x), Y(y), Z(z) {};
#else
  IntPoint(cInt x = 0, cInt y = 0): X(x), Y(y) {};
#endif

  friend inline bool operator== (const IntPoint& a, const IntPoint& b)
  {
    return a.X == b.X && a.Y == b.Y;
  }
  friend inline bool operator!= (const IntPoint& a, const IntPoint& b)
  {
    return a.X != b.X  || a.Y != b.Y;
  }
};
```
在6.0版本以上，IntPoint可以拥有Z坐标，但是在intpoint类中并没有去使用这个坐标。      
在intpoint类中宏定义了几个函数，用来做数据类型的转换，由于整点类中的成员变量以微米为单位，所以宏定义的函数主要用来做整数的微米单位和毫米单位之间的转换。      
### Point3类
#### 成员变量
存储三维的常量整数点坐标，int32_t x,y,z。               
#### 成员函数
- **运算符重载**。重载了几个常用的运算符，包括和Point3类的+,-,+=,-=,==,!=、和int32_t类型数据的*/以及和double类型的*；          
- **int32_t max() const**。用于检测坐标值的最大值；          
- **bool testLength(int32_t len) const**。用于检测坐标点距离原点长度是否满足要求；        
- **int64_t vSize2() const**。用于返回三维整数点距离原点距离的平方；          
- **int32_t vSize() const**。用于返回三维整数点距离原点的距离；          
- **double vSizeMM() const**。用于返回三维点距离原点的距离（mm）；          
- **int64_t dot(const Point3& p) const**。用于返回两个三维向量的点积；          
- **<span style="color:red;">~~Point3 cross(const Point3& p)~~</span>**。用于返回两个三维向量的叉积，但是由于可能会造成溢出（当向量长度超过4.6cm时），所以不建议使用这个函数。          
#### 全局变量
定义了一个简单占位符坐标点（brief Placeholder coordinate point (3D)），作用不明。该坐标点是由三个int32_t类型所能提供的最小值构成的点。           
### IntPoint类
####成员变量
存储二维坐标变量，int X, Y。          
####成员函数
- **Point p()**。用于返回Point类型的x，y坐标；          

### Point类
对clipper中的IntPoint类定义了一个别名，即为这里的Point类。
####成员函数
- **运算符重载**。重载了几个常用的运算符，包括和Point类的+,-,+=,-=,==,!=、和int32_t类型数据的*/以及和任意类型的*,\*=,/=；          
####相关内联函数
- **Point normal(const Point& p0, int64_t len)**。用于返回len长度的Point向量；          
- **bool shorterThen(const Point& p0, int32_t len)**。用于检测坐标点距离原点长度是否满足要求；        
- **int64_t vSize2(const Point& p0)**。用于返回二维整数点距离原点距离的平方；          
- **float vSize2f(const Point& p0)**。用于返回二维浮点距离原点距离的平方；          
- **int64_t vSize(const Point& p0)**。用于返回二维整数点距离原点的距离；          
- **double vSizeMM(const Point& p0)**。用于返回二维点距离原点的距离（mm）；          
- **int64_t dot(const Point& p0, const Point& p1)**。用于返回两个二维向量的点积；          
- **Point turn90CCW(const Point& p0)**。用于返回p0向量逆时针旋转90°；          
- **Point rotate(const Point& p0, double angle)**。用于返回p0向量逆时针旋转angle(弧度)；          
- **int angle(const Point& p)**。用于返回p0向量与x轴的夹角（角度）；          

### PointMatrix类           
PointMatrix类是用来对二维点进行旋转变换的矩阵类。              
####成员变量           
定义了double类型的大小为4的一维数组。          
####成员函数
- **构造函数**。不带参数的构造函数默认matrix数组为{1,0,0,1}；可以用弧度制和Point类型的向量值来构造变换矩阵。
- **Point apply(const Point p) const**。Point p逆时针旋转。                 
- **Point unapply(const Point p) const**。Point p顺时针旋转。                 

### Point3Matrix类        
Point3Matrix类是用来对三维点进行旋转变换的矩阵类。              
####成员变量         
定义了double类型的大小为9的一维数组。           
####成员函数          
- **构造函数**。不带参数的构造函数默认matrix数组为{1,0,0,0,1,0,0,0,1}；可以用PointMatrix类型的向量值来构造缺少第三个方向值的变换矩阵。
- **Point3 apply(const Point3 p) const**。Point3 p逆时针旋转。                 
- **Point apply(const Point p) const**。Point p逆时针旋转（只在xy平面投影）。           
- **static Point3Matrix translate(const Point p)**。构造平移Point p向量的矩阵。           
- **Point3Matrix compose(const Point3Matrix& b)**。实现矩阵相乘。           
