---
title: 从APT中获取运行时类信息
tags:
  - Java
  - APT
  - Annotation
categories:
  - Development
date: 2016-04-14 20:48:58
---


## 概述
从JDK1.6开始提供了一个新的被称为APT(Annotation Processing Tool)的工具，使用其提供的APT我们可以通过类似数据结构的方式来访问被编译的Java的源代码。

利用这个新的工具提供的API我们可以在编译Java源代码的同时对现有代码进行增强和生成代码，比之以往通过运行时的反射以及通过Java的动态代理或者运行时字节码修改来增强要来的更简单并且运行时的效率要更高（指启动时间）

## 问题描述
在Java程序运行时可以通过反射来获取类的Meta信息，但是在APT中处理的是Java源代码，此时无法直接获取定义在Annotation里面的类型信息，因为没有反射API可以使用。
例如我们定义了一个注解：
``` java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface Service {

    Class[] value();
}
```
该注解接受一个Class的参数，注意该注解仅可见于源代码编译过程。

如果想在APT中获取Service.value，下面代码是不可行的：
``` java
...
Service service = ...
Class[] cls = service.value();
...
```
运行这段代码会在编译时抛出异常，因为在编译时类型信息无法直接获取，只有Native数据和String可以直接获取。

<!-- more -->

## 解决方案
如何解决该问题呢？我们需要使用APT提供的*Mirror API获取Class的相关信息。

### 获取AnnotationMirror对象

首先获取AnnotationMirror对象：
``` java
AnnotationMirror svcAnnoMirror =
  MoreElements.getAnnotationMirror(classElement, Service.class).get();
```
MoreElements是Google Auto库里面的一个工具类，getAnnotationMirror方法比较简单，它返回Service Annotation对应的AnnotationMirror对象。

上面的classElement是一个javax.lang.model.element.Element实例，在这里它代表了申明了Service这个Annotation的类的元素。
>在APT里面，Java源代码会被解释称类似XML的结构，比如类，字段，方法，方法的参数等都会被解释称一个元素，每个元素都是Element的实例

### 获取Annotation定义的元素列表

然后获取该Annotation里面定义的所有元素的列表:
``` java
Set<Element> elementSet = svcAnnoMirror.getElementValues().entrySet();
```
过滤出key是value的那个元素
``` java
if (entry.getKey().getSimpleName().toString().equals("value")) {
  AnnotationValues annoValue = entry.getValue();
}
```

### 获取TypeElement

有了AnnotationValue，我们就可以取得内部的值：
``` java
List<AnnotationValue> values = (List<AnnotationValue>) annoValue.getValue();
```
因为我们定义了Service.value是一个Class数组，所以这里我们需要把返回值强制转换成List。

为了获取Service.value里面定义的Class信息，我们需要使用TypeMirror API
``` java
DeclaredType declaredType = (DeclaredType) value.getValue();
TypeElement typeElement = (TypeElement) declaredType.asElement();
```

有了TypeElement，我们就可以访问该类型相关的信息了，比如获取其完整类名：
``` java
typeElement.getQualifiedName().toString();
```
当然获取它的实现的接口或者定义在它内部的方法或者字段都是可以的。

### 完整代码
``` java
AnnotationMirror svcAnnoMirror =
    MoreElements.getAnnotationMirror(classElement, Service.class).get();
List<String> types = new ArrayList<>();
Observable.from(svcAnnoMirror.getElementValues().entrySet())
      .filter(entry -> "value".equals(entry.getKey().getSimpleName().toString()))
      .map(Map.Entry::getValue)
      .flatMap(annoValue -> Observable.from((List<AnnotationValue>) annoValue.getValue()))
      .map(annoValue -> (DeclaredType) annoValue.getValue())
      .map(declaredType -> (TypeElement) declaredType.asElement())
      .map(typeElem -> typeElem.getQualifiedName().toString())
      .subscribe(types::add, logger::error);
```
上面代码使用rxJava以及Lambda表达式来简化代码。

参见我的项目源码：[这里](https://github.com/minjing/uapi/blob/master/uapi.kernel.annotation/src/main/java/uapi/annotation/AnnotationsHandler.java)
