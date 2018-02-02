---
layout: post
title:  "Annotation in Android"
date:   2018-02-02 18:04:00
categories: annotation
tags: 注解 annotation
share: true
---

* content
{:toc}

现在annotation在Android越来越普遍和主流了，学习下Annotation API 和 自定义 Annotation

![](https://tip.duke.edu/independent_learning/language_arts/wj2_lesson/annotation_icon_final.jpg)





## Annotation


### 基础
开发这么久了对于注解到现在还云里雾里的，所以最近在做把项目拆分成组件，看到一些思路，其中觉得最好的就是用`annotation`，
可以减少重复代码的编写，极大的方便我们快速开发。既然要使用这个必须要了解其内部的使用原理，否则后期遇到了什么问题，
都不知道怎么解决了。所以准备写个`Annotation`探索文章，解开想象中比较高大尚的工具。

#### What is the annotation?

`Annotation`是元数据的一种形式，向外提供程序的信息，但它本身并不是这个程序的一部分，它可以被添加到包，类，方法，变量中，
并且可以在某个生命周期中（`java`源码中，编译期，`Runtime`）被反射获取。`Annotation`并不是直接影响它所注解的代码 。

简单来说，`Annotation`为我们在代码中添加信息提供了一种形式化的方法，是我们可以在稍后某个时刻方便地使用这些
数据（通过 解析注解 来使用这些数据）

#### Annotation用来做什么？ 

1. 给编译器提供信息--例如提供给编译器探测错误和压制警告等等
2. 编译期生成代码  
3. `Runtime`处理注解

#### 元注解
用来注解注解类的注解就是元注解，java提供了五种元注解，分别是`@Documented`，`@Inherited`，`@Repeatable`,
 `@Target`, `@Retention`
 
 `@Documented` 它代表着此注解的元素会被javadoc工具提取成文档
 
 `@Inherited `允许子类继承父类中的注解
 
 `@Repeatable` Java SE8引入的注解，表示这个注解可以在同一处多次声明
 
 `@Target` 是用来描述该注解标记哪一种类型在java源码中，它的取值可为：
  >ElementType.ANNOTATION_TYPE 可以使用在注解类型上      
  >ElementType.CONSTRUCTOR 可以使用在构造方法上       
  >ElementType.FIELD 可以使用在属性（成员变量）上      
  >ElementType.LOCAL_VARIABLE 可以使用在局部变量上      
  >ElementType.METHOD 可以使用在方法上   
  >ElementType.PACKAGE 可以使用在包声明上    
  >ElementType.PARAMETER 可以使用在方法参数上        
  >ElementType.TYPE 可以使用在类中任何元素     
  
  `@Retention` 代表这个注解的生命周期，可以存活到什么时期:
  > RetentionPolicy.SOURCE 存在在java源码中      
  > RetentionPolicy.CLASS 存活到编译成Class中      
  > RetentionPolicy.RUNTIME 存活到运行时期     


#### 自定义注解
new a module type : `Java Library` name : `lib_annotation` ，并新建一个`TestSourceAnnotation.java`注解，如下：
```java
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
public @interface TestSourceAnnotation {
}
```

新建一个



> 参考 : 
>>[Android 注解--（一）注解基础](https://www.jianshu.com/p/9edbbdc43fc6)   
>>
>>[Android 注解--(二)利用注解技术在编译期生成代码](https://www.jianshu.com/p/1b261d7b0834)


