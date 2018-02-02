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

![来一张图](https://zeroturnaround.com/wp-content/uploads/2016/02/gradle-logo.png)






## Gradle dependencies之implement、api 和compile区别

### 前言  
2017年google大会后，Android studio版本更新至3.0。现在gradle工具也升级到了3.0.1，
在3.0.1中使用了最新的Gradle 4.1里程碑版本作为gradle的编译版本，该版本gradle编译速度有所加速，更加欣喜的是完全支持Java8。

> ### 参考
>> #### [google android developer](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html#new_configurations)
>> #### [gradle docs](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_separation)
>> #### [http://blog.csdn.net/marvinhq/article/details/73477128](http://blog.csdn.net/marvinhq/article/details/73477128)

在3.0以前的`gradle dependencies`中声明写法
```groovy
compile fileTree(dir: 'libs', include: ['*.jar'])
```
在3.0之后`gradle dependencies`中声明写法
```groovy
implementation fileTree(dir: 'libs', include: ['*.jar'])
//或者
api fileTree(dir: 'libs', include: ['*.jar'])
```
在3.0之后`compile`指令就是过时的了，当然从它引出了`api`和`implementation`下面就讲讲新的指令表示的意思和用法

### api
`api`其实和`compile`没区别，如果想偷懒把`compile`全部替换成`api`就可以了，但是这种思维还是要避免的，否则google也不会把`compile`
分成`api`和`implementation`。当一个库包含`api`依赖项时，其实就是让`Gradle`知道模块想要将该依赖项传递到其他模块，以便在运行时和编译
时都可以使用它。这个配置的行为就像`compile`（现在已经被弃用了），你通常应该只在库模块中使用它。这是因为，如果`api`依赖项更改了外部API，
`Gradle`会在编译时重新编译所有有权访问该依赖项的模块。所以，拥有大量的`api`依赖可以大大增加构建时间。
除非你想将一个依赖的API暴露给一个单独的测试模块，`implementation` 依赖。

### implementation
当你的模块配置一个`implementation`依赖项时，让`Gradle`知道这个模块在编译的时候不需要把依赖项泄露给其他模块。也就是说，只有在运行时，
依赖才可用于其他模块。使用这种依赖配置代替`api`或`compile`可以导致显着的构建时间改进，因为它减少了构建系统需要重新编译的项目的数量。例如，
如果一个`implementation`依赖关系改变了它的API，`Gradle`只重新编译这个依赖项以及直接依赖它的模块。大多数应用程序和测试模块应使用此配置。

### compileOnly
这个配置的行为就像`provided`（现在已被弃用）。`Gradle`仅将编译类路径添加到编译类路径（它不会被添加到编译输出）。这在创建Android库模块
时非常有用，并且在编译期间需要依赖关系，但是在运行时存在则是可选的。也就是说，如果你使用这个配置，那么你的库模块必须包含一个运行时条件来检
查依赖是否可用，然后适当地改变它的行为，所以如果没有提供，它仍然可以运行。这有助于减少最终APK的大小，而不会增加不重要的临时依赖关系。

### runtimeOnly
`Gradle`将依赖关系添加到构建输出中，以便在运行时使用。也就是说，它不会被添加到编译类路径中。这个配置的行为就像 `apk`（现在已被弃用）

您可以使用它implementation来使所有变体都可用，当然也可以用variant+`Implementation`，例如`debugImplementation`,`testImplementation`

### annotationProcessor
在以前的插件版本中，compile classpath的依赖项会自动添加到处理器类路径中
```groovy
dependencies {
    compile 'com.google.dagger:dagger-compiler:<version-number>'
}
```
当使用Android plugin 3.0.0时，必须使用注解处理器依赖配置将注释处理器添加到处理器类路径，如下所示:
```groovy
dependencies {
    annotationProcessor 'com.google.dagger:dagger-compiler:<version-number>'
}
```



>注： `compile`，`provided`以及`apk` 目前仍然可用。但是，他们将在Android插件的下一个主要版本中被删除。


> 各个依赖关系的图
> ![](https://docs.gradle.org/current/userguide/img/java-library-ignore-deprecated-main.png)

### transitive
transitive dependencies 被称为依赖的依赖，称为“间接依赖”比较合适。

```
compile('com.xxxxx.xxxxx.xxxxx:xxxxx:xxxx@aar'){
        transitive = true
        exclude module: 'xxxxxx'
        exclude module: 'xxxxxx'
        exclude group:'com.xxxxx.xxxxxxxx' module:'xxxxxx'
    }
```
>在后面加上`@aar`，意指你只是下载该`aar`包，而并不下载该`aar`包所依赖的其他库，那如果想在使用`@aar`的前提下还能
下载其依赖库，则需要添加`transitive=true`的条件。 

### 排除 transitive dependencies 
通过configuration或者dependency可以除去 transitive dependencies：
````
build.gradle

configurations {
    compile.exclude module: 'xxxxx'
    all*.exclude group: 'com.xxxxx.xxxxx.xxxxx', module: 'xxxxx'
}

dependencies {
    compile('com.xxxxx.xxxxx.xxxxx:xxxxx:xxxx@aar') {
        exclude module: 'xxxx_model'
    }
}
````
>如果在configuration中定义一个exclude,那么所有依赖的transitive dependency (指定的)都会被去除。 
>定义exclude时候，或只指定group, 或只指定module名字，或二者都指定。






