## 网站性能优化项目

##优化点：
1. 图片裁剪为合适尺寸并压缩，JavaScript代码添加媒体类型，首屏css集成到首页文件中，index.html 在移动设备和桌面上的 PageSpeed 分数为97分
2. views/pizza.html 在滚动时一直保持 60fps 的帧速。
3. changePizzaSizes() 函数中，将获取尺寸和设置尺寸分开，防止在同循环中出现强制同步布局 FSL，确保views/pizza.html 页面上的 pizza 尺寸滑块调整 pizza 大小的时间小于5毫秒



##这里还有疑问，优化点在哪里：
Part 2: 优化 pizza.html 的 FPS（每秒帧数）

你需要编辑 views/js/main.js 来优化 views/pizza.html，直到这个网页的 FPS 达到或超过 60fps。你会在 main.js 中找到一些对此有帮助的注释。
