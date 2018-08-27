---
layout: post
title:  "有限元分析基础理论"
date:   2018-8-26 22:35:56
categories: jekyll update
---

这段时间由于项目原因去了企业教企业员工关于ANSYS的一些操作，虽然说如果只教软件的话，可能对于有限元理论并不是需要那么了解，但是知其然知其所以然，正好趁着这个机会巩固了一下有限元分析的一些基础。

### FEA的基本概念  
有限元分析（Finite Element Analysis, FEA）是现代工程计算中不可忽略的一部分，应用于汽车，飞机，机械部件，桥梁，建筑物等的变形和应力分析中。各类工程问题基本上都可以通过一些常微分方程或者偏微分方程进行求解，但是只有在某些特殊情况下，这些方程的解析解才能够求解得出，在更多的情况下，只能通过计算机采用一些数值计算方法得出足够精度的数值解。对于绝大多数工程问题来说，所需求解的问题是连续的，有少数情况下连续的介质是会自动离散化的，比如桁架结构。对于这类问题，它的几何变形或者是物理响应是可以通过结点位置的变化以及相关物理响应来进行精确求解的，数值解可以简单地用一组有限的数值表示。但是对于大多数常见问题来说，对于没有自动离散化的求解介质，是无法利用一组有限自由度的数值解来表达准确的，对于这类问题来说，数值解可以通过一组采样点和特定的插值方程来进行表示。从数学的角度来看，样本点以及插值方程的选择很多。但是，在工程应用中，有限元方法采用分段插值法，如下所示：  
- **离散化**。通过特定方法在需要求解区域选取一组采样点（节点，nodes），并通过这些节点将连续区域离散化为有限的，具有相同拓扑特征的小部分（单元，elements）。这个过程在ANSYS中对应的就是网格划分（mesh）。
- **选取插值方程**。选取一组多项式方程（形函数，shape function），通过形函数和每个单元的节点值的线性组合表达单元的数值解。
- **建立FEA方程**。当网格划分完成，通过物理关系建立一组具有所有节点值的代数方程，并将所求解问题的自由度作为未知量进行求解。
- **求解节点值**。在求解节点值的同时，自由度也会被求解。
- **得到近似的数值解**。问题的数值解可以通过计算得出的每个节点值以及形函数从而对每个单元的分段插值求解得出。    

通过一个例子来解释一下形函数所建立的插值方法。
![2D近似解](https://github.com/conceptclear/conceptclear.github.io/raw/master/images/FEA/example.png "Example")



### Reference
[1]Moaveni S. Finite Element Analysis:Theory and Applications with ANSYS:[M]. Prentice Hall, 2013.
[2]ANSYS Help

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
