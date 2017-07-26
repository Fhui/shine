---
title: 探寻Android中MVP设计模式
---

<img src="http://i.imgur.com/8KhioGQ.png" width = "600" height = "300" align=center />

<!--more-->

Android中mvp现在炙手可热的设计模式之一，在讲解mvp模式之前我们先看看图。
![这里写图片描述](http://img.blog.csdn.net/20160526115303276)
此图是我从[泡网](http://www.jcodecraeer.com/)上拔下来的，如果不可以这么做请联系我，我会删除的， 谢谢。

MVP是MVC升级来的， 如果有对MVC不了解的同学请自行百度， 我在这儿就说下MVC的缺点， android其实并没有标准的mvc模式， 而我们习惯性的把Activity当作Controller去使用， 而且在MVC中是允许M层和V层有交互的， 然而我们把一些逻辑写到controller中会使activitry或者fragment中代码巨多， 增加了代码的维护性，没准下一个开发者去维护代码是都要崩溃了![这里写图片描述](http://img.blog.csdn.net/20160526131655850)
下面我们进入今天的主题---------->MVP
在MVP中， 每个字母都代表着什么？因为它是从MVC升级而来， 所以除了P其余都相同

 - M:Module层， 也就是我们平常写的模型(JavaBean), 业务逻辑也是在这儿。
 - V:View层， Activity， 自定义的View等等
 - P:Presenter层，相当于中间人的角色，module去和view进行交互都是通过presenter

这样mvp三层已经介绍完了，当然它的优缺点也有很多，我就不在这儿一一列举的， 我们今天主要是学如何使用mvp。当然， 我们要以代码的形式体现，这次的代码就是简单的点击按钮展示json数据， url和以前我讲volley的一样， 都是获取经纬度，返回json数据，而网络请求用的是volley。volley是google在2013年推出的网络请求框架，有不懂得同学可以去看我的另一篇[博客](http://blog.csdn.net/huiiiiiiii/article/details/51446511).
![这里写图片描述](http://img.blog.csdn.net/20160526132901604)
下面我们首先来展示的就是**Module层**
在下手之前我们要思考一下我们展示数据都需要什么， 首先展示数据是一个动作，所以我们定义一个接口showData， 然后里面写一个方法load, 它需要什么参数呢？我们展示数据其实就是请求网络接口，然后把它给我们返回的数据展示出来。所以我们需要一个url和一个加载数据的监听(接口)。

```
public interface ILoadListener {


    void loadSuccess(Data data);
    void loadError();

}

```

```
public interface IShowData {

    Data load(String url, ILoadListener listener);

}

```

```
public class LoadData implements IShowData{

    private Data data;

    @Override
    public Data load(String url, final ILoadListener listener) {


        HttpLoader.get(url, Data.class, 200, new ResponseListener() {
            @Override
            public void onGetResponseSuccess(int requestCode, RBResponse response) {
                data = (Data) response;
                listener.loadSuccess(data);
            }

            @Override
            public void onGetResponseError(int requestCode, VolleyError error) {
                listener.loadError();
            }
        }, true);

        return data;
    }
}
```
上文已经说过， 我是用的是volley+gson封装，返回的就是一个javabean， 所以很是方便。我们从代码中可以看到在监听成功的接口中是有参数的，为什么呢？继续往下看![这里写图片描述](http://img.blog.csdn.net/20160526134308987)

下面我们展示view层。
首先xml代码很是简单

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="startLoad"
        android:text="load"/>

    <TextView
        android:id="@+id/tv_show_load_data"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="waiting for load data ."/>


</LinearLayout>
```
在加载数据的时候需要设置url, 然后加载成功了会做什么，加载失败或做什么，所以我定义了接口中有三个方法
```
public interface ILoadData {

    String getUrl();
    void successFor(Data data);
    void errorFor();

}
```

```
public class LoadDataActivity extends AppCompatActivity implements ILoadData{

    private String url = "http://gc.ditu.aliyun.com/geocoding?a=苏州市";
    private TextView tv_show_load_data;
    private LoadPresenter loadPresenter = new LoadPresenter(this);

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_load);
        initView();
    }

    public void initView(){
        tv_show_load_data = (TextView) findViewById(R.id.tv_show_load_data);
    }

    public void startLoad(View view){
        loadPresenter.startLoad();
    }

    @Override
    public String getUrl() {
        return url;
    }

    @Override
    public void successFor(Data data) {
        tv_show_load_data.setText(data+"");
    }

    @Override
    public void errorFor() {
        tv_show_load_data.setText("load error .");
    }
}
```
最后我们展示的是**Prenenter**层放出代码

```
public class LoadPresenter {

    private IShowData showData;
    private ILoadData loadData;

    public LoadPresenter(ILoadData loadData){
        this.loadData = loadData;
        showData = new LoadData();

    }


    public void startLoad(){
        showData.load(loadData.getUrl(), new ILoadListener() {
            @Override
            public void loadSuccess(Data data) {
                loadData.successFor(data);
            }

            @Override
            public void loadError() {
                loadData.errorFor();
            }
        });
    }
}
```
我们创建了先前的两个接口的实例，然后再构造器中初始化， 然后写一个load方法来供module和view层进行数据交互。
到此为止， mvp版的展示数据就已经写完了，有的同学就说了，好麻烦啊， 仅仅是一个展示数据就写如此多的接口，![这里写图片描述](http://img.blog.csdn.net/20160526135957650)
对，兄弟，你说对了，就是比较麻烦，但是总比一个类上千行爽吧？这样层次分明，但是写起来要费一些时间，如果你有充分的时间和人力，并且提高代码的可维护性，mvp模式确实是挺好的选择，另外推荐下retrofit2+rxjava+mvp会让你一直高潮的，有兴趣的同学了解下吧。




	


