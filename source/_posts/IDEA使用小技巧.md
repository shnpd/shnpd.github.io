---
title: IDEA使用小技巧
toc: true
date: 2023-12-12 12:55:00
tags: intellij-idea java ide
categories: Java
---

​​点击阅读更多查看文章内容<!--more-->

# 常用的基本设置
- 界面字体
	- File | Settings | Appearance & Behavior | Appearance
- 编辑区字体
	- File | Settings | Editor | Color Scheme | Color Scheme Font
	**Use color scheme font instead of the default**
- 控制台字体
	- File | Settings | Editor | Color Scheme | Console Font  
	**Use console font instead of the default**
- 通过ctrl+鼠标滚轮控制字体大小
File | Settings | Editor | General 勾选 **change font size with Ctrl+Mouse Wheel** 
- 将编码全部改为UTF-8
在settings中搜索encode，将编码都改为utf-8
- JDK设置
Project Structure - Project Settings - Project - SDK
- 单击目录的文件自动打开并定位在编辑区
	项目目录始终定位在编辑区打开的文件
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/7782846f600daf7601999dc8d95d39f6_1740930200442.png%20=400x200)
- 自动导入(import)
File | Settings | Editor | General | Auto Import
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/c3d890ff1051144a0dad48099cc66fca_1740930200442.png)
# 编辑区设置
- 显示行号
File | Settings | Editor | General | Appearance
**Show line numbers**
- tabs位置
File | Settings | Editor | General | Editor Tabs
**Tab placement**
- tabs排序
File | Settings | Editor | General | Editor Tabs
**Sort tabs alphabetically**

# 代码编辑
- 复制
复制一行代码时，可以直接把光标放在该行任意位置，Ctrl+C
复制文件名时，直接在左侧的项目目录选择文件，Ctrl+C
复制光标所在行，Ctrl+D
复制多行，先选中多行，Ctrl+D
查看复制历史，Ctrl+shift+V，双击即可粘贴内容

- 粘贴
普通粘贴，会自动格式化，Ctrl+V
纯文本粘贴，不会格式化，Ctrl+alt+shift+V
- 格式化代码
文件格式化：Ctrl+alt+L
局部格式化：选中需要格式化的部分，Ctrl+alt+L
- 剪切
剪切光标所在行（不需要选中），可以当删除用，Ctrl+X

- 移动
Alt+Shift+上/下：当前行向上/下移动一行
Ctrl+Shift+上/下：带格式移动
选中多行可以移动多行

# 快速跳转
- 行内跳转
Home键跳到行首，End键跳到行尾
Ctrl+左/右：光标一次跳过一个词
Ctrl+Shift+左/右：选中一个词
- 根据行号定位
 Ctrl+G：跳到指定行
- Tabs快速切换
Alt+左/右：左/右切换Tabs
- 查看最近浏览过的文件
Ctrl+E

# 快速查找和替换
- 当前文件查找
Ctrl+F
- 当前文件替换
Ctrl+R
- 全局搜索（Find in Files）
Ctrl+Shift+F（可以选择项目或目录等）
- 全局替换
Ctrl+Shift+R
- 万能查找
Shift+Shift，可以查找文件、操作、文本等


# 万能快捷键
Alt+Enter
智能辅助提示。给出的提示与当前光标所在的位置有关系。
- 见到红色报错就按
- 见到波浪线警告就按
- 没报错没警告也可以按（删除无用变量，自动生成构造方法）

# 键鼠配合
- 竖向选择
alt+鼠标左键拖动
- 进入方法
Ctrl+鼠标左键
**跳回刚才的位置：Ctrl+Alt+方向键左**

# 调试项目

- Step Over：执行到当前方法的下一句
- Step Into：进入当前行调用的方法体里
- Step Out：执行完当前的方法
- Run to Cursor：运行到光标所在处
- 删除断点、失效断点、条件断点
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/23a86155d001a4a62d0bcf36c7901ad1_1740930200442.png)
- Mute Breakpoints：失效所有断点
- 异常断点：当抛出某个异常时执行断点![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/d5697e8740f9a983ea7e9c80f132011a_1740930200442.png)
# 代码生成Generate
在类中使用快捷键Alt+Insert 或者 右键-Generate
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/15fe80260ff6d28aebb435429a5589e6_1740930208396.png%20=200x200)
 - 生成Get/Set方法 
Getter and Setter
- 生成构造函数
Constructor
- toString
toString()：默认使用+拼接，建议使用stringbuffer
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/beeaf93546cea42423ec5a0a4ceca514_1740930208396.png%20=300x200)
- equals() and hashCode()
生成时可以选择判断相等或生成哈希的属性
# 代码重构
- 重命名
选中后，Shift+F6或右键-Refactor-Rename
变量、函数、类
在改动函数名时，idea会同步选择项目中相同的地方进行修改，如果idea筛选的改动位置不是我们希望改动的，可以右键-exclude，排除当前行，如果某个包下都不想改，可以在包上右键-exclude，统一排除。
- 抽取方法
将部分代码抽取出一个新的方法
选中代码-右键-Refactor-Extract Method
 - 生成变量
Ctrl+Alt+V：调用方法自动生成返回值；实例化对象自动生成变量
- 文件移动/复制/删除
移动：选中文件，F6 或 右键-Refactor-Move
复制：F5
删除：Delete

# 代码模板
File | Settings | Editor | Live Templates（可以自定义）
**live templates** （直接打快捷键）
 - 生成Main函数
psvm
- 生成输出语句
sout
- 生成for循环
fori

File | Settings | Editor | General | Postfix Completion（不能自定义）
**postfix**（先打变量或表达式，再打.快捷键）
- 10.fori：for (int i = 0; i < 10; i++) {  }
- i==1.if：if (i\=\=1) {  }
- user.null：if (user \== null) {  }
- user.sout：System.out.println(user);

# 更多实用技巧
- tab分屏和独立
右键-split
tab变为独立窗口：拖动出idea/选择文件 Shift+F4
- 本地修改历史
选择文件-右键Local History-Show History
- 查看方法调用情况
选择方法 Ctrl+Alt+H 或 点击Hierarchy窗口
Caller：调用该方法的
Callee：该方法调用的
- 多选
选择文件中出现的所有同一字符串：选择字符串-Ctrl+Alt+Shift+J

# 常用插件
git插件

