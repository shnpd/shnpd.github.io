---
title: 常用Git命令（简洁版）
toc: true
date: 2022-05-13 23:42:27
tags: git github
categories: 杂项
---

​​点击阅读更多查看文章内容<!--more-->

# 一、添加SSH

1. 本地生成密钥，存储在用户目录(~)下的.ssh目录中，.ssh是隐藏文件要使用ls -a查看
`ssh-keygen -t rsa -C "your_email@example.com"`
2. id_rsa是私匙，id_rsa.pub是公匙，在github上添加公钥
`cat /root/.ssh/id_rsa.pub`：查看公钥内容
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/fbd5997c1048413f78e576f1013dca45_1740930577064.png)

# 二、拉取项目
在github页面找到要拉取项目的url，使用git clone命令拉取到当前目录下
`git clone https://github.com/shn-1/HyperledgerFabric_Learning.git`
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/fbf0844d85bba60efa45de3c929320d4_1740930577064.png)

# 三、上传项目
上传项目主要用到三个命令
`git add`
`git commit`
`git push`
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/7a82e61b4ac2fdcbd9ac59db82e78037_1740930577064.png%20=500x)


1. 把当前目录变成 Git 可以管理的仓库 
`git init`
2. 执行将目录下的所有文件加入到暂存区
`git add *`
3. 将暂存区的内容提交到版本库
`git commit -m "注释"`

至此，在本地仓库的操作已经完成，下面是远程仓库的操作

---
[remote 及 push 命令详解](https://blog.csdn.net/wq6ylg08/article/details/89028412)

4. 与远程仓库建立连接，其中origin可以视为远程仓库在本地的别名
`git remote add origin https://github.com/shn-1/HyperledgerFabric_Learning.git`
5. 将本地分支推送到远程仓库的分支即可，其中origin为远程仓库，master为本地分支（默认）
`git push origin master`

>这里可能需要输入用户名和密码，**注意这里的密码不是登录密码而是Personal Access token**
>![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/94e2c788e9c6ef3060718784801c3c37_1740930577064.png%20=500x)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b4208248fa6aa8204959e48e260b3019_1740930577064.png%20=500x)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/bd0af52c5744689252e08cdcc8a8c8e8_1740930584194.png%20=500x)
>如果token忘记了或是过期了，点击对应的token直接重新生成即可
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/0ca3844f0f9274d284c92edf4e6e0213_1740930584194.png%20=500x)
# 四、分支操作

1. 查看本地所有分支，前面带*的是当前所处的分支
`git branch`
2. 查看远程所有分支
`git branch -r`
3. 创建分支
`git branch [name]`
4. 切换分支
`git checkout [name]`
5. 合并分支
`git merge [name]`

# 五、部分命令的详细用法
**关于git push命令的用法**
[参考文章](https://lijunde.blog.csdn.net/article/details/89028412)
>`git push <远程仓库名> <本地分支名>:<远程分支名>`：将本地分支推送到远程仓库的远程分支。（注意：这里的远程仓库名依然是在本地仓库中对远程仓库起的别名）
`<远程仓库名>`：在本地仓库中对远程仓库起的别名，如上面命令解析2(1)中设置的origin。
`<本地分支名>`：本地分支的名称，比如我们在项目开发，一般主分支(也是默认分支)叫做master，一些新功能开发的分支叫做develop或feature。这些我们在我们自己电脑本地用git branch创建的分支就是本地分支。
`<远程分支名>`：在远程仓库的普通分支，比如远程仓库上的master，自己在远程仓库创建的分支，以及自己推送到远程仓库上去的在远程仓库上的分支。
（注意：`<远程分支名>`与 `<远程仓库名>`的情况不同：
(i)`<远程分支名>`的取名由git push中的远程分支名决定，一般Git使用者会省略<远程分支名>这个参数，所以Git会默认把`<本地分支名>`设置为`<远程分支名>`;
(ii)`<本地分支名>`无论在远程仓库还是本地仓库就只有一个名字，不像<远程分支名>有一个绝对URL地址名字和一个在本地仓库中的别名。）
