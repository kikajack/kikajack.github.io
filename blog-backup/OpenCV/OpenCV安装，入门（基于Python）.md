##1.安装Anaconda
https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/
anaconda集成了很多python科学计算的第三方库，安装方便，如果不使用anaconda，那么安装起来会比较痛苦，各个库之间的依赖性就很难连接的很好。
官网下载比较慢，这里推荐用清华的镜像库，快的飞起。找一个适合你机器的版本就行（注意Anaconda2代表支持Python2，Anacon3代表支持Python3）。
![清华镜像的Anaconda资源列表](http://img.blog.csdn.net/20170417201952178?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
下载完成后安装，注意安装路径最好不要有空格或中文，Windows下一直下一步，Linux下执行./xxxx.sh。
![这里写图片描述](http://img.blog.csdn.net/20170417204059391?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
安装完成后，直接在命令行敲`python`，进入Python环境则表示安装成功，记住版本号，后面安装OpenCV时需要对应。
##2.安装OpenCV3
方式比较多，列举常用的几种：
1. 直接在cmd命令行输入：`conda install --channel https://conda.anaconda.org/menpo opencv3`
然后根据提示，输入一些简单的命令如[y]\n? 输入y，等待更新安装即可
2. 用轮子文件安装（.whl文件）
######1. http://www.lfd.uci.edu/~gohlke/pythonlibs/，下载与当前Python版本对应的 opencv 文件（但这个网站卡死，建议用它搜索出文件名后，去其他地方下载），如 opencv_python-3.2.0-cp36-cp36m-win_amd64.whl，
![这里写图片描述](http://img.blog.csdn.net/20170417203917610?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
######2. 切换到该文件所在的目录，在命令行环境下使用：
  pip install opencv_python-3.2.0-cp35-cp35m-win_amd64.whl
######进行安装，文件名不能有空格或中文。
![这里写图片描述](http://img.blog.csdn.net/20170417205633682?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##3.入门
1. 安装完成的Anaconda默认带的IDE是Spyder，虽然比PyCharm弱一点，但对入门来说够用，而且占用内存小。
![这里写图片描述](http://img.blog.csdn.net/20170417205738399?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2. OpenCV图像的读取，显示，保存
```python
import cv2  #导入OpenCV库

src = cv2.imread("1.jpg")  #读图片，需要和当前Python文件同一个目录
cv2.imshow("src", src)  #显示
cv2.imwrite("2.jpg", src)  #保存

cv2.waitKey(0)  #等待按键按下
cv2.destroyAllWindows()  #关闭所有打开的窗口
```

3. OpenCV视频的读取，显示，保存
- 从硬盘读
```python
import cv2  
  
cap = cv2.VideoCapture('1.wmv')  #打开视频文件：其实就是建立一个VideoCapture结构  
    
fps = cap.get(cv2.CAP_PROP_FPS)  #获得视频的格式
size = (int(cap.get(cv2.CAP_PROP_FRAME_WIDTH)),
int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT)))  #获得码率及尺寸

while(cap.isOpened()):  #检测是否正常打开:成功打开时，isOpened返回ture  
    ret, frame = cap.read()  #读取下一帧  
    if ret == True:  
        cv2.imshow("video", frame) #显示
        k = cv2.waitKey(int(1000/fps)) #延迟
        if (k & 0xff == ord('q')):  #判断是否按下按键q
            break  
    else:  
        break  
  
cap.release()  
cv2.destroyAllWindows()
```
- 从摄像头读
```python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)
while(1):
    ret, frame = cap.read()
    cv2.imshow("video", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
cv2.destroyAllWindows() 
```
4.OpenCV基本数据结构
```
CV_8UC1// 8位无符号单通道  
CV_8UC3// 8位无符号3通道  
CV_8UC4  
CV_32FC1// 32位浮点型单通道  
CV_32FC3// 32位浮点型3通道  
CV_32FC4  
```
mat=cv.CreateMat(3, 3, cv.CV_8UC1)