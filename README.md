# react-lifecycle
this is a rep introduce react-lifeclcle

## 本文着重介绍react的生命周期 覆盖至最新版本

### react组件的生命周期总共可以用三个阶段来描述

### 1.初始化阶段

#### 组件被创建的时候，可以拥有自己的state和defaultProps,这个阶段通常在constructor来进行初始化。(暂不做介绍)

#### 执行流程 getDerivedStateFromProps -> componentWillMount(UN_SAFE) -> render -> componentDidMount

#### 这是初始化阶段执行的生命周期，我们先探讨个问题 何为初始化，有以下几个问题，先思考。

- 组件创建的时候，可以有自己的state和defaultProps 那么组件接受父组件传递的props是否在初始化就渲染？
- 在初始化阶段 在componentWillMount中进行setState操作，我们都知道setState不会立刻执行，那么初始化渲染会包含此次状态改变吗？

#### getDerivedStateFromProps

-  object (nextProps, prevState)
-  该组件的生命周期是在react v16.3加入的 是一个静态方法
-  取代componentWillMount 与 componentWillReceiveProps生命周期 无法获得实例的this
-  该方法的主要目的在于 使当前的state能够响应传入的props(即根据传入的props来改变组件的 状态)
-  该方法接受两个参数 nextProps与prevState 返回值为object
-  触发时机： 传入的props改变或者 普通的setState触发（初始化渲染的时候，为defaultProps 到 父组件的props ）所以初始化渲染会执行这个方法。
-  返回一个对象，作用相当于setState，不同于setState的是 setState会触发getDerivedStateFromProps这个方法，而该方法返回的不会触发(直接触发 shouldComponentUpdate 注：初始化渲染不触发) 其他都和setState的作用一样，这就是为什么在该方法中进行改变状态不会触发死循环的原因。如果不需要改变状态 返回null
-  该生命周期在整个组件的生命周期内只会执行多次

#### componentWillMount
- void
- 属于不安全的生命周期 react v16.3 被废除
- 因为react 将推出的async-render 中 凡是在Vdom形成之前的生命周期都有可能会渲染多次 导致不安全的因素出现如：可能重复调用ajax请求， react为了避免这些情况的发生，直接废弃
- 在该生命周期内进行setState的时候，如果setState都是在主线程中，那么react会合并这些状态，在初始化渲染的时候一次性渲染，而避免无谓的多次渲染。
- 目前来看 该生命周期在整个组件的生命周期内只会执行一次
- 该方法不可以和getDerivedStateFromProps, react会给出警告，这是一个不安全的生命周期。

#### componentDidMount
- void
- 该生命周期是在组件初始化之后 render之后挂载到页面上执行，全局只会调用一次，属于安全当生命周期
- 此时 render已经生成了Vdom并且patch成了真实的dom挂载到页面上了。
- 网络请求当数据都应该在该生命周期调用而不应该在componentWillMount时调用，理由如下：
- 对于请求的数据 属于异步操作 而render是同步执行，也就是当数据请求到了，组件已经挂载到页面上了，会进行一次空渲染 这时候会出现一个短暂的loading期间，等待数据进行render。这就意味着在componentWillMount进行数据请求并setSetate意义不大，不如放置安全的生命周期中。

### 更新阶段

#### 这个阶段 组件处于活性状态。状态的改变以及父组件的rerender，props的改变都会造成组件的更新

#### 该阶段生命周期执行的流程（加入新的生命周期）

#### componentWillReceiveProps / getDerivedStateFromProps -> shouldComponentUpdate(return true) ->componentWillUpdate -> render -> getSnapshotBeforeUpdate -> componentDidUpdate

#### componentWillReceiveProps

-  object (nextProps, prevState)
- 初始化渲染的时候已经介绍过这个生命周期了。
- ·