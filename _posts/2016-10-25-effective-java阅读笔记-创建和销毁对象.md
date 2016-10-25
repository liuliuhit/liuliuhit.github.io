---
layout: post
title: effective java阅读笔记-创建和销毁对象
---

之前看过一遍，有一些章节看的比较仔细，有一些章节看的比较粗略，打算再仔细精读一遍，顺便留点笔记便于记录和回忆每一条的重点。

第一节，创建和销毁对象，何时以及如何创建对象，何时以及如何避免创建对象...

## 1. 考虑用静态工厂方法替代构造器

### tips:
* 静态工厂方法不是指设计模式中的工厂方法
* 构造器参数不能正确描述返回的对象时，由于静态工厂方法有名称，可以有效的描述返回的对象，提高代码的可阅读性
* 不必每次调用的时候都返回一个新对象，返回预先创建的实例，避免不必要的重复创建，如(1.1)
* 可以返回原返回类型的任何子类型的对象

### sample code:

{% highlight java %}
// sample code 1.1
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
{% endhighlight %}

---

## 2. 遇到多个构造器参数的时候考虑使用构建器(builder模式)

### tips:
* 可以类比一下JavaBean，但是JavaBean无法实现immutable，builder模式可以避免这个很好的解决这个问题
* 抽象工厂(Abstract Factory)，如(2.1)

### sample code: 
{% highlight java %}
// sample code 2.1
public interface Builder<T> {
    public T Build();
}
{% endhighlight %}

---

## 3.用私有构造器或者枚举类型强化Singleton属性

### tips:
* 单例模式，懒汉实现方式，如(3.1)
* 使用包含单个元素的枚举类型实现Singleton，如(3.2)

### sample code:

{% highlight java %}
// sample code 3.1
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() {
        // private constructor
    }

    public static Singleton getInstance() {
            return INSTANCE;
    }

    public void doSomething() {}
}
{% endhighlight %}

{% highlight java %}
// sample code 3.2
public enum Singleton {
    INSTANCE;

    public void doSomething() {}
}
{% endhighlight %}

---

## 4.通过私有构造器强化不可实例化的能力

### tips:
* 例如实现工具类(utility class)，创建实例毫无意义，但是编译器会创建一个缺省的无参构造器
* 不要通过使用抽象类实现不可实例化，抽象类可以被继承，子类往往可以实例化

### sample code:

{% highlight java %}
// sample code 4.1
public class UtilityClass {
    // Suppress default constructor for noninstantiablity
    private UtilityClass() {
        // private构造器已经足以避免外部对该类进行实例化，throw Exception是为了避免在类的内部进行实例化 
        throw new AssertionError();
    }
}
{% endhighlight %}

---

## 5.避免创建不必要的对象

### tips:
* 当对象是immutable的时候，它就可以始终被重用，也可以重用一些不会被修改的对象，如(1.1)，使用Boolean.valueOf(String)几乎总是优先于构造器Boolean(String)
* 避免无意识的自动装箱，优先使用基本类型而不是装箱类型
* 维护对象池来避免重复创建对象，只有当对象池中的对象的创建是非常重量级的，否则避免使用这种方式，例如数据库连接池
* 小对象的创建和回收动作是非常廉价的，通过创建附加的对象来提高程序的清晰性和简洁性
* 保护性拷贝：**当你应该重用现有对象的时候，请不要创建新的对象**，**当你应该创建新对象的时候，请不要重用现有的对象**，参考第39条

---

## 6.消除过期的对象引用

### tips:
* 避免无意识的持有过期的对象引用导致的内存泄漏
* 消除过期的对象引用，最好的方式是让包含改引用的变量结束生命周期
* 缓存中，使用WeakHashMap

---

## 7.避免使用终结方法

### tips:
* 使用显式的try-finally
