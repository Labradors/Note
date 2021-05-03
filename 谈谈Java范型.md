---
title: 谈谈Java范型
date: 2021-05-03 14:30:24
tags:  Java
categories: Java
---

![2017031158256cat1.jpg](http://7xk0q3.com1.z0.glb.clouddn.com/2017031158256cat1.jpg)

工作一年有余，用过一些范型，但很少。做移动端的时候，看过`Okhttp`源码，其中使用了众多范型和设计模式的内容。学习后端后，`Spring`也使用了范型，这篇文章是作为工作以后作为范型的回顾。也是深入学习范型。范型作为Java高级部分的内容，不容忽视。Java使用范型的目的是加强类型安全和减小类的转换次数。Java范型的参数只能代表类，不能代表个别对象，Java编译时会进行类型擦除，所以无法在运行时得知参数的具体类型，也不能使用基本值类型。运行时是使用生成的字节码进行反射的调用。 

<!--more-->

## 范型类

范型的标志`<T>`

声明范型类和一般的类没有什么差别，只是在类的后面使用`<T>`即表示范型类。如下:

```java
package tech.jiangtao.fouction;

import java.util.ArrayList;

/**
 * Java范型学习和回顾 范型类 范型接口 范型方法
 */
public class Generics<T> {
    private ArrayList<T> list;

    public Generics() {
        list = new ArrayList<T>();
    }

    private ArrayList<T> add(T t) {
        list.add(t);
        return list;
    }

    private void print() {
        for (T aList : list) {
            System.out.println(aList.toString());
        }
    }

    public static void main(String[] args) {
        Generics<Integer> generics = new Generics<>();
        generics.add(1);
        generics.add(2);
        generics.add(3);
        generics.print();
        Generics<Integer> generics1 = new Generics<>();
      	generics.add("a");
        generics.add("b");
        generics.add("c");
        generics.print();
    }
}

```

如果要对范型类型进行限定。如:

```java
public class Generics<T extends Integer> 
```



## 范型接口

范型接口其实与范型类并没有什么特别的差别，使用场景可以使用范型来作为生成器。比如:

```java
public interface GenericsInterface<T> {
    public ArrayList generate();
}
```

```java
package tech.jiangtao.fouction;

import java.util.ArrayList;

/**
 * Java范型学习和回顾 范型类 范型接口 范型方法
 */
public class Generics<T> implements GenericsInterface<T> {
    private ArrayList<T> list;

    public Generics() {
        list = new ArrayList<T>();
    }

    @Override
    public ArrayList generate() {
        return list;
    }
}

```



## 范型方法

范型方法使用方式有所差别，范型方法声明是在函数返回值前面声明。

```java
    public <C> void next(C c){
        if (c instanceof Integer){
            System.out.println("范型类型是Integer类型");
        }
    }
```

## 实例分析

`Java Collection`中有很多范型的案例。先来看看`List`：

![](http://7xk0q3.com1.z0.glb.clouddn.com/2017031131874Screen Shot 2017-03-11 at 7.58.26 PM.png)

从上面可以看出`AbstractList`实现了遍历接口`Iterable`,集合接口`Collection`,列表接口`List`。实现类有`AbsctractSequentialList`,`ArrayList`,`Vector`。 而集合中保存的数据并不确定，并不是一类数据，为了容器能够保存多种数据，所以必须使用范型。看看最简单的`Iterable`源码。

```java
package java.lang;

import java.util.Iterator;
import java.util.Objects;
import java.util.Spliterator;
import java.util.Spliterators;
import java.util.function.Consumer;

public interface Iterable<T> {
    
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}

```

`Iterator`包含集合遍历的功能，其中包含了接口和函数的范型使用。

最后看到`super`关键字，提一下，`extends`关键字表示`T`可以是该类和其子类。而`super`关键字表示 `T`可以是该类和其父类直至`Object`.

就这样吧。不想写了。看算法玩去。