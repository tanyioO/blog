---
title: ant-modal中forceRender属性及其作用
tags: ['ant-modal']
date: 2019-06-24 16:27:55
permalink:
categories: UI组件库
description:
image:

---

<p class="description"></p>

#### ant-modal/forceRender

以下是[ant design](<https://ant.design/index-cn>)对于对话框的使用场景介绍：需要用户处理事务，又不希望跳转页面以致打断工作流程时，可以使用 modal 在当前页面正中打开一个浮层，承载相应的操作。对话框可细分成两类：模态对话框和非模态对话框。模态对话框会阻断应用程序，如果出现，直到对话框关闭，则其是应用程序当前唯一活动的东西；非模态对话框不会阻断应用程序，因此同时可以有多个非模态的对话框被激活。ant-modal是其组件库中的模态对话框，与之类似如bootstrap-modal也是模态对话框。

ant-modal属性`forceRender`属性的主要作用是：无论modal是隐藏或者显示，都会强制性的将modal渲染到DOM中，以满足在实际开发过程中的一些需求，例如：在进入modal前，提前获取到模态框中的DOM元素，作为图片加载容器，从而解决在网络环境较差时打开modal图片加载缓慢的问题。

<!--more -->

#### 强制渲染 or not？

在我所使用的一些"历史比较悠久"的UI组件库如bootstrap、layUI中，对于modal的处理是将其作为一个绝对定位元素渲染到DOM中，设置它的CSS display属性用于控制其显示和隐藏。这种方式在现在的”数据驱动“式开发中存在很多问题，例如：

* 首屏加载问题：当modal中包含数据请求时，数据量较大或者页面中的modal个数比较多时，可能会导致页面加载缓慢。
* 数据“残留”：如果某个操作表格列中包含modal，那么所有的表格行使用的其实是同一个modal。当modal中存在表单时，由于modal只是隐藏，并不销毁，再次打开modal时，表单中的数据为上一次修改的数据，需要在关闭modal时手动清除或重置表单。

所以，一开始大佬们在开发[rc-dialog](<https://github.com/react-component/dialog>)的时候考虑到了强制渲染所带来的问题，为此做了优化：

~~~javascript
const style = this.getWrapStyle();
// clear hide display
// and only set display after async anim, not here for hide
if (props.visible) {
  style.display = null;
}
~~~

ant-modal是基于rc-dialog封装的对话框组件（contributors是同一批大佬），因此7.3.0前的版本也就天生携带“不可见时不渲染”的属性。这种优化在一定程度上比较完美地解决上述问题，首先，modal在父节点挂载时不渲染，不会对首屏加载造成压力；同时，关闭modal时将其进行销毁，那么下一次打开的就是一个新的modal，从而解决了数据残留的问题。

当然，并不是所有的改变都是百利而无一害的。在实际开发时这种“强制不渲染”带来的问题就是：**无法在父组件挂载时拿到modal中的子元素**。然而在某些情况下在父组件挂载时提前获取modal中的子元素是很有必要的操作。

于是有开发者提了[issue #12503](<https://github.com/ant-design/ant-design/issues/12503>)：

![issue](<https://github.com/tanyioO/image-lib/raw/master/blog/ant-modal/issue.png>)

由此引发了ant design维护者大佬们的一番“激烈”的讨论，最终的讨论结果就是：合并之前开发者提交到rc-dialog的[pull request](<https://github.com/react-component/dialog/commit/f9a7c2169df21f521dcae57a568adcad08b192b2>)，添加`forceRender`属性，让开发者可以根据实现情况决定是否需要强制渲染。（撒花）

PR的实现原理是根据`props.forceRender`属性值，来判断应该设置`display`属性值为`none`或者`null`，源码如下：

~~~javascript
const style = this.getWrapStyle();
// clear hide display
// and only set display after async anim, not here for hide
// if the forceRender is true, change display value to none
if (props.forceRender && !props.visible) {
  style.display = 'none';
} else if (props.visible) {
  style.display = null;
}
~~~

rc-dialog 7.3.0及以后版本，and design 3.12.0及以后版本，正式支持`forceRender`属性。

#### 关注及使用

rc-dialog 7.3.0更新于2018年12月，在这之前在项目开发中遇到过这样一个使用场景：打开modal，使用ActiveX插件播放视频。在播放视频之前，需要拿到modal中的子元素<object\>标签。

> <object\>标签定义一个嵌入的对象。比如图像、音频、视频、Java applets、ActiveX、PDF 以及 Flash。

提前拿到<object\>标签的主要原因是为了在父元素挂载后就开始对ActiveX插件进行初始化及后台登录等操作，如果等到ant-modal渲染以后再执行以上操作，就会出现明显的卡顿现象，十分影响用户体验。在`forceRender`没有实现之前，受开发进度影响，只能退而求其次的使用layUI组件库的对话框layer作为视频播放对话框的载体。

2019年1月6日，ant design更新`forceRender`属性以后，我马上对项目中的组件进行了替换，然而即使添加了`forceRender`属性，也无法拿到modal中的子元素。在提了一个[issue #14160](<https://github.com/ant-design/ant-design/issues/14160>)以后，发现是ant design维护者的疏忽，只更新了API文档却没在interface里面添加新属性。在解决上述问题以后，依然无法按照预期拿到DOM元素，官方为此提供了一个[demo](<https://codesandbox.io/s/xv68643y14>)，里面使用了`requestAnimationFrame`这个`Web API`，有关知识可以查看另一篇文章。

#### 总结

不管是简单还是复杂的问题，追本溯源或者仔细分解以后其实都是由各种小问题导致的。在不了解具体实现时，我会认为`forceRender`的原理必定是很复杂，需要多方面考虑的。在看了ts的源码以后会觉得大佬们确实很厉害，只是利用CSS的display属性就实现页面优化或者API的设计，简单并且优雅。

学习代码是一种快乐，一步一步由浅到深具体地去了解造轮子的过程更是一种快乐。

<hr />

