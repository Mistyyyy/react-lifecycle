# react-lifecycle
this is a rep introduce react-lifeclcle

## 本文着重介绍react的生命周期 覆盖至最新版本

### react组件的生命周期总共可以用三个阶段来描述

### 1.初始化阶段

#### 组件被创建的时候，可以拥有自己的state和defaultProps,这个阶段通常在constructor来进行初始化。(暂不做介绍)

#### 执行流程 getDerivedStateFromProps -> componentWillMount(UN_SAFE) -> render -> componentDidMount

#### 这是初始化阶段执行的生命周期，我们先探讨个问题 何为初始化，有以下几个问题，先思考。

  1. 组件创建的时候，可以有自己的state和defaultProps 那么组件接受父组件传递的props是否在初始化就渲染？
  2. 在初始化阶段 在componentWillMount中进行setState操作，我们都知道setState不会立刻执行，那么初始化渲染会包含此次状态改变吗？


#### getDerivedStateFromProps

  1. 返回值 object 
  2. 参数(nextProps, prevState)
  3. 该组件的生命周期是在react v16.3加入的 是一个静态方法
  4. 取代componentWillMount 与 componentWillReceiveProps生命周期 无法获得实例的this
  5. 该方法的主要目的在于 使当前的state能够响应传入的props(即根据传入的props来改变组件的 状态)
  6. 返回一个对象，作用相当于setState，不同于setState的是 setState会触发getDerivedStateFromProps这个方法，而该方法返回的不会触发(直接触发 shouldComponentUpdate 注：初始化渲染不触发) 其他都和setState的作用一样，这就是为什么在该方法中进行改变状态不会触发死循环的原因。如果不需要改变状态 返回null
  7. 该生命周期在整个组件的生命周期内会执行多次


#### componentWillMount
  1. 返回值 void
  2. 属于不安全的生命周期 react v16.3 被废除
  3. 因为react 将推出的async-render 中 凡是在Vdom形成之前的生命周期都有可能会渲染多次 导致不安全的因素出现如：可能重复调用ajax请求， react为了避免这些情况的发生，直接废弃
  4. 在该生命周期内进行setState的时候，如果setState都是在主线程中，那么react会合并这些状态，在初始化渲染的时候一次性渲染，而避免无谓的多次渲染。
  5. 目前来看 该生命周期在整个组件的生命周期内只会执行一次
  6. 该方法不可以和getDerivedStateFromProps, react会给出警告，这是一个不安全的生命周期。


#### componentDidMount
  1. 返回值 void
  2. 该生命周期是在组件初始化之后 render之后挂载到页面上执行，全局只会调用一次，属于安全的生命周期
  3. 此时 render已经生成了Vdom并且patch成了真实的dom挂载到页面上。
  4. 网络请求的数据都应该在该生命周期调用而不应该在componentWillMount时调用，理由如下：
  5. 对于请求的数据 属于异步操作 而render是同步执行，也就是当数据请求到了，组件已经挂载到页面上了，会进行一次空渲染 这时候会出现一个短暂的loading期间，等待数据进行render。这就意味着在componentWillMount进行数据请求并setSetate意义不大，不如放置安全的生命周期中。


### 更新阶段

#### 这个阶段 组件处于活性状态。状态的改变以及父组件的rerender，props的改变都会造成组件的更新

#### 该阶段生命周期执行的流程（加入新的生命周期）

#### componentWillReceiveProps / getDerivedStateFromProps -> shouldComponentUpdate(return true) ->componentWillUpdate -> render -> getSnapshotBeforeUpdate -> componentDidUpdate

#### componentWillReceiveProps

  1. 返回值 object  参数(nextProps, prevState)
  2. 初始化渲染的时候已经介绍过这个生命周期了。这里不在做具体介绍
  3. 在该生命周期内 返回除了null以外的对象都相当于 执行了setState方法 但是跟普通的setState区别在于 普通的setState即状态改变 在触发渲染之前会进入该生命周期。而该方法返回的不会再次触发该生命周期。所以不会造成死循环

#### shouldComponentUpdate

  1. 返回值 boolean 参数（nextProps, nextState）
  2. 仅在render之前（更细一步是在componentWillUpdate之前）进行调用 可以细粒度来控制是否进行组件的重渲染
  3. 可以利用this.props 和 this.state获取 oldprops 和 oldState 进行更深层次的比较
  4. 该方法默认是返回true 也就是重渲染
  5. 该方法最大的用处就是 当无相关的props或者state改变时，可以人为的控制组件是否渲染 利用好可以极大的提升性能 避免无谓的渲染。最典型的就是父组件rerender的时候，无关属性修改时，可以利用该生命周期避免无所谓的渲染。
  6. 应该避免在该方法内进行 setState 可能会陷入死循环 实在有必要 请一定要进行有条件的setState
  7. 为什么是可能？ 理由如下 shouleComponentUpdate => setState => shouldComponent => setState => ....

#### componentWillUpdate

  1. 返回值 void 参数 （nextProps, nextState）
  2. 该生命周期被废弃了 在react v16.3 版本中 
  3. 该方法在render之前 shouldComponentUpdate 之后进行调用
  4. 在该方法中 尽量避免setState 因为可能陷入死循环 如果有必要 请务必进行有条件的setState
  5. 理由 componentWillUpdate => setState => shouldComponentUpdate => componentWillUpdate => setState => ....
  6. 首次渲染不会调用

#### render

  1. 返回值 Vdom 
  2. 在该方法中 更新后的state和props会被获取到

#### getSnapshotBeforeUpdate

  1. 返回值any 参数(prevProps, prevState)
  2. 该生命周期新增在react v16.3 版本中 
  3. 可以获取到dom信息,返回值作为下一个生命周期componentDidUpdate的第三个参数传入。
  4. 在该方法中 尽量避免setState 因为可能陷入死循环 如果有必要 请务必进行有条件的setState
  5. 理由 getSnapshotBeforeUpdate => setState => shouldComponentUpdate => componentWillUpdate => render => getSnapshotBeforeUpdate => setState => ....
  6. 首次渲染不会调用

#### componentDidUpdate

  1. 返回值void 参数(prevProps, prevState, snapshot)
  2. 该生命周期 getSnapshotBeforeUpdate 之后调用，组件已经重新渲染了并且vdom已经经过patch后渲染在页面上了
  3. 此时 可以用this.props 和 this.state 获取到最新的 props 和 state了
  4. 在该生命周期中 可以操作实际 的dom 因为已经渲染到页面上了
  5. 在该方法中 尽量避免setState 因为可能陷入死循环 如果有必要 请务必进行有条件的setState
  6. 理由 componentDidUpdate => setState => shouldComponentUpdate => componentWillUpdate => render => componentDidUpdate => setState => ...
  7. 首次渲染不会调用


### 卸载阶段

#### componentUnMount

  1.在组件卸载的时候调用，通常在这个生命周期中释放内存,解绑事件。防止出错 内存泄漏