---
layout: post
title:  "面片模型的曲率估计"
date:   2019-10-12 02:35:56
category: CG
---

在计算机图形学中，面片模型是最常用的表示三维模型的表达方式。从具有大量小面的多面近似中估计曲面的主曲率和主方向，例如由等值面构造算法生成的那些面，已成为许多计算机视觉算法中的基本步骤。Gabriel Taubin[1]在95年提出了一种计算多面体近似顶点处的曲面曲率的方法。通过计算对称矩阵的特征值和特征向量可以得出主曲率和主方向，而该对称矩阵是由积分公式定义并且与曲率张量的矩阵表达密切相关。

## 顶点的法向量
三角网格模型一般可以由一对线性表来表示，

$$
M=(V,F)
$$

其中，$V={v_i:1\le i \le n_v}$用于表示顶点集，$n_v$代表顶点数量，$F={f_k:1 \le k \le n_f}$用于表示三角面片集，$n_f$代表面片数量。对于任意一点$v_i$，所有包含$v_i$的三角面片集合记为$F^i$，若面片$f_k$属于$F^i$，记为$f_k \in F^i$，$f_k$的绝对值记为三角形面片的面积，包含点$v_i$的三角面片的面积之和记为$N(v_i)$。$F_i$中除了顶点$v_i$之外的其他顶点组成的集合记为$V^i$，如果顶点$v_i \in V^I$，则$v_j$是$v_i$的相邻点，$V^i$中顶点的个数称为顶点的度，用$N(i)$的绝对值来表示。                  

在离散三角网格面片模型中，任意顶点的法向量是一个非常重要的量，对顶点法向量的求解一般采用对该顶点所连接的三角形的某些几何量加权平均值得到。最简单的加权方法为$F_i$中所有三角面片的法向量的平均值：

$$
N_{v_i}=\frac{1}{N(i)} \sum_{f_k \in F^i}N_{f_k}
$$

Taubin给出的面积加权和为：

$$
N_{v_i}=\frac{\sum_{f_k \in F^i}|f_k|N_{f_k}}{||\sum_{f_k \in F^i}|f_k|N_{f_k}||}
$$

对于三角形网格上的任一点$v_i$，法曲率通常使用公式

$$
\kappa_{ij}=\frac{2N_{v_i}^T(v_j-v_i)}{||v_j-v_i||}
$$

具体证明详见文献[1]，这里不做赘述。

## Taubin主曲率估计方法
对于曲面$S$上的点$p$，设${T_1,T_2}$为其切平面上的一组正交基，设$T$为点$p$上的单位切向量，有$T=t_1T_1+t_2T_2$，这样可以表示出点p沿着T方向的法曲率为

$$
\kappa_p(T)= \left[
 \begin{matrix}
   t_1 & t_2
  \end{matrix}
  \right]\left[
   \begin{matrix}
     \kappa_p^{11} & \kappa_p^{12} \\
     \kappa_p^{21} & \kappa_p^{22}
    \end{matrix}
    \right]\left[
     \begin{matrix}
       t_1 \\ t_2
      \end{matrix}
      \right]
$$

其中，$\kappa_p^{11}=\kappa_p(T_1),\kappa_p^{22}=\kappa_p(T_2),\kappa_p^{12}=\kappa_p^{21}$。当$\kappa_p^{12}=\kappa_p^{21}=0$时，$\{T_1,T_2\}$为S点的两个主曲率。此时用$\kappa_p^1$和$\kappa_p^2$来代替$\kappa_p^{11}$和$\kappa_p^{22}$。             
若在正交基中加入法向量$N$，可以得到三个正交基$\{N,T_1,T_2\}$，这样p点的法曲率也可以表示为：

$$
\kappa_p(T)= \left[
 \begin{matrix}
   n & t_1 & t_2
  \end{matrix}
  \right]\left[
   \begin{matrix}
      0 & 0 & 0 \\
     0 & \kappa_p^{1} & 0 \\
     0 & 0 & \kappa_p^{2}
    \end{matrix}
    \right]\left[
     \begin{matrix}
       n \\ t_1 \\ t_2
      \end{matrix}
      \right]
$$

其中$T=nN+t_1T_1+t_2T_2$可以表示p点上的任意向量。将$t_1,t_2$用该向量与主方向$T_1,T_2$来表示，即

$$
T_{\theta}=\kappa_p^1cos^2\theta+\kappa_p^2sin^2\theta
$$

定义对称矩阵：

$$
M_p = \frac{1}{2\pi}\int_{-\pi}^{+\pi}\kappa_p(T_{\theta})T_{\theta}T_{\theta}^Td\theta
$$

由于$T_{\theta}T_{\theta}^T$的秩为1，因此法向量N为该矩阵特征值为0时所对应的特征向量。            
令$T_{12}=[T_1\ T_2]$，对称矩阵可以分解为

$$
M_p=T_{12}\left[
 \begin{matrix}
   m_p^{11} & m_p^{12} \\
   m_p^{21} & m_p^{22}
  \end{matrix}
  \right]T_{12}^T
$$

其中，

$$
m_p^{11}=T_1^TM_PT_1\\
m_p^{12}=m_p^{21}=T_1^TM_PT_2\\
m_p^{22}=T_2^TM_PT_2\\
$$

通过积分运算可以得到

$$
m_p^{11}=\frac{3}{8}\kappa_p^1+\frac{1}{8}\kappa_p^2 \\
m_p^{22}=\frac{1}{8}\kappa_p^1+\frac{3}{8}\kappa_p^2 \\
m_p^{12}=m_p^{21}=0
$$

转换可得

$$
\kappa_p^1=3m_p^{11}-m_p^{22}\\
\kappa_p^2=3m_p^{22}-m_p^{11}\\
$$

这样只要确定了$M_p$中的$\kappa_p(T_{\theta})$，就可以通过特征值和特征向量反求出主曲率和主方向。在离散三角面片模型中，可以利用上面提到的公式来计算：

$$
\kappa_{ij}=\frac{2N_{v_i}^T(v_j-v_i)}{||v_j-v_i||}
$$

## 算法实现
对于任意一个输入的三维离散三角网格模型，可以通过公式求解出顶点的法向量：

$$
N_{v_i}=\frac{\sum_{f_k \in F^i}|f_k|N_{f_k}}{||\sum_{f_k \in F^i}|f_k|N_{f_k}||}
$$

计算矩阵$M_p$的值，这里通过离散化近似的方法使用以下公式计算：

$$
M_{v_i}=\sum_{v_j \in V^i}\omega_{ij}\kappa_{ij}T_{ij}T_{ij}^T
$$

对于每个$v_i$的相邻点$v_j$，单位向量$T_{ij}$为向量$v_i-v_j$在i点的切平面上的投影：

$$
T_{ij}=\frac{(I-N_{v_i}N_{v_i}^T)(v_i-v_j)}{||(I-N_{v_i}N_{v_i}^T)(v_i-v_j)||}
$$

对法曲率$\kappa_{ij}$的求解，直接通过公式：

$$
\kappa_{ij}=\frac{2N_{v_i}^T(v_j-v_i)}{||v_j-v_i||}
$$

$\omega_{ij}$采用面积加权平均值，即与所有包含$v_i,v_j$的三角面片面积成正比，如果模型是封闭的，则有两个面片，即

$$
\omega_{ij}=\frac{|f_{k_i}+f_{k_j}|}{2\sum|f_k|}
$$

利用Householder变化，令

$$
E=\left[
 \begin{matrix}
   1 \\ 0 \\ 0
  \end{matrix}
  \right], \  W_{v_i}=\frac{E\pm N_{v_i}}{||E\pm N_{v_i}||}
$$

其中，当$E-N_{v_i}的量>E+N_{v_i}的量$时符号取负，反之取正。
得到Householder矩阵为：

$$
Q_{v_i}=I-2W_{v_i}W_{v_i}^T
$$

该矩阵是正交的切第一列为$N_{v_i}$或者$-N_{v_i}$(取决于之前的符号选择)，其余两列决定了一组标准正交基,定义这两列向量为$\widetilde{T_1},\widetilde{T_2}$。由于$N_{v_i}$是矩阵特征值为0的特征向量，所以有：

$$
Q_{v_i}^TM_{v_i}Q_{v_i}=\left[
 \begin{matrix}
    0 & 0 & 0 \\
   0 & \widetilde{m_{v_i}^{11}} & \widetilde{m_{v_i}^{12}} \\
   0 & \widetilde{m_{v_i}^{21}} & \widetilde{m_{v_i}^{22}}
  \end{matrix}
  \right]
$$

其中有，$\widetilde{m_{v_i}^{12}}=\widetilde{m_{v_i}^{21}}$。通过Givens变换可以得到：

$$
\widetilde{m_{v_i}^{11}}cos\theta+\widetilde{m_{v_i}^{21}}sin\theta=0\\
-\widetilde{m_{v_i}^{12}}sin\theta+\widetilde{m_{v_i}^{22}}cos\theta=0
$$

通过计算可以得到$cos\theta,sin\theta$，从而可以求出主方向：

$$
T_1=cos\theta \widetilde{T_1}-sin\theta \widetilde{T_2}\\
T_2=sin\theta \widetilde{T_1}+cos\theta \widetilde{T_2}
$$

计算出主方向后即可以求出主曲率的大小。

## Reference
[1] Taubin G . Estimating the tensor of curvature of a surface from a polyhedralapproximation[C]// Computer Vision, 1995. Proceedings. Fifth International Conference on. IEEE Computer Society, 1995.
