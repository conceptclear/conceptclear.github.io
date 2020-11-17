---
layout: post
title:  "三维空间中根据线段变换前后坐标求解变换矩阵"
date:   2019-12-04 02:35:56
category: CG
---

对于三维空间中的一条线段$P_1P_2$如下图所示，若已知变换前后的线段端点坐标$P_1(x_1,y_1),P_2(x_2,y_2)$以及$P_1'(x_1',y_1'),P_2'(x_1',y_1')$，如何求解该线段的三维变换矩阵？本文针对该问题进行分析。线段的空间三维变换可以主要分解为线段所构成向量的旋转，顶点的平移，以及大小的变换三个变换矩阵，这里分开讨论。

<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/CG/line_transformation.png"></div>   

## 旋转矩阵
不考虑线段的绕着自身的自旋，求解线段的旋转矩阵可以简化为所构成向量的旋转。已知旋转前的向量为$\boldsymbol {P_1P_2}$，旋转后的向量为$\boldsymbol {P_1'P_2'}$，可以计算出向量之间的夹角为：

$$
\theta = arccos \left( \frac{\boldsymbol{P_1P_2} \cdot \boldsymbol{P_1'P_2'}}{\|\boldsymbol {P_1P_2}\|\|\boldsymbol {P_1'P_2'}\|} \right)
$$

为了方便表示，令

$$
\boldsymbol{P_1P_2}=\left[
 \begin{matrix}
   x_2-x_1 \\ y_2-y_1 \\ z_2-z_1
  \end{matrix}
  \right]=\left[
   \begin{matrix}
     a_1 \\ a_2 \\ a_3
    \end{matrix}
    \right] \qquad
    \boldsymbol{P_1'P_2'}=\left[
     \begin{matrix}
       x_2'-x_1' \\ y_2'-y_1' \\ z_2'-z_1'
      \end{matrix}
      \right]=\left[
       \begin{matrix}
         b_1 \\ b_2 \\ b_3
        \end{matrix}  
        \right]
$$  

则夹角$\theta$可以表示为：

$$
\theta = arccos \left( \frac{a_1b_1+a_2b_2+a_3b_3}{\sqrt{a_1^2+a_2^2+a_3^2}\cdot\sqrt{b_1^2+b_2^2+b_3^2}} \right)
$$

根据两个向量可以求出向量旋转的旋转轴，旋转轴垂直于两个向量所构成的平面，利用向量的叉乘可以计算出旋转轴$\boldsymbol{k}$的向量：

$$
\boldsymbol{k}=\boldsymbol{P_1P_2} \times \boldsymbol{P_1'P_2'}=\left[
 \begin{matrix}
   a_2b_3-a_3b_2 \\ a_3b_1-a_1b_3 \\ a_1b_2-a_2b_1
  \end{matrix}
  \right]
$$

得到了向量的旋转角和旋转轴之后，旋转轴$\boldsymbol{k}$和原始向量$\boldsymbol{P_1P_2}$的外积可以表示成矩阵积的形式：

$$
\boldsymbol{k} \times \boldsymbol{P_1P_2}=\left[
 \begin{matrix}
   k_ya_3-k_za_2 \\ k_za_1-k_xa_3 \\ k_xa_2-k_ya_1
  \end{matrix}
  \right]=\left[
   \begin{matrix}
     0  &  -k_z  &  k_y\\ k_z  &  0  &  -k_x \\ -k_y  &  k_x  &  0
    \end{matrix}
    \right]\left[
     \begin{matrix}
       a_1 \\ a_2 \\ a_3
      \end{matrix}
      \right]
$$

用矩阵$\boldsymbol{K}$来表示向量$\boldsymbol{k}$的叉乘矩阵，即

$$
\boldsymbol{K}=\left[
 \begin{matrix}
   0  &  -k_z  &  k_y\\ k_z  &  0  &  -k_x \\ -k_y  &  k_x  &  0
  \end{matrix}
  \right]
$$

可以利用罗德里格旋转公式来得到旋转矩阵，旋转矩阵$\boldsymbol{R}$表示如下：

$$
\boldsymbol{R}=\boldsymbol{E}+sin\theta\boldsymbol{K}+(1-cos\theta)\boldsymbol{K^2}
$$

公式中的$\boldsymbol{K}$需要归一化，即

$$
{\|\boldsymbol{K}\|}_2=1
$$

为了和后面的平移缩放矩阵保持一致性，这里将三维旋转矩阵拓展至四维，即

$$
{\boldsymbol{R}}_{4 \times 4}=\left[
 \begin{matrix}
   {\boldsymbol{R}}_{3 \times 3}  &  {\boldsymbol{0}}_{1 \times 3}\\ {\boldsymbol{0}}_{3 \times 1}  &  1
  \end{matrix}
  \right]
$$

## 比例变换矩阵
经过旋转之后，由于之前做过了归一化处理，这里比例变换因子即为$\|\boldsymbol {P_1'P_2'}\|$，直接通过比例变换矩阵${\boldsymbol{S}}$进行整体放缩即可：

$$
{\boldsymbol{S}}_{4 \times 4}=\left[
 \begin{matrix}
   \|\boldsymbol {P_1'P_2'}\|  &  0  &  0  &  0  \\ 0  &  \|\boldsymbol {P_1'P_2'}\|  &  0  &  0 \\ 0  &  0  &  \|\boldsymbol {P_1'P_2'}\|  &  0  \\ 0  &  0  &  0  &  1
  \end{matrix}
  \right]
$$

## 平移矩阵
已知起点$P_1$和终点$P_1'$，平移矩阵${\boldsymbol{T}}$很容易表示：

$$
{\boldsymbol{T}}_{4 \times 4}=\left[
 \begin{matrix}
   1  &  0  &  0  &  0  \\ 0  &  1  &  0  &  0 \\ 0  &  0  &  1  &  0  \\  x_1'-x_1  &  y_1'-y_1  &  z_1'-z_1  &  1
  \end{matrix}
  \right]
$$

## 示例
针对于三维空间中的回转体，若已知回转体旋转轴上任意不相同两点空间变换前后的坐标，即可以利用该方法对整个回转体进行空间变换。如图所示：                  
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/CG/clinder1.png"></div>   

已知圆柱体的中心轴的两个端点位置，该圆柱体的方向为(0,0,1)，若想旋转至(1,1,1)，且保证底部圆的圆心位置处于原点处不变，则可以利用该方法进行三维变换，变换后结果如图所示：
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/CG/clinder2.png"></div>   

matlab的源码如下：
````
v1=[0;0;1];
v2=[1;1;1];
theta = acos(dot(v1,v2)/(norm(v1)*norm(v2)));
k=cross(v1,v2);
K=[0 -k(3) k(2);k(3) 0 -k(1);-k(2) k(1) 0];
K=K/norm(K);
R=eye(3)+sin(theta)*K+(1-cos(theta))*(K*K);

[x,y,z]=cylinder;%创建以(0,0)为圆心，高度为[0,1]，半径为R的圆柱
v3=[x(1,:);y(1,:);z(1,:)];
m=R*v3;
v4=[x(2,:);y(2,:);z(2,:)];
n=R*v4;
x1=[m(1,:);n(1,:)];
y1=[m(2,:);n(2,:)];
z1=[m(3,:);n(3,:)];
%surf(x,y,z)%重新绘图
surf(x1,y1,z1)%重新绘图
````

## Reference
[1] https://en.wikipedia.org/wiki/Rodrigues%27_rotation_formula                         
[2] https://www.cnblogs.com/xpvincent/archive/2013/02/15/2912836.html                           
[3] https://blog.csdn.net/piaoxuezhong/article/details/70171525                       
