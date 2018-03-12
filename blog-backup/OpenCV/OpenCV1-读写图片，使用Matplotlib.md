OpenCv中，图片的读取，显示，保存分别使用imread(), imshow(), imwrite()。Matplotlib库是Python的2D绘图库，可以轻松地将数据图形化。
###1.cv2.imread(fileName，colorType)
- fileName：图片名，含完整路径（与当前项目同一个文件夹下则可以忽略路径）。
- colorType：图片色彩。
1. cv2.IMREAD_UNCHANGED : 返回图像的深度不变，含alpha通道，对应-1（即原图）。
2. cv2.IMREAD_GRAYSCALE : 8位深度，1通道灰度，对应0（黑白图）。
3. cv2.IMREAD_COLOR : 8位深度，3通道彩色，是默认的读取图片方式，对应数字1（彩图）。
4. cv2.IMREAD_ANYDEPTH : 返回图像的深度不变，1通道，对应2。
5. cv2.IMREAD_ANYCOLOR : 8位深度，3通道，对应4。
>如果需要alpha 通道，必须用-1。
>通道表示每个像素点由几个数字组成，类似于RGB彩色图中的每个像素点有三个值，即三通道的。
>深度表示每个值由多少位来存储，图片是8bit（位）的，则图片的深度是8.

###2.cv2.imshow(windowName，image)
- windowName：窗口名。
- image：要显示的图片。

###3.cv2.imwrite(fileName，image)
- fileName：文件名，不指定路径的话，默认为当前程序所在目录。
- image：要保存的图片。

```python
'''
图片读写，保存的示例
'''
import cv2

src = cv2.imread("1.jpg", 0)
cv2.imshow("src", src)
cv2.imwrite("2.jpg", src)

cv2.waitKey(0)
cv2.destroyAllWindow()
```

###4.Matplotlib
Matplotlib十分适合交互式制图，语法和 Matlab 十分相近。Gallery页面（http://matplotlib.org/gallery.html）中有上百幅缩略图，打开之后都有源程序。Matplotlib实际上是一套面向对象的绘图库，它所绘制的每个元素，如线条Line2D、文字Text、刻度等在内存中都有一个对象与之对应。Matplotlib 里的常用类的包含关系为 Figure -> Axes -> (Line2D, Text, etc.)一个Figure对象可以包含多个子图(Axes)，在matplotlib中用Axes对象表示一个绘图区域，可以理解为子图。
```python
import cv2
from matplotlib import pyplot as plt

src = cv2.imread("1.jpg", 1)
plt.figure(1)
plt.imshow(src, cmap = 'gray', interpolation = 'bicubic')
#plt.xticks([]), plt.yticks([])  #隐藏栅格
plt.show()
```

1. 折线图plot(x, y)
```python
import cv2
from matplotlib import pyplot as plt

x = range(6)
plt.plot(x, [xi**2 for xi in x])
plt.show()
```

```python
import numpy as np
import pylab as pl
 
x = [1, 2, 3, 4, 5]# Make an array of x values
y = [1, 4, 9, 16, 25]# Make an array of y values for each x value
 
pl.plot(x, y)# use pylab to plot x and y
pl.show()# show the plot on the screen
```
2. 散点图plt.plot(x, y, 'o')
3. 颜色plt.plot(x, y, 'r')