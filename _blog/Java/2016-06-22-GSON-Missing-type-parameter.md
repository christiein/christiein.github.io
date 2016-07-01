---
layout: blog_item
title: 'Android使用GSON后，混淆运行程序出现异常：Missing type parameter'
date: 2016-06-22 18:00:00 +0800
author: christiein
categories: [原创]
---


>在Android或Java项目中，经常会使用到google的GSON这个十分方便的json库。然而在Android混淆打包后，会出现一个Missing type parameter的问题，百思不得其解。这里不得不说<http://stackoverflow.com/>是个好网站，很多问题都在上面找到了解决答案。

# 我的配置

首先，在使用GSON的时候，我们知道它是使用了Java的反射机制来实现了JSON的解析，它通过类的字段（属性，failed）来匹配json的解析。由此我们知道，我们的json数据类的字段是不能混淆的；再则gson属于第三方库，我们也没有必要混淆。那么我加入以下的配置，保证gson的解析数据类的字段不混淆，gson库不混淆。

```
......
##---------------Begin: proguard configuration for Gson  ----------
-keep class com.google.gson.** {
    *;
}

-keepclassmembers class com.christiein.json.beans.** {
    <fields>;
}

##---------------End: proguard configuration for Gson  ----------
......
```

# 出现问题

然而当混淆后，运行apk出现了下面的问题：

```java
11-15 01:46:26.818: W/System.err(21810): java.lang.RuntimeException: Missing type parameter.
11-15 01:46:26.828: W/System.err(21810):    at da.<init>(Unknown Source)
11-15 01:46:26.828: W/System.err(21810):    at gc.<init>(Unknown Source)
11-15 01:46:26.828: W/System.err(21810):    at fx.f(Unknown Source)
11-15 01:46:26.828: W/System.err(21810):    at com.yourshows.activity.UnwatchedActivity.onResume(Unknown Source)
```

# 解决方案

通过搜索找到了答案，解决网页地址<http://stackoverflow.com/questions/8129040/proguard-missing-type-parameter>，下面是它给出的混淆配置文件。

```
##---------------Begin: proguard configuration for Gson  ----------
# Gson uses generic type information stored in a class file when working with fields. Proguard
# removes such information by default, so configure it to keep all of it.
-keepattributes Signature


# Gson specific classes
-keep class sun.misc.Unsafe { *; }
#-keep class com.google.gson.stream.** { *; }

# Application classes that will be serialized/deserialized over Gson
-keep class com.google.gson.examples.android.model.** { *; }

##---------------End: proguard configuration for Gson  ----------
```

在这里面，加上`-keep class sun.misc.Unsafe { *; }`就能解决问题了。