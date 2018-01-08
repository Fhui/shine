---
title: RXJava入门
tags: Library
---

<img src="http://i.imgur.com/LbiBFu3.png" width = "600" height = "300" align=center />

<!--more-->

现在RXJava可谓是赤手可热，没接触过的同学见了RXJava风格代码会不会吐槽？“TMD这个是什么？和屎一样”哈哈，我初次见反正吐槽了，看不明白什么意思，但是经过了解以后看到了它的魅力，简直是代码中的宋仲基啊。帅的一逼。

![这里写图片描述](http://img.blog.csdn.net/20160606133625762)

首先感谢抛物线大大谢了这么好的[**文章**](http://gank.io/post/560e15be2dca930e00da1083)，供我们充分了解RXJava，原谅博主的愚笨看了好多遍,千万不要嘲笑朕，所以做一个笔记我们一起交流。
....

![这里写图片描述](http://img.blog.csdn.net/20160606133312370)

博主总结的或许还是不是那么容易全, 但是起到一个交流的作用，见谅。
废话不多说，见正文。
老规矩----->GIT:https://github.com/ReactiveX/RxJava ,
------------------>https://github.com/ReactiveX/RxAndroid
使用只需要添加依赖就好:

```
compile 'io.reactivex:rxjava:1.0.14'
compile 'io.reactivex:rxandroid:1.0.1'
```
其实说白了RXJava就是异步的代名词， 在Android上面，好多操作都必须是异步的， 在Android的主线程中不允许有过多的耗时操作，否则系统抛出ANR异常，是的，你的程序被系统抛弃了。

我们先来说说RXJava的关键API。

Observable: 被观察者，Observer: 观察者， subscribe : 绑定观察
对，刚才忘了说了，其实这就是观察者模式，API也很直白，观察者，被观察者，订阅。一目了然。

##创建Hello, world
我们在学编程的时候首先就会写最最最基础的helloworld， 那么我先给大家吧RXJava版的HelloWorld写出来，让大家过目。

```
 Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("hello, world.");
                subscriber.onCompleted();
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i(TAG, "onNext:"+s);
            }
        });
```
![这里写图片描述](http://img.blog.csdn.net/20160606140523108)
尼玛，我只不过输出依据hello,world竟然要这么多行，还让不让活了？
看官，且听我慢慢讲解。
上面代码就是举得最简单的例子，当然现实中写代码肯定不会这样的嘛，特么一个hello， world也要用rxjava写，除非你是RXJava骨灰粉丝，要不然你就是神经病，为了让大家容易快速掌握RXJava的使用用法，所以例子都会比较简单的。
代码很容易理解，创建一个Observable然后实现OnSubcribe中的call方法，这个方法中的参数是
Subscriber subscriber  那么subcriber是什么？
它是Observer中内置的抽象类罢了，但是重点来了，每个subcriber都会被转成Observer来使用的，所以他们都可以实现功能只不过subcriber比Observer多了几个方法而已，所以写代码的时候可以直接写subcriber.当Observable被订阅的时候 自动触发call()然后通过onNext()来触发，最后调用onCompleted(), 然后再observer中的onNext()方法来展示结果。
是不是特别简单？那么我们再来学习RXJava提供的另外几个更方便的API
Observable

```
  Observable.just("hello", "world").subscribe(new Observer<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i(TAG, "onNext:"+s);
            }
        });
```
前面突然出现了just(...)方法，其实Observable.just("hello", "world")就是等同于subcriber.onNext("hello");subcriber.onNext("world");subcriber.onCompleted();是不是简化了许多？还有一种是操作数组的方法from()

```
String[] strArray = {"hello", "world"};
        Observable.from(strArray).subscribe(new Subscriber<String>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                Log.i(TAG, "onNext:"+s);
            }
        });
```
操作起来都是一样的，非常的简单。好了，最基础的hello,world已经搞定了，下面我们继续搞一下filter， map， flatMap.

##flter, map, flatMap

首先我们来看看flter方法。flter翻译过来的意思是滤过，其实它在rxjava中的含义也差不多，过滤嘛。我们来写个Demo玩玩。

```
String[] params = {"android", "ios", "java", "c", "c++"};
        Observable.from(params)
                .filter(new Func1<String, Boolean>() {
                    @Override
                    public Boolean call(String s) {
                        return s.length() > 3;
                    }
                })
                .subscribe(new Subscriber<String>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(String s) {
                        Log.i(TAG, "onNext:" + s);
                    }
                });
```
运行结果----->
`06-06 15:34:38.468 10201-10201/ALOG: onNext:android
06-06 15:34:38.468 10201-10201/ALOG: onNext:java`

上面代码就是过滤params数组中字符串长度小于3的参数，然后再onNext方法中打印出来, 使用filter过滤是不是挺爽的。
下面我们看看map， map可以起到变换的作用，超吊的哦。

```
  Observable.just(1, 2, 3, 4, 5)
                .filter(new Func1<Integer, Boolean>() {
                    @Override
                    public Boolean call(Integer integer) {
                        return integer > 3;
                    }
                })
                .map(new Func1<Integer, String>() {
                    @Override
                    public String call(Integer integer) {
                        return integer+"rxJava";
                    }
                })
                .subscribe(new Subscriber<String>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(String s) {
                        Log.i(TAG, "onNext:"+s);
                    }
                });
```
打印的结果是:

```
06-06 15:50:11.688 23667-23667/ALOG: onNext:4rxJava
06-06 15:50:11.688 23667-23667/ALOG: onNext:5rxJava
```
从代码可以看出，首先just进去五个数字，然后通过filter过滤大于3的，然后再通过map把过滤后的添加字符串'rxjava'，在onNext方法打印输出.是不是map也是非常简单的，感觉到rxjava的强大了吧。我们再来看看flateMap

```
   Observable.create(new Observable.OnSubscribe<Person>() {
           @Override
           public void call(Subscriber<? super Person> subscriber) {
                subscriber.onNext(Person.createPerson());
           }
       }).flatMap(new Func1<Person, Observable<Person.PersonProperty>>() {
           @Override
           public Observable<Person.PersonProperty> call(final Person person) {
               return Observable.create(new Observable.OnSubscribe<Person.PersonProperty>() {
                   @Override
                   public void call(Subscriber<? super Person.PersonProperty> subscriber) {
                       for(int i = 0;i < person.getList().size(); i++){
                           subscriber.onNext(person.getList().get(i));
                       }
                   }
               });
           }
       }).subscribe(new Subscriber<Person.PersonProperty>() {
           @Override
           public void onCompleted() {

           }

           @Override
           public void onError(Throwable e) {

           }

           @Override
           public void onNext(Person.PersonProperty personProperty) {
               Log.i(TAG, "onNext:"+personProperty);
           }
       });
```

打印结果：

```
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=100, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=101, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=102, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=103, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=104, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=105, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=106, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=107, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=108, name='张三'}
06-06 16:54:12.288 15529-15529/ALOG: onNext:PersonProperty{sex='男', age=109, name='张三'}
```

其实我就是创建了一个javaBean， 然后打印姓名以及和这个人的属性， flatMap变换比较难理解，其实就是返回的是一个Obsverable对象，当创建一个Obsverable的时候，他会将里面的参数全部读取出来，然后统一发给Subcribe去打印，其实在上面的代码中我为了大家更容易理解我没有写Obsverable.from，而是利用for循环打印，这样更好理解，更容易看清变换的过程。

##线程控制Scheduler 

Schedulers.immediate(): 直接在当前线程运行，默认。
Schedulers.newThread(): 创建一个线程在新线程执行操作。
Schedulers.io(): 其实就是里面含有一个无数量上限的线程池，可以重用空闲的线程。
Schedulers.computation(): 一般进行复杂计算所使用的线程。AndroidSchedulers.mainThread():在主线程中运行。

##结束语
也是谢了不少内容，RXJava的基本内容反正是在这儿了，还有很多没有列举出来，大家可以多实践， 我们多多交流。


#**共勉**。





	


