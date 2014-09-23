---
layout: post
type: android
title: "创建自定义和混合视图教程"
modified: 2014-9-18 18:21:30
excerpt: "如何创建自定义和混合视图"
tags: [android, custom view, tutorial]
comments: true
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>主要内容</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

[原文链接 Creating custom and compound Views in Android - Tutorial](http://www.vogella.com/tutorials/AndroidCustomViews/article.html#tutorial_compoundcontrols3)

## 1.Custom View-自定义视图
### 1.1.默认视图
Android框架提供了几种默认的视图，但开发者也可以为自己的应用自定义视图。最基础的视图时View,如下图所示

<figure>
   <a href="http://chiemyblog.qiniudn.com/xandroid_viewhierarchy10.png.pagespeed.ic.nQTkkQQ3Y8.png"><img src="http://chiemyblog.qiniudn.com/xandroid_viewhierarchy10.png.pagespeed.ic.nQTkkQQ3Y8.png"></a>
   <figcaption></figcaption>
</figure>

### 1.2.Android是如何绘制视图层级的
一旦Activity获得焦点，它必须为Android系统提供根节点的布局。然后，Android系统开始着手绘制。

绘制从布局的根节点开始。根据声明的顺序对布局层级进行遍历，例如，父视图在子视图之前被绘制，子视图按照声明顺序绘制。

绘制布局需要两个过程：

- 测量过程 - 在`measure(int, int)方法内执行`，从上到下遍历视图层级。每个视图保存自己的尺寸。
- 布局过程 - 	在`layout(int, int, int, int)` 方法内实现，也是从上到下进行遍历。在此过程中，每个布局负责它的子视图的位置。使用的视图大小是measure的结果。

>注意<br/>
>1.measure和layout过程总是同时发生<br/>
>2.布局管理器（Layout Managers）会多次调用measure过程。For example LinearLayout supports the weight attribute which distributes the remaining empty space among views and RelativeLayout measures child views several times to solve constraints given in the layout file.

视图或Activity可以通过调用`requestLayout()`方法触发`measure`和`layout`过程。

在`measure`和`layout`之后，`view`就开始绘制了。绘制过程可以通过在View内部调用`invalidate()`方法触发。

### 1.3.创建视图的理由
一些情况下通过默认视图不能提供独特的用户交互功能，通过自定义视图的方式可以更好的满足特定操作。

### 1.4.视图的职责
View对自己的测量，布局，绘制负责，同时也负责保存UI状态，控制点击事件。

### 1.5.创建自定义视图的方式
1. 复合视图（Compound View）
2. 自定义视图
	- 通过继承已有的视图
	- 直接继承View类

### 1.6.视图的截屏

{% highlight java %}
//Build the Drawing Cache
view.buildDrawingCache();

//Create Bitmap
Bitmap cache = view.getDrawingCache();

//Save Bitmap
saveBitmap(cache);
view.destroyDrawingCache(); 
{% endhighlight %}

## 2.复合视图
复合视图是基于已有的视图，而预先配置的`ViewGroup`。

对于这样一个控制，你需要定义一个布局文件,并将它分配给你的复合视图。在复合视图的实现中预定义好视图的交互。你需要创建一个布局文件，并继承相应的`ViewGroup`类，在这个类的内部`inflate`布局文件，并实现视图间的逻辑。

## 3.创建自定义视图

### 3.1.创建
你可以通过继承`View`类或其子类创建自定义视图。

在`onDraw()`方法中，你会得到一个`Canvas`对象，我们在它上实现一些绘制操作。例如，画线，圆，文字或者是位图。如果视图需要被重新绘制，那么就调用`invalidate()`方法，它会再次触发`onDraw()`方法。

>提示：`ViewConfiguration`类保存了一些常量，可能在自定义视图时用到，建议你了解下这个类。

绘制视图通常要用到2D Canvas API.

### 3.2.测量
布局管理器调用`onMearsure()`方法。视图从布局管理器中获得布局参数。布局管理器决定了子视图的大小。

视图必须将结果赋给`setMeasuredDimenstion(int, int)`方法。

### 3.3.自定义布局管理器
你可以通过继承`ViewGroup`类来实现自定义的布局管理器。	This allows you to implement more efficient layout managers or to implement effects which are currently missing in the Android platform.？？？？

自定义的布局管理器，可重写`onMeasure()`和`onLayout()`方法，并计算子类大小。例如，它可以省去对LinearLayout类中耗时的`layout_weight`属性的支持。？？？？？

>提示：`ViewGroup`的`measureChildWithMargins()`方法，可以实现对子视图尺寸的计算。

在你的`ViewGroup`实现中保存一些额外的布局参数是一个好的方法。例如，ViewGroup.LayoutParams的实现控制布局参数， LinearLayout.LayoutParams有其特定的参数，比如，layout_weight。

## 4.生命周期

### 4.1.和窗口相关的声明周期

当窗口可用时，回调`onAttachedToWindow()`方法；

当视图从父视图（和窗口关联）移除时，会调用`onDetachedFromWindow()`方法，例如，activity被回收（如通过`finish()`方法），或者在`ListView`中的视图被回收时。此方法可以用来停止动画和清除被视图使用的资源。

### 4.2.遍历声明周期事件
包括`Animate`,`Measure`,`Layout`以及`Draw`

<figure>
   <a href="http://chiemyblog.qiniudn.com/xview_traversallifecycle10.png.pagespeed.ic.xtwuGrv__z.png"><img src="http://chiemyblog.qiniudn.com/xview_traversallifecycle10.png.pagespeed.ic.xtwuGrv__z.png" align="middle"></a>
   <figcaption></figcaption>
</figure>

调用`requestLayout()`方法，会触发`measure`和`layout`过程，因为可能会影响到其他视图的布局，所以要调用父类的`requestLayout()`方法。

>提示：因为递归过程，可能会使得`measure`和`layout`过程有很大的开销，所以不建议布局嵌套的太深。

`onMeasure()`方法决定了视图自身和子视图的大小。在方法返回结果之前，必须调用`setMeasuredDimension()`方法。

`onLayout()`方法基于`onMeasure`的结果，进行视图的布局。

## 5.定义额外的属性

在`res/values`文件夹下，创建一个`attrs.xml`文件，名称任意。举例：

定义属性
<script src="https://gist.github.com/chiemy/2a8f10cc32886cf40c24.js?file=attrs.xml"></script>

引用属性
<script src="https://gist.github.com/chiemy/2a8f10cc32886cf40c24.js?file=attrs_use_demo.xml"></script>

在代码中获得属性值
<script src="https://gist.github.com/chiemy/2a8f10cc32886cf40c24.js?file=access_attrs_demo.java"></script>


##未完待续……