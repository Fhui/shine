---
title:  Mac上使用Hexo+github搭建自己的博客 
---

<img src="http://i.imgur.com/Efu1afv.png" width = "600" height = "300" align=center />

<!--more-->

推荐大家购买一个域名，这样使用起来方便的多。购买域名有好多种方式，为了避免冲突，大家自行购买。另外博主使用的是mac，所以，今天的教程主题是mac系统，不过windows系统大同小异，原理是一样的，只不过Mac集成了不少环境，变得更方便。
因为hexo基于node.js, 所以我们直接下载[nodejs](https://nodejs.org/en/)。当然也得下载[Git](https://git-scm.com/), 因为我们是结合Github吗，所以工具是少一不可。

安装好，开始安装hexo，
控制台－－> `sudo npm install -g hexo` 自动安装
找一个比较显著的位置建立一个文件夹, 楼主是强迫症，所以在applications里面创建的文件夹

```
cd /applications/hexo
```

进入这个hexo文件夹目录下，然后`hexo init`

初始化hexo，它会生成一大堆静态网站文件。

这时候在`hexo server`

它会给你一个ip地址，然后你直接copy到浏览器访问就能看到欢迎页面"hello world"了


接着输入－－> `hexo init`
最好在写一下这个指令－－> `npm install hexo-deployer-git --save`

为了防止最后部署到Github上面的时候出现 `Deployer not found: github`或者部署到服务器的时候出现错误所以最好搞一下。

好了，这个时候我们基本搞好了，其他的都先不管，我们先部署到Github上再说。前方高能预警！
![这里写图片描述](http://img.blog.csdn.net/20160627162706968)
首先我们登录Github
![这里写图片描述](http://img.blog.csdn.net/20160627162921475)
登录成功后点击创建一个新的仓库
![这里写图片描述](http://img.blog.csdn.net/20160627163145380)
输入仓库名以及描述，接着创建成功。
好，我们此时再回到终端创建私钥与我们创建的远程仓库进行关联。
我们首先输入`cd ~/.ssh`看看是否有安装过这个ssh私钥如果没有会提示没有找到文件。
其中会让你输入一次密码，四位数以上的，谨记，以后会用到。
接着输入`ssh-keygen -t rsa -C "email"`
然后它会出现这个

```
Enter file in which to save the key (/Users/HIMan/.ssh/id_rsa):
```
就是询问你的ssh保存到哪儿，以及让你输入你的id_rsa，你可以自定义id_rsa但是最好不要改变路径，要不然后面会变得特别恶心。
好了，然后打开Finder找到保存的公钥，然后打开Github仓库，点击setting，然后创建一个ssh， 然后吧公钥的内容复制进去。
这时候已经连接成功。
现在回到Github
![这里写图片描述](http://img.blog.csdn.net/20160627165322083)
打开我们创建的仓库，点击设置，找到Github pages选项，然后点击大按钮
![这里写图片描述](http://img.blog.csdn.net/20160627165451830)
然后选择一个内容体，点击create
![这里写图片描述](http://img.blog.csdn.net/20160627165543289)
然后选择主题点击publish page
这时候我们就生成完毕了。
然后Github会给我们自动创建一个分支
![这里写图片描述](http://img.blog.csdn.net/20160627165639534)
这时候我们在gh-pages里面打开设置，我们可以看到Github page已经创建完成，并且给我们生成了这个连接。
![这里写图片描述](http://img.blog.csdn.net/20160627165811003)
到此为止我们的Github page已经创建完成。
这时候我们再来到hexo。
打开终端输入指令`hexo generate`会生成一套静态网页，在你创建hexo的文件夹下面生成了一个public文件夹，里面全部都是静态网页文件。
这时候我们创建一个文件夹作为Github的仓库。博主用的不是命令而是sourcetree，比较简单，然后我们打开sourcetree
， 创建新仓库，选择添加已存在的本地仓库
![这里写图片描述](http://img.blog.csdn.net/20160627170225614)
路径就写你仓库的路径，然后创建完毕后打开，点击设置，将Github上的ssh地址拿到，
![这里写图片描述](http://img.blog.csdn.net/20160627170356193)
然后创建远程仓库，将此ssh地址输进去即可。
这时候把刚才创建的public 文件夹下的文件全部复制进去，然后更新－－推送。如果出错那就是sourcetree没有关联此ssh, 那么我们重新写命令关联。

```
ssh-add ~/.ssh/ID_rsa
```
后面跟着的就是你创建的ssh
然后推送上去，这时候在访问Github给你提供的地址就是你创建的这个hexo的index了。
现在再来说说绑定域名的问题。
域名可以在好多地方买到，我感觉最好不要在国外的网站买，怕不怕被墙？我选的是dnspod，进入官网需要注册一个，然后在里面购买一个域名，点击域名解析，如果你购买了域名它会自动给你创建，如果没有活着在别的途径购买点击添加域名即可，很简单。
然后点击添加记录，
@	A	192.30.252.153

@	A	192.30.252.154

www		CNAME	***.github.io
添加这三个地址，然后我们在回到Github里面找到我们的这个仓库，创建一个文件就叫CNAME，里面就写我们创建的域名即可。

然后漫长的等待－－－－－－－－－－－－－－－

等待解析完毕，就可以访问试试看了。





















	


