---
title: 基于opencv的图片模板匹配及其简单应用
date: 2018-12-22 15:06:59
tags: 
- Python
- Opencv
- 脚本开发

categories: 个人项目
---



# 基础知识

基于opencv的图片模板匹配
----

<!--more-->

注: python及其相关包的安装不在讨论范围内

        

opencv提供了图片模板匹配的方法, 

 

```python
cv2.matchTemplate(目标图,模板,相似度计算方法)

```

 

该方法的思路大致如下：

 

1. 确定当前图片的长宽 x，y

2. 确定模板图片的长宽 w，h

3. 从(0，0)为起点，依次比对到(x-w，y-h), 计算在每个点(i,j)处图片(i,j)~(i+w,j+h)与模板的的相似度

4. 比对完成后返回每个点的相似度

 
![](https://i.loli.net/2018/12/22/5c1de3f0816e4.png)


 

此外，opencv还提供了计算图片相似度的方法：

 

- CV_TM_SQDIFF 平方差匹配法：该方法采用平方差来进行匹配；最好的匹配值为0；匹配越差，匹配值越大。 

- CV_TM_SQDIFF_NORMED 相关匹配法：该方法采用乘法操作；数值越大表明匹配程度越好。 
- CV_TM_CCORR 相关系数匹配法：1表示完美的匹配；-1表示最差的匹配。 
- CV_TM_CCORR_NORMED 归一化平方差匹配法 
- CV_TM_CCOEFF 归一化相关匹配法 
- CV_TM_CCOEFF_NORMED 归一化相关系数匹配法


 [公式文档](https://docs.opencv.org/3.0-beta/modules/imgproc/doc/object_detection.html?highlight=matchtemplate#cv2.matchTemplatehttps://docs.opencv.org/3.0-beta/modules/imgproc/doc/object_detection.html?highlight=matchtemplate#cv2.matchTemplate)



 

具体的函数公式我没看懂……流下了不学无术的泪水.jpg

 

使用adb(usb调试工具)实现电脑操控手机
---

安卓提供了usb调试功能，将其打开就可以通过usb数据线和命令行来进行模拟点击，滑动等操作。

 

具体要用到的命令只有几条：

```

adb shell screencap -p ./sdcard/now.png

adb pull ./sdcard/now.png D:\now.png

adb shell input tap x y

adb shell input swipe x1 y1 x2 y2

```

其中第一条是截图命令, 截取当前屏幕命名为now.png并保存在sdcard目录中，-p参数是使其保存当前截图为png图片。

 

第二条是将截到的now.png拉到电脑的d盘里，方便接下来进行模板匹配

 

第三条是在(x,y)处进行点击。手机默认左上角为0，0。

 

第四条是模拟从x1,y1滑动到x2,y2的操作

 

最后再通过python的subprocess包中的run方法就可以实现仅用python来完成这些操作了。

 

-------

# 简单应用

作为手游玩家，对于模板匹配自然是想应用到重复的刷图上，用科技代替肝。

![](https://i.loli.net/2018/12/22/5c1de3f2c7515.png)

如图是肝一场的循环，其中进入和退出地图、结束回合的ui按钮位置固定，不需要进行模板匹配。需要模板匹配解决的是识别位置不固定的敌人、补给点和自己的角色，以完成部署、迎击和补给的操作。

 

以我玩的比较多我一款手游为例：

![](https://i.loli.net/2018/12/22/5c1de3fcaa2e5.png)
希望实现的是识别机场并部署己方梯队。由于敌方与己方相对位置固定，在这张图中确定了己方位置就能间接得到敌方位置，所以不需要对杂兵的位置进行匹配。

 

为节省篇幅，这里假设其他功能已经完善，仅就确定机场位置进行演示，代码如下：

```python
import cv2
import numpy as np

def locate(target,tpl,rate):
'''
target和tpl为两图片，本函数将在target中对tpl进行模板匹配，标记出所有匹配结果大于rate的结果
'''
    #将两图转换为灰度图
    tpl=cv2.cvtColor(tpl,cv2.COLOR_BGR2GRAY)
    target_gray=cv2.cvtColor(target,cv2.COLOR_BGR2GRAY)

    #模板匹配
    res=cv2.matchTemplate(tpl,target_gray,cv2.TM_CCOEFF_NORMED)

    #取得模板长宽
    th,tw=tpl.shape[:2]

    #过滤匹配结果
    locs = np.where(res >=rate)

    #为超过阈值的结果绘制矩形
    for loc in zip(*locs[::-1]):
        img =cv2.rectangle(target, loc, (loc[0] + tw, loc[1] + th), (0, 0, 255), 3)
    cv2.imshow('locate',img)

#读入测试用图
target=cv2.imread(r'D:\now.png')
tpl=cv2.imread(r'D:\airplain.png')

cv2.imshow('tpl',tpl)
locate(target,tpl,0.8)
```

程序中设置了rate来过滤匹配值低于0.8的结果，并在高于0.8的结果的位置绘制一个模板大小的矩形。应用在具体脚本里时可以在for循环中把匹配结果存在一个列表里，之后用return返回列表就得到了匹配的坐标。再对坐标进行一些处理就可以得到应当点击的位置了。

运行结果：
---

![](https://i.loli.net/2018/12/22/5c1de3f06ec25.png)

能够发现左边的机场由于被小兵的战斗力标签挡住了而没能成功识别，可以通过修改匹配的参数来解决这个问题。可以认为算法完成的还是很不错的。

 

---

# 总结

模板匹配在辅助游戏方面也可以有着更多的应用，比如微信的跳一跳就可以通过这样来确定当前位置和下一位置的距离，再通过模拟点击等完成外挂制作。~~碧蓝航线和fgo应该也行的~~

此外，通过模板匹配也可以做到判断两图是否相似，能够给后续的脚本开发节省代码实现难度

 

本例中模板匹配只实现了基础脚本中的确定点击位置这一个功能。一个合格脚本的反反脚本功能如随机点击位置，随机间隔延时等并未实现。至于后续的脚本的功能完善，那就是另一篇博客了。
![](https://i.loli.net/2018/12/22/5c1de40780496.jpg)
（图文无关，侵删）