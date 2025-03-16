---
title: 博客搭建（hexo+github）
date: 2024-04-05 14:07:56
toc: true
categories: [技术总结,其它]
tags: 博客搭建
---
## 简介

**搭建完成网站的如下所示**
[https://polarday.top/](https://polarday.top/)<!-- more -->

使用github托管博客，完全免费不需要购买服务器
博客框架：hexo
hexo主题：ICARUS
图床：github+PicGo
编辑：vscode
> 为什么使用hexo框架？因为hexo是静态框架，我们使用github托管博客的页面只能使用静态的框架，不支持像wordpress等需要请求数据库的动态框架，这类框架必须具有自己的服务器

## 搭建步骤

1. 搭建hexo框架发布到github
2. 更换ICARUS主题
3. 配置vscode的markdown环境
4. 连接图床

## 搭建hexo框架发布到github

先安装node.js和git
[参考教程](https://zhuanlan.zhihu.com/p/26625249)
[hexo官方文档](https://hexo.io/zh-cn/docs/)

### 1.github创建个人仓库

点击GitHub中的New repository创建新仓库，仓库名应该为：用户名.github.io这个用户名使用你的GitHub帐号名称代替，这是固定写法，比如我的仓库名为:
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240405142119.png" width="50%">

### 2.安装hexo

Hexo就是我们的个人博客网站的框架， 这里需要自己在电脑常里创建一个文件夹,可以命名为Blog,进入文件夹中,使用npm命令安装Hexo，输入：
`npm install -g hexo-cli`
安装完成后，初始化我们的博客，输入：
`hexo init blog`
初始化完成后进入blog目录，输入以下三条命令检测网站是否安装成功：

- 生成一篇文章
`hexo new test_my_site`
- 生成静态文件
`hexo generate`
- 启动服务器，默认情况下，访问网址为： <http://localhost:4000/>
`hexo server`

> 这里hexo server命令在执行之前不需要执行hexo g生成静态文件，执行该命令默认会从本地文件中启动而不是使用hexo generate生成的静态文件，使用hexo server -s	命令才会使用静态文件启动本地服务器

### 3.本地静态文件推送到github

上面只是在本地预览，接下来要做的就是就是推送网站，可以通过github的域名访问
blog目录下的_config.yml是网站的配置信息，包括网站标题等自定义内容可以参考hexo官方文档设置，在本地都调试完成后将其推送到github，在_config.yml的最后有deploy的配置，修改为如下配置，其中repo是你之前创建的仓库的路径

```yml
deploy:
  type: git
  repo: https://github.com/shnpd/shnpd.github.io.git
  branch: main
```

执行命令，就可以一键部署，具体命令细节可以参考[hexo官方文档](https://hexo.io/zh-cn/docs/)
`hexo clean`
`hexo g`
`hexo d`

然后访问shnpd.github.io就可以看到我们的博客了
**组成我们网站的文件是blog目录下的public目录，执行clean命令会清楚public目录，执行generate命令会生成public目录，部署命令也是将public目录部署到github上**

### 4.域名绑定（可选）

绑定域名是可选的，我们使用shnpd.github.io本身就可以访问博客了，如果有同学想绑定自己的域名可以参考本节

首先需要购买自己的域名，购买的途径有很多，像腾讯云、阿里云都可以，我使用的是腾讯云

在腾讯云控制台域名解析中添加两条记录
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240405150447.png">

登录github之前创建的仓库，settings-pages-custom domain填入自己的域名，点击save保存
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240405150735.png" width="50%">

进入blog的source目录下新建txt，输入你的域名。如果带有www，那么以后访问的时候必须带有www完整的域名才可以访问，但如果不带有www，以后访问的时候带不带www都可以访问。所以建议不要带有www。保存命名为CNAME，无后缀名。
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240405151023.png" width="50%">

域名绑定完成，这里因为是基于github搭建的所以不需要对域名进行备案

## 更换hexo主题

hexo的主题有很多，我们这里使用的是icarus
[hexo-theme-icarus](https://github.com/ppoffice/hexo-theme-icarus)
[官方文档](https://ppoffice.github.io/hexo-theme-icarus/)

### 1.主题安装

在blog目录下执行如下命令：
`npm install -S hexo-theme-icarus hexo-renderer-inferno`

在网站配置_config.yml文件中启用icarus主题：
`theme: icarus`
或使用hexo命令修改主题为Icarus
`hexo config theme icarus`

启动本次服务器测试是否成功：
`hexo server`

### 2.主题配置

icarus的具体主题配置都可以参考[官方文档](https://ppoffice.github.io/hexo-theme-icarus/tags/Icarus%E7%94%A8%E6%88%B7%E6%8C%87%E5%8D%97/)，细节配置可以参考文档自行定义，下面主要介绍几个关键的地方

**图片**
我们在设置网站的logo或其他配置时可能会用到图片，这里图片的路径为/img/xxx.jpg，这是相对路径，相对blog目录的路径为：/node_modules/hexo-theme-icarus/source/img/xxx.jpg

**About页面**
about页面在初始化的时候是没有的，需要我们自己创建，创建命令为：
`hexo new page about`

需要执行以下命令重新生成静态文件

```bash
hexo clean
hexo g
```

创建其他页面时也按照这个步骤，新建的页面在source目录下

**写文章**
新建文章使用`hexo new "第一篇文章"`命令，就可以在source/_posts目录下创建文章的markdown文件直接编辑即可
文章分类和标签的设置参考[hexo官方文档Front-matter](https://hexo.io/zh-cn/docs/front-matter)
文章在首页默认会显示全部内容，需要在文章中添加\<!-- more -->标签。 标签前面的文章内容会被标记为摘要，而其后的内容不会显示在文章列表上。

## 配置vscode markdown环境（可选 推荐）

这里因为我们需要编辑markdown来写文章，这里我们推荐使用vscode，通过vscode打开blog目录，可以方便的查看目录结构，同时也可以直接在vscode中开启终端执行相关命令，使用vscode编写markdown非常方便，具体配置教程有很多，这里主要就是安装几个插件

- Markdown All in One （基本语法）
- Markdown Preview Enhanced（预览）
- markdownlint（语法检查）

安装完成后就可以直接编写markdown了

## 配置图床（可选 推荐）

我们文章中会存在大量的图片，直接将图片保存在网站的文件中会相当的臃肿，所以我们推荐使用图床将图片保存在其他位置并生成外链，在我们的文章中通过外链来引用图片即可。

可以用来充当图床的服务有很多，包括七牛云、阿里云OSS、腾讯云COS、GitHub等，其中七牛云需要备案域名、阿里云和腾讯云不是完全免费的，我们这里使用的是GitHub，但是GitHub本身存在访问较慢的问题，还是推荐大家使用七牛云、阿里云等。

### 1.新建github仓库

新建github仓库作为图床用来保存我们的图片，仓库名字可以随便起我的名字是blog-pic，仓库需要设置为public

### 2.下载PicGo

[PicGo官方文档](https://picgo.github.io/PicGo-Doc/zh/guide/#picgo-is-here)
PicGo是一个用于快速上传图片并获取图片URL链接的工具，如果使用vscode的话可以直接安装PicGo的扩展，安装完成后配置github图床

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240405160928.png" width="50%">

配置名：随意起
仓库名：新建仓库的名称
分支名：设置为main即可
Token：需要在github中申请；settings——Developer Settings——Personal access tokens——tokens——Generate new token

vscode的扩展也是相同的配置方法，这里主要说一下vscode中的使用

**复制自动生成链接**
通过使用Upload image from clipboard就可以从粘贴板上传图片，这里的快捷键我们设置为ctrl+alt+e比较方便，当我们复制了一张图片后，使用该快捷键就可以自动将图片上传并生成链接插入到对应的位置
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240405162011.png" width="50%">

**自定义图片的输出格式**
这里定义了前一步插入的格式，考虑到通用性我们设置为html的格式，同时添加了width="50%"默认对每张图片缩放50%
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240405161317.png" width="50%">
