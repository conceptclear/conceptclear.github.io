---
layout: post
title:  "手游fgo网易mumu模拟器自动挂机py脚本"
date:   2020-06-17 02:35:56
category: MobileGame
---

之前曾经写过一篇利用按键精灵来进行fgo挂机的脚本[手游fgo网易mumu模拟器自动挂机刷本脚本](http://conceptclear.cn/mobilegame/2020/02/18/MobileGame-fgo.html)，但是按键精灵本身存在很多限制（比如图像识别算法的问题，不能后台识图的问题等）而且运行起来CPU占用率也不低，这里提出一种利用python脚本来实现的方法（2020年9月1日更新了一部分内容）。

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

更新：由于实现了助战角色和助战角色所带礼装分开检测的功能，所以进行模板匹配时，需要先对整个游戏截图进行一下裁剪。

```
def template_matching(check_pic, template_pic, resolution, check_area):
    """check the pic whether existing or not"""
    template = cv.imread(template_pic)
    size = template.shape
    target = cv.imread(check_pic)
    if resolution != 0:
        target = cv.resize(target, resolution)
    if check_area != 0:
        target = target[check_area[0]:check_area[1], check_area[2]:check_area[3]]

    result = cv.matchTemplate(target, template, cv.TM_CCOEFF_NORMED)
    min_val, max_val, min_loc, max_loc = cv.minMaxLoc(result)

    return [min_val, max_val, min_loc, max_loc, size[0], size[1]]
```

## 自动补充体力
自动吃苹果只需要在每次进本的时候识别一下是否需要吃苹果即可，即点击了该本之后是否弹出补充体力这一窗口。为了保证程序的稳定性，这里并不是认为没有弹出该窗口就是不需要补充体力，需要再对后续界面进行一次识别，若失败证明脚本出现问题，需要停止继续操作。

```
def check_apple(handle, width, height, resolution, state):
    """check whether eat apple"""
    print('Checking whether to eat apple')
    basic_function.get_bitmap(handle, width, height, resolution)
    [min_val1, max_val1, min_loc1, max_loc1, th, tw] = basic_function.template_matching('img_check.bmp',
                                                                                        'source/apple.jpg',
                                                                                        (1280, 720), 0)
    [min_val2, max_val2, min_loc2, max_loc2, th, tw] = basic_function.template_matching('img_check.bmp',
                                                                                        'source/assist.jpg',
                                                                                        (1280, 720), 0)
    if max_val1 > 0.95:
        print("eat apple")
        if state:
            basic_function.press_keyboard(handle, button_dict['T'], 1)
            basic_function.press_keyboard(handle, button_dict['H'], 2)
        else:
            print("end loop")
            sys.exit()
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

更新：由于有时候刷本不需要确定的助战角色，只需要该角色身上带的礼装就行（比如刷QP本），有时候只需要确定的助战角色，不在乎带什么礼装（比如高难本），有时候都不需要有时候都需要，这就给识别带来了一定的难度，如果每种情况都事先截图然后进行匹配，那每次出新的礼装时都要更新一大波模板文件，这里更新了一种分开检测的方法。思路很简单，就是先判断是否要检测角色，如不需要检测角色就直接检测礼装，若需要检测角色就先检测角色，再检测该角色下方区域对应的礼装是否匹配所需要的礼装。

```
def check_character(handle, width, height, character, equipment, resolution):
    """find character in assist"""
    basic_function.get_bitmap(handle, width, height, resolution)
    if character == str(0):
        print("don't need to find character")
        if equipment == str(0):
            print("don't need to find equipment")
            basic_function.press_keyboard(handle, button_dict['N'], 3)
            basic_function.press_keyboard(handle, button_dict['4'], 22)
            return None
        else:
            print("finding " + equipment)
            [min_val, max_val, min_loc, max_loc, th, tw] = basic_function.template_matching('img_check.bmp',
                                                                                            'source/' + equipment + '.jpg',
                                                                                            (1280, 720), 0)
            while 1:
                count = 0
                while max_val < 0.95 and count < 10:
                    count += 1
                    basic_function.press_keyboard(handle, button_dict['up'], 1.5)
                    basic_function.get_bitmap(handle, width, height, resolution)
                    [min_val, max_val, min_loc, max_loc, th, tw] = basic_function.template_matching('img_check.bmp',
                                                                                                    'source/' + equipment + '.jpg',
                                                                                                    (1280, 720), 0)
                if max_val >= 0.95:
                    break
                else:
                    time.sleep(10)
                    print('could not find ' + equipment + ', refresh')
                    basic_function.press_keyboard(handle, button_dict['5'], 1)
                    basic_function.press_keyboard(handle, button_dict['H'], 1)
            print('OK')
    else:
        print('finding ' + character)
        [min_val, max_val, min_loc, max_loc, th, tw] = basic_function.template_matching('img_check.bmp',
                                                                                        'source/' + character + '.jpg',
                                                                                        (1280, 720), 0)

        while 1:
            count = 0
            while max_val < 0.95 and count < 10:
                count += 1
                basic_function.press_keyboard(handle, button_dict['up'], 1.5)
                basic_function.get_bitmap(handle, width, height, resolution)
                [min_val, max_val, min_loc, max_loc, th, tw] = basic_function.template_matching('img_check.bmp',
                                                                                                'source/' + character + '.jpg',
                                                                                                (1280, 720), 0)
            if max_val >= 0.95:
                print('OK')
                if equipment == str(0):
                    print("don't need to find equipment")
                    break
                else:
                    print("checking " + equipment)
                    if max_loc[1] + th + 70 > 720:
                        print("equipment did not match")
                        continue
                    [min_val1, max_val1, min_loc1, max_loc1, th1, tw1] = basic_function.template_matching(
                        'img_check.bmp',
                        'source/' + equipment + '.jpg',
                        (1280, 720),
                        [max_loc[1] + th,
                         max_loc[
                             1] + th + 70,
                         max_loc[0] - 20,
                         max_loc[
                             0] + tw + 20])
                    if max_val1 >= 0.95:
                        break
                    else:
                        print("equipment did not match")
            else:
                time.sleep(10)
                print('could not find ' + character + ', refresh')
                basic_function.press_keyboard(handle, button_dict['5'], 1)
                basic_function.press_keyboard(handle, button_dict['H'], 1)
        print('OK')

    tl = max_loc
    br = (tl[0] + tw, tl[1] + th)

    point = [0, 0]
    point[0] = int((tl[0] + br[0]) / 2 / 1280 * width)
    point[1] = int((tl[1] + br[1]) / 2 / 720 * height)

    basic_function.press_mouse(handle, point, 3)
    basic_function.press_keyboard(handle, button_dict['4'], 22)
    return None
```

## 战斗操作
之前的版本需要在程序里事先输入战斗策略，然后调用该策略的函数来运行，但是这样的话无法适用于每个人的刷本策略，不同的人使用代码的话需要重新修改代码，这无疑增加了使用难度。现在采取了读取策略的方式来实现个性化刷本，提供了一个strategy.xlsx的excel文件，只需要在该文件中设定好每一面的技能释放顺序以及宝具及等待需要时间，则可以实现针对不同队伍的刷本。需要更换队伍时，只需要修改excel文件即可，不需要重新修改代码，读取部分代码如下：

```
    wb = xlrd.open_workbook('strategy.xlsx')
    for battle in range(3):
        if battle != check_fight(handle, width, height, resolution)-1:
            print("error occurred in fight " + str(battle+1))
            sys.exit()
        else:
            print("Start fight in side " + str(battle+1))
        side = wb.sheet_by_name('Side' + str(battle + 1))
        for repeat in range(9):
            if side.cell(2 + repeat, 1).value != 1:
                continue
            else:
                if side.cell(2 + repeat, 2).value == 0:
                    basic_function.press_keyboard(handle, button_dict[chr(65 + repeat)], 3)
                elif side.cell(2 + repeat, 2).value > 6 or side.cell(2 + repeat, 2).value < 0:
                    print("error input")
                    sys.exit()
                elif side.cell(2 + repeat, 2).value < 4:
                    basic_function.press_keyboard(handle, button_dict[chr(65 + repeat)], 0.5)
                    basic_function.press_keyboard(handle, button_dict[chr(78 + int(side.cell(2 + repeat, 2).value))], 3)
                else:
                    basic_function.press_keyboard(handle, button_dict[chr(57 + int(side.cell(2 + repeat, 2).value))],
                                                  0.5)
                    basic_function.press_keyboard(handle, button_dict[chr(65 + repeat)], 3)

        for repeat in range(3):
            if side.cell(11 + repeat, 1).value != 1:
                continue
            else:
                basic_function.press_keyboard(handle, button_dict['S'], 0.5)
                if side.cell(11 + repeat, 2).value == 0:
                    basic_function.press_keyboard(handle, button_dict[chr(84 + repeat)], 3)
                elif side.cell(11 + repeat, 2).value > 4 or side.cell(11 + repeat, 2).value < 0:
                    print("error input")
                    sys.exit()
                else:
                    basic_function.press_keyboard(handle, button_dict[chr(84 + repeat)], 0.5)
                    basic_function.press_keyboard(handle, button_dict[chr(78 + int(side.cell(11 + repeat, 2).value))],
                                                  3)

        basic_function.press_keyboard(handle, button_dict['J'], 3)
        for repeat in range(3):
            if side.cell(15 + repeat, 1).value != 1:
                continue
            else:
                if side.cell(15 + repeat, 2).value == 0:
                    basic_function.press_keyboard(handle, button_dict[chr(75 + repeat)], 0.5)
                elif side.cell(15 + repeat, 2).value > 4 or side.cell(15 + repeat, 2).value < 0:
                    print("error input")
                    sys.exit()
                else:
                    basic_function.press_keyboard(handle, button_dict[chr(60 + int(side.cell(15 + repeat, 2).value))],
                                                  0.5)
                    basic_function.press_keyboard(handle, button_dict[chr(75 + repeat)], 0.5)

        rand_card(hwnd)
        time.sleep(side.cell(18, 1).value)

    for repeat in range(5):
        basic_function.press_keyboard(handle, button_dict['4'], 1)
    print('battle finish')
    return None
```
脚本现在加入了一段测试代码，用于测试每一面释放技能开始之前是否游戏已经运行到了这一界面，以防止出现游戏卡顿导致的时间轴出错或者是宝具回收不够导致上一面没有成功运行脚本的情况，因为每个副本的游戏背景不同，简单地采用模板匹配并不能准确检测当前的战斗面，所以可能出现bug，现在处于测试阶段。

## 实际代码
代码共享在[github](https://github.com/conceptclear/gamescript)上，可以根据自己需求使用。

## Reference
[1]百度百科       
[2]http://conceptclear.cn/mobilegame/2020/02/18/MobileGame-fgo.html                             
[3]http://conceptclear.cn/mobilegame/2018/10/07/MoblieGame-ArkOrder-blog.html                          
[4]http://conceptclear.cn/mobilegame/2019/10/02/homedream.html        
