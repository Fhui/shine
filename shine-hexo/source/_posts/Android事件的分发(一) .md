---
title:  Android事件的分发(一) 
---

<img src="http://i.imgur.com/y1lLWjh.png" width = "600" height = "300" align=center />

<!--more-->

Android的事件分发是一个重点，同样也是一个难点，用到的地方非常的多，比如复杂的listview和srcoview嵌套， 带有侧拉功能的listView和viewPager相结合，listView下拉刷新和viewPager，很多个例子都是需要事件分发的，碰巧这个玩意又比较复杂，所以我们就来聊一聊Android的事件分发。

![这里写图片描述](http://img.blog.csdn.net/20160607135247766)

#View的事件分发

View事件的分发相对ViewGroup的分发来说还是比较简单的。我们先从View事件分发先来说起。上代码---------->

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/btn_test"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="test_onTouch"
        android:layout_centerInParent="true"/>
</RelativeLayout>
```
xml布局很简单，就有一个button, 我们再来看java代码

```
 public class MainActivity extends AppCompatActivity implements View.OnClickListener, OnTouchListener{

     private static final String TAG = "MainActivity";
     private Button btn_test;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
    }


     public void initView(){
         btn_test = (Button) findViewById(R.id.btn_test);
         btn_test.setOnClickListener(this);
         btn_test.setOnTouchListener(this);
     }

     @Override
     public void onClick(View v) {
         Log.i(TAG, "onclick:btn_test");
     }

     @Override
     public boolean onTouch(View v, MotionEvent event) {
         switch(event.getAction()){
             case MotionEvent.ACTION_DOWN:
                 Log.i(TAG, "ACTION_DOWN");
                 break;
             case MotionEvent.ACTION_MOVE:
                 Log.i(TAG, "ACTION_MOVE");
                 break;
             case MotionEvent.ACTION_UP:
                 Log.i(TAG, "ACTION_UP");
                 break;
         }
         return false;
     }
 }
```
我在点击事件上面和touch时间上面分别打上了tag，我们运行一下看看。

```
06-07 14:33:08.458 25345-25345/MainActivity: ACTION_DOWN
06-07 14:33:08.528 25345-25345/MainActivity: ACTION_UP
06-07 14:33:08.528 25345-25345/MainActivity: onclick:btn_test
```
运行结果是down, up, onclick。意料之中。
![这里写图片描述](http://img.blog.csdn.net/20160607144749656)
我们再看看上面的onTouch方法，如果把返回值改成true会发生什么？

```
06-07 14:51:06.638 25345-25345/MainActivity: ACTION_DOWN
06-07 14:51:06.698 25345-25345/MainActivity: ACTION_UP
```
咦，好像少了一个LOG日志打印欸，What's wrong? 这我们就得慢慢聊一聊onTouch这个方法。
我们注册touch事件调用了这个方法setOnTouchListener(), 正如所有的可见控件都继承了view， 所以我们去直接去它的祖先类中view找setrOnTouchListener(), 我们看下Android2.2的源码，因为高版本的代码太多了，更复杂 不容易理解，所以我选择android2.2的源码。

```
 public void setOnTouchListener(OnTouchListener l) {  
    mOnTouchListener = l;  
}  
```

找到了，和我们使用完全一样，传入一个OnTouchListener, 然后在内部赋值给mOnTouchListener， 其实mOnTouchListener是一个接口， 我们可以看到mOnTouchListener在dispathTouchEvent()方法中被调用到了， 我来说下这个dispathTouchEvent方法， 当我们触摸到手机上的控件的时候，会调用这个方法，dispatchTouchEvent()方法，

```
 public boolean dispatchTouchEvent(MotionEvent event) {  
    if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
            mOnTouchListener.onTouch(this, event)) {  
        return true;  
    }  
    return onTouchEvent(event);  
}  
```
我们来观摩下mOnTouchListener里面的判断条件， mOnTouchListener！= null， 我们刚才看到mOntouchListener是什么东西了， 就是我们是否激活onTouchListener方法， 然后第二个条件(mViewFlags & ENABLED_MASK) == ENABLED这个说白了就是空间是否是可用的，肯定是true， 最后这个条件mOnTouchListener.onTouch(this, event)， 是不是有点面熟， mOnTouchListener中的onTouch方法， 刚才有说过mOnTouchListener是个接口， 那么我们实现了这个方法的时候重写了该方法，关键就是它的返回值了，我们刚才测试， 如果返回值为true， 那么我们的onclick事件就不会响应了，这意味着该事件被消耗掉了。所以，当这三个条件都满足的时候返回true, 反之 调用onTouchEvent， 那么我们继续看看onTouchEvent方法。

```
 public boolean onTouchEvent(MotionEvent event) {  
    final int viewFlags = mViewFlags;  
    if ((viewFlags & ENABLED_MASK) == DISABLED) {  
        // A disabled view that is clickable still consumes the touch  
        // events, it just doesn't respond to them.  
        return (((viewFlags & CLICKABLE) == CLICKABLE ||  
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));  
    }  
    if (mTouchDelegate != null) {  
        if (mTouchDelegate.onTouchEvent(event)) {  
            return true;  
        }  
    }  
    if (((viewFlags & CLICKABLE) == CLICKABLE ||  
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {  
        switch (event.getAction()) {  
            case MotionEvent.ACTION_UP:  
                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;  
                if ((mPrivateFlags & PRESSED) != 0 || prepressed) {  
                    // take focus if we don't have it already and we should in  
                    // touch mode.  
                    boolean focusTaken = false;  
                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {  
                        focusTaken = requestFocus();  
                    }  
                    if (!mHasPerformedLongPress) {  
                        // This is a tap, so remove the longpress check  
                        removeLongPressCallback();  
                        // Only perform take click actions if we were in the pressed state  
                        if (!focusTaken) {  
                            // Use a Runnable and post this rather than calling  
                            // performClick directly. This lets other visual state  
                            // of the view update before click actions start.  
                            if (mPerformClick == null) {  
                                mPerformClick = new PerformClick();  
                            }  
                            if (!post(mPerformClick)) {  
                                performClick();  
                            }  
                        }  
                    }  
                    if (mUnsetPressedState == null) {  
                        mUnsetPressedState = new UnsetPressedState();  
                    }  
                    if (prepressed) {  
                        mPrivateFlags |= PRESSED;  
                        refreshDrawableState();  
                        postDelayed(mUnsetPressedState,  
                                ViewConfiguration.getPressedStateDuration());  
                    } else if (!post(mUnsetPressedState)) {  
                        // If the post failed, unpress right now  
                        mUnsetPressedState.run();  
                    }  
                    removeTapCallback();  
                }  
                break;  
            case MotionEvent.ACTION_DOWN:  
                if (mPendingCheckForTap == null) {  
                    mPendingCheckForTap = new CheckForTap();  
                }  
                mPrivateFlags |= PREPRESSED;  
                mHasPerformedLongPress = false;  
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());  
                break;  
            case MotionEvent.ACTION_CANCEL:  
                mPrivateFlags &= ~PRESSED;  
                refreshDrawableState();  
                removeTapCallback();  
                break;  
            case MotionEvent.ACTION_MOVE:  
                final int x = (int) event.getX();  
                final int y = (int) event.getY();  
                // Be lenient about moving outside of buttons  
                int slop = mTouchSlop;  
                if ((x < 0 - slop) || (x >= getWidth() + slop) ||  
                        (y < 0 - slop) || (y >= getHeight() + slop)) {  
                    // Outside button  
                    removeTapCallback();  
                    if ((mPrivateFlags & PRESSED) != 0) {  
                        // Remove any future long press/tap checks  
                        removeLongPressCallback();  
                        // Need to switch from pressed to not pressed  
                        mPrivateFlags &= ~PRESSED;  
                        refreshDrawableState();  
                    }  
                }  
                break;  
        }  
        return true;  
    }  
    return false;  
}  
```
看完刚才那个方法再看这个，是不是感觉突然代码量大多了，不过没关系，我们只需要看这个条件判断下的就好了
` if (((viewFlags & CLICKABLE) == CLICKABLE ||  
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE))`
判断成立后直接进入switch语句中

```
 switch (event.getAction()) {  
            case MotionEvent.ACTION_UP:  
                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;  
                if ((mPrivateFlags & PRESSED) != 0 || prepressed) {  
                    // take focus if we don't have it already and we should in  
                    // touch mode.  
                    boolean focusTaken = false;  
                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {  
                        focusTaken = requestFocus();  
                    }  
                    if (!mHasPerformedLongPress) {  
                        // This is a tap, so remove the longpress check  
                        removeLongPressCallback();  
                        // Only perform take click actions if we were in the pressed state  
                        if (!focusTaken) {  
                            // Use a Runnable and post this rather than calling  
                            // performClick directly. This lets other visual state  
                            // of the view update before click actions start.  
                            if (mPerformClick == null) {  
                                mPerformClick = new PerformClick();  
                            }  
                            if (!post(mPerformClick)) {  
                                performClick();  
                            }  
                        }  
                    }  
                    if (mUnsetPressedState == null) {  
                        mUnsetPressedState = new UnsetPressedState();  
                    }  
                    if (prepressed) {  
                        mPrivateFlags |= PRESSED;  
                        refreshDrawableState();  
                        postDelayed(mUnsetPressedState,  
                                ViewConfiguration.getPressedStateDuration());  
                    } else if (!post(mUnsetPressedState)) {  
                        // If the post failed, unpress right now  
                        mUnsetPressedState.run();  
                    }  
                    removeTapCallback();  
                }  
                break;  
            case MotionEvent.ACTION_DOWN:  
                if (mPendingCheckForTap == null) {  
                    mPendingCheckForTap = new CheckForTap();  
                }  
                mPrivateFlags |= PREPRESSED;  
                mHasPerformedLongPress = false;  
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());  
                break;  
            case MotionEvent.ACTION_CANCEL:  
                mPrivateFlags &= ~PRESSED;  
                refreshDrawableState();  
                removeTapCallback();  
                break;  
            case MotionEvent.ACTION_MOVE:  
                final int x = (int) event.getX();  
                final int y = (int) event.getY();  
                // Be lenient about moving outside of buttons  
                int slop = mTouchSlop;  
                if ((x < 0 - slop) || (x >= getWidth() + slop) ||  
                        (y < 0 - slop) || (y >= getHeight() + slop)) {  
                    // Outside button  
                    removeTapCallback();  
                    if ((mPrivateFlags & PRESSED) != 0) {  
                        // Remove any future long press/tap checks  
                        removeLongPressCallback();  
                        // Need to switch from pressed to not pressed  
                        mPrivateFlags &= ~PRESSED;  
                        refreshDrawableState();  
                    }  
                }  
```
这里面有几个状态， 我们想一下刚才打印的log， 首先是down, 其次是up， 最后才是onclick， 那我们找一下up，`performClick();`  咦， 这是什么方法？好可疑，我们点进去看看

```
 public boolean performClick() {  
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);  
    if (mOnClickListener != null) {  
        playSoundEffect(SoundEffectConstants.CLICK);  
        mOnClickListener.onClick(this);  
        return true;  
    }  
    return false;  
}  
```
有特么看到mOnclickListener了，阴魂不散啊， 这次判断就很简单了，只要mOnclickListner ！= null就会调用它的onClick方法，返回true。
看到这儿是不是瞬间脑动大开了，哈哈，把我累够呛。 最后在上一张流程图。
![这里写图片描述](http://img.blog.csdn.net/20160608111144447)








	


