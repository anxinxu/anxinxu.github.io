---
layout: post
title:  "Set gradle global proxy"
date:   2018-02-01 15:00:00
categories: gradle
tags: gradle proxy
share: true
---

* content
{:toc}


相信大多数公司都有自己的代理服务器，所以gradle必须设置好代理才能下载项目的依赖包，比如在GitHub上clone一个项目，每次都
要修改项目的的gradle-wrapper.properties,或者gradle.properties和project的gradle.properties文件，可谓是无比痛苦无奈。


![](http://igorpopov.io/images/understanding-gradle-nirvana.jpg)




## Set Gradle Global Proxy
 
 1. 确认`%UserProfile%`（Windows ：`C:/Users/{userName}/.gradle/`; Mac | Linux : `~/.gradle/`）,没有就新建一个
 2. 在`.gradle`目录下，看有没有`gradle.properties`（没有就新建一个）
 3. 编辑`gradle.properties`，代理服务器的设置 : `systemProp.{代理类型}.{代理属性}={代理值}`
     
      1. 代理类型 : http、https、socks等
      2. 代理属性 ：proxyHost（代理的IP或域名）、proxyPort（代理的端口号）、proxyUser（用户名）、
         proxyPassword（密码）、nonProxyHosts（不使用代理的列表）。代理属性中的proxyUser（用户名）、
         proxyPassword（密码）、nonProxyHosts（不使用代理的列表）不是必须的，如果不需要的话，可以不用写
      3. 举个列子：
      
      ````
      
      #
      #systemProp.http.proxyHost=www.somehost.org
      #systemProp.http.proxyPort=8080
      #systemProp.http.proxyUser=userid
      #systemProp.http.proxyPassword=password
      #systemProp.http.nonProxyHosts=*.nonproxyrepos.com|localhost
      #
      #systemProp.https.proxyHost=www.somehost.org
      #systemProp.https.proxyPort=8080
      #systemProp.https.proxyUser=userid
      #systemProp.https.proxyPassword=password
      #systemProp.https.nonProxyHosts=*.nonproxyrepos.com|localhost
      
      
      ````
 
 
 