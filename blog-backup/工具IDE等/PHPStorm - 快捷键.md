[官方文档中关于快捷键的部分](https://www.jetbrains.com/help/phpstorm/keyboard-shortcuts-by-keystroke.html)
#1. 编辑
- Ctrl + Q：查看光标所在的函数或类的注释内容。注释应采用标准的注释格式：/**这是注释*/。
- Ctrl + 光标移动到函数或类上：按住Ctrl，光标移动到函数或类上，可查看简短的介绍。按住 Ctrl 点击该函数或类即可跳转到该函数或类定义的位置。
- Alt + Insert：生成代码段，包括函数或类注释（内容可以定制），版权信息，构造方法，抽象方法等。
- Ctrl + /：添加“//”形式的注释，会添加到光标所在行的最前端。
- Ctrl + Shift + /：添加“/**/”形式的注释，会添加到选中代码段的两端。
- Ctrl + W：增量式的选中代码，从光标所在处开始，每按一次，选中更大的代码块，通常以单词，引号，括号为界限。
- Ctrl + Shift + W：与 Ctrl +  W 相反，减小选中范围。
- Ctrl + Alt + I：自动缩进。标准缩进采用四个空格或者按一下Tab键。
- Tab / Shift + Tab：缩进/取消缩进。
- Ctrl + D：复制当前行到下一行，或复制选择的内容到选择内容末尾处。
- Ctrl + Y：删除光标所在的行。
- Ctrl + Shift + ] / [：以块（大括号{}）为单位，从光标处 向后/向前 选择，再次点击增加选择范围。常用。
#2. 搜索替换
- Ctrl + F 查找。
- Ctrl + R 替换。
- Ctrl + Shift + F 在目录或项目中查找。
- Ctrl + Shift + R 在目录或项目中替换。
#3. 导航
- **Ctrl + Shift + Backspace：返回到上次编辑的位置。**
- **Ctrl + F12：打开文件结构的弹出窗。**
- **Alt + Up/Down 上下切换函数。**
- Ctrl + N：搜索类。全项目范围。
- Ctrl + Shift + N：根据文件名搜索文件。全项目范围。
- Ctrl + Alt + Shift + N：搜索函数。全项目范围。
- Alt + Right/Left：左右切换在 tab 中打开的文件。
- Alt + Tab：切换为上一次打开的文件。
- Ctrl + G：按行号快速定位。
- Ctrl + E 打开最近打开过的文件列表。
#4. 版本控制
- Ctrl + K：提交代码到svn。
- Ctrl + T：更新代码到本地。