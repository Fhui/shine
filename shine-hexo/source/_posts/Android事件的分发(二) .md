---
title:  Android事件的分发(二) 
tags: Android
---

<img src="http://i.imgur.com/y1lLWjh.png" width = "600" height = "300" align=center />

<!--more-->

这是Android事件分发讲解的第二篇， 在上篇我讲过View的事件分发，感兴趣的或者没有看过的可以再去[**温习一遍**](http://blog.csdn.net/huiiiiiiii/article/details/51603720)。
一定要有不屈的精神， 哈哈。
![这里写图片描述](http://img.blog.csdn.net/20160613092042036)


今天我们来搞下ViewGroup的事件分发，这个博主感觉要比View事件分发复杂一点点，但是也不难。

#ViewGroup

上篇博客我们在讲View的事件分发的时候讲过了， 当手指触摸屏幕的时候肯定会调用dispathTouchEvent方法，那么View和ViewGroup有什么区别？我简要的说下，我们所有能看得到的控件都是直接或间接的继承自View, 比如Button，TextView等等，太多了。那么ViewGroup是什么？说白了我们常用的布局都是继承自ViewGroup, LinearLayout, RelativeLayout， 那么，当我们用到写自定义控件的时候， 感觉会更强烈， 自定义View和自定义ViewGroup, 好， 那么我们先来看看ViewGroup是如何事件分发的？

```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/btn_test"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="test_onTouch"
        android:layout_centerInParent="true"/>

    <com.customview.CustomViewGroup
        android:id="@+id/custom_group"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:clickable="true">
    </com.customview.CustomViewGroup>
</LinearLayout>
```
这是目前的布局， customViewGroup是一个自定义的LinearLayout, 间接继承了ViewGroup， 里面目前没有包含任何的View， 下面见代码

```
public class CustomViewGroup extends LinearLayout {
    public CustomViewGroup(Context context) {
        super(context);
    }

    public CustomViewGroup(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CustomViewGroup(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
}
```

```
 public class MainActivity extends AppCompatActivity {

     private static final String TAG = "ALOG";
     @BindView(R.id.btn_test)
     public Button btn_test;
     @BindView(R.id.custom_group)
     CustomViewGroup custom_group;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
    }

     @OnClick({R.id.btn_test,R.id.custom_group})
     public void click(View v){
         switch(v.getId()){
             case R.id.btn_test:
                 Log.i(TAG, "Button:onclick");
                 break;
             case R.id.custom_group:
                 Log.i(TAG, "viewGroup:onclick");
                 break;
         }
     }

     @OnTouch({R.id.btn_test,R.id.custom_group})
     public boolean touch(View v, MotionEvent event){
         switch(v.getId()){
             case R.id.btn_test:
                 switch(event.getAction()){
                     case MotionEvent.ACTION_DOWN:
                         Log.i(TAG, "Button:ACTION_DOWN");
                         break;
                     case MotionEvent.ACTION_MOVE:
                         Log.i(TAG, "Button:ACTION_MOVE");
                         break;
                     case MotionEvent.ACTION_UP:
                         Log.i(TAG, "Button:ACTION_UP");
                         break;
                 }
                 return false;
             case R.id.custom_group:
                 switch(event.getAction()){
                     case MotionEvent.ACTION_DOWN:
                         Log.i(TAG, "viewGroup:ACTION_DOWN");
                         break;
                     case MotionEvent.ACTION_MOVE:
                         Log.i(TAG, "viewGroup:ACTION_MOVE");
                         break;
                     case MotionEvent.ACTION_UP:
                         Log.i(TAG, "viewGroup:ACTION_UP");
                         break;
                 }
                 return false;
         }
         return false;
     }

 }
```
我是这样响应的， 打印的log是

```
06-13 10:26:57.450 22708-22708/ALOG: viewGroup:ACTION_DOWN
06-13 10:26:57.550 22708-22708/ALOG: viewGroup:ACTION_UP
06-13 10:26:57.550 22708-22708/ALOG: viewGroup:onclick
06-13 10:26:59.830 22708-22708/ALOG: Button:ACTION_DOWN
06-13 10:26:59.920 22708-22708/ALOG: Button:ACTION_UP
06-13 10:26:59.920 22708-22708/ALOG: Button:onclick
```
和上次几乎一样，那么我们改一改，我们在customViewGroup中添加了一个button

```
 <com.iscs.mobilewcs.customview.CustomViewGroup
        android:id="@+id/custom_group"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:clickable="true">

        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:id="@+id/btn_custom"
            android:text="btn_custom"/>

    </com.iscs.mobilewcs.customview.CustomViewGroup>
```
然后再来响应这个button的onclick和touch事件，customViewGroup中没有做任何改变。打印结果

```
06-13 11:04:52.710 23390-23390/ALOG: btn_custom:ACTION_DOWN
06-13 11:04:52.800 23390-23390/ALOG: btn_custom:ACTION_UP
06-13 11:04:52.830 23390-23390/ALOG: btn_custom:onclick
```
可以响应btn_custom的响应事件，但是缺少了customViewGroup的响应时间， 因为cunstomViewGroup是包含这个button，为什么没有响应呢？那么我们做一下改变，重写customViewGroup中的onInterceptTouchEvent方法，这个方法其实就是ViewGroup事件分发有很大关系，ViewGroup事件分发也就比View事件分发多了这么一个方法。也就是ViewGroup事件分发一共有三个方法，分别是:onTouchEvent，dispatchTouchEvent，onInterceptTouchEvent

```
@Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return super.onInterceptTouchEvent(ev);
    }
```
我们先来看看这个方法里面有什么，查看源码

```
   public boolean onInterceptTouchEvent(MotionEvent ev) {
        return false;
    }
```
哈哈，你没有看错，就一个返回值，这是做了什么？
我们看看文档怎么说的，我就用我渣渣的英语大改翻译一下
![这里写图片描述](http://img.blog.csdn.net/20160613112206016)
For as long as you return false from this function, each following，event (up to and including the final up) will be delivered first here，and then to the target's onTouchEvent()。
If you return true from here, you will not receive any following events: the target view will receive the same event but with the action {@link MotionEvent#ACTION_CANCEL}, and all further events will be delivered to your onTouchEvent() method and no longer appear here。
意思大概就是如果你返回false，事件会先传递到这儿，然后再到对应的onTouchEvent方法中，如果你返回true, 你将不会收到任何事件。
我就能看懂这么多，哎，回头赶紧去学英语。
![这里写图片描述](http://img.blog.csdn.net/20160613112930429)
意思上面已经表达清楚了，这个方法是有多么关键，因为默认返回false，那么我们重写返回true试试看。Log如下

```
06-13 11:31:18.810 13751-13751/ALOG: custom_group:ACTION_DOWN
06-13 11:31:18.870 13751-13751/ALOG: custom_group:ACTION_UP
06-13 11:31:18.870 13751-13751/ALOG: viewGroup:onclick
06-13 11:31:21.760 13751-13751/ALOG: custom_group:ACTION_DOWN
06-13 11:31:21.810 13751-13751/ALOG: custom_group:ACTION_UP
06-13 11:31:21.810 13751-13751/ALOG: viewGroup:onclick
```
我点击了一下btn_custom和custom_group但是Log上只打印custom_group，纳尼，为什么呢？
因为文档上已经写的很清楚了，当你返回true， 你将不会收到任何事件，我们来看看dispatchTouchEvent方法吧。

```
  if (disallowIntercept || !onInterceptTouchEvent(ev)) {  
            ev.setAction(MotionEvent.ACTION_DOWN);  
            final int scrolledXInt = (int) scrolledXFloat;  
            final int scrolledYInt = (int) scrolledYFloat;  
            final View[] children = mChildren;  
            final int count = mChildrenCount;  
            for (int i = count - 1; i >= 0; i--) {  
                final View child = children[i];  
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE  
                        || child.getAnimation() != null) {  
                    child.getHitRect(frame);  
                    if (frame.contains(scrolledXInt, scrolledYInt)) {  
                        final float xc = scrolledXFloat - child.mLeft;  
                        final float yc = scrolledYFloat - child.mTop;  
                        ev.setLocation(xc, yc);  
                        child.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
                        if (child.dispatchTouchEvent(ev))  {  
                            mMotionTarget = child;  
                            return true;  
                        }  
                    }  
                }  
            }  
        }  
```
在dispathTouchEvent中有这么一个判断，他判断的是disallowIntercept和!onInterceptTouchEvent的返回值判断符是||, 我们刚才onInterceptTouchEvent返回值是true， 然而判断是onInterceptTouchEvent取反， 而disallowIntercept的默认值是false，两个都是false这个判断就进不来了，直接消耗掉了。
我们在继续看源码，我们看到如果没有拦截，也就是返回false的时候，通过层层阻碍，到了这儿

```
 for (int i = count - 1; i >= 0; i--) {  
                final View child = children[i];  
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE  
                        || child.getAnimation() != null) {  
                    child.getHitRect(frame);  
                    if (frame.contains(scrolledXInt, scrolledYInt)) {  
                        final float xc = scrolledXFloat - child.mLeft;  
                        final float yc = scrolledYFloat - child.mTop;  
                        ev.setLocation(xc, yc);  
                        child.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
                        if (child.dispatchTouchEvent(ev))  {  
                            mMotionTarget = child;  
                            return true;  
                        }  
                    }  
                }  
            }  
```
上面代码判断viewgroup是否有子view，最后的

```
if (child.dispatchTouchEvent(ev))  {  
                            mMotionTarget = child;  
                            return true;  
                        }  
```

如果有的话又调用的子View的dispathTouchEvent， 其实ViewGroup也是继承自View， 这样又转到了我们前几天写的第一篇View的事件分发。
#总结
首先上一张总结图
![这里写图片描述](http://img.blog.csdn.net/20160613143320170)
事件分发传递是先传到ViewGroup的，然后判断onInterceptTouchEvent的返回值，如果是false，继续向下传递，可以看成一个递归状态，但是如果是true，意味着拦截掉事件，然后响应dispathTouchEvent, 处理Touch事件。













	


