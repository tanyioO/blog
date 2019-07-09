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

在我所使用的一些"历史比较悠久"的UI组件库如bootstrap，layUI中，对于modal的处理是将其作为一个绝对定位元素渲染到DOM中，设置它的CSS display属性用于控制其显示和隐藏。这种方式在现在的”数据驱动“式开发中存在很多问题，例如：

* 首屏加载问题：数据量较大或者页面中的modal个数比较多时，可能会导致页面加载缓慢

* 数据“残留”：当modal中存在表单时，由于modal不销毁，再次打开modal时，表单中的数据为上一次修改的数据，需要手动清除或重置表单

* 

  未写完...

<hr />

