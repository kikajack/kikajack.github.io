OpenCV的视频相关操作封装在VideoCapture和VideoWriter两个类中。
###1.视频读取VideoCapture
1. 读取视频文件或摄像头
```python
#方法一：定义类的时候指定文件或摄像头，默认摄像头为0。
cap = cv2.VideoCapture("1.avi") #读取本地视频文件
cap = cv2.VideoCapture(0) #读取摄像头，参数为ID

#方法二：用open()方法指定文件或摄像头
cap = cv2.VideoCapture();
cap.open("1.flv")
```
2. 获取视频参数（帧率、总帧数、尺寸、格式等）：get(propID)
```python
fps = cap.get(cv2.CAP_PROP_FPS)  
size = (int(cap.get(cv2.CAP_PROP_FRAME_WIDTH)),   
        int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))) 
```
3. 读取视频帧
```python
#read()获取下一帧，成功则返回True和图像
success, frame = cap.read() 

```
4. 设置参数，读取起始帧：set(propID, value)
cap.set(cv2.CV_CAP_PROP_POS_FRAMES, 100.0)   #第100帧

```python
import cv2

cap = cv2.VideoCapture()
cap.open("../spyder/1.flv")

fps = cap.get(cv2.CAP_PROP_FPS) #获取帧率
cap.set(cv2.CAP_PROP_POS_FRAMES, 200) #从200帧开始读

success, frame = cap.read()

while success:
    cv2.imshow("video", frame)
    key = cv2.waitKey(int(1000/fps))
    if (key & 0xff == ord('q')):
        break
    success, frame = cap.read()

cv2.waitKey(0)
cv2.destroyAllWindows()
```

###2.视频写入VideoWriter
OpenCV里对视频的编码解码支持不是很好，要实现摄像头图像的获取与转码可以参考FFmpeg库。
视频格式包括: 
I420(大文件) -> .avi;
PIMI -> .avi;
MJPG -> .avi & .mp4
THEO -> .ogv;
FLV1(flash video, 流媒体视频) -> .flv
1. 