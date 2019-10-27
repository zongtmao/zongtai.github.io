---
title: hexo + git博客搭建
date: 2019-10-26 19:24:19
tags:
	- hexo
	- 分享
	- 总结
categories: 博客
---

# hexo + Github 博客搭建

## #前言#
 	17年开始做前端的时候搭建了自己的第一个git博客，当时只是简单的玩了下，域名也没有申请备案，这也算为自己的懒惰找个借口吧，再没有花精力弄过，现在域名备案成功！想从新搭建下博客，网上也有一堆详细教程。我在此稍稍总结一下具体的搭建步骤，并且修改博客源码的一些坑和经验分享给大家，希望有所帮助！！废话不多说，开始正题~

首先是搭建博客用到的框架，我选用基于node.js 的Hexo，可以直接用Markdown语法撰写博客，写好部署到git上就行

## Node.js
	node.js可以直接去官网down，推荐使用当前稳定版本，以前安装过的，记着更新下版本，版本过低，hexo安装后执行命令可能会报错（坑已踩过），下载好以后机械式安装就好，这里不做过多介绍，最后通过
	```
	node -v
	```
查看下node版本信息，如果有版本信息说明安装成功了

## Git安装
为了让我们本地开发的博客能在外网上访问，我们借助GitHub，安装教程网上很多，自己搜下，安装完成后在命令提示符中输入
```
git -v
```
验证是否安装成功；安装成功后自己按照教程注册一个属于你自己的GitHub账号（作为程序猿，git应该是要有的哦~）
然后新增一个项目：如下图
![新增工程](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/new_project.png)
然后如下图所示，输入自己的项目名字，后面一定要加.github.io后缀，README初始化最好也要勾上
![工程信息](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/project_mess.png)

项目就建成了，点击Settings，向下拉到最后有个GitHub Pages，点击Choose a theme选择一个主题。然后等一会儿，再回到GitHub Pages，会变成下面这样：
![gitHub Pages](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/git_pages.png)

点击标红的链接，就可以看到自己的网页信息啦，效果图如下：
![博客网上效果](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/hexo_show.png)

## 安装hexo
在本地电脑硬盘在新增一个文件夹用来存放博客源码文件，如我的在 F:\blog_manage\myBlog目录下，点击鼠标右键，点击：Git Bash Here 打开git控制窗口，也可使用window自带的控制台

在myBlog根目录下，输入npm install hexo-cli -g 安装Hexo ,如果有提示，忽略，安装完成后输入hexo -v 验证是否安装成功！

然后就要初始化我们的网站，输入hexo init初始化文件夹，接着输入npm install安装依赖，这样本地的网站配置也弄好啦，hexo g生成静态网页，然后输入hexo s打开本地服务器，然后浏览器打开http://localhost:4000/，就可以看到新建的本地博客了，看到下图说明本地博客OK了：
![目录](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/mulu.png)
![hexo本地效果](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/xiaoguo.png)

按ctrl+c关闭本地服务器。
> 备注：如果4000端口被占用，个人认为最简单方法：可以通过node_modules\hexo-server\index.js文件，可以修改默认的port值，重新hexo s 启动项目

## Git ssh配置
首先设置好git全局配置name和email
```
git config --global user.name "xxx"
git config --global user.email "xxx@qq.com"
```
然后生成ssh key:
```
ssh-keygen -t rsa -C "xxx@qq.com"
```
控制台中输入cat ~/.ssh/id_rsa.pub将输出的内容复制到框中，点击确定保存
最后输入ssh -T git@github.com 如果出现下图所示，说明成功了
![ssh](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/git_ssh.png)
打开博客根目录下的_config.yml文件，这是博客的配置文件，在这里你可以修改与博客相关的各种信息,修改最后一行的配置：
deploy:
  type: git
  repository: https://github.com/zongtmao/zongtai.github.io  //自己的地址
  branch: master
> 注意：type repository branch前面两个空格，后面一个，格式错误的话也是部署不成功！！

## 发布文章
首先在博客根目录下右键打开git bash，安装一个扩展npm i hexo-deployer-git。

然后输入hexo new post "test"，新建一篇文章。

然后打开D:\study\program\blog\source\_posts的目录，可以发现下面多了一个文件夹和一个.md文件，一个用来存放你的图片等数据，另一个就是你的文章文件啦。

编写完markdown文件后，根目录下输入hexo g生成静态网页，然后输入hexo s可以本地预览效果，最后输入hexo d上传到github上。这时打开你的github.io主页就能看到发布的文章啦

## 域名绑定
现在博客默认的域名还是xxx.github.io,首先你要有备案成功的域名，我已我的阿里云为例，如下图，添加两条解析记录
![域名解析](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/yuming.png)
然后打开你的github博客项目，点击settings，拉到下面Custom domain处，填上你自己的域名，保存
![域名绑定](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/success.png)

## 个性主题设置
接下来，就是根据自己的喜好，设置自己喜欢的博客主题，本人已Next主题为例
[Next主题文档链接](https://theme-next.iissnan.com/)

### 下载主题
git bash 到博客根目录，输入命令
```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
此时themes文件夹下多了一个next文件夹：
F:\blog_manage\myBlog\themes\next
说明主题下载成功！

### 启用主题
修改根目录下_config.yml文件，将theme改成next,重新启动可以看到效果

![NEXT](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/theme_next.png)

### 主题设定
定位到F:\blog_manage\myBlog\themes\next文件夹下，打开next文件夹下的_config.yml文件，关键词定位到Scheme，根据自己爱好修改下面的主题：
![主题](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/scheme.png)

其他的个性化配置都可按文档类似配置，比如留言，搜索，标签，添加git入口之类的就不一一介绍了，看下最终的效果图吧：
![博客](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/my_blog.png)

