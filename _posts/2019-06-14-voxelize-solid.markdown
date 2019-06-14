---
layout: post
title:  "三维模型的体素化————实体体素化"
date:   2019-06-14 10:35:56
category: voxelization
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
未完待续。。。

## Reference
[1]https://zh.wikipedia.org/wiki/Flood_fill                     
[2]https://en.wikipedia.org/wiki/Breadth-first_search
[3]杨钦, 徐永安, 翟红英. 计算机图形学[M]. 清华大学出版社有限公司, 2005.
