---
title: 从APT中获取运行时类信息
tags:
    - Java
    - APT
    - Annotation
Categories: Development
---

JDK提供了APT(Annotation Processing Toll)使得我们在编译Java源代码的时候可以针对某些注解进行代码生成和修改，比在运行时通过反射来生成或修改字节码要相对简单并且运行时的效率要更高。

## 问题描述
在Java程序运行时可以通过反射来获取类的Meta信息，但是在APT中处理的是Java源代码，此时无法直接获取定义在Annotation里面的类型信息，因为没有反射API可以使用。
例如我们定义了一个注解：
``` java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface Service {

    Class<?> value();
}
```
该注解接受一个Class的参数，注意该注解仅可见于源代码编译过程
如果想在APT中获取Service.value，下面代码是不可行的：
``` java
...
Service service = classElement.getAnnotation(Service.class);
Class<?> cls = service.value();
...
```
classElement是一个javax.lang.model.element.Element实例，在这里它代表了申明了Service这个Annotation的类的元素。
运行这段代码会在编译时抛出异常。

## 解决方案
TODO:
