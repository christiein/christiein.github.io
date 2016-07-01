---
layout: blog_item
title: 'volley retry和发送请求慢的问题'
date: 2016-06-23 10:00:00 +0800
author: christiein
categories: [原创]
---


>在Android项目中，经常会使用到google的volley库。然而在使用中发现，调用一次get或post方法后，服务器端居然收到了多次请求。

# 查找原因

首先，我仔细的查了代码，我的确只是调用了一次，但是服务器的确是收到了几次请求。那问题就是应该是超时重试的问题了，volley默认的超时时间是2.5秒，所有我增加超时时间到10秒，也可以按照具体的问题设置合适的超时时间。下面是设置超时时间：

```
......
request.setRetryPolicy(new DefaultRetryPolicy(
                10*1000,//默认超时时间，应设置一个稍微大点儿的
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES,//默认最大尝试次数
                DefaultRetryPolicy.DEFAULT_BACKOFF_MULT
        ));
......
```

通过这样设置以后，服务器端收到的请求次数就是一次了。但是问题没有得到根本解决，因为为什么我的请求会超时，我通过其他的工具和浏览器测试，向服务器发送请求是很快得到回应的，为什么我使用volley就这么慢，需要4秒，但是浏览器明细在一秒以内。后来通过设置volley请求的http头信息，使请求在100毫秒以内的到了服务器的回应。下面是设置http请求的头信息：

```
JsonObjectRequest request = new JsonObjectRequest(url,
                postJsonObj,
                new Response.Listener<JSONObject>() {
                    @Override
                    public void onResponse(JSONObject json) {
                        Log.i(TAG, "onResponse" + json);
                    }
                },
                new Response.ErrorListener() {
                    @Override
                    public void onErrorResponse(VolleyError volleyError) {
                        Log.e(TAG, "onError..");
                    }
                }
        ) {

            //修改默认的http头部信息，大大优化了连接时间。完全模拟一个浏览器
            @Override
            public Map<String, String> getHeaders() throws AuthFailureError {
                Map<String, String> headers = new HashMap<>();
                headers.put("connection", "keep-alive");
                headers.put("accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8");
                headers.put("user-agent", "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/33.0.1750.154 Safari/537.36");
                headers.put("accept-encoding", "gzip,deflate,sdch");
                headers.put("accept-language", "zh-CN,zh;q=0.8,en;q=0.6");
                return headers;
            }
        };
```

# 解决方案

通过设置超时时间和http请求头信息，使得volley发送请求到接收服务器回应的数据的时间从4至10秒，缩短到了几十毫秒的级别，问题得到了解决。