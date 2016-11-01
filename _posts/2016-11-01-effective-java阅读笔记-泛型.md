---
layout: post
title: effective java阅读笔记-泛型
---

第五章，泛型确保了编译时的类型安全，有效的避免了运行时由于类型转换导致的错误

<!--more-->

## 23. 不要在新代码中使用原生态类型

### tips:
* `List<E>`的原生态类型是`List`，`List<String>`和`List<Integer>`公用一个Class对象即`List.class`
* 支持原生态类型，主要是向前兼容老版本java的版本
* 泛型可以实现编译时的类型安全检查
* 泛型信息在运行时被**擦除**
* 泛型子类型化规则，`List<String>`是List的子类型，不是`List<Object>`的子类型
* 例外：类文字(class literal)和`instanceof`，`List.class`合法，`List<String>.class`不合法

## 24. 消除非受检警告

### tips:
* 可以确认代码是类型安全的的时候，使用`@SuppressWarnings("unchecked")`
* 在尽可能小的范围内使用SuppressWarnings注解

## 25. 列表优于数组

### tips:
* 数组是协变的(covariant)，Sub是Super的子类型，`Sub[]`就是`Super[]`的子类型；相反的，泛型是不可变的(invariant)，对于任意两个不同的Type1和Type2，`List<Type1>`既不是`List<Type2>`的超类型，也不是其子类型
* 泛型只是在编译时强化他们的类型信息，并在运行时丢弃（擦除），擦除就是使泛型可以与没有使用泛型的代码随意进行互用
* 创建泛型数组是非法的
* 数组是协变且可以具体化的，泛型是不可变且可以擦除的；数组提供了运行时的类型安全，但是没有编译时的类型安全，泛型则相反

## 26. 优先考虑泛型

## 27. 优先考虑泛型方法

### tips:
* 泛型类和泛型方法

## 28. 利用有限制通配符来提升API的灵活性

### tips:
* **PESC**: producer-extends, consumer-super，如(28.1)
* 如果某个输入参数既是生产者，又是消费者，这时候需要严格的类型匹配
* Comparable和Comparator都是消费者
* 如果类型参数只在方法声明中出现一次，就可以用通配符取代它

### sample code:

{% highlight java %}
//sample code 28.1 pushAll
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}

//sample code 28.1 popAll
public void popAll(Collection<? super E> dst) {
    while(!isEmpty())
        dst.add(pop());
}
{% endhighlight %}

## 29. 优先考虑类型安全的异构容器

### tips:
* 将键(key)进行参数化而不是将容器(container)参数化，如(29.1)
* class literal用于传达编译和运行时的类型信息时，被称作type token
* 使用原生类型使用Class对象，可以破坏这种实现的类型安全
* 不能用在不可具体化(non-reifiable)的类型中，可以保存`String`, `String[]`，但是不能保存`List<String>`，因为`List<String>.class`本身就是不合法的

### sample code:

{% highlight java %}
// sample code 29.1
public class Favorites {
    private Map<Class<?>, Object> m = 
        new HashMap<>();

    public <T> void put(Class<T> type, T instance) {
        if (type == null) {
            throw new NullPointerException("type is null");
        }
        m.put(type, instance);
    }

    public <T> T get(Class<T> type) {
        return type.cast(m.get(type));
    }
}
{% endhighlight %}

