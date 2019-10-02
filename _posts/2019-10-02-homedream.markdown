---
layout: post
title:  "手游家国梦网易mumu模拟器挂机收火车及收钱脚本"
date:   2019-10-02 02:35:56
category: MobileGame
---

最近腾讯和人民日报出了个手游，虽然游戏本质上是放置类手游，但是因为游戏无法氪金，玩了几天感觉如果光靠人工的话太肝了，针对于游戏里比较肝的两个部分分别编写了一个脚本。           

## 游戏机制                 
游戏是通过抽卡来收集建筑，建筑经过抽卡升星和金币升级之后可以加速金币的获取速度。但是随着游戏的进行，光靠建筑的等级升级来提升金币获取速度会越来越慢，所以同时需要提升建筑的星级。                
游戏中建筑的星级提升是通过抽卡来实现的，抽卡即购买商店中的**福气红包**，**多福红包**，**满福红包**这三类红包，如图所示：
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/envelope.png"></div>               
其中，
- **福气红包**。抽卡耗费蓝星币20（抽卡用货币），开启福气红包必得1份建筑卡牌，并有几率额外获得蓝星币。额外获得蓝星币的概率为30%，各类建筑的出率为：普通90%，稀有9%，史诗1%；                  
- **多福红包**。抽卡耗费蓝星币100，开启多福红包必得2份建筑卡牌，并必然额外获得蓝星币。各类建筑的出率为：普通45%，稀有50%，史诗5%；                  
- **满福红包**。抽卡耗费蓝星币500，开启满福红包必得3份建筑卡牌，并必然额外获得蓝星币与金币。各类建筑的出率为：稀有53%，史诗47%。                  

游戏中可以通过升级建筑来获得蓝星币和福气红包，但是只会在25级的奇数倍时获得少量的蓝星币，在100级的倍数时获得一个福气红包。除了升级以及游戏的各种礼包之外，唯一获得蓝星币的方法就是通过收集火车上的供货。

## 供货
游戏提供了一个火车供货来获取金币，蓝星币以及贡献值的方法，方法即为火车停下之后火车上会随机刷出一定的物资，通过将特定的物资移动到所对应的建筑上，既可以获得一定的奖励，如图所示：                
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/train.png"></div>     
对于不同类型的建筑，所获得的奖励实际上是不同的，其中，
- **普通建筑**。有几率获得1个福气红包，或者1张相片，或者少量蓝星币，或者少量金币；                     
- **稀有建筑**。有几率获得1个福气红包，或者1张相片，或者一定蓝星币，或者一定金币；                     
- **史诗建筑**。有几率获得2个福气红包，或者1个多福红包，或者2张相片，或者适当蓝星币，或者适当金币；                     

其中，获得的物资数量是收到游戏内供货加成这个数值影响的，但是游戏采用的是截尾取整的算法，比如供货加成为90%时，普通建筑获得的福气红包一次最多只能为1个，史诗建筑获得的福气红包最多为3个。火车送货的次数只和最终成功移动的次数有关，所以可以通过遇到普通建筑或者稀有建筑的货不送，只送史诗建筑的货这样达到获益的最大值。

## 火车只送史诗建筑脚本
虽然只送史诗建筑可以达到最大收益，但是会使得收货时间大大加长，如果人工来进行收货的话实在太肝，这里提供一种基于按键精灵脚本的火车自动送货方法。                       
游戏中一共有6个史诗建筑，但是为了将供货加成拉满，只能上5个史诗建筑，其他的将有供货加成的建筑放满即可。由于游戏中送货送去不对应的建筑上时，并不会被接收，所以可以采用将3截车厢分别只往5个史诗建筑上送货，这样可以保证史诗建筑对应的货肯定会被接收，而普通和稀有建筑对应的货物不会被接收。送货时，建筑收获成功的范围很大，所以实际的操作中不需要每个车厢都送满5个建筑，只需要往两个点上送货即可，如图所示：                  
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/pushpoint.png"></div>             
对于史诗建筑所对应的货物，一般上每个车厢只会携带一个，偶尔会携带两个，所以只需要让每个车厢都往这两个点送货两次即可，即每次火车来了之后进行12次鼠标拖动的操作。利用网易mumu模拟器自带的键鼠功能，用键盘按键来模拟鼠标操作，分别用ABCDEF按键记录下每次鼠标滑动的操作，如图所示：                    
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/mousesimulate.png"></div>         
其中由于游戏有时候火车来了并没有刷新出货物，可以通过采集一下金币来实现火舞的刷新，所以设置了一个按键1实现金币收集。                 
因为火车有火车之间的时间间隔是不确定的，这里利用颜色识别的方法来判定火车是否到站，具体代码如下：
```
Hwnd = Plugin.Window.MousePoint()
Do
    Call Plugin.Bkgnd.KeyPress(Hwnd, 49)
    Delay 10000
    GetColor = Plugin.Bkgnd.GetPixelColor(Hwnd, 248, 874)
    If GetColor = "928E44" Then
    Delay 5000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 49)
    Delay 2000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 65)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 65)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 66)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 66)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 67)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 67)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 68)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 68)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 69)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 69)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 70)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 70)
    Delay 1000   
    Do
    	Delay 500
    	GetColor = Plugin.Bkgnd.GetPixelColor(Hwnd, 248, 874)
    	    If GetColor <> "928E44" Then
    	    Exit Do
    	    End If
    Loop
    End If
Loop

```
实现结果如图所示：
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/trainexample.gif"></div>                           

## 自动收集金币升级
相对于收货，这个脚本要简单的多，只需要事先录制好几组收集金币的鼠标滑动路径，为了防止检测出利用脚本，这里使用了一个随机函数，每次随机调用一个路径来收集金币，代码如下：
```
Hwnd = Plugin.Window.MousePoint()
For 100
	Call Plugin.Bkgnd.KeyPress(Hwnd,54)
    Delay 1000
	Call Plugin.Bkgnd.KeyPress(Hwnd,83)
    Delay 1000
	Call Plugin.Bkgnd.KeyPress(Hwnd,65)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd,85)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd,83)
    Delay 1000
    For 3
    Randomize //初始化随机数生成器
    d=int(Rnd()*4+1)//d的值为随机数1-5
    Select Case d
    	Case 1
    	Call Plugin.Bkgnd.KeyPress(Hwnd,49)
    	Delay 60000
    	Case 2
    	Call Plugin.Bkgnd.KeyPress(Hwnd,50)
    	Delay 60000
    	Case 3
    	Call Plugin.Bkgnd.KeyPress(Hwnd,51)
    	Delay 60000
    	Case 4
    	Call Plugin.Bkgnd.KeyPress(Hwnd,52)
    	Delay 60000
    	Case 5
    	Call Plugin.Bkgnd.KeyPress(Hwnd,53)
    	Delay 60000
    End Select
	Next
Next
```
模拟按键设置如图：
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/selectcoin.png"></div>                 

## Reference
[1]http://nga.178.com/thread.php?fid=678       
[2]http://zy.anjian.com/?action-study
