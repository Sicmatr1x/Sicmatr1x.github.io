---
title: Java复习笔记：注解
date: 2021-05-13 14:49:28
categories:
    - 设计模式
tags: 
    - JavaScript
cover: /img/article-cover/jigsaw_puzzle_hires.jpg
---


## Java 内置注解

### `@Override`

定义在：`java.lang.Override` 中

只能用于修辞方法，表示一个方法声明打算重写超类中的另一个方法。

### `@Deprecated`

定义在：`java.lang.Deprecated` 中

可以用于修辞方法、属性、类，表示不推荐程序员使用的元素，通常意味着它存在危险或有更好的选择

### `@SuppressWarnings`

定义在：`java.lang.SuppressWarnings` 中

用于抑制编译时的警告信息

- `@SuppressWarnings("all")`: 抑制所有类型的警告
- `@SuppressWarnings(value={"unchecked", "rawtypes"})`: 抑制所多种类型的警告
- `@SuppressWarnings("unchecked")`: 抑制unchecked类型的警告

参数：
- `all`: to suppress all warnings
- `boxing` : to suppress warnings relative to boxing/unboxing operations
- `cast`: to suppress warnings relative to cast operations
- `dep-ann`: to suppress warnings relative to deprecated annotation
- `deprecation`: to suppress warnings relative to deprecation
- `fallthrough`:  to suppress warnings relative to missing breaks in switch statements
- `finally` : to suppress warnings relative to finally block that don’t return
- `hiding`: to suppress warnings relative to locals that hide variable
- `incomplete-switch`:  to suppress warnings relative to missing entries in a switch statement (enum case)
- `nls`:  to suppress warnings relative to non-nls string literals
- `null`: to suppress warnings relative to null analysis
- `rawtypes`: to suppress warnings relative to un-specific types when using generics on class params
- `restriction`: to suppress warnings relative to usage of discouraged or forbidden references
- `serial`: to suppress warnings relative to missing serialVersionUID field for a serializable class
- `static-access`: o suppress warnings relative to incorrect static access
- `synthetic-access` :  to suppress warnings relative to unoptimized access from inner classes
- `unchecked`:  to suppress warnings relative to unchecked operations
- `unqualified-field-access`: to suppress warnings relative to field access unqualified
- `unused`: to suppress warnings relative to unused code

## 元注解(meta-annotation)

用于注解其它注解的注解

### 1. `@Target`

用于描述注解的使用范围

example:

```java
@Target(value = ElementType.TYPE)
```

可用`{}`来给value传入数组

```java
@Target(value = {ElementType.TYPE, ElementType.METHOD})
```

value参数值为枚举类型，定义如下：

```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

### 2. `@Retention`

表示需要什么级别保存该注解信息，用于描述注解的生命周期

取值：`SOURCE` < `CLASS` < `RUNTIME`

- `SOURCE`: 保留到源码
- `CLASS`: 保留到字节码
- `RUNTIME`: 保留到虚拟机

```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

### 3. `@Document`

表示该注解将被包含在javadoc中

### 4. `@Inherited`

表示子类可以继承父类中的该注解

## 自定义注解

## 反射(Reflection)

反射(Reflection)是Java被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类型的内部信息，并能直接操作任意对象的内部属性及方法。

加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个类只有一个Class对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。




