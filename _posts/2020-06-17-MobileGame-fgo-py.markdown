---
layout: post
title:  "手游fgo网易mumu模拟器自动挂机py脚本"
date:   2020-06-17 02:35:56
category: MobileGame
---

之前曾经写过一篇利用按键精灵来进行fgo挂机的脚本[手游fgo网易mumu模拟器自动挂机刷本脚本](http://conceptclear.cn/mobilegame/2020/02/18/MobileGame-fgo.html)，但是按键精灵本身存在很多限制（比如图像识别算法的问题，不能后台识图的问题等）而且运行起来CPU占用率也不低，这里提出一种利用python脚本来实现的方法。

## 句柄获取
利用win32gui可以比较简单的获取模拟器的句柄，要实现后台操作的话首先要获取模拟器窗体句柄的子句柄，对该子句柄发送消息即可以模拟后台按键。

```
def get_child_windows(parent):
    """get child handle"""
    if not parent:
        return None
    hwndChildList = []
    win32gui.EnumChildWindows(parent, lambda hwnd, param: param.append(hwnd), hwndChildList)
    return hwndChildList


def get_handle(name):
    """get handle of execute program"""
    hwnd = win32gui.FindWindow(None, name)
    hwnd1 = get_child_windows(hwnd)
    hwnd = hwnd1[0]
    return hwnd
```

## 后台操作
利用win32api.PostMessage可以给模拟器发送后台消息，用以模拟鼠标或者键盘操作，fgo大多数情况下按键位置都是确定的，可以通过模拟器的键鼠操作来固定按键位置，从而简化程序，我的设置如下所示：

![key_location](https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/mumu_settings.bmp "key_location")

## 图像模板匹配
fgo刷本里面必须进行图像识别的部分只有是否要吃苹果补充体力和选择助战上，这两个部分都可以利用模板匹配的方式来进行判定。事先存储好分辨率确定的所需识别部分的图形，之后利用opencv库的cv.matchTemplate函数来进行判断是否存在这块图片以及这块图片在模拟器中所处的位置。由于opencv库自带resize的函数，所以对于模拟器窗口的大小没有限制，若利用按键精灵来实现则必须保证每次模拟器窗口大小一致。模板匹配中归一化后主要有三种方法，分别是平方差匹配，相关性匹配，相关性系数匹配，其中平方差匹配有时候会出现匹配错误的情况，这里采用的是相关性系数的匹配方法，匹配速度相对于平方差来说会慢一些，但是这种小图片速度是足够的。

```
def template_matching(handle, width, height, pic, resolution):
    """check the pic whether existing or not"""
    tpl = cv.imread(pic)
    target = cv.imread("img_check.bmp")
    target = cv.resize(target, resolution)

    result = cv.matchTemplate(target, tpl, cv.TM_CCOEFF_NORMED)
    min_val, max_val, min_loc, max_loc = cv.minMaxLoc(result)

    return [min_val, max_val, min_loc, max_loc]
```

## 自动补充体力
自动吃苹果只需要在每次进本的时候识别一下是否需要吃苹果即可，即点击了该本之后是否弹出补充体力这一窗口。为了保证程序的稳定性，这里并不是认为没有弹出该窗口就是不需要补充体力，需要再对后续界面进行一次识别，若失败证明脚本出现问题，需要停止继续操作。

```
def check_apple(handle, width, height):
    """check whether eat apple"""
    basic_function.press_keyboard(handle, button_dict['M'], 2)
    print('Checking whether to eat apple')
    basic_function.get_bitmap(handle, width, height)
    [min_val1, max_val1, min_loc1, max_loc1] = basic_function.template_matching(handle, width, height, 'apple.bmp',
                                                                                (821, 462))
    [min_val2, max_val2, min_loc2, max_loc2] = basic_function.template_matching(handle, width, height, 'assist.bmp',
                                                                                (821, 462))
    if max_val1 > 0.95:
        print("eat apple")
        basic_function.press_keyboard(handle, button_dict['T'], 1)
        basic_function.press_keyboard(handle, button_dict['H'], 2)
    elif max_val2 > 0.95:
        print("don't need to eat apple")
        return None
    else:
        print("error occurred")
        sys.exit()

    return None
```

## 自动选取助战
fgo助战在很多情况下无法保证第一个角色就是我想要借的角色，利用按键精灵的简单图像识别不仅速度慢，而且操作起来比较麻烦。之前文章中提到的方法是只检测第一个助战位，若不是则刷新，这样大幅度的降低了程序运行的效率，这里利用python实现了识别所需角色位置的方法。在mumu模拟器中，不仅可以用键盘按键模拟鼠标点击的操作，还可以模拟鼠标拖动的操作，这里通过模拟鼠标上拖来实现助战界面的下滑。在每次上拖之后，检测当前窗口下是否存在我需要借用的角色，若不存在则继续下滑。由于助战显示是有限制的，有时候正好所有显示的助战都不是我所需要的角色，这里采用计数的方法，记录操作的次数，若次数到达一定之后还没有找到所需角色，则刷新助战，具体代码如下：

```
def check_character(handle, width, height, character):
    """find character in assist"""
    basic_function.get_bitmap(handle, width, height)
    print('finding ' + character)
    [min_val, max_val, min_loc, max_loc] = basic_function.template_matching(handle, width, height,
                                                                            character, (821, 462))
    th = 91
    tw = 105

    while max_val < 0.95:
        count = 0
        while max_val < 0.95 and count < 10:
            count += 1
            basic_function.get_bitmap(handle, width, height)
            basic_function.press_keyboard(handle, button_dict['up'], 1.5)
            [min_val, max_val, min_loc, max_loc] = basic_function.template_matching(handle, width, height,
                                                                                    character, (821, 462))
        if max_val >= 0.95:
            break
        else:
            time.sleep(10)
            print('could not find ' + character + ', refresh')
            basic_function.press_keyboard(handle, button_dict['5'], 1)
            basic_function.press_keyboard(handle, button_dict['H'], 1)
    print('OK')

    tl = max_loc
    br = (tl[0] + tw, tl[1] + th)

    point = [0, 0]
    point[0] = int((tl[0] + br[0]) / 2 / 821 * width)
    point[1] = int((tl[1] + br[1]) / 2 / 462 * height)

    basic_function.press_mouse(handle, point, 3)
    basic_function.press_keyboard(handle, button_dict['4'], 22)
    return None
```

## 实际代码
代码共享在[github](https://github.com/conceptclear/gamescript)上，可以根据自己需求使用。

## Reference
[1]百度百科       
[2]http://conceptclear.cn/mobilegame/2020/02/18/MobileGame-fgo.html                             
[3]http://conceptclear.cn/mobilegame/2018/10/07/MoblieGame-ArkOrder-blog.html                          
[4]http://conceptclear.cn/mobilegame/2019/10/02/homedream.html        
