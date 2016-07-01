---
layout: blog_item
title: 'Android TextView无焦点的跑马灯效果'
date: 2016-07-01 12:00:00 +0800
author: christiein
categories: [原创]
---

TextView的跑马灯效果在网上能搜索岛很多内容，我看大部分都直接使用的是TextView控件，但是这存在一些问题。什么问题呢？那就是如果这个TextView没有焦点的时候，它是不会滚动的，也就是说只有焦点在TextView上的时候才能看到跑马灯效果，但是如果我们有时候希望即使TextView没有交点文字依然滚动，那要怎么办呢?
        
其实也很简单，只需要自定义一个TextView的子类，然后在isFocused方法中返回true就行了。代码如下：

```java
public class MarqueeTextView extends TextView {

    public MarqueeTextView(Context context) {
        super(context);
    }

    public MarqueeTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
    } 

    public MarqueeTextView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }  

    @Override
    public boolean isFocused() {
        return true;//这里返回true就好了
    }

}
```
  
在布局xml文件中引用这个控件就行了。

```xml
<com.example.marquee.MarqueeTextView
        android:layout_width="50dp"
        android:layout_height="wrap_content"
        android:ellipsize="marquee"
        android:focusable="true"
        android:focusableInTouchMode="true"
        android:freezesText="true"
        android:marqueeRepeatLimit="marquee_forever"
        android:scrollHorizontally="true"
        android:singleLine="true"
        android:text="我就是一个真正的跑马灯效果" />
```