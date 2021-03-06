---
layout: post
title:  "三维模型的体素化————实体体素化"
date:   2019-06-14 10:35:56
category: CG
---

表面体素化主要是对三维模型表面进行体素化，而实体体素化的目标则是在离散空间中建立一个封闭且水密的体素模型，并且模型内部的体素需要全部填充。对于实体体素化，主要分为以下几种方法：

## 漫水填充法
模型经过表面体素化之后一般可以得到一个水密的表面体素模型，通过漫水填充算法，可以比较轻易地得到所需要的实体体素模型。实现方法主要有基于深度优先搜索的迭代递归的方法和基于广度优先搜索的循环两种，其中最简单直观的漫水填充算法是采用深度优先搜索通过栈的递归方式来实现的，6邻域的实现方法如下：        
```
void FloodFill(int x, int y, int z)
{
    //判断该位置体素是否在表面体素上或者不在体素空间中，若是则返回，若不是则填充
    if(CheckVoxel(x,y,z))
        return;
    //根据情况将该体素填充为内部或外部
    SetVoxel(x,y,z);
    FloodFill(x-1,y,z)；
    FloodFill(x+1,y,z)；
    FloodFill(x,y-1,z)；
    FloodFill(x,y+1,z)；
    FloodFill(x,y,z-1)；
    FloodFill(x,y,z+1)；
}
```
具体如图所示：                         
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/voxelization/Recursive_Flood_Fill_4.gif"></div>     

这种方法需要逐点进行搜索，效率比较低，而且由于函数递归需要压栈，系统资源是有限的,递归层次一旦多起来就很容易堆栈溢出。为了解决这个问题可以采用广度优先搜索的方法将迭代转换为循环通过队列来实现，6邻域的实现方法如下：
```
void FloodFill_bfs(int x, int y, int z)
{
    unsigned int nummax=1<<(max_height-1);
    nummax--;
    //判断该位置体素是否在表面体素上或者不在体素空间中，若是则返回
    if(CheckVoxel(x,y,z))
        return;
    queue<Voxel> Q;
    //将起始体素点压入队列
    Q.push(Voxel(x,y,z));
    //根据情况将该体素填充为内部或外部
    SetVoxel(x,y,z);
    //对6邻域进行填充
    int x[6]={-1,1,0,0,0,0};
    int y[6]={0,0,-1,1,0,0};
    int z[6]={0,0,0,0,-1,1};
    while(!Q.empty())
    {
        Voxel qpoint = Q.front();
        Q.pop();
        SetVoxel(qpoint);
        //对6邻域的位置首先判断是否在体素空间中
        bool check[6]={qpoint.x==0,qpoint.x==nummax,qpoint.y==0,qpoint.y==nummax,qpoint.z==0,qpoint.z==nummax};

        //分别对6邻域的位置点进行判断填充
        for(int i=0;i<6;i++)
        {
            if(check[i])
                continue;
            Voxel mpoint(qpoint.x+x[i],qpoint.y+y[i],qpoint.z+z[i]);
            if(CheckVoxel(mpoint))
                continue;

            SetVoxel(mpoint);
            Q.push(mpoint);
        }
    }
    return;
}
```
利用BFS可以将迭代转化为循环，可以解决堆栈溢出的问题。然而，漫水填充算法在易实现的同时存在着较多的缺陷，主要缺陷有如下几点：           

- 漫水填充算法需要一个起始节点。在表面体素化完成之后，体素空间中的体素被分割成为了两个部分：处于模型上的体素和不在模型上的体素。由于实体体素化需要区分出模型内部和外部的体素，利用漫水填充算法不管是从内部进行填充还是从外部进行填充，都需要首先寻找到一个起始的填充节点，由于体素空间大小相当于模型的AABB包围盒，空间边缘的点只能在模型的表面上或者在模型的外部，这样通过循环判断的方式可以比较容易的寻找到外部空间的起始填充点，但是同样存在某些特殊情况，外部空间被表面体素分割成了几个部分，比如体素化的是一个空心的正方体，在这种情况下利用上述办法则无法在边缘处寻找到空间外部的起始点。           

- 漫水填充可能存在未填充区域或过填充区域。由于在表面体素化的过程中可能会形成表面局部的漏洞或孤立体素，在填充过程中采用6邻域体素填充则可能产生为填充区域，而采用26邻域填充则可能产生过填充的现象。这就对表面体素化结果的6-separability和6-minimality或者是26-separability和26-minimality提出了要求。

- 算法效率较低，资源消耗量大，易导致堆栈溢出，当然可以采用扫描线的方法来进一步优化算法，但是本质上还是漫水填充。

## 边界标志算法
边界标志算法，在有些地方也叫边缘填充算法，在二值二维图像中是求出多边形的每一条边与扫描线的交点，对交点右侧的所有像素颜色全部取反，按照顺序完成多边形所有边之后，就完成了多边形内部的填充。其原理是一个像素的值被求反两次之后就会恢复原值，即任意像素被求偶数次反之后保持原值，被求奇数次后取反。对于任意一个封闭的图形，如果点在图形内部，通过这个点引出一条射线，这条射线与原图形的边的相交次数必为奇数次（若在顶点上，则认为与两条边都相交），实现如下图所示：                        
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/voxelization/edge_fill_2d.png"></div>     
将这个二维图像填充中使用的算法拓展到三维空间中，即首先对面片模型的每个三角面片都进行遍历，对于每一个三角面片，首先求解其在yoz平面上投影面的包围盒，并且根据所得到的包围盒计算得出所覆盖的体素中心范围。若所得到的体素中心范围不为空，则遍历所得到的所有体素列。对于每个位于包围盒中的体素中心首先对其和三角面片在yoz平面上的投影进行检测，判断其是否在投影三角形内部，可以采用碰撞检测中常用的分离轴（SAT)算法或是Schwarz[4]提出的改进算法来进行检测。这里可以采用一个简单的方法来判断，对于体素中心点P和投影三角形ABC，若点P在三角形ABC内部，有：                      
- PA×PB，PB×PC，PC×PA所得值必为同号；                                
- 若存在符号相反则点P必在三角形外部；                             
- 若PA×PB=0，则点P在AB连线上，其中若PA=k*PB，k>0时，P在AB线段的延长线上，k<0时，P在AB线段上，k=0时，P与点A重合，以此类推；                

实现方法如下：
```
int check_point_triangle(vec2 v0, vec2 v1, vec2 v2, vec2 point)
{
	 vec2 PA = point - v0;
	 vec2 PB = point - v1;
	 vec2 PC = point - v2;

   //由于浮点数不是精确数，有可能不能正好达到0，所以不用等于0来进行判断
	 float t1 = PA.x*PB.y - PA.y*PB.x;
	 if (fabs(t1) < float_error && PA.x*PB.x <= 0 && PA.y*PB.y <= 0)
		  return 1;//点在线段AB上

	 float t2 = PB.x*PC.y - PB.y*PC.x;
	 if (fabs(t2) < float_error && PB.x*PC.x <= 0 && PB.y*PC.y <= 0)
		  return 2;//点在线段BC上

	 float t3 = PC.x*PA.y - PC.y*PA.x;
	 if (fabs(t3) < float_error && PC.x*PA.x <= 0 && PC.y*PA.y <= 0)
		  return 3;//点在线段CA上

	 if (t1*t2 > 0 && t1*t3 > 0)
		  return 0;//点在三角形内部
	 else
		  return -1;//点在三角形外部
}
```
若体素中心点在三角形内部，将该体素中心沿着x轴进行投影到三角面片上，对该投影点在体素空间中的映射体素q到原检测体素之间的所有体素值全部求反。遍历完所有面片之后，则完成了三维模型的实体体素化。                      
与二维像素空间的填充相似，边界标志算法在边界位置上存在着一定的问题。在二维图形中，若凸多边形的顶点正好位于像素中心上，其所连接的边数必为偶数，这样映射到像素空间中时，顶点像素总会进行两次翻转，这样该顶点最后都会产生填充错误。从二维映射到三维时，若存在体素的中心经过某个面片的边，这样会导致该体素所对应的体素列重复填充，引起整条体素列的填充错误。而且若体素中心在某面片的顶点之上时，由于顶点连接的三角面片数是不确定的，这样会导致顶点所投影的体素列翻转也出现问题。                     
针对于边重复所对应的体素列重复翻转的问题，可以采用光栅化中的左上填充规则（top-left rule），对于每个三角面片的边，只对其左边和上边对应的体素列进行翻转。左上填充规则实现起来其实比较容易，由于三角形顶点方向是一定的，可以通过对每条边两个顶点的高低来判断是不是左边或上边。对于逆时针排列的三个顶点，左边的边的第一个顶点一定比第二个顶点高，如果是上边的话则第一个顶点和第二个顶点一样高，但是第一个顶点一定在第二个顶点的右边。实现方法如下：
```
bool TopLeftEdge(vec2 v0, vec2 v1)
{
    return ((v1.y<v0.y)||(v1.y==v0.y&&v0.x>v1.x));
}
```
检测三角形的三个顶点顺时针或逆时针实现起来也很容易，可以利用矢量叉乘来判断。                  
对于三角形ABC,若AB*AC>0，则三角形ABC是逆时针的；                         
对于三角形ABC,若AB*AC<0，则三角形ABC是顺时针的；                         
对于三角形ABC,若AB*AC=0，则三点共线。                         
实现方法如下：
```
bool checkCCW(vec2 v0, vec2 v1, vec2 v2)
{
	 vec2 e0 = v1 - v0;
	 vec2 e1 = v2 - v0;
	 float result = e0.x*e1.y - e1.x*e0.y;
	  if (result > 0)
		  return true;
	  else
		  return false;//共线的情况在之前排除掉
}
```



## Reference
[1]https://zh.wikipedia.org/wiki/Flood_fill                     
[2]https://en.wikipedia.org/wiki/Breadth-first_search                     
[3]杨钦, 徐永安, 翟红英. 计算机图形学[M]. 清华大学出版社有限公司, 2005.                
[4]Schwarz M, Seidel H P. Fast parallel surface and solid voxelization on GPUs[J]. ACM Transactions on Graphics (TOG), 2010, 29(6): 179.                    
