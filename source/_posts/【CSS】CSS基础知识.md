---
title: 【CSS】CSS基础知识
toc: true
date: 2022-12-27 17:45:43
tags: css 前端 html
categories: 前端
---

​点击阅读更多查看文章内容<!--more-->

CSS：Cascading Style Sheets 层叠式样式表

原始示例代码：
```html
<div>
    <div>租辆酷车</div>
    <div>css教学</div>
</div>
<div>
    <div>
        <div>中关村到天安门</div>
        <div>价格: 120元</div>
    </div>
    <div>
        <div>陆家嘴到迪士尼</div>
        <div>价格: 50元</div>
    </div>
    <div>
        <div>天河体育中心到广州塔</div>
        <div>价格: 80元</div>
    </div>
</div>
```
在代码后添加如下片段即可修改所有div元素的字体
```html
<style>
    div{
        font-size: xx-large;
    }
</style>
```

# 选择器
- element
直接选择全部的元素
如：`div`，选择所有的div元素

- #id
选择某一id的元素
如：`#title`，选择id为title的元素
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/1324341323a67f338f816f8252c31daa_1740930243307.png)
- .class
选择包含某个class的部分元素
如：`.item`，选择class为item的元素（以下三个全选）
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/a81985df5f7eda7488aa894012d5b73e_1740930243307.png)
`.item` 选择class包含item的元素（以下三个全选）
![在这里插入图片描述](https://raw.githubusercontent.com/shnpd/blog-pic/undefined/csdn/b9a7b50f3bfb509741faf35fa53f49d8_1740930250430.png)
- 同时满足
 `.item.blue` 选择class为item及blue的元素，中间无空格（1和3）
 `div.blue` 选择class为blue的div元素（1和3）
![在这里插入图片描述](https://raw.githubusercontent.com/shnpd/blog-pic/undefined/csdn/b9a7b50f3bfb509741faf35fa53f49d8_1740930250430.png)
- 父子关系
中间加上空格
`.item div` 选择class为item的元素的**所有**div子元素（不只是第一层儿子是所有的子元素）
`.item >div` 添加`>`选择class为item的元素的**第一层**div子元素
# 宽高
- height/width：设置高度/宽度，可以设置数值也可以设置百分比（相对于父元素大小的百分比，若父元素没有设置高度，则相对于父元素的父元素设置）

> 宽度**默认**填满页面，父元素高度**默认**为子元素高度之和（display: block;）

- overflow（当父元素高度小于子元素高度之和时）
	- hidden：隐藏多余元素
	- scroll：添加滚动条



# 缩进
margin：边框到父元素的距离
border：元素的边框宽度
padding：元素到边框的距离

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/0e11c79192021cf10c00ed8e87cca929_1740930250430.png)

---

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/84721885e9bc9d41a48de55bcb644615_1740930250430.png%20=700x300)

---


元素高度 = height + padding-top + padding-bottom + 2 * border（上下两个边框）

元素宽度 = width + padding-left + padding-right + 2 * border

---

如下示例

margin-left: 50px; **左边界与父元素的距离**
border: 2px solid; **元素边界宽度**
padding-left: 50px **元素与左边界距离**
height: 100px;**元素高度**
padding-top: 50px; **元素与上边界距离**
margin-top: 50px; **上边界与父元素的距离**

元素高度为 height + padding-top + 2 \* border = 154
元素宽度为 width + padding-left + 2 \* border = 510
 Ps：这里的width没有设置，是根据子元素得来的

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/0f58e7d11b576454139119b9e04c8d9b_1740930250430.png%20=700x300)

---

- box-sizing: border-box
设置之后元素的高度/宽度直接等于height/width
对元素本身的高度进行调整以使得最终高度等于height，默认情况的宽高只针对元素本身，最终宽高还要加上padding和border
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/35e4225036f20622f43edd5d9aced0cf_1740930256119.png%20=300x500)
# 位置
top\bottom\left\right 与 **position** 组合使用

- position
  - relative：原位置不会被顶替，相对于原位置进行位移
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/8a40e20b8eb877a150dbac3b6691a61c_1740930256119.png%20=200x)
  - absolute：原位置会被顶替，相对于页面进行位移，会因为滚动条而滚出页面
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/585c90abea5b07e2ef4f23bd8ad21652_1740930256119.png%20=200x)

  - fixed：原位置会被顶替，相对于页面进行位移，不会因为滚动条而滚出页面，始终固定在同一位置
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ff21289306f202910c0b51d0fdda4990_1740930256119.png%20=200x)

将父元素设置为position:relative，子元素设置为position:absolute，可以将子元素相对父元素设置

> 小练习：在陆家嘴到迪士尼的右上角添加一个小方框
> 1. 在陆家嘴中新建一个div，class设置为star
> 2. 将陆家嘴position设置为relative
> 3. 设置star大小和颜色
> 4. 将star的positon设置为absolute使其飘在上面而不占据空间
> 5. 将right和top设置为0就移动到右上角了
    .item {
        border: 2px solid;
        position: relative;
    }
    .star {
        width: 20px;
        height: 20px;
        background-color: red;
        position: absolute;
        right: 0px;
        top: 0px;
    }
        ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/2dca021b6ca5882adc7a2b8234f8e5f2_1740930256119.png%20=200x)
# 文本样式
font-size：设置大小
font-family：设置字体
font-style：设置斜体
font-weight：设置粗体
color：设置颜色
text-align：文字对齐

**折行**
默认情况超宽换行
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/8e6c20b7e8c3ad7738c4bd5da577208e_1740930262410.png%20=200x)

white-space:nowrap：超出宽度不换行
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/94b578a9df2cf211789145d36db00a9f_1740930262410.png%20=200x)

overflow:hidden：超出部分隐藏 （与前一个结合使用）
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/988f6b655dfdd1a4519f9a3cf2eb8d3b_1740930262410.png%20=200x)

text-overflow:ellipsis：超出部分省略号截断（与前两个结合使用）
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/3671ddf9d732cc768b79c1d32e569f84_1740930262410.png%20=200x)

word-break：英文换行按照字母/单词

---
# flex布局
flex是一种布局方法

- display（设置flex）
	- flex 设置为flex布局
- flex-direction （确定flex方向）
	- column 竖向排列
	- column-reverse 反向竖排
	- row 横向排列（默认）
	- row-reverse 反向横排

**以竖排为例**

- align-items 设置横向的对齐（**垂直于排列方向**，若元素横向排列则设置纵向对齐）
	- stretch 横向拉伸填满
	- center 横向居中 
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/79a297a5b9b3ab7fa122550141dc2232_1740930262410.png%20=500x)

- justify-content 设置竖向的对齐
	- flex-end：竖向末尾 (与flex-direction配合，如果flex-direction为reverse那么flex-end就是开头)
	- flex-start：竖向开头 
	- space-around 元素之间的距离为上下距离的二倍
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/eeb0f1992aff1ca41112fc8bfff8f0f6_1740930268882.png%20=500x)

	- space-between 上下距离为零元素之间距离相等
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/7b8b0673e6fc9fa2de64d4ef97353cc3_1740930268882.png%20=500x)

	- space-evenly 上下及元素之间的距离都相等
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/00d6536bf8666c5daabf949f7c723835_1740930268882.png%20=500x)

**flex可以嵌套使用**
- align-item：设置内部元素的对齐
- align-self：设置自身的对齐

# 小程序
微信小程序中的大小使用rpx相对像素
不管机型大小，宽度都是750rpx（高度750rpx，宽度100%，在任何机型上都是正方形，高度可能不同）
>rpx最初是根据iphone6来的，750rpx对应750/2=375像素
