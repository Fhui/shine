---
title: AndroidSocketClient开源库的使用 
tags: Library
---

<img src="http://i.imgur.com/se1WmJg.jpg" width = "600" height = "300" align=center />

<!--more-->

我发现网上关于socket的库介绍比较少啊，难道socket太简单了，大家不喜欢做？哈哈哈![这里写图片描述](http://img.blog.csdn.net/20160519111525096)

我给大家推荐一个socketclient库，用起来特别方便，主要是源于前几天公司需要长连接，正好逛GitHub的时候发现的这个库，挺爽挺方便的，推荐给大家。

git:https://github.com/vilyever/AndroidSocketClient

下载完导入，然后再gradle中添加		
					
					repositories {
				  maven { url "https://jitpack.io" }
				  }
				  dependencies {
	  compile 'com.github.vilyever:AndroidSocketClient:1.4.1'
	}

至于用法也是非常简单
		
		 private SocketClient socketClient;
创建scoketClient.
	
		socketClient = new SocketClient(url, prot);
里面传入socketClient的url地址和端口号。
				socketClient.registerSocketDelegate(new SocketClient.SocketDelegate(){

            @Override
            public void onConnected(SocketClient client) {
                socketClient.send("hello, server !--------------------------->Android");
                socketClient.setHeartBeatMessage("hello, server !--------------------------->Android");
            }

            @Override
            public void onDisconnected(SocketClient client) {
                Log.i("Server", "timeout");
                String error = client.getCharsetName();
                Log.i("Server", "timeoutData:"+error);

            }

            @Override
            public void onResponse(SocketClient client, @NonNull SocketResponsePacket responsePacket) {
                String responseMsg = responsePacket.getMessage();
                int i = 1;
                Log.i("Server", responseMsg);
            }
        });
这个方法直接实现了socket发送信息到服务器，在onDisconnected()错误信息也可以及时返回并打印。

		socketClient.setConnectionTimeout(1000 * 15);
        socketClient.setHeartBeatInterval(1000);
        socketClient.setRemoteNoReplyAliveTimeout(1000 * 60);
        socketClient.setCharsetName("UTF-8");
        socketClient.connect();
这边设置的是超时时间，心跳包的间隔， 自动连接时间，编码格式。
是不是用起来特别爽？
因为库也是国人写的，注释都是中文，大家可以扒一扒源代码看如何实现， 毕竟学东西不只是学如何使用，还要追求原理的。

