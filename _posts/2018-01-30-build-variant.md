---
layout: post
title:  "全局修改Gradle Maven 仓库地址"
date:   2018-01-30 16:37:05
categories: gradle
tags: gradle variant flavor
share: true
---

* content
{:toc}

Build Variants（构建变种版本）

![](http://www.devapp.it/wordpress/wp-content/uploads/2016/10/gradle-build-variant-android-studio-how-to-642x336.png)






## Build Variants（构建变种版本）

新构建系统的一个目标就是允许为同一个应用创建不同的版本。

> 1. [原文地址](https://developer.android.com/studio/build/index.html)
> 2. [参考博客一](http://www.cnblogs.com/yydcdut/p/4821156.html)
> 3. [参考博客二](http://blog.csdn.net/qinxiandiqi/article/details/37906449)

> 这里有两个主要的使用情景：
> 1. 同一个应用的不同版本。例如一个收费版本，一个免费版本。
> 2. 同一个应用需要打包成不同的apk以发布在App Store上[更多详情](http://developer.android.com/google/play/publishing/multiple-apks.html)


### Build Type (构建类型)
默认情况下，Android插件自动为项目构建一个debug和一个release版本的应用。这两个版本的不同主要体现在在非开发机
上的调试功能以及APK的签名方式。debug版本使用一个用公开的name/password创建的key来签名（这样构建的时候就
不需要提示输入密码了）。release版本在构建的时候不会进行签名，而是稍后在做。

这个可以使用gradle中的BuildType对象来进行配置。默认情况下，2个BuildType的实例会被创建，一个debug，一个release。
Android插件允许自定义这两个实例，当然你也可以创建其他的build type。配置由buildTypes这个DSL来完成：
```

android {  
    buildTypes {  
        debug {  
            applicationIdSuffix ".debug"  
        }  
  
  
        jnidebug.initWith(buildTypes.debug)  
        jnidebug {  
            packageNameSuffix ".jnidebug"  
            jniDebuggable true  
        }  
    }  
}  

```

上面的代码完成了下列配置：

* 配置默认的debug Build Type

  设置debug版本的包名为<应用id>.debug，这样就可以在设备上同时安装debug和release版本了。

* 创建一个新的BuildType，名字是jnidebug，同时配置它是复制自debug Build Type。

  配置jnidebug开启debug版本的JNI组件，添加一个不同的包名后缀。

创建一个新的的Build Types非常简单，只需要在buildTypes下面通过调用initWith或者使用闭包添加一个新的元素。下表是可以
配置的属性以及默认值：

|属性名|debug版本默认值|release或其他版本默认值|
|-|-|-|
|debuggable|true|false|
|jniDebuggable|false|false|
|renderscriptDebuggable|false|false|
|renderscriptOptimLevel|3|3|
|applicationIdSuffix|null|null|
|versionNameSuffix|null|null|
|signingConfig|android.signingConfigs.debug|null|
|zipAlignEnabled|false|true|
|minifyEnabled|false|false|


除了这些属性，Build Types还可以用来配置代码和资源文件。针对每一个Build Type，一个新的对应的
sourceSet会被创建，这个sourceSet使用一个默认的路径`src/< buildtype名字>/`。这就意味着Build Type
的名字不能是main或者androidTest（这是由插件强制的），同时每个Build Type的名字必须是唯一的。


### Product Flavors（不同定制的产品）

一个Product Flavor定义了从项目中构建了一个应用的自定义版本。一个单一的项目可以同时定义多个不同的`flavor`来
改变应用的输出。

这个新的设计概念是为了解决不同的版本之间的差异非常小的情况。虽然最项目终生成了多个定制的版本，但是它们本质上
都是同一个应用，那么这种做法可能是比使用库项目更好的实现方式。

在gradle3.0之后 Product Flavor需要在`productFlavors`这个DSL容器中声明:

```

android {  
    // Specifies a flavor dimension.
    flavorDimensions "color"
    
    productFlavors {
         red {
          // Assigns this product flavor to the 'color' flavor dimension.
          // This step is optional if you are using only one dimension.
          dimension "color"
          ...
        }
    
        blue {
          dimension "color"
          ...
        }
    }

}  

```
这里创建了两个flavor，名为red和blue。

>注意：flavor的命名不能与已存在的`Build Type`或者`androidTest`这个`sourceSet`有冲突。

### Build Variant（构建变种版本）

 Build Variant = Build Type  + Product Flavor
 
 正如前面章节所提到的，每一个Build Type都会生成一个新的APK。
 
 
 Product Flavor同样也会做这些事情：项目的输出将会拼接所有可能的Build Type和Product Flavor（如果有Flavor定义存在的话）的组合。
 
 
 每一种组合（包含Build Type和Product Flavor）就是一个Build Variant（构建变种版本）。
 
 
 例如，在上面的Flavor声明例子中与默认的debug和release两个Build Type将会生成4个Build Variant：
 
   * red - debug
 
   * red - release
 
   * blue - debug
 
   * blue - release
 
 
 
 项目中如果没有定义flavor同样也会有Build Variant，只是使用的是默认的flavor和配置。默认的flavor是没有名字的，所以生成的Build Variant列表看起来就跟Build Type列表一样。
 
 ### Building and Tasks（构建和任务）
 
 我们前面提到每一个Build Type会创建自己的assemble<name>task，但是Build Variant是Build Type和Product Flavor的组合。
 
 
 当使用Product Flavor的时候，将会创建更多的assemble-type task。分别是：
 
   1. assemble<Variant Name>：允许直接构建一个Variant版本，例如assembleRedDebug。
 
   2. assemble<Build Type Name>：允许构建指定Build Type的所有APK，例如assembleDebug将会构建
   RedDebug和BlueDebug两个Variant版本。
 
   3. assemble<Product Flavor Name>：允许构建指定flavor的所有APK，例如assembleRed
   将会构建RedDebug和RedRelease两个Variant版本。
 
 另外assemble task会构建所有可能组合的Variant版本。
 
 
 ### variantFilter (过滤变体)
 
 Gradle 会为您配置的产品风味与构建类型的每个可能的组合创建构建变体。不过，某些特定的构建变体在您的项目环境
 中并不必要，也可能没有意义。您可以在模块级 build.gradle 文件中创建一个变体过滤器，以移除某些构建变体配置。
 
 ```
 
 android {
   ...
   buildTypes {...}
 
   flavorDimensions "api", "mode"
   productFlavors {
     demo {...}
     full {...}
     minApi24 {...}
     minApi23 {...}
     minApi21 {...}
   }
 
   variantFilter { variant ->
       def names = variant.flavors*.name
       // To check for a certain build type, use variant.buildType.name == "<buildType>"
       if (names.contains("minApi21") && names.contains("demo")) {
           // Gradle ignores any variants that satisfy the conditions above.
           setIgnore(true)
       }
   }
 }
 ...
 
```
 
### sourceSets (创建源集)

> [https://developer.android.com/studio/build/build-variants.html#sourcesets](https://developer.android.com/studio/build/build-variants.html#sourcesets)

默认情况下，Android Studio 会创建 main/ 源集和目录，用于存储您要在所有构建变体之间共享的一切资源。然而，
您可以创建新的源集来控制 Gradle 要为特定的构建类型、产品风味（以及使用风味维度时的产品风味组合）
和构建变体编译和打包的确切文件。例如，您可以在 main/ 源集中定义基本的功能，使用产品风味源集针对不同的客户更
改应用的品牌，或者仅针对使用调试构建类型的构建变体包含特殊的权限和日志记录功能。

Gradle 要求您按照与 main/ 源集类似的特定方式组织源集文件和目录。例如，Gradle 要求您的“调试”构建类型所特定的
 Java 类文件位于 src/debug/java/ 目录中。
 
 
 ![](https://developer.android.com/images/tools/debug-directories_2-1_2x.png)
 
 更改默认源集配置
 如果您的源未组织到 Gradle 期望的默认源集文件结构中（如上面的创建源集部分中所述），您可以使用 sourceSets {}
  代码块更改 Gradle 希望为源集的每个组件收集文件的位置。您不需要重新定位文件；只需要为 Gradle 提供相对于模块级
   build.gradle 文件的路径，Gradle 应当可以在此路径下为每个源集组件找到文件。要了解您可以配置哪些组件，
   以及是否可以将其映射到多个文件或目录，请阅读适用于 Gradle 的 Android 插件参考。
 
 下面的代码示例可以将 app/other/ 目录中的源映射到 main 源集的某些组件，并更改 androidTest 源集的根目录。
 
 ```
 
 android {
   ...
   sourceSets {
     // Encapsulates configurations for the main source set.
     main {
       // Changes the directory for Java sources. The default directory is
       // 'src/main/java'.
       java.srcDirs = ['other/java']
 
       // If you list multiple directories, Gradle uses all of them to collect
       // sources. Because Gradle gives these directories equal priority, if
       // you define the same resource in more than one directory, you get an
       // error when merging resources. The default directory is 'src/main/res'.
       res.srcDirs = ['other/res1', 'other/res2']
 
       // Note: You should avoid specifying a directory which is a parent to one
       // or more other directories you specify. For example, avoid the following:
       // res.srcDirs = ['other/res1', 'other/res1/layouts', 'other/res1/strings']
       // You should specify either only the root 'other/res1' directory, or only the
       // nested 'other/res1/layouts' and 'other/res1/strings' directories.
 
       // For each source set, you can specify only one Android manifest.
       // By default, Android Studio creates a manifest for your main source
       // set in the src/main/ directory.
       manifest.srcFile 'other/AndroidManifest.xml'
       ...
     }
 
     // Create additional blocks to configure other source sets.
     androidTest {
 
       // If all the files for a source set are located under a single root
       // directory, you can specify that directory using the setRoot property.
       // When gathering sources for the source set, Gradle looks only in locations
       // relative to the root directory you specify. For example, after applying the
       // configuration below for the androidTest source set, Gradle looks for Java
       // sources only in the src/tests/java/ directory.
       setRoot 'src/tests'
       ...
     }
   }
 }
 ...

 
 ```
 

