---
title: 在同一个git Repository备份hexo文章及配置
date: 2016-01-02 18:10:33
categories: Hexo博客
tags: hexo
---

使用场景
===

使用 Hexo 的一键发布文章很方便，但是有多个地点需要同步文章的时候就麻烦了。
因为GitHub的repo上 只有Hexo 编译好的html文件，
并没有Hexo的source文件（也就是文章）。
如果需要在不同地点进行文章发布的话，就没有办法完成了（先不考虑草稿的事）
所以，我们需要在其他repo进行文章的备份。
<!-- more -->
参考链接：<http://www.jianshu.com/p/f746f8f7b32d>

操作步骤
===

### 上传操作

进入 Hexo 所在的文件夹。
打开 git bash 窗口，进行初始化repo：
```
git init
```


完成之后，添加修改的文件，本来 Hexo 就自带了 .gitignore 文件，需要忽略的文件 都已经默认配置好了。接下来进行第一次提交。
当然了，git流程还是要走正确的，首先应该是`add`：(`.`的意思将所有文件进行添加)
```
git add .
```
然后commit：
```
git commit -m "commit first time"
```
提交成功之后，接下来就是 push 到github了。
我们先把本地这个文件夹 映射到 远程 repo 上：
```
git remote add origin https://github.com/your-name/your-name.github.io.git
```
接下来就是把刚刚的提交 push 上去。
**关键点到了：**
Hexo 部署的 page 是在 master 上的，如果我们push 到 master 分支上的话，虽然程序还能运行，但是目录结构就会变得很乱了。

**我们的做法是**：把这些源文件（包括文章和配置）push到另外一个分支中。

**怎么操作呢？**

#### 1. 先新建一个分支名字叫 source：
```
git branch source
```
然后把刚刚的东西全部 push 到 source 分支上：
```
git push -u origin source
```
输入账号密码后上传，一会就成功了。

去 GitHub 上的 repo 看一下，有两个分支：
一个是 master,里面的内容是 Hexo 生成了 page 页面；
一个是 source, 里面的内容是我们的文章还有 Hexo 的配置等源文件。

这样的话，不管在什么地方都可以进行最新文章的获取和继续操作了。

当然了，修改了文章或者其他东西，在 deploy 前还是先 commit 和push 一下哦。
**记得：所有的本地操作都在 source 分支里面**。

### 下载操作

现在我们切换视觉到 一台新的电脑上，在这台电脑上没有 Node.js,没有 Hexo，没有 Git。

1. 先安装必要的程序 node.js 和 git，过程不再赘述
2. 把原来的文章下载下来
3. 配置 git ssh key，你才拥有从这台电脑发布文章的权利（请不要在公共电脑操作）
4. 然后就是走 Hexo 的发布流程，也就是上面的上传操作了

#### 具体操作如下

+ **1.把文章下载下来**

找到自己喜欢的路径，新建一个文件夹，命名随意，自己认识就好
打开 git bash
```
git clone xxxxxxxxx.xx (输入你的 github page 的 repo 地址)
```
等待下载完成之后，默认的分支是 master，master分支是 Hexo 编译之后的 网站程序，我们的文章在 source 分支内
```
git checkout source
```
这个时候文件夹内的内容已经是我们的博客源文件（包括文章和配置）

+ **2.配置 git ssh key**

还是在当前的 git bash 界面，打开 ssh-key 所在文件夹：
```
cd ~/.ssh
```
生成 ssh-key：
```
ssh-keygen -t rsa -C "your_email@youremail.com"
```
弹出一下提问，我们不修改生成路径，不设置密码短语，直接回车。
1.找到.ssh文件夹，用文本编辑器打开“id_rsa.pub”文件，复制内容到剪贴板。
*注意：*最后的空格不需要复制到（可能会引起一些问题）。
2.然后就是去 github setting 里面设置 ssh-key 了，打开 <https://github.com/settings/ssh>，点击 Add SSH Key 按钮，粘贴进去保存即可。

配置完成之后，测试一下
```
ssh -T git@github.com
```
如果返回 Hello <username> (username 表示你的用户名)，表示成功

+ **3.配置 Hexo 程序了**

安装 hexo:
```
npm install hexo-cli -g
npm install hexo --save
```

检查 hexo 是否安装成功：
```
hexo -v
```
这个时候就不做初始化了.

+ **4.自动安装需要的组件**

```
npm install
```

安装 git 部署的插件：
```
npm install hexo-deployer-git --save
```
操作完成，现在这台新电脑 就跟原来那台电脑一样操作就可以发布文章了。



