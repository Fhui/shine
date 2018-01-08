---
title: EventBus消息类型重复解决方案 
tags: Library
---

<img src="http://i.imgur.com/1FbR8Ql.png" width = "600" height = "300" align=center />

<!--more-->

我们平常在开发中各个组件进行数据交互也是常有的事，EventBus很好的解决了我们平常各组件数据交互时的代码量，但是我们在使用的时候会发现，如果每个页面发送的都是String类型的，在想接收的页面上接收也是String类型的，那么它会全部接收到，所以需要加一个判断，下面演示下解决方案吧，当然还有好多，另外还有很多类似于EventBus这种开源库，RXJava等等。




EventBus在V1.0.4的时候发布粘性事件，可以发送对象，但是需要手动注销掉。所以我们借助这个粘性事件来搞起。


我们首先创建一个Event类来存储信息 


    public T data;
    public Message msg;
    public static enum Message{
        test;
    }

    public AppEvent(){

    }

    public AppEvent(Message msg, T data){
        this.msg = msg;
        this.data = data;
    }

    public Message getMessage() {
        return msg;
    }

    public <T>T getData(){
        if(data == null){

        }
        return (T)data;
    }

    public static <T> void post(Message msg, T data){
        EventBus.getDefault().postSticky(new AppEvent(msg, data));
    }

我们放入一个枚举类， 里面存储一些信息，然后我们发送的时候直接发送该类的post方法，然后接收onEvent(Appevent event), 然后通过message来switch判断，简单方便便捷。
说白了就是加个判断呗。简单记录下。
