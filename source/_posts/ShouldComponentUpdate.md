---
title: 使用ShouldComponentUpdate避免重复渲染
tags: [React生命周期]
date: 2019-07-08 09:58:08
permalink:
categories: React
description:
image:
---
<p class="description"></p>

#### 问题

在开发一个带有Echarts图表的页面时，每次切换图表类型，都会出现图表反复渲染的问题。通常情况下，页面多次渲染并不会影响用户体验（如果没有动画效果），而在这个页面下Echarts数据填充的动画反复进行多次就会让人明显的感觉到页面在多次刷新，降低用户体验。问题代码如下：

~~~jsx
  handleChangeDisplayType = e => {
    const { dispatch } = this.props;
    // 获取echarts图表数据
    dispatch({ type: 'securitySystem/fetchSpaceAlarmData' });
    // 受控组件Button.Group，需手动改变state中的值
    this.setState({ displayType: e.target.value }); 
  };
~~~

~~~jsx
  componentDidUpdate() {
    const { spaceAlarmData } = this.props;
    const el = document.getElementById('parking_space_alarm')
    echarts.dispose(el);
    this.parkingSpaceAlarmChart = echarts.init(el);
    // 渲染图表组件
    this.renderSpaceAlarmChart(spaceAlarmData, this.parkingSpaceAlarmChart);
  }
~~~

<!-- more -->

页面涉及到多个图表类型之间的切换，所以需要在每次切换时请求数据并手动保存目标图表类型。本以为多次渲染echarts图表的`dispose`和`init`同步渲染引起的，在后来仔细了解了React生命周期发现`state`和`props`的异步过程不会合并到一次页面渲染，会依次进行。

#### 页面渲染的触发时机

页面重新渲染，需要触发React生命周期中的Render函数，以下的React生命周期图可便于理解Render函数是如何被触发的：

![react_lifecycle](react_lifecycle.png)

这张图将React生命周期分成了三个阶段：生成期、存在期、销毁期。由图可以比较清楚的看出触发Render函数的内容可能包括：

* 生成期初次Render
* Props变更
* State变更

在React的渲染机制中，组件内的某一个props或state变化，会导致该组件内的所有子组件都重写render函数，尽管绝大多数子组件的props没有变化。例如在上面的问题中，受控组件Button.Group和Echarts图表作为页面的子组件，当页面内非自身状态更新时该子组件也会重新渲染。

![target](target.png)

例如在项目中，点击切换图表类型的主要目的是想设置页面上某一个按钮为选中状态，只需要更新页面上Button.Group中相应的DOM即可。事实上在更新Button组件状态时，还触发了图表组件的Render函数，当父组件中用于填充图表数据的props更新时，再次触发该组件内所有子组件的重绘，因此才会发生问题当中所描述的图表动画加载到一半页面刷新再次触发图表动画的过程。

#### 使用ShouldComponentUpdate避免重复触发Render函数

**shouldComponentUpdate(nextProps, nextState)**

当 props 或 state 发生变化时，`shouldComponentUpdate` 会在渲染执行之前被调用。返回值默认为 true。首次渲染(`ComponentDidMount`前的`Render`)或使用 `forceUpdate` 时不会调用该方法。

手动重写这个函数时，可以将 `this.props` 与 `nextProps` 以及 `this.state` 与`nextState` 进行比较，并返回`true`或 `false` 以告知 React 是否需要跳过更新。需要注意的是，**返回 `false` 并不会阻止子组件在 state 更改时重新渲染。**

> 这里的子组件是自身还是父组件下的所有子组件还不太清楚。
>
> [官方](<https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate>)的解释是：目前，如果 `shouldComponentUpdate` 返回 `false`，则不会调用 [`UNSAFE_componentWillUpdate`](https://zh-hans.reactjs.org/docs/react-component.html#unsafe_componentwillupdate)，[`render`](https://zh-hans.reactjs.org/docs/react-component.html#render) 和 [`componentDidUpdate`](https://zh-hans.reactjs.org/docs/react-component.html#componentdidupdate)。后续版本，React 可能会将 `shouldComponentUpdate` 视为提示而不是严格的指令，并且，当返回 `false` 时，仍可能导致组件重新渲染。
>
> 官方建议此方法仅作为性能优化的方式而存在。不能依靠此方法来“阻止”渲染，因为这可能会产生 bug，应该考虑使用内置的 PureComponent 组件，而不是手动编写 `shouldComponentUpdate`。`PureComponent` 会对 props 和 state 进行浅层比较，并减少了跳过必要更新的可能性。

在项目中遇到的问题暂时可以使用如下方式解决：

~~~jsx
  shouldComponentUpdate(nextProps, nextState) {
    const { displayType } = this.state;
    // 改变state的时候不更新图表组件，改变props的时候更新
    if (displayType === nextState.displayType) {
      return true;
    }
    return false;
  }
~~~

**React.PureComponent**

`React.PureComponent` 与 `React.Component` 几乎完全相同，但 `React.PureComponent` 通过`props`和`state`的**浅对比**来实现 `shouldComponentUpate`。如果对象包含复杂的数据结构，它可能会因深层的数据不一致而产生错误的否定判断(表现为对象深层的数据已改变视图却没有更新）。

> * 无论组件是否是 `PureComponent`，如果定义了` shouldComponentUpdate`，那么会调用它并以它的执行结果来判断是否 update。只有在组件未定义 `shouldComponentUpdate` 的情况下，才会判断该组件是否是 `PureComponent`。如果是的话，则会对新旧 props、state 进行 [shallowEqual](<https://github.com/facebook/fbjs/blob/c69904a511b900266935168223063dd8772dfc40/packages/fbjs/src/core/shallowEqual.js#L39>) 比较。
> * 浅对比，只会比较到两个对象的 hasOwnProperty 是否符合`Object.keys()`判等，不会递归地去深层比较。

在项目中，由于图表数据使用嵌套数组的数据结构，无法简单的使用`PureComponent`来进行优化，如果需要兼顾复杂的数据结构以及较好的用户体验，可以使用第三方库如immutable.js来进行优化，此处不做涉及。

#### 总结

使用`ShouldComponent`作为页面性能的一种方式，能够有效的避免页面重复渲染的问题，从而提升用户体验。作为一个不常用的React生命周期钩子，在之前的项目开发中没能很好的理解并运用。希望能通过写文章的方式记录自己学习和了解其原理的过程，加深对于React相关知识的理解。

<hr />
