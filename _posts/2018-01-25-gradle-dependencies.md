---
layout: post
title:  "Gradle dependencies之implement、api 和compile区别"
date:   2018-01-25 16:37:05
categories: blog
tags: gradle
share: true
---

* content
{:toc}

Gradle dependencies之implement、api 和compile区别

![来一张图](https://crackberry.com/sites/crackberry.com/files/styles/small/public/topic_images/2013/ANDROID.png?itok=xhm7jaxS)






## Gradle dependencies之implement、api 和compile区别

### 前言  
2017年google大会后，Android studio版本更新至3.0。现在gradle工具也升级到了3.0.1，
在3.0.1中使用了最新的Gradle 4.1里程碑版本作为gradle的编译版本，该版本gradle编译速度有所加速，更加欣喜的是完全支持Java8。


在3.0以前的`gradle dependencies`中声明写法
```groovy

compile fileTree(dir: 'libs', include: ['*.jar'])

```
{% codeblock %}
Awesome code snippet
{% endcodeblock %}