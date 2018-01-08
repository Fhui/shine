---
title:  关于ListView数据显示错乱的解决方案  
tags: Android
---

<img src="http://i.imgur.com/lkqd2pb.png" width = "600" height = "300" align=center />

<!--more-->


我们在平常开发Android中经常用到listview, 然而, 这个空间如果不进行优化的写法, 是非常吃内存的. 但是google在2013年IO大会上提出的viewholder写法显然已经是现在的优化标准了, 然而这个优化带来的烦恼也挺多的.
首先使用viewholder优化其实就是复用的创建好的item, 然后如果我们在创建好的item上面现实图片或者是在条目上做一些操作显然BUG是挺多的, 比如:我们listview显示的item里面是从网络获取的图片, 那么我们用viewholder优化后, 滑动listview图片会错乱, 会闪烁; 再比如, 我们常见的点赞操作, 我点击某一个条目, 它会给该条目设置一个标识, 然后当我在滑动的时候, 你会发现很奇怪, 怎么下面的item也会有该标识.
![这里写图片描述](http://img.blog.csdn.net/20160830185625962)

所以我们为了让listview做到尽量节约内存, 我们还是使用这个优化方案, 反之, 他出现的bug也要解决掉.

如果是网络请求获取图片显示在listview上面, 我们可以设置标签, 也就是tag, 可以使用他的URL来做他的标签, 然后加一个 if, else判断, 假如我们要在item上面做一些操作,
 比如加载的是本地的图片, 假设我现在点击listview的条目, 要在条目上显示一个图片 类似于空间微信点赞操作, 那么我们应该怎么做?
 ![这里写图片描述](http://img.blog.csdn.net/20161004154721234)

```
package com.himan.blogdemo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.ListView;
import android.widget.RelativeLayout;
import android.widget.TextView;

import org.w3c.dom.Text;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {


    private ListView lv_test;
    private List<CheckBean> strList;
    private StrListAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        lv_test = (ListView) findViewById(R.id.lv_test);
        strList = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            strList.add(new CheckBean(false, "item"+i));
        }
        adapter = new StrListAdapter();
        lv_test.setAdapter(adapter);
    }


    class StrListAdapter extends BaseAdapter {

        ViewHolder holder;

        @Override
        public int getCount() {
            return strList.size();
        }

        @Override
        public Object getItem(int position) {
            return strList.get(position);
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(final int position, View convertView, ViewGroup parent) {
            if (convertView == null) {
                convertView = View.inflate(MainActivity.this, R.layout.list_item_layout, null);
                holder = new ViewHolder(convertView);
                convertView.setTag(holder);
            } else {
                holder = (ViewHolder) convertView.getTag();
            }
            final CheckBean bean = strList.get(position);
            if(bean.isClick()){
                holder.iv_check.setVisibility(View.VISIBLE);
            }else{
                holder.iv_check.setVisibility(View.GONE);
            }

            holder.tv_content.setText(strList.get(position).getStr());

            holder.rl_items.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    bean.setClick(!bean.isClick());
                    notifyDataSetChanged();
                }
            });

            return convertView;
        }
    }


    class ViewHolder {
        TextView tv_content;
        ImageView iv_check;
        RelativeLayout rl_items;

        public ViewHolder(View view) {
            tv_content = (TextView) view.findViewById(R.id.tv_content);
            iv_check = (ImageView) view.findViewById(R.id.iv_check);
            rl_items = (RelativeLayout) view.findViewById(R.id.rl_items);
        }
    }
}
```
这个是MainActivity代码, List的类型正是我创建的bean类的类型

```
package com.himan.blogdemo;

/**
 * Created by HIMan on 2016/10/4.
 */

public class CheckBean {

    private boolean isClick;
    private String str;

    public CheckBean(boolean isClick, String str) {
        this.isClick = isClick;
        this.str = str;
    }

    public String getStr() {
        return str;
    }

    public void setStr(String str) {
        this.str = str;
    }

    public boolean isClick() {
        return isClick;
    }

    public void setClick(boolean click) {
        isClick = click;
    }
}
```
在convertView里面做判断的时候有if同时也要有else, 这样不会有bug也会更严谨一些, 另外如果加载网络图片的话使用第三方库好多都有带防止图片错乱机制, 如果没有的话, 我们直接可以setTag(), 然后在根据getTag()来判断刚才的tag值, 可以用URL作为tag, 用法上大同小异.

[-------代码下载--------](http://download.csdn.net/detail/huiiiiiiii/9645503)































	


