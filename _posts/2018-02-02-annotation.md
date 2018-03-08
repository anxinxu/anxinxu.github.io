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


#### @Retention
  1. new a module type : `Java Library` name : `lib_annotation` ，并新建`TestSourceAnnotation.java`：
```java
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
public @interface TestSourceAnnotation {
}
```

`TestClassAnnotation.java`:
```java
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.TYPE)
public @interface TestClassAnnotation {
}
```

`TestRuntimeAnnotation.java`:
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface TestRuntimeAnnotation {
}
```

  2. new test class

`TestSourceAnnotationClass.java`
```java
@TestSourceAnnotation
public class TestSourceAnnotationClass {
}
```

`TestClassAnnotationClass.java`
```java
@TestClassAnnotation
public class TestClassAnnotationClass {
}
```

`TestRuntimeAnnotationClass.java`
```java
@TestRuntimeAnnotation
public class TestRuntimeAnnotationClass {
}
```

  3. 然后编译一下生成class文件`compileJava`task

![](https://github.com/anxinxu/resource/blob/master/blog_image/annotation/gradle_build_task.png?raw=true)

然后在build文件夹classes中查找到生成的class文件
![](https://github.com/anxinxu/resource/blob/master/blog_image/annotation/compile-class.png?raw=true)

编译生成的`class`如下

`TestSourceAnnotationClass.class`
```java

public class TestSourceAnnotationClass {
}
```

`TestClassAnnotationClass.class`
```java
@TestClassAnnotation
public class TestClassAnnotationClass {
}
```

`TestRuntimeAnnotationClass.class`
```java
@TestRuntimeAnnotation
public class TestRuntimeAnnotationClass {
}
```
  4. 结果发现如下：
  * `RetentionPolicy.SOURCE` ： 对比发现注解`@TestSourceAnnotation()`已经不存在了，使注解仅存在与java源码中。
  * `RetentionPolicy.CLASS`和`RetentionPolicy.RUNTIME` 的class中注解依然存在。接下来分析它两的区别：
  
  5. 注解信息的获取
```java
public class TestAnnotation {

    public static void main(String[] args) {
//        testClass();
        testRuntime();
    }

    private static void testClass(){
        Class<TestClassAnnotation> tClass = TestClassAnnotation.class;
        Class<TestClass> tAClass = TestClass.class;
        TestClass tAnnotation = tClass.getAnnotation(tAClass);
        String tValue = tAnnotation.value();
        System.out.print("testClass() ->  default value = " + tValue);
    }

    private static void testRuntime(){
        Class<TestRuntimeAnnotation> tClass = TestRuntimeAnnotation.class;
        Class<TestRuntime> tAClass = TestRuntime.class;
        TestRuntime tAnnotation = tClass.getAnnotation(tAClass);
        String tValue = tAnnotation.value();
        System.out.print("testRuntime() ->  default value = " + tValue);
    }

    @TestClass
    private class TestClassAnnotation{

    }

    @TestRuntime
    private class TestRuntimeAnnotation{

    }

    @Retention(RetentionPolicy.CLASS)
    @interface TestClass{
        String value() default "test class annotation";
    }

    @Retention(RetentionPolicy.RUNTIME)
    @interface TestRuntime{
        String value() default "test runtime annotation";
    }
}
```
运行`main()`
![](https://github.com/anxinxu/resource/blob/master/blog_image/annotation/run_test_main.png?raw=true)

`testRuntime()`日志如下：
![](https://github.com/anxinxu/resource/blob/master/blog_image/annotation/run_fun_test_runtime.png?raw=true)

`testClass()`日志如下：
![](https://github.com/anxinxu/resource/blob/master/blog_image/annotation/run_fun_test_class.png?raw=true)

>`RetentionPolicy.CLASS`和`RetentionPolicy.RUNTIME`的区别，`RetentionPolicy.CLASS`的注解是不会存活到
运行时期的，在运行时期要想通过反射获得注解，那么你定义这个注解的时候需要使用`RetentionPolicy.RUNTIME`。
       
       
### 自定义注解，编译时生成代码

#### demo需求
做一个类似`butter knife`的注解

#### 创建Android工程
创建一个android的项目
在根目录的`build.gradle`
```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    apply from : 'version.gradle'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'

        // apt
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

App Module的`build.gradle`
```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion versions.targetSdk
    defaultConfig {
        applicationId "com.anxin.annotation"
        minSdkVersion versions.minSdk
        targetSdkVersion versions.targetSdk
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation deps.support.appcompat
    implementation deps.support.constraint_layout
    testImplementation deps.junit
    androidTestImplementation deps.support.test.runner
    androidTestImplementation deps.support.test.espresso.espresso_core
    annotationProcessor project(':lib_annotation_compiler')
    implementation project(':lib_annotation')
    implementation project(':inject')
}
```

#### 创建一个Annotation Library（Java Library）
`build.gradle`
```groovy
apply plugin: 'java-library'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    api deps.support.annotations
}

sourceCompatibility = "1.8"
targetCompatibility = "1.8"
```
首先新建一个注解类`BindView`

```java
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.FIELD)
public @interface BindView {
    @IdRes int value();
}
```
接下来就要接触生成代码的处理器，使用的APT；
>APT是Annotation-Processing-tool的简写，称为注解处理器，一般来说，自定义注解是在运行时使用的，
通过反射获取class上的注解，并进行解析处理，使用apt可以让我们在编译时处理注解

在这里用的是`RetentionPolicy.CLASS`而不是`RetentionPolicy.RUNTIME`，这是因为我要在编译时就生成代码，不是运行时通过反射
得到注解的信息。

现在的问题就是编译的时候如何获取注解的信息，接下来就使用到注解处理器Processor。

#### 创建一个提供inject module
这个主要作用是让使用的类不去直接使用自动生成的代码，而是用这个自动取调用
```groovy
apply plugin: 'com.android.library'

android {
    compileSdkVersion versions.targetSdk



    defaultConfig {
        minSdkVersion versions.minSdk
        targetSdkVersion versions.targetSdk
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])

    testImplementation deps.junit
    androidTestImplementation deps.support.test.runner
    androidTestImplementation deps.support.test.espresso.espresso_core
}

```

主要包括了两个`Inject.java`和`Unbinder.java`
```java
package com.anxin.inject;

/**
 * Created by anxin on 2018/2/7.
 * <p>
 */

public interface Unbinder {

    void unbind();
}
```

```java
package com.anxin.inject;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

/**
 * Created by anxin on 2018/2/8.
 * <p>
 */

public class Inject {

    private Inject() {
    }

    @SuppressWarnings("unchecked")
    public static Unbinder bind(Object target) {
        if (target != null) {
            Class<?> tClass = target.getClass();
            String tClassName = tClass.getName();
            if (tClassName.startsWith("android.") || tClassName.startsWith("java.")) return null;

            try {
                Class<?> cls = tClass.getClassLoader().loadClass(tClassName + "_BindView");
                Constructor<? extends Unbinder> tConstructor = (Constructor<? extends Unbinder>) cls.getConstructor(tClass);
                return tConstructor.newInstance(target);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } catch (NoSuchMethodException e) {
                e.printStackTrace();
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```
> 主要机制就是根据类名创建一个自动生成的的代码的对象，从而初始化构造器的代码。

#### 注解处理器Processor
了解一下注解处理器，注解处理器有什么作用呢，首先它会在编译期被调用，可以扫描特定注解的信息，你可以为你自己的的注解注册
处理器（如何注册后面会讲），一个特定的注解处理器以java源码作为输入，然后生成一些文件作（通常为java）为输出，
这些java文件同样会被编译。这意味着，你可以根据注解的信息和被注解类的信息生成你想生成的代码！

#### JavaPoet 
Java代码生成借助square公司的JavaPoet开源库

使用javapoet前需要了解这几个经常使用类
* JavaFile 用于构造输出包含一个顶级类的Java文件
* TypeSpec 生成类，接口，或者枚举   
* MethodSpec 生成成员变量或字段    
* FieldSpec 生成成员变量或字段   
* ParameterSpec 用来创建参数
* AnnotationSpec 用来创建注解

创建方法中肯定不可缺少倒包`JavaPoet` 帮我们定义了如下几种专门描述类型的类。其关系图如下:

|分类|生成的类型|`JavaPoet`写法|`Java`写法|
|-|-|-|-|
|内置类型|`int`|`TypeName.INT`|`int.class`|
|数组类型|`int[]`|`ArrayTypeName.of(int.class)`|`int[].class`|
|需要引入包名的类型|`java.io.File`|`ClassName.get(“java.io”, “File”)`|`java.io.File.class`|
|参数化类型 `ParameterizedTypeName`|`List<String>`|`ParameterizedTypeName.get(List.class, String.class)`|-|
|类型变量 `TypeVariableName`泛型|T|`TypeVariableName.get(“T”)`|-|
|通配符类型|`? extends String`|`WildcardTypeName.subtypeOf(String.class)`|-|

>这些类型之间可以相互嵌套， 比如 `ParameterizedTypeName.get(List.class, String.class)` 其中 `List.class` 等价于 `ClassName.get("java.util", "List")`。 因此，
>`ParameterizedTypeName.get(List.class, String.class)`   
>可以写为 `ParameterizedTypeName.get(ClassName.get("java.util", "List"), ClassName.get("java.lang", "String"))`。
>前者的好处是简洁， 后者的好处是 使用 `ClassName` 代表某个类型而无需引入该类型。 比如： 由于在 `java` 工程中是没有 `android` 的 `sdk`，
 所以你在 `java` 工程中想生成 `android.app.Activity` 这种类型是不能直接 `Activity.class`。这种情况下只能通过 `ClassName` 进行引用。”
 
常用api:
* addStatement() 方法负责分号和换行
* beginControlFlow()
* endControlFlow() 需要一起使用，提供换行符和缩进。
* addCode() 以字符串的形式添加内
* .returns 添加返回值类型
* .constructorBuilder() 生成构造器函数
* .addAnnotation 添加注解
* addSuperinterface 给类添加实现的接口
* superclass 给类添加继承的父类
* ClassName.bestGuess(“类全名称”) 返回ClassName对象，这里的类全名称表示的类必须要存在，会自动导入相应的包
* ClassName.get(“包名”，”类名”) 返回ClassName对象，不检查该类是否存在
* TypeSpec.interfaceBuilder(“HelloWorld”)生成一个HelloWorld接口
* MethodSpec.constructorBuilder() 构造器
* addTypeVariable(TypeVariableName.get(“T”, typeClassName)) 会给生成的类加上泛型

占位符可以在addStatement中使用
* $T 类型替换, 一般用于 `("$T foo", List.class) => List foo`， `$T `的好处在于 `JavaPoet` 会自动帮你补全文
件开头的 `import`. 如果直接写 `("List foo")` 虽然也能生成 `List foo`， 但是最终的 `java` 文件就不会自动帮你添加
 `import java.util.List`.
* $L 字面量替换, 比如 `("abc$L123", "FOO") => abcFOO123`. 也就是直接替换.
* $S 字符串替换, 比如: `("$S.length()", "foo") => "foo".length()` 注意 `$S`是将参数替换为了一个带双引号的字符串.
 免去了手写 `"\"foo\".length()"` 中转义 `(\")` 的麻烦.
* $N 名称替换, 比如你之前定义了一个函数 `MethodSpec methodSpec = MethodSpec.methodBuilder("foo").build()`;
 现在你可以通过 `$N` 获取这个函数的名称 `("$N", methodSpec) => foo`.
 

#### 创建一个Annotation Compiler
新建一个Compile Module（Java Library）
`build.gradle`:
```groovy
apply plugin: 'java-library'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation deps.auto.service
    implementation deps.javapoet
    implementation project(':lib_annotation')
}

sourceCompatibility = "1.8"
targetCompatibility = "1.8"

```
1. AutoService 主要的作用是注解 processor 类，并对其生成 META-INF 的配置信息。
2. JavaPoet 这个库的主要作用就是帮助我们通过类调用的形式来生成代码。
3. 自定义的Annotation
依赖上面创建的annotation Module。

```java
@AutoService(Processor.class)
public class BindViewProcessor extends AbstractProcessor {


    @Override
    public synchronized void init(ProcessingEnvironment processingEnvironment) {
        super.init(processingEnvironment);
    }

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        return super.getSupportedAnnotationTypes();
    }

    @Override
    public SourceVersion getSupportedSourceVersion() {
        return super.getSupportedSourceVersion();
    }

    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        return false;
    }
}
```
>我们需要重写的方法有这么几个:   
> * `init(ProcessingEnvironment processingEnv)` ：所有的注解处理器类都必须有一个无参构造函数。然而，
有一个特殊的方法`init()`，它会被注解处理工具调用，以`ProcessingEnvironment`作为参数。ProcessingEnvironment 
提供了一些实用的工具类Elements, Types和Filer。我们在后面将会使用到它们。
> * `process(Set<? extends TypeElement> annotations, RoundEnvironment env)` ：这类似于每个处理器的
`main()`方法。你可以在这个方法里面编码实现扫描，处理注解，生成 `java`文件。使用`RoundEnvironment`
 参数，你可以查询被特定注解标注的元素。
> * `getSupportedAnnotationTypes()`：在这个方法里面你必须指定哪些注解应该被注解处理器注册。注意它的返回值是一
个`String`集合，包含了你的注解处理器想要处理的注解类型的全称。换句话说，你在这里定义你的注解处理器要处
理哪些注解。
> * `getSupportedSourceVersion()` ： 用来指定你使用的 `java` 版本，建议使用
`SourceVersion.latestSupported()`。
 
 
接下来就是在process中处理需要注解的信息，需要生成的`Java`文件
```
    @Override
    public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnvironment) {
        note(" -> process () set : " + set + " , roundEnvironment : " + roundEnvironment);

        Map<Element, JavaFile> bindMap = findAndParseTarget(roundEnvironment);
        for (Map.Entry<Element, JavaFile> tFileEntry : bindMap.entrySet()) {
            Element tKey = tFileEntry.getKey();
            JavaFile tValue = tFileEntry.getValue();
            note("key : %s ", tKey.toString());
            try {
                tValue.writeTo(processingEnv.getFiler());
            } catch (IOException e) {
                e.printStackTrace();
                StringWriter tWriter = new StringWriter();
                e.printStackTrace(new PrintWriter(tWriter));
                printMessage(Kind.ERROR, tKey, "process() -> e : %s", tWriter);
            }
        }

        return false;
    }

    private Map<Element, JavaFile> findAndParseTarget(RoundEnvironment roundEnvironment) {
        Map<Element, JavaFile> bindingMap = new LinkedHashMap<>();
        // Process each @Test element.
        for (Element tElement : roundEnvironment.getElementsAnnotatedWith(Test.class)) {
            JavaFile tJavaFile = parseTest(tElement);
            bindingMap.put(tElement, tJavaFile);
        }
        // Process each @BindView element.
        for (Element tElement : roundEnvironment.getElementsAnnotatedWith(BindView.class)) {
            JavaFile tJavaFile = parseBindView(tElement);
            if (tJavaFile != null) {

                bindingMap.put(tElement, tJavaFile);
            } else {
                error("parse bind view error", tElement, null);
            }
        }

        return bindingMap;
    }

    private JavaFile parseBindView(Element typeElement) {
        BindView tAnnotation = typeElement.getAnnotation(BindView.class);
        try {
            int tValue = tAnnotation.value();
            TypeElement tTypeElement = (TypeElement) typeElement.getEnclosingElement();
            ClassName targetClass = ClassName.get(tTypeElement);
            TypeSpec.Builder tBuilder = TypeSpec.classBuilder(String.format("%s_BindView", tTypeElement.getSimpleName()))
                    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
                    .addJavadoc("don't edit this code!\n")
                    .addSuperinterface(INJECT_CLASS_NAME);
            MethodSpec unbind = MethodSpec.methodBuilder("unbind")
                    .addModifiers(Modifier.PUBLIC)
                    .addAnnotation(Override.class)
                    .returns(void.class)
                    .build();
            MethodSpec constructor = MethodSpec.constructorBuilder()
                    .addModifiers(Modifier.PUBLIC)
                    .addParameter(targetClass, "target", Modifier.FINAL)
                    .addStatement("$L.$L = $L.findViewById($L)", "target", typeElement, "target", tValue)
                    .build();
            tBuilder.addMethod(unbind);
            tBuilder.addMethod(constructor);
            return JavaFile.builder(typeUtils.getPackageOf(typeElement).getQualifiedName().toString(), tBuilder.build()).build();

        } catch (Exception e) {
            StringWriter tWriter = new StringWriter();
            e.printStackTrace(new PrintWriter(tWriter));
            error("parseBindView() -> annotation : %s ", typeElement, e, tAnnotation);
        }

        return null;
    }

    private JavaFile parseTest(Element typeElement) {
        MethodSpec constructor = MethodSpec.constructorBuilder()
                .addJavadoc(String.format("constructor() ->  tElement = %s , tTypeElement = %s", typeElement.toString(), typeElement.toString()))
                .addParameter(TypeName.get(typeElement.asType()), "target")
                .addParameter(String.class, "msg")
                .addStatement(String.format("android.util.Log.d(%s,\"%s\")", "TAG", "init<>"))
                .addStatement("name = \"test annotation\"")
                .addStatement("int count = 0;")
                .beginControlFlow("for(int i = 0;i < 10;i++)")
                .addCode("//add code ========>\n")
                .addStatement("count++")
                .endControlFlow()
                .addStatement("target.count = count")
                .addStatement(String.format("android.util.Log.d(%s,\"%s -> count = \" + %s)", "TAG", "init<>", "count"))
                .build();
        MethodSpec main = MethodSpec.methodBuilder("main")
                .addModifiers(Modifier.PUBLIC, Modifier.STATIC, Modifier.FINAL)
                .returns(void.class)
                .addParameter(String[].class, "args")
                .build();
        FieldSpec TAG = FieldSpec.builder(String.class, "TAG", Modifier.PRIVATE, Modifier.FINAL, Modifier.STATIC)
                .initializer("\"Test_" + typeElement.getSimpleName() + "\"")
                .build();
        TypeSpec tTypeSpec = TypeSpec.classBuilder("Test_" + typeElement.getSimpleName())
                .addMethod(constructor)
                .addMethod(main)
                .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
                .addField(TAG)
                .addField(String.class, "name", Modifier.PRIVATE)
                .build();
        return JavaFile.builder(typeUtils.getPackageOf(typeElement).getQualifiedName().toString(), tTypeSpec).build();
    }
```

对应生成的Java文件`MainActivity_BindView.java`
```java
package com.anxin.annotation;

import com.anxin.inject.Unbinder;
import java.lang.Override;

/**
 * don't edit this code!
 */
public final class MainActivity_BindView implements Unbinder {
  public MainActivity_BindView(final MainActivity target) {
    target.mTextView = target.findViewById(2131165307);
  }

  @Override
  public void unbind() {
  }
}

```
`Test_MainActivity.java`
```java
package com.anxin.annotation;

import java.lang.String;

public final class Test_MainActivity {
  private static final String TAG = "Test_MainActivity";

  private String name;

  /**
   * constructor() ->  tElement = com.anxin.annotation.MainActivity , tTypeElement = com.anxin.annotation.MainActivity */
  Test_MainActivity(MainActivity target, String msg) {
    android.util.Log.d(TAG,"init<>");
    name = "test annotation";
    int count = 0;;
    for(int i = 0;i < 10;i++) {
      //add code ========>
      count++;
    }
    target.count = count;
    android.util.Log.d(TAG,"init<> -> count = " + count);
  }

  public static final void main(String[] args) {
  }
}

```

### Demo地址
[Demo地址：https://github.com/anxinxu/annotation](https://github.com/anxinxu/annotation)



> 参考 : 
>>[square公司JavaPoet源码](https://github.com/square/javapoet) 
>>
>>[Android 注解--（一）注解基础](https://www.jianshu.com/p/9edbbdc43fc6)   
>>
>>[Android 注解--(二)利用注解技术在编译期生成代码](https://www.jianshu.com/p/1b261d7b0834)
>>
>>[JavaPoet 看这一篇就够了](https://juejin.im/entry/58fefebf8d6d810058a610de)


