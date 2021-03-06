---
layout: post
title:  "手游方舟指令捞UR脚本"
date:   2018-10-07 10:35:56
category: MobileGame
---

前段时间方舟指令这个游戏全平台开服，就下载下来玩了一会儿，虽然感觉不是特别有意思，但是这个游戏在某些关卡刷图的时候有几率会掉ur，我玩游戏的时候又有收集欲，就一直用模拟器在电脑上来刷。游戏虽然提供了自动战斗功能，但是只能用于进入战斗之后，结束战斗以及进入战斗还需要自己去点感觉十分麻烦，而且由于一次战斗花的时间是不确定的（有攻击miss的存在），所以简单的计时脚本来按键不是特别的方便，但是总的来说这个游戏是一个一直重复的过程，只不过在时间上不是那么容易控制，正好发现按键精灵现在提供了可以检测颜色的方法，这里就提供一下我的思路。

## 方舟指令  
方舟指令这个游戏乱七八糟的玩法整了一大堆，核心玩法其实上还是刷图，主线关卡是点进每个关卡，杀掉一些特定的怪就可以通过该关卡。每个关卡完成一些条件可以解锁一些等级，最多是三星。当第一次将该关卡所有的怪都清掉通关之后，就可以解锁自动搜索敌人的功能，这个功能对自动刷图很重要，简化了脚本的工作量。      
一般来说，主线本进去开启自动刷图之后，关卡里面的怪分为普通小怪和boss怪，击败了boss怪这个关卡就结束了，而且如果要刷ur的话只有特定关卡的boss怪才会掉落，所以只需要尽可能快的去打boss怪就可以了，但是boss怪往往是需要清掉一些特定的小怪之后才会出现，自动搜索敌人这个功能是实现了这个尽快去打boss的功能的，所以刷图的话只需要进到这个关卡，打开自动搜索敌人功能，然后再在每个具体的战斗中将敌人击败就可以了。      
- **自动战斗功能**。游戏提供了自动战斗功能，虽然来说自动战斗的ai非常蠢，但是这个游戏很看等级，当你等级足够高的时候，随便打也可以过本，这种情况之下完全可以用自动战斗来节省自己的时间。这个刷ur的脚本也是建立在你的队伍等级已经足够自动战斗的情况下的。       
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/AutoFind.png"></div>         

- **协助战斗**。游戏在小怪和boss怪进入战斗的时候有一点不同，进入boss战斗的时候需要选择一个好友或者一个陌生人来协助战斗，不过这里与其说是类似于fgo那种直接用好友的servant来战斗，还不如说是给自己的队伍加上一个可有可无的buff，所以为了简化起见，这里直接选取第一个buff。    
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/HelpTeam.png"></div>         

- **卡牌掉落**。因为该游戏除了在boss怪的地方会掉落ur之外，在普通小怪的地方也有可能有卡牌掉落，只不过不会是ur，会掉落哪些卡牌在战斗开始的界面上可以点进去查看，如果掉落了一张你之前没有的卡牌，那么游戏会提醒你是否需要锁住这张卡牌，这种情况下和一般战斗结束是不一样的，简单的方法就是一直点确定的那个地方，不管结束的时候到底有没有出现新卡牌，复杂一点的话就要写一个判断脚本来执行，由于我基本上sr和r都出全了所以这里没有考虑这个问题。        
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/CardDrop.png"></div>         

## 安卓模拟器
安卓模拟器我使用的是网易的MuMu模拟器，由于之前用过蓝叠和腾讯等的模拟器来玩挂机牧羊人之心这个游戏的时候都有各种各样的问题，只有网易的这个显示上没有什么问题还不错，所以这次还是使用的网易的这个模拟器来玩。         
本身网易的这个MuMu模拟器是自带脚本软件的，但是不知道为啥我用的时候有问题，所以只用这个模拟器提供的键盘功能，具体的脚本交给按键精灵来实现。
模拟器提供了一个自定义键鼠方案，实际上就是通过键盘按键来模拟在屏幕上某个区域手指点击的功能，由于每次刷本需要按得键的位置是确定的，按键次数也是确定的，这里利用自定义键鼠方案的办法将鼠标点击模拟手指触碰改为键盘按键模拟手指触碰更为方便编写脚本。

## 按键精灵
按键精灵经过这么多年发展现在功能算是比较强大了，可以实现后台模拟按键功能和简单的图像识别功能（区域颜色识别），这样结合一下就可以编写简单的重复刷本的操作了。

- **窗口句柄**。在Windows中，句柄是一个系统内部数据结构的引用。例如当你操作一个窗口，或说是一个Delphi窗体时，系统会给你一个该窗口的句柄，系统会通知你：你正在操作142号窗口，就此你的应用程序就能要求系统对142号窗口进行操作——移动窗口、改变窗口大小、把窗口最小化等等。当获取了模拟器这个窗口的句柄之后，所有需要进行重复的操作都可以确保会在这个窗口程序上执行，这样就可以在脚本运行的时候将模拟器最小化去干别的事了。     
- **后台按键**。不添加句柄的按键只能实现特定的点击，这里采用的是像该句柄发送一个后台操作信息，比如模拟按了某个键，在这种情况之下就不需要将窗口显示在屏幕上。
- **颜色识别**。按键精灵现在可以实现判断某个确定点的颜色，区域找色，区域模糊找色，区域中心找色以及区域找图功能，这里实际上只需要用到确定某个确定点的颜色就行。由于只有在胜利界面的时候，会显示出来一小块战斗胜利的黄色，直接利用这个地方进行判断是否需要进行操作即可。

## 源码
```
Hwnd = Plugin.Window.MousePoint()
For 100
    Call Plugin.Bkgnd.KeyPress(Hwnd,67)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd,68)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd,68)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd,83)
    Delay 1000
    Delay 5000
    For 5
        Delay 4000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 70)
        Delay 1000
        Do
            Delay 1000
            GetColor = Plugin.Bkgnd.GetPixelColor(Hwnd, 170, 123)
            If GetColor = "0EE9FF" Then
                Exit Do
            End If
        Loop
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
        Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
        Delay 1000
    Next
    Delay 10000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 70)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 90)
    Do
        Delay 1000
        GetColor = Plugin.Bkgnd.GetPixelColor(Hwnd, 170, 123)
        If GetColor = "0EE9FF" Then
            Exit Do
        End If
    Loop
    Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
    Delay 1000
    Call Plugin.Bkgnd.KeyPress(Hwnd, 84)
    Delay 1000
Next
```

## Reference
[1]百度百科       
[2]http://zy.anjian.com/?action-study
