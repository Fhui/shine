---
title: Volley以及自定义Request详解
date: 2016-05-18 19:09:00
tags: Library
---

<img src="http://i.imgur.com/cN4P4Af.png" width = "600" height = "300" align=center />

<!--more-->

google在2013年I/O大会上发布了volley, 主打频繁的网络通信(小请求)。
  
本身Android就提供了HttpClient和HttpUrlConnection来进行网络请求，可是随着需求不断的提升，比如增加缓存啊，就变得更加复杂了，当然除了volley还有其他很多的优秀的网络请求。比如：OkHttp， Retrofit，xutils等等。当然，今天我们的主题还是volley。

volley:https://android.googlesource.com/platform/frameworks/volley
下载volley然后导入添加依赖就好了。

volley提供了四种请求方式:StringRequest , JsonObjectRequest, JsonArrayRequest, ImageRequest.
首先，我们在使用volley的时候先创建一个请求队列RequestQueue
		
		RequestQueue queue = Volley.newRequestQueue(this)
还要添加网络权限		
		
		<uses-permission android:name="android.permission.INTERNET"/>

#StringRequest 

StringRequest 字符串请求，其实用起来很简单。
	
	 StringRequest stringRequest = new StringRequest(int method, String url, Listener<String> listener, Response.ErrorListener errorListener);
	 
StringRequest中有两个构造参数， 其实就多了一个请求方式（int method）要么就是post请求， 要么就是get请求。其他的参数依次是:url地址, 请求成功监听，请求失败监听。下面为了更直观，我就找了一个查询经纬度接口的url做测试，代码如下:

	 StringRequest stringRequest = new StringRequest(url, new Response.Listener<String>() {
            @Override
            public void onResponse(String s) {
                LogUtils.i("info:"+s);
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError volleyError) {
                LogUtils.i("error infos :" + volleyError.toString());
            }
        });
        queue.add(stringRequest);

代码很简单，就是创建一个StringRequest，然后两个回调，最后将该请求添加到请求队列中。
下面我在给大家演示一遍StringRequest的post请求。

	  StringRequest stringRequest = new StringRequest(url, new Response.Listener<String>() {
            @Override
            public void onResponse(String s) {
                LogUtils.i("info:"+s);
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError volleyError) {
                LogUtils.i("error infos :" + volleyError.toString());
            }
        }){
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                return map;
            }
        };
        queue.add(stringRequest);

当发起post请求的时候，会调用父类getParams()方法来查找参数，我们看到参数返回的是一个map类型的，我们只需要重写该方法，然后添加map集合参数即可。


#JsonObjectRequest
	
JsonObjectRequest与JsonArrayRequest没有什么实质性的区别，都是请求json数据，无非一个是json对象，另外一个是json数组罢了， 为了节约时间，我就给大家演示JsonObjectRequest。

	 public JsonObjectRequest(int method, String url, JSONObject jsonRequest, Listener<JSONObject> listener, ErrorListener errorListener)

JsonObjectRequest在new的时候看起来和StringRequest基本类似，只不过多了一个请求参数json对象。

	JsonObjectRequest jsonObjectRequest = new JsonObjectRequest(url, null, new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject jsonObject) {
                LogUtils.i("info:"+jsonObject.toString());
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError volleyError) {
                LogUtils.i("error infos :" + volleyError.toString());
            }
        }){
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                return map;
            }
        };

        queue.add(jsonObjectRequest);

因为笔者没有找到传递json的接口，所以使用map代替，但是如果有这方面需求，其实同理， 就不需要重写父类的getParams()方法，直接将json对象作为一个参数传入方法中即可。JsonArrayRequest同理。
volley使用就是这么简单，基本都是首先创建请求队列---->创建请求方式----->添加到请求队列

#ImageRequest

其实使用volley请求图片有三种，我这边介绍两种，一种是ImageRequest, 另外一种是ImageLoader

###ImageRequest

	 ImageRequest imageRequest = new ImageRequest(url, new Response.Listener<Bitmap>() {
            @Override
            public void onResponse(Bitmap bitmap) {
                iv_show_pic.setImageBitmap(bitmap);
            }
        }, 0, 0, Bitmap.Config.RGB_565, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError volleyError) {
                LogUtils.i("error infos :" + volleyError.toString());
            }
        });
        queue.add(imageRequest);
我们首先看到ImageRequest需要传入的构造参数, 其中除了第三四五个参数或许有些不明白，其余的都和上面讲的类似，其中第三个和第四个参数就是获取到图片的宽高，写多少就是会压缩到多少，如果写0的话就是不压缩，直接显示出来，然后第五个就是图片的格式。

###ImageLoader

	  ImageLoader imageLoader = new ImageLoader(queue, new ImageLoader.ImageCache() {
            @Override
            public Bitmap getBitmap(String s) {
                return null;
            }

            @Override
            public void putBitmap(String s, Bitmap bitmap) {

            }
        });
        ImageLoader.ImageListener imageListener = ImageLoader.getImageListener(iv_show_pic, R.mipmap.ic_launcher, R.mipmap.ic_launcher);
        imageLoader.get(url, imageListener, 300, 100);
![ImageLoader](http://img.blog.csdn.net/20160518171514846)
ImageLoader的底层其实还是ImageRequest, 但是它比ImageRequest更有趣，为什么这么说呢？因为它添加了缓存功能，在当new ImageLoader的时候传入两个参数，一个是请求队列，另外一个是ImageCache（图片缓存）。
图片缓存是怎么做的呢？其实图片缓存技术就是二级缓存。它的缓存过程就是缓存到本地，在下次请求的时候本地查找，没有在请求网络，其中有两个算法，LruCache和DiskLruCache算法，有兴趣的同学可以去查查，毕竟这不是我们的主题嘛。

![这里写图片描述](http://img.blog.csdn.net/20160518172033637)

好啦，Volley最基础的用法已经学完啦，so easy不是吗
然而![这里写图片描述](http://img.blog.csdn.net/20160518172303857)
难道每次请求都需要 写这么多代码么？其实这也是volley的有点，可扩展性强。哈哈。大家可以脑补下。
**因为接下来到高潮了。**![这里写图片描述](http://img.blog.csdn.net/20160518172621655)

#**自定义Reuqest**

大家在平时开发中post请求接口无非就是两种方式嘛，一种请求提示map，另外就是json，然而服务器返回的也大多是两种，一种xml, 一种json。然而现在大多数都是json咯。
何为自定义Reqeust？就是自己写一个Reuqest，此处容我笑两声![这里写图片描述]
(http://img.blog.csdn.net/20160518173036141)

首先我们来做第一种，服务器返回的是json数据，我们开发时都知道Gson可以解析json数据，所以我们将Request和Gson封装一起，那岂不是爽死了？
为了个Gson封装一起 我们先一个javabean的基类，然后解析的时候更方便。

首先我们创建一个类来集成Reqeust泛型类，在里面初始化Gson并且重写里面的parseNetworkResponse(), deliverResponse(), 其实我们在手写自定义Reqeust的时候要扒一下源码，我们会发现都会有parseNetworkResponse()方法，so...我们发现这个方法的重要性，处理网络请求。
所以我们的主要实现还是在这里完成
		
		 try {
            String json = new String(
                    response.data,
                    HttpHeaderParser.parseCharset(response.headers));
            Log.i("GsonRequest", "content"+json);

            T result = null;
            try {
                result = gson.fromJson(json, clazz);    //解析json
                if (isCache) {
                    FileCopyUtils.copy(response.data, new File(MobileApplication.application.getCacheDir(), "" + MD5Utils.encode(getUrl())));
                }
            } catch (JsonSyntaxException e) {
                result = (T) gson.fromJson(json, ErrorResponse.class);//解析失败，按规范错误响应解析
            }  catch (IOException e) {
                e.printStackTrace();
            }
            return Response.success(
                    result,
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        }
这样基本我们自定义Request完成了，然后我们要写一个网络请求类来封装Request。然后根据刚刚自定义Request的构造器来创建一个，代码如下
		
		private static Request makeGsonRequest(int method, String url, Map<String, String> params, Class<? extends RBResponse> clazz, int requestCode, ResponseListener listener, boolean isCache) {
        HttpListener httpListener = new HttpListener(listener, requestCode, mInFlightRequests);
        GsonRequest gsonRequest = new GsonRequest<RBResponse>(method, url, params, clazz, httpListener, httpListener, isCache);
        gsonRequest.setRetryPolicy(new DefaultRetryPolicy());
        return gsonRequest;
    }


在这儿出现了一个陌生的方法 setPetryPolicy(); 这个方法是设置超时时间，这儿是写死的，但是有的需求或许长一点，比如服务器需要一个比较复杂的运算过程，那么这个超时时间就得设置长一点。这个方法就是单步封装了刚刚自定义Request里面的代码显而易见的，而还有一个HttpListener， HttpListener封装的请求的监听，里面只有成功失败，传入的是listener, 请求码(稍后再说)， 和一个map(稍后再说)。

自定义Request创建完成了，下面应该还有一层封装，为什么呢？因为volley之所以方便，它可以根据每个请求设置一个请求码，正如刚才HttpListener中的构造一样，而我为什么还要写一个HashMap呢？因为我用HashMap来保存请求记录，它的键值对的类型是Integer, request看到类型是不是有种恍然大悟的感觉？所以我们在封装一个方法来请求自定义Request, 代码如下。

	 private static Request request(int method, String url, Map<String, String> params , Class<? extends RBResponse> clazz, int requestCode, ResponseListener listener, boolean isCache) {
        Request request = mInFlightRequests.get(requestCode);
        if (request == null) {
                request = makeGsonRequest(method, url + buildParams(params), null, clazz, requestCode, listener, isCache);
            tryLoadCacheResponse(request, requestCode, listener);
            return addRequest(request, requestCode);
        } else {
            return request;
        }

大家先看代码，传入的参数先不管，因为在自定义Reqeust的时候全部都有用到，它里面做了什么？首先map先查找了一下请求码，返回的是一个reqeust, 然后判断是否空，如果空则是一个新的请求，去创建自定义Request请求，在makeGsonRequest方法中有个方法 buildParams(), 这个方法主要就是拼接map字符串, 然后又看到一个陌生的方法，tryLoadCacheResponse(), 这个方法是什么，从字面的意思是尝试缓存网络，里面传了一个request请求，请求码， 监听，这个方法里面做了什么？看代码。

	 private static void tryLoadCacheResponse(Request request, int requestCode, ResponseListener listener) {
        if (listener != null && request != null) {
            try {
                File cacheFile = new File(MobileApplication.application.getCacheDir(), "" + MD5Utils.encode(request.getUrl()));
                StringWriter sw = new StringWriter();
                FileCopyUtils.copy(new FileReader(cacheFile), sw);
                if (request instanceof GsonRequest) {
                    GsonRequest gr = (GsonRequest) request;
                    RBResponse response = (RBResponse) gr.getGson().fromJson(sw.toString(), gr.getClazz());
                    listener.onGetResponseSuccess(requestCode, response);
                }
            } catch (Exception e) {
            }
        }
    }

这个方法主要就是将数据缓存到本地，先判断请求和监听是否为空，然后将请求的url通过MD5算法编译保存到本地，然后下次去的时候会有限匹配改请求的缓存，就是办了这个一个事。

最后我们直接在封装一成我们方便调用
	
	 public static Request post(String url, Map<String, String> params, Class<? extends RBResponse> clazz, final int requestCode, final ResponseListener listener, boolean isCache) {
        return request(Request.Method.POST, url, params, clazz, requestCode, listener, isCache);
    }
自此，自定义Request post带参map已经全部完成。但是有的哥们服务器响应的是json， 怎么办呢？这个更简单，只需要修改自定义Request就好了。直接放代码。为了方便我直接集成了JsonRequest

	 protected Response<T> parseNetworkResponse(NetworkResponse response) {
        try {
            String json = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
            T result = null;
            try {
                result = gson.fromJson(json, clazz);
                if (isCache) {
                    FileCopyUtils.copy(response.data, new File(MobileApplication.application.getCacheDir(), "" + MD5Utils.encode(getUrl())));
                }
            } catch (JsonSyntaxException e) {
                result = (T) gson.fromJson(json, ErrorResponse.class);
            }  catch (IOException e) {
                e.printStackTrace();
            }
            return Response.success(result,
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
           return Response.error(new ParseError(e));
        }catch (JsonSyntaxException e){
            return Response.error(new ParseError(e));
        }
    }
典型的换汤不换药，其他基本一样，我在这么写一遍不是侮辱大家的智商么。
最后使用的时候请求成功根据传入的泛型数据返回，然而我们将Gson一起封装里面了，so..爽不爽，然后我们直接get*** 即可。

那好吧 就先这样了 
有什么问题留言撒
![这里写图片描述](http://img.blog.csdn.net/20160518190851294)