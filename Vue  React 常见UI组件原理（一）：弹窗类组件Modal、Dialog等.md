## 前言

在某个月黑风高的晚上...没剧刷的我无意想起以前处理的一些弹窗的坑。

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zi0p0jrj30dd0b4qah.jpg)
然后又无意间刷到“`Portal`”，才知道`Modal`的实现还有如此妙的方式，顺而想着干脆把`UI`组件库的实现原理看完。

本文将讲述以下三种UI组件的实现原理：
1. `Modal`弹窗类。
2. `Steps`步骤条。
3. `Transfer`穿梭框。


## 1. `Modal`弹窗的基本原理

我给弹窗类的定义是脱离固定的层级关系，不再受制于层叠上下文的组件。

常见的`Modal`模态框、`Dialog`对话框、`Notification`通知框等都是最最常用的交互方式。

![img](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zlzqfkcj30zk0k0tbw.jpg)
在我们页面有时需要一些特定的弹窗时，通过改`UI`组件过于麻烦。

这时切图仔级别的会想：简单啊，创建一个`<div/>` 给绝对定位不就得了。

倘若只是当前路由页用，也还凑合。**可一旦涉及到了组件复用以及抽象为声明式，就会有很大的隐患**：
1. 若无封装，组件代码需要处处粘贴。
2. 即使封装了，都是在每个路由页下创建`<div/>`，易造成样式污染。
3. 类购物车的弹窗，又该如何处理数据及渲染？
4. 再进一步想，万一组件库会作为绩效考核，拿到每个环境都长得不一样，咋整？


![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zi4v9hlj305i03qjrm.jpg)

### 1.1 `Jquery`时代的弹窗实现

初初入行时，去各种资源站，找`Jquery`的`UI`组件，想必三四年经验的前端们都曾乐此不疲。

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zi82k16j307n053t8i.jpg)
这个时代（也就三四年前）的弹窗，因为没有`React`/`Vue`根节点的概念，普遍都是：

1. **直接操作真实 dom，使用熟知的dom 操作方法将指令所在的元素 append 到另外一个 dom 节点上去。** 如：` document.body.appendChild`。
2. 再通过`overflow: hidden`或`display:none`（或调整`z-index`）来隐藏。

这种操作真实`dom`的代价，在大型项目中不停触发重绘/回流，是很糟糕的，且内部数据/样式不易更改。像以下这种情况就容易出现：
1. 原本图片固定在区域内。
![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zib3vhej30ka05r472.jpg)
2. 小弹窗展示后，溢出了。
![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zid2ursj30ka08sds6.jpg)
随着`React / Vue`先进库的发展，也陆续有了多种方案选择。。。

### 1.2 `React / Vue`早期实现。
其实`React / Vue`早期的实现和`Jquery`时代的并无二异：**依赖于父节点数据，在当前组件内挂载弹窗。**

`Vue`的情况稍好，有自定义指令这条路走。

> 以下引自：[《Vue中的Portal技术》](https://www.dazhuanlan.com/2019/10/05/5d9817395c1a8/)

以`vue-dom-portal`为例，代码非常简单无非就是将当前的 `dom` 移动到指定地方:

```
function (node = document.body) {
  if (node === true) return document.body;
  return node instanceof window.Node ? node : document.querySelector(node);
}

const homes = new Map();

const directive = {
  inserted(el, { value }, vnode) {
    const { parentNode } = el;
    const home = document.createComment("");
    let hasMovedOut = false;

    if (value !== false) {
      parentNode.replaceChild(home, el); // moving out, el is no longer in the document
      getTarget(value).appendChild(el); // moving into new place
      hasMovedOut = true;
    }

    if (!homes.has(el)) homes.set(el, { parentNode, home, hasMovedOut }); // remember where home is or should be
  },
  componentUpdated(el, { value }) {
    // 对比子组件更新
    const { parentNode, home, hasMovedOut } = homes.get(el); // recall where home is

    if (!hasMovedOut && value) {
      parentNode.replaceChild(home, el);
      getTarget(value).appendChild(el);

      homes.set(el, Object.assign({}, homes.get(el), { hasMovedOut: true }));
    } else if (hasMovedOut && value === false) {
      parentNode.replaceChild(el, home);
      homes.set(el, Object.assign({}, homes.get(el), { hasMovedOut: false }));
    } else if (value) {
      getTarget(value).appendChild(el);
    }
  },
  unbind(el, binding) {
    homes.delete(el);
  }
};

function plugin(Vue, { name = "dom-portal" } = {}) {
  Vue.directive(name, directive);
}

plugin.version = "0.1.6";

export default plugin;

if (typeof window !== "undefined" && window.Vue) {
  window.Vue.use(plugin);
}
```

可以看到在 `inserted` 的时候就拿到实例的 el（真实 dom），然后进行替换操作，在 `componentUpdated` 的时候再次根据指令的值去操作 dom。

为了能够在不同声明周期函数中使用缓存的一些数据，这里在 `inserted` 的时候就把当前节点的父节点和替换成的 `dom` 节点（一个注释节点）,以及节点是否移出去的状态都记录在外部的一个 `map` 中，这样可以在其他的声明周期函数中使用，可以避免重复计算。

但是`React / Vue`的实现都有类似的通病：
1. 生命周期的执行会很混乱。
2. 需要通过`redux`或`props`管理数据，可这对于一个`UI`组件来说过于臃肿了。

`React`官方也意识到构建脱离于父组件的组件挺麻烦的，于是在`v16`版本推了一个叫“`Portal` ”的功能。而`Vue3`也是借鉴并吸纳了优秀插件，将`Portal`作为内置组件了。


### 1.3 传送门`Portal`方案

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zig8cxtj30zk0k01ky.jpg)
`React / Vue`的第二套方案都是基于操作虚拟`dom`：

**定义一套组件，将组件内的 `vnode/ReactDOM` 转移到另外一个组件中去，然后各自渲染。**

## 2. `React`的`Portal`

`React Portal`之所以叫`Portal`，因为做的就是和“传送门”一样的事情：`render`到一个组件里面去，实际改变的是网页上另一处的`DOM`结构。
```
ReactDOM.createPortal(child, container)
```
1. 第一个参数（`child`）是任何可渲染的 `React` 子元素，例如一个元素，字符串或碎片。
2. 第二个参数（`container`）则是一个 `DOM` 元素。


在`v16`中，使用`Portal`创建`Dialog`组件简单多了，不需要牵扯到`componentDidMount`、`componentDidUpdate`，也不用调用`API`清理`Portal`，关键代码在render中，像下面这样就行:

```
import React from 'react';
import {createPortal} from 'react-dom';

class Dialog extends React.Component {
  constructor() {
    super(...arguments);

    const doc = window.document;
    this.node = doc.createElement('div');
    doc.body.appendChild(this.node);
  }

  render() {
    return createPortal(
      <div class="dialog">
        {this.props.children}
      </div>, //塞进传送门的JSX
      this.node //传送门的另一端DOM node
    );
  }

  componentWillUnmount() {
    window.document.body.removeChild(this.node);
  }
```
当然，我们作为一个`React Hooks`选手，不骚一下咋行。


### 2.1 热门组件库`Ant Design`中的实现

原本是想从`Ant Design`库中一窥究竟，却发现事情并不简单。。

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zijfj5dj308b03idhw.jpg)

前后寻址了三个库/地方，才发现实现的关键：

1. `import Dialog from 'rc-dialog';`
2.  `import Portal from 'rc-util/lib/PortalWrapper';`
3.  `import Portal from './Portal';`

具体实现也算如我所料：
```
import React from 'react';
import ReactDOM from 'react-dom';
import PropTypes from 'prop-types';

export default class Portal extends React.Component {
  static propTypes = {
    getContainer: PropTypes.func.isRequired,
    children: PropTypes.node.isRequired,
    didUpdate: PropTypes.func,
  }

  componentDidMount() {
    this.createContainer();
  }

  componentDidUpdate(prevProps) {
    const { didUpdate } = this.props;
    if (didUpdate) {
      didUpdate(prevProps);
    }
  }

  componentWillUnmount() {
    this.removeContainer();
  }

  createContainer() {
    this._container = this.props.getContainer();
    this.forceUpdate();
  }

  removeContainer() {
    if (this._container) {
      this._container.parentNode.removeChild(this._container);
    }
  }

  render() {
    if (this._container) {
      return ReactDOM.createPortal(this.props.children, this._container);
    }
    return null;
  }
}
```

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zim55nnj302201yjra.jpg)
`render`里用了`ReactDOM.createPortal`

**这也是为什么多数`Modal`组件不会提供篡改整体样式的`API`,只能通过全局重置样式。`

### 2.1 `React Hooks`版弹窗：`useModal`

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zio0pi7j30c80b1q8t.jpg)

#### 步骤一：创建一个`Modal`组件
```
import React from 'react'
import ReactDOM from 'react-dom'

type Props = {
  children: React.ReactChild
  closeModal: () => void
}

const Modal = React.memo(({ children, closeModal }: Props) => {
  const domEl = document.getElementById('modal-root')

  if (!domEl) return null
  return ReactDOM.createPortal(
    <div>
      <button onClick={closeModal}>Close</button>
      {children}
    </div>,
    domEl
  )
})

export default Modal
```

#### 步骤二：自定义`useModal`
```
import React, { useState } from 'react'

import Modal from './Modal'

// Modal组件最基础的两个事件，show/hide
export const useModal = () => {
  const [isVisible, setIsVisible] = useState(false)
  
  const show = () => setIsVisible(true)
  const hide = () => setIsVisible(false)
  
  const RenderModal = ({ children }: { children: React.ReactChild }) => (
    <React.Fragment>
      {isVisible && <Modal closeModal={hide}>{children}</Modal>}
    </React.Fragment>
  )

  return {
    show,
    hide,
    RenderModal,
  }
}
```

很好理解，不懂的建议转行写`Vue`。

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zirgy5mj30c80cugvn.jpg)

#### 步骤三：使用它
```
import React from 'react'

import { useModal } from './useModal'

const App = React.memo(() => {
  const { show, hide, RenderModal } = useModal()
  return (
    <div>
      <div>
        <p>some content...</p>
        <button onClick={show}>打开</button>
        <button onClick={hide}>关闭</button>
        <RenderModal>
          <p>这里面的内容将会被渲染到'modal-root'容器里.</p>
        </RenderModal>
      </div>
      <div id='modal-root' />
    </div>
  )
})

export default App
```

## 3. `Vue 3`的`Portal`

`Vue`虽说是借鉴，但使用方式可容易多了。
```
<OtherComponent>
  <Portal target="#popup-target">
    <Modal />
  </Portal>
</OtherComponent>
....
<div id="popup-target"></div>
```

在上面的示例中，该` <Modal />`组件将在`id=portal-target`的容器中渲染，即使它位于`OtherComponent`组件内。

这，这...这也太香了吧。
进一步的用法如下：
```
<!-- UserCard.vue -->
<template>
  <div class="user-card">
    <b> {{ user.name }} </b>  
    <button @click="isPopUpOpen = true">Remove user</button>
    <Portal target="#popup-target">
      <div v-show="isPopUpOpen">
        <p>Are you sure?</p>
        <button @click="removeUser">Yes</button>
        <button @click="isPopUpOpen = false">No</button>
      </div>
    </Portal>
  </div>
</template>
```

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zivir09j301j01c0sk.jpg)

然后我再去找了下`Vue 3`的源码实现：
在`packages/runtime-core/src/components/Portal.ts`目录中：
```
import { ComponentInternalInstance } from '../component'
import { SuspenseBoundary } from './Suspense'
import { RendererInternals, MoveType } from '../renderer'
import { VNode, VNodeArrayChildren, VNodeProps } from '../vnode'
import { isString, ShapeFlags, PatchFlags } from '@vue/shared'
import { warn } from '../warning'

export const isPortal = (type: any): boolean => type.__isPortal

export interface PortalProps {
  target: string | object
}

export const PortalImpl = {
  __isPortal: true,
  process(
    n1: VNode | null,
    n2: VNode,
    container: object,
    anchor: object | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    optimized: boolean,
    {
      mc: mountChildren,
      pc: patchChildren,
      pbc: patchBlockChildren,
      m: move,
      o: { insert, querySelector, setElementText, createComment }
    }: RendererInternals
  ) {
    const targetSelector = n2.props && n2.props.target
    const { patchFlag, shapeFlag, children } = n2
     if (n1 == null) {
      // insert an empty node as the placeholder for the portal
      insert((n2.el = createComment(`portal`)), container, anchor)
      if (__DEV__ && isString(targetSelector) && !querySelector) {
        warn(
          `Current renderer does not support string target for Portals. ` +
            `(missing querySelector renderer option)`
        )
      }
    } else {
    //....中间忽略了，大致的意思就是对比两个VNode,以及在不同生命周期的边界处理
    // 核心就是通过createComment，创建注释节点，将其插入不同节点中。
    // 最后setElementText，重置插入节点的内容。
    }
  }
}

// Force-casted public typing for h and TSX props inference
export const Portal = (PortalImpl as any) as {
  __isPortal: true
  new (): { $props: VNodeProps & PortalProps }
}

```
重要的解释，都在上述注释中了，临时看的，说得不对的谢谢指正。

其中：`createComment`是`Vue`对`DOM.createComment`的进一步封装。


## 结语&参考

这篇算是自己半夜无聊折腾出来的，原定计划是一篇写三种组件，但弹窗类的实现比较有意思。

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zj0smznj30u018ygpy.jpg)

这个系列我会看着写，不出意外下一篇就是讲`Steps`步骤条和`Transfer`穿梭框的实现（当然，太难了就忽悠一下，嘿嘿。）

**参考文章：**
> 1. [《Building a simple and reusable modal with React hooks and portals》](https://sandstorm.de/de/blog/post/reusable-modal-with-react-hooks-and-portals.html)
> 2. [《Vue中的Portal技术
> 》](https://www.dazhuanlan.com/2019/10/05/5d9817395c1a8/)
> 3. [《Portal – a new feature in Vue 3》](https://vueschool.io/articles/vuejs-tutorials/portal-a-new-feature-in-vue-3/)
> 4. [《React Portal的前世今生
> 》](https://juejin.im/post/5aa777e351882555867f15be)


## ❤️ 看完三件事
如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章

![](https://tva1.sinaimg.cn/large/00831rSTgy1gd3zj4pzvfj30sg0lcq6a.jpg)
劝退师个人微信：**huab119**

也可以来我的`GitHub`博客里拿所有文章的源文件：

**前端劝退指南**：https://github.com/roger-hiro/BlogFN
一起玩耍呀。~