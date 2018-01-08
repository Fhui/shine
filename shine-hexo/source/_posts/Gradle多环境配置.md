---
title: Gradle多环境配置
tags: Gradle
---

<img src="http://i.imgur.com/DUJ76kt.jpg" width = "600" height = "300" align=center />

<!--more-->

在开发中遇到了多个不同的环境, 比如测试环境, 上线环境, 甚至根据不同开发组有多个环境, 平常做法就是建一个类来存储, 然后在里面不停的注释来达到切换环境的效果, 其实gradle提供了多环境配置, 配置起来也挺简单的, 也不用之前的无脑注释了.
![这里写图片描述](http://img.blog.csdn.net/20161017185930104)
首先打开自己的gradle, 在buildType里面有当前的两个环境, 一般都是debug环境和replace环境, 如果我们需要两个环境, 比如Test环境和上线环境, 我们只需要在BuildType里面去添加自己所需要的环境即可

```
        Test {
            buildConfigField 'String', 'Base_URL', '"http://192.168.1.1"'
        }
```
如果有多个环境的话另起一行即可, 这时候gradle会自动编译一下, 但是如果你的app没有打包配置的话会报错, 错误原因就是没有进行打包签名, 这时你只需要
![这里写图片描述](http://img.blog.csdn.net/20161017190356750)
点这里进行打包操作, 生成自己应用的签名即可, 然后再点击
![这里写图片描述](http://img.blog.csdn.net/20161017190448935)
这儿来配置你刚刚生成的key-store即可
具体就是signing栏里面去添加一个配置, 配置好打开BuildType栏里面选择你刚刚配置的环境, 点击signing config去选择你刚刚配置的key即可
一定要每个都要去配置, 不然还是会报和刚才相同的错误, 找不到key
这时候基本就配置好了, 你只需要在BuildVariants里面选择不同的环境即可
![这里写图片描述](http://img.blog.csdn.net/20161017190901296)
都配置好以后, 它默认会在你配置的module里面生成一个buildConfig文件, 里面就是你刚刚配置的环境
目录:build----->generate----->source----->buildConfig----->Test---->BuildConifg.java

Test就是我刚才配置的环境名字, 
这时候就方便的很了, 我们只需要在工程里面buildConfig.Base即可
如果需要更换环境, 在BuildVariants更改即可.


