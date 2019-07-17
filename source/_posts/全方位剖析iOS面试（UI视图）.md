---
title: 全方位剖析iOS面试 — UI视图
date: 2019-04-22 16:26:44
tags:
---


### 目录
1. UITableView相关
2. 事件传递&视图响应
3. 图像显示
4. 绘制原理&异步绘制
5. 离屏渲染



### 一、UITableView相关

#### 重用机制

通过为每个单元格指定一个重用标识符，即指定了单元格的种类,当屏幕上的单元格滑出屏幕时，系统会把这个单元格添加到重用队列中，等待被重用，当有新单元格从屏幕外滑入屏幕内时，从重用队列中找看有没有可以重用的单元格，如果有，就拿过来用，如果没有就创建一个来使用。


#### 数据源同步

我们一般在主线程中刷新UI，然后在子线程中去加载网络数据和数据解析。这时候，假如我们用户要在点击删除广告这一操作，这时候子线程又在加载数据（显然我们是在不同线程对同一资源做操作了），我们怎么通知子线程让他在显示数据的时候知道我们删除了这条广告呢。



#### 解决方案

- **并发访问，数据拷贝**

  并发及多个线程都可以执行在同一段时间，不需要互相等待，主线程与用户互动，子线程加载cell所需要的网络数据以及预排版。

  **解决方法：**如下图，主线程首先拷贝一份数据给子线程完成预排版，网络请求与数据解析(json xml转化)，这时候如果主线程需要删除某些数据源操作，他就记录这条删除操作，在子线程完成各种加载操作后将这条操作与子线程进行同步一下，然后再回到主线程刷新界面。

  **缺点：**可能需要拷贝大量数据，比较消耗内存。

  ![并发访问数据拷贝](https://upload-images.jianshu.io/upload_images/2910208-c242578c18728ac1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  

- **串行访问**

  创建一个 `GCD` 串行队列，主线程增删改操作需要等待子线程操作完成。

  **解决方法：**如图，我们首先使用 `GCD` 创建一个串行队列，子线程先加入队列完成网络加载操作，如果这时候主线程需要修改数据源，这个操作就要等待子线程完成才去进行(串行执行)。

  **缺点：**可能子线程的网络请求速度慢，主线程UI操作等待时间长。

  ![串行访问](https://upload-images.jianshu.io/upload_images/2910208-3a643b0fb266206b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






### 二、事件传递&视图响应

#### UIView和CALayer的关系

  UIView为其提供内容，以及负责处理触摸等事件，参与响应链
  CALayer负责显示内容contents
  使用了单一职责原则来区分这两者



#### 三种 iOS 事件
Touch Events(触摸事件)
Motion Events(运动事件，比如重力感应和摇一摇等)
Remote Events(远程事件，比如用耳机上得按键来控制手机)

  

#### Touch Events(触摸事件)
`Touch Events(触摸事件)` 事件的整个过程可以分为 `传递` 和 `响应` 2 个阶段。
   **传递**：是当我们触摸屏幕时，为我们找出最适合的 `View`，
   **响应**：当我们找出最适合的 `View` 后，此时只是找到了最合适的 `View`，但未必此 `View` 可以响应此事件，所以需要继续找出能响应此事件的 `View`。

  

#### 事件传递

- **传递过程**

  > 每当手指接触屏幕，操作系统会把事件传递给当前的 `App`， 在 `UIApplication ` 接收到手指的事件之后，就会去调用 `UIWindow` 的 `hitTest:withEvent:`，看看当前点击的点是不是在 `window` 内，如果是则继续倒序遍历 `subview` 并调用其 `hitTest:withEvent:` 方法，直到找到最后需要的 `view`。调用结束并且 `hit-test view   ` 确定之后，便可以确定最合适的 `View`。

  

  ![事件传递](https://upload-images.jianshu.io/upload_images/2910208-fcb10a588db2b424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  

  ![事件传递具体实现](https://upload-images.jianshu.io/upload_images/2910208-a4ab8ebd62b4187f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  ```objective-c
  // 内部实现(https://github.com/BigZaphod/Chameleon/blob/master/UIKit/Classes/UIView.m)
    - (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
      if (self.hidden || !self.userInteractionEnabled || self.alpha < 0.01 || ![self pointInside:point withEvent:event] || ![self _isAnimatedUserInteractionEnabled]) {
          return nil;
      } else {
          for (UIView *subview in [self.subviews reverseObjectEnumerator]) {
              UIView *hitView = [subview hitTest:[subview convertPoint:point fromView:self] withEvent:event];
              if (hitView) {
                  return hitView;
              }
          }
          return self;
      }
  }
  ```

  

#### 视图响应

当我们知道最合适的 View 后，事件会由上向下【子view -> 父view，控制器view -> 控制器】来找出合适响应事件的 View，来响应相关的事件。如果当前的 View 有添加手势，那么直接响应相应的事件，不会继续向下寻找了，如果没有手势事件，那么会看其是否实现了如下的方法：

```objective-c
  - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
  - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
  - (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
  - (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```

如果有实现那么就由此 View 响应，如果没有实现，那么就会传递给他的下一个响应者【子view -> 父view，控制器view -> 控制器】

这里我们可以做一个简单的验证，在默认情况下 UIView 是不响应事件的，UIControl 就算没有添加手势一样的会由他来响应， 这里可以使用 runtime 查看 UIView 和 UIControl 的方法列表， 或查看 [UIKit 源码](https://github.com/BigZaphod/Chameleon/blob/master/UIKit/Classes/UIView.m) 可知， UIView 没有实现如上的 `touchesBegan`方法，而 `UIControl` 是实现了如上的相关方法，所以验证了刚才的 UIView 不响应和 UIControl 的响应。一旦找到最合适响应的 View 就结束, 在执行响应的绑定的事件，如果没有就抛弃此事件（需要注意的是应用不会崩溃）。



#### 问题解答

- 为什么父 View 关闭了事件响应时，子 View 就无法响应事件？
> 因为在事件传递的时，先到父 view，当父 view 无法响应事件，直接就跳过了遍历其子 view ，故只要父类关闭了事件，子 view 就已经没有机会响应事件了。



- 如何扩大 Button 的点击范围？
> 扩大点击范围，无非就是想本来没有点击 btn 但想让 btn 响应事件，那么可以在 hitTest 方法中用做适当的操作，当满足xxx条件时，强行返回 btn 来达到最佳点击范围的效果。



- 为什么子 View 关闭了事件，但其父 View 开启事件的情况下，点击子 View 时，父 View 可以响应事件？
> 子 view 关闭了事件，事件的传递是父 view 到子 view，在父 view 时，父 view 可以响应，那么会继续访问其 子 view 是否可以响应，如果此时子 view 不可以响应，那么他会直接返回父 view，所以子 View 关闭了事件 父 View  正常执行事件是必然的。



- 如何让超出父视图范围的子视图响应事件，在UIView范围外响应点击？
> ```objective-c
> // 重写父视图里的hitTest方法
> - (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
> 	UIView *view = [super hitTest:point withEvent:event];
> 	if (view == nil) {
>  		 for (UIView *subView in self.subviews) {
>      		CGPoint tp = [subView convertPoint:point fromView:self];
>      		if (CGRectContainsPoint(subView.bounds, tp)) {
>          		view = subView;
>      		}
>  		 }
> 	}
> 	return view;
> }
> ```



#### 参考链接
[浅谈 iOS 事件的传递和响应过程](https://www.jianshu.com/p/481465fc4f2d)





### 三、图像显示

#### 屏幕的成像原理

在屏幕成像的过程中，CPU和GPU起着至关重要的作用。
CPU(Central Processing Unit，中央处理器) ：对象的创建和销毁、对象属性的调整、布局计算、文本的计算和排版、图片的格式转换和解码、图像的绘制（Core Graphics）。
GPU(Graphics Processing Unit，图形处理器) ：纹理的渲染。



#### CPU、GPU的工作流程

![[工作流程]
](https://upload-images.jianshu.io/upload_images/2910208-f124bd3f15affc80.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![工作流程](https://upload-images.jianshu.io/upload_images/2910208-37723aef7dbdc45f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![工作流程](https://upload-images.jianshu.io/upload_images/2910208-737c0086b5277938.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 卡顿&掉帧的原因
- 在iOS中是双缓冲机制，有前帧缓存、后帧缓存
- 屏幕显示的时候，是由垂直同步信号 (VSync) 和水平同步信号 (HSync) 组成，先发出垂直同步信号，再一行一行的发出水平同步信号进行显示
  ![同步信号](https://upload-images.jianshu.io/upload_images/2910208-6e682a6525deaceb.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 按照每秒 60FPS 的刷帧率，每隔16.7ms 就会有一次 VSync 信号，由于垂直同步的机制，如果某一次 CPU 和 GPU 工作的总耗时超过 16.7ms，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变，就会造成卡顿掉帧
  ![卡顿原因](https://upload-images.jianshu.io/upload_images/2910208-9835adfe9267f9bd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 卡顿优化方案

`CPU`

- 尽量用轻量级的对象，比如用不到事件处理的地方，可以考虑使用 CALayer 取代 UIView
- 不要频繁地调用 UIView 的相关属性，比如 frame、bounds、transform 等属性，尽量减少不必要的修改
- 尽量提前计算好布局，在有需要时一次性调整对应的属性，不要多次修改属性
- Autolayout 会比直接设置 frame 消耗更多的 CPU 资源
- 图片的 size 最好刚好跟 UIImageView 的 size 保持一致
- 控制一下线程的最大并发数量
- 尽量把耗时的操作放到子线程
- 预先排版（布局计算、文本计算)
- 预先渲染（图片编解码、文本等异步绘制）



`GPU`

- GPU能处理的最大纹理尺寸是 4096x4096，一旦超过这个尺寸，就会占用 CPU 资源进行处理，所以纹理尽量不要超过这个尺寸
- 尽量减少视图数量和层次
- 减少透明的视图（alpha<1），不透明的就设置 opaque 为 YES
- 尽量避免出现离屏渲染





### 四、绘制原理&异步绘制

#### UIView的绘制原理

![UIView的绘制原理](https://upload-images.jianshu.io/upload_images/2910208-7d95fc5ed8e7f256.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`UIView` 调用 `setNeedsDisplay` 方法后，实际上并没有发生当前视图的绘制工作，而是在之后的某一时机进行绘制工作，为什么会在之后的某一时机进行绘制工作呢？当 `UIView` 调用 `setNeedDisplay   
`之后，系统会调用`view`对应 `layer  `的 `setNeedsDisplay` 方法,相当于在当前 `layer` 上打上了一个脏标记，然后会在当前 `runloop` 即将结束的时候调用 `CALayer` 的 `display` 方法，才会真正的进入当前视图的绘制流程当中，所以视图的绘制时机，是在当前 `runloop` 即将结束的时候才会开始。

`CALayer `的 `display ` 方法的内部实现，首先会判断 `layer ` 的 `delegete` 是否响应 `display` 方法，如果代理不响应就会进入到系统的绘制流程当中，如果响应，实际上就为我们提供了异步绘制的接口，这样就构成了 `UIView` 的绘制原理。



#### 系统的绘制流程

同样看一副流程图

![系统的绘制流程](https://upload-images.jianshu.io/upload_images/2910208-2f2875499d0a5214.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先 `CALayer `会在内部创建一个 `backing store(CGContextRef)` ，我们一般在 `drawRect` 中可以通过上下文堆栈当中拿到当前栈顶的 `context` 。然后 `layer` 判断是否有代理，如果没有代理会调用 `layer` 的 `drawInContext` 方法，如果实现了代理就会调用 `delegete` 的 `drawLayer:inContext `方法，这是在发生在系统内部当中的，然后在合适的时机给予回调方法，也就是 `View` 的 `drawRect`(默认是什么都不做的) 方法。可以通过 `drawRect` 方法做一些其他的绘制工作，然后无论哪两个分支，都有 `calayer `上传 `backing store`  (最终的位图)到 `CPU` ，然后结束系统的绘制流程。



#### 异步绘制

怎么进行异步绘制呢，其实就是基于系统给我们开的口子 `layer.delegate` ,如果遵从或者实现了 `displayLayer` 方法,我们就可以进入到异步绘制流程当中，在异步绘制的过程当中

1. 就由 `delegete `去负责生成 `bitmap` 位图
2. 设置改 `bitmap` 作为 `layer.content` 属性的值

通过一副时序图来了解异步绘制的机制和流程
![异步绘制](https://upload-images.jianshu.io/upload_images/2910208-c0bd2f4b452c1f43.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





### 五、离屏渲染

#### GPU屏幕渲染的两种方式

- On-Screen Rendering (当前屏幕渲染) 
  指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区进行。

- Off-Screen Rendering (离屏渲染)
  指的是在GPU在当前屏幕缓冲区以外开辟一个缓冲区进行渲染操作。

当前屏幕渲染不需要额外创建新的缓存，也不需要开启新的上下文，相对于离屏渲染性能更好。但是受当前屏幕渲染的局限因素限制(只有自身上下文、屏幕缓存有限等)，当前屏幕渲染有些情况下的渲染解决不了的，就使用到离屏渲染。



#### 为什么要避免离屏渲染

相比于当前屏幕渲染，离屏渲染的代价是很高的，主要体现在两个方面：
- 创建新缓冲区
  要想进行离屏渲染，首先要创建一个新的缓冲。

- 上下文切换
  离屏渲染的整个过程，需要多次切换上下文环境：先是从当前屏幕（On-Screen）切换到离屏（Off-Screen），等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上又需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是要付出很大代价的。



#### 既然离屏渲染这么耗性能,为什么有这套机制呢？

有些效果被认为不能直接呈现于屏幕，而需要在别的地方做额外的处理预合成。图层属性的混合体没有预合成之前不能直接在屏幕中绘制，所以就需要屏幕外渲染。屏幕外渲染并不意味着软件绘制，但是它意味着图层必须在被显示之前在一个屏幕外上下文中被渲染（不论CPU还是GPU）。



#### 何时触发离屏渲染？

- 为图层设置遮罩蒙版（layer.mask）
- 设置圆角，将图层的 layer.masksToBounds / view.clipsToBounds 属性设置为 true
- 为图层设置阴影（layer.shadow ）
- 将图层 layer.allowsGroupOpacity 属性设置为 YES 和 layer.opacity 小于1.0
- 光栅化，为图层设置 layer.shouldRasterize = true
- 具有 layer.cornerRadius，layer.edgeAntialiasingMask，layer.allowsEdgeAntialiasing （抗锯齿）的图层