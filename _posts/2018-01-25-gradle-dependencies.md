---
layout: post
title:  "Gradle dependencies之implement、api 和compile区别"
date:   2018-01-25 17:47:05
categories: gradle
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
在3.0之后`gradle dependencies`中声明写法
```groovy
implementation fileTree(dir: 'libs', include: ['*.jar'])
or
api fileTree(dir: 'libs', include: ['*.jar'])
```
在3.0之后`compile`指令就是过时的了，当然从它引出了`api`和`implementation`下面就讲讲新的指令表示的意思和用法

### api



{% codeblock %}
Awesome code snippet
{% endcodeblock %}