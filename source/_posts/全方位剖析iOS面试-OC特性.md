---
title: 全方位剖析iOS面试 — OC特性
date: 2019-04-27 15:41:39
tags:
---



### 目录

1. 分类 & 扩展
2. KVC
3. KVO
4. 属性关键字


### 一、分类 & 扩展



### 分类

##### 分类能干什么？
- 为现有类添加实例方法

- 为现有类添加类方法（静态方法）

- 为现有类添加属性（手动实现 `getter` 和 `setter` 的方式）

- 为现有类添加协议

- 添加实例变量（关联对象技术）

- 公开私有方法

  


##### 分类的特点

- 分类是运行时决议，也就是通过 `runtime` 方法在运行时才将添加的方法、属性添加到宿主类上。

- 可以为系统类添加分类

- 为同一个宿主类添加多个分类时，如果存在同名方法，那么最后编译的方法会生效。

- 名字相同的分类会引起编译报错

- 分类添加的方法可以“覆盖”原类方法（本质上是分类的方法优先调用）

  


##### 相关面试题
- Category的实现原理？
> 1. 通过 `runtime` 加载某个类的所有 `Category` 数据。
> 2. 把所有 `Category` 的方法、属性、协议数据合并到一个大数组中，后参与编译的 `Category` 数据，会存放在数组的前面。
> 3. 将合并后的分类数据（方法，属性，协议），插入到类原来数据的前面。



- Category 为什么只能加方法不能加属性？
>`Category` 可以添加属性，只是不会自动生成成员变量及 `set/get` 方法。因为 `category_t` 结构体中并不存在成员变量。成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法在程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量。



##### 分类的实现原理

参考文章：[https://juejin.im/post/5aef0a3b518825670f7bc0f3](https://juejin.im/post/5aef0a3b518825670f7bc0f3)



### 扩展
扩展是分类的一种特殊形式，也被称作匿名分类。



##### 一般用扩展做什么
- 声明私有属性

- 声明私有方法

- 声明私有成员变量

  

##### 扩展的特点 (和分类的区别)
- 编译时决议

- 只以声明的形式存在，多数情况下寄生于宿主类的.m中

- 不能为系统类添加扩展

  


### 二、KVC
`KVC` 即是指 [NSKeyValueCoding](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Protocols/NSKeyValueCoding_Protocol/Reference/Reference.html#//apple_ref/occ/cat/NSKeyValueCoding)，一个非正式的 `Protocol`，提供一种机制来间接访问对象的属性。



主要有这俩个方法

```
- (id)valueForKey:(NSString *)key
- (void)setValue:(id)value forked:(NSString *)key;
```


通过一副流程图看一下 `valueForKey` 的实现逻辑
![实现逻辑](https://upload-images.jianshu.io/upload_images/2910208-7ac64e3cf32327cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



首先系统会判断访问的 `key` 是否有对应的 `getter` 方法，存在就直接进行调用，不存在就判断实例变量是否存在，通过 `accessInstanceVariablesDirectly` 来判断 ，默认是 `YES`。 如果不存在，会调用 `UndefinedKey` ，抛出异常。





### 三、KVO

##### 概述
`KVO` 全称 `KeyValueObserving`，是苹果提供的一套事件通知机制。允许对象监听另一个对象特定属性的改变，并在改变时接收到事件。`KVO` 就是基于 `KVC` 实现的关键技术之一。由于 `KVO` 的实现机制，所以对属性才会发生作用，一般继承自`NSObject`的对象都默认支持 `KVO`。

`KVO` 和 `NSNotificationCenter` 都是 `iOS` 中`观察者模式`的一种实现。区别在于，相对于被观察者和观察者之间的关系，`KVO` 是一对一的，`NSNotificationCenter` 是一对多的。`KVO` 对被监听对象无侵入性，不需要修改其内部代码即可实现监听。 

`KVO` 可以监听单个属性的变化，也可以监听集合对象的变化。通过 `KVC` 的 `mutableArrayValueForKey:` 等方法获得代理对象，当代理对象的内部对象发生改变时，会回调 `KVO` 监听的方法。集合对象包含 `NSArray` 和 `NSSet`。



##### 实现原理

`KVO` 是通过 `isa-swizzling(混写)` 技术实现的(这句话是整个 `KVO` 实现的重点)。在运行时根据原类创建子类，并重写了被观察属性的 setter 方法，最后动态修改当前对象的 `isa` 指针(`isa` 指针告诉 Runtime 系统这个对象的类是什么)指向中间类。
![实现原理](https://upload-images.jianshu.io/upload_images/2910208-40d7356e19e3db2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```
// 重写 setter 方法
- (void)setAge:(int)age {
    if (age != _age) {
        [self willChangeValueForKey:@"age"];
        _age = age;
        [self didChangeValueForKey:@"age"];
    }
}
```


##### 参考文章
- [https://www.jianshu.com/p/badf5cac0130](https://www.jianshu.com/p/badf5cac0130)
- [https://www.jianshu.com/p/703aafde0c40](https://www.jianshu.com/p/703aafde0c40)
- [https://tech.glowing.com/cn/implement-kvo/](https://tech.glowing.com/cn/implement-kvo/)

  

### 四、属性关键字
- 读写权限
  `readonly`
  `readwrite`

- 原子性
  `atomic`
  `nonatomic`

- 引用计数
`retail / strong`

  `assign`
修饰基本数据类型，如 `int` ,`BOOL` 等
修饰对象类型时，不改变其引用计数

  `weak`
不改变被修饰对象的引用计数
所指对象在被释放之后会自动置为nil
那么问题来了，weak对象修饰的对象为什么在被释放之后会置为nil？
[https://www.jianshu.com/p/df3c118cfbb0](https://www.jianshu.com/p/df3c118cfbb0)
[https://blog.csdn.net/xiaohuoziooo/article/details/88029300](https://blog.csdn.net/xiaohuoziooo/article/details/88029300)

  `copy`
浅拷贝和深拷贝的概念
![浅拷贝](https://upload-images.jianshu.io/upload_images/2910208-d0f93c3b741173ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![深拷贝](https://upload-images.jianshu.io/upload_images/2910208-c49c0587ae48dd3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/2910208-6b651bdace34ce03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

