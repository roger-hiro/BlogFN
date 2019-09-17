## 前言
在本篇文章你将会学到：
* `IntersectionObserver API` 的用法，以及如何兼容。
* 如何在`React Hook`中实现无限滚动。
* 如何正确渲染多达10000个元素的列表。
![](https://user-gold-cdn.xitu.io/2019/9/16/16d3a490697f87cf?w=880&h=406&f=png&s=35969)
无限下拉加载技术使用户在大量成块的内容面前一直滚动查看。这种方法是在你向下滚动的时候不断加载新内容。

当你使用滚动作为发现数据的主要方法时，它可能使你的用户在网页上停留更长时间并提升用户参与度。随着社交媒体的流行，大量的数据被用户消费。无线滚动提供了一个高效的方法让用户浏览海量信息，而不必等待页面的预加载。

![](https://user-gold-cdn.xitu.io/2019/9/18/16d400aa8dbeb465?w=180&h=180&f=png&s=52846)

**如何构建一个体验良好的无限滚动**，是每个前端无论是项目或面试都会碰到的一个课题。


## 1. 早期的解决方案

关于无限滚动，早期的解决方案基本都是依赖监听scroll事件：
```
function fetchData() {
  fetch(path).then(res => doSomeThing(res.data));
}

window.addEventListener('scroll', fetchData);
```

然后计算各种`.scrollTop()`、`.offset().top`等等。

手写一个也是非常枯燥。而且：

* `scroll`事件会频繁触发，因此我们还需要手动节流。
* 滚动元素内有大量`DOM`，容易造成卡顿。

![](https://user-gold-cdn.xitu.io/2019/9/18/16d400b58be25b86?w=180&h=180&f=png&s=46124)
后来出现交叉观察者`IntersectionObserver API `
，在与`Vue`、`React`这类数据驱动视图的框架后，无限滚动的通用方案就出来了。

## 2. 交叉观察者：`IntersectionObserver`
```
const box = document.querySelector('.box');
const intersectionObserver = new IntersectionObserver((entries) => {
  entries.forEach((item) => {
    if (item.isIntersecting) {
      console.log('进入可视区域');
    }
  })
});
intersectionObserver.observe(box);
```
敲重点：
**IntersectionObserver API是异步的，不随着目标元素的滚动同步触发，性能消耗极低。**

### 2.1 `IntersectionObserverEntry`对象

![](https://user-gold-cdn.xitu.io/2019/9/17/16d3f3a10f8a9452?w=800&h=240&f=png&s=412875)
这里我就粗略的介绍下需要用到的：
> [IntersectionObserve初试](https://anata.me/2019/02/28/IntersectionObserve%E5%88%9D%E8%AF%95/)

**`IntersectionObserverEntry`对象**

`callback`函数被调用时，会传给它一个数组，这个数组里的每个对象就是当前进入可视区域或者离开可视区域的对象(`IntersectionObserverEntry`对象)

这个对象有很多属性，其中最常用的属性是：


* `target`: 被观察的目标元素，是一个 DOM 节点对象
* `isIntersecting`: 是否进入可视区域
* `intersectionRatio`: 相交区域和目标元素的比例值，进入可视区域，值大于0，否则等于0

### 2.3 `options`
调用`IntersectionObserver`时，除了传一个回调函数，还可以传入一个`option`对象，配置如下属性：
* `threshold`: 决定了什么时候触发回调函数。它是一个数组，每个成员都是一个门槛值，默认为[0]，即交叉比例（intersectionRatio）达到0时触发回调函数。用户可以自定义这个数组。比如，[0, 0.25, 0.5, 0.75, 1]就表示当目标元素 0%、25%、50%、75%、100% 可见时，会触发回调函数。
* `root`: 用于观察的根元素，默认是浏览器的视口，也可以指定具体元素，指定元素的时候用于观察的元素必须是指定元素的子元素
* `rootMargin`: 用来扩大或者缩小视窗的的大小，使用css的定义方法，10px 10px 30px 20px表示top、right、bottom 和 left的值

```
const io = new IntersectionObserver((entries) => {
  console.log(entries);
}, {
  threshold: [0, 0.5],
  root: document.querySelector('.container'),
  rootMargin: "10px 10px 30px 20px",
});
```

### 2.4 `observer`
```
observer.observer（nodeone）; //仅观察nodeOne 
observer.observer（nodeTwo）; //观察nodeOne和nodeTwo 
observer.unobserve（nodeOne）; //停止观察nodeOne
observer.disconnect（）; //没有观察任何节点
```

## 3. 如何在`React Hook`中使用`IntersectionObserver`

在看`Hooks`版之前，来看正常组件版的：
```
class SlidingWindowScroll extends React.Component {
this.$bottomElement = React.createRef();
...
componentDidMount() {
    this.intiateScrollObserver();
}
intiateScrollObserver = () => {
    const options = {
      root: null,
      rootMargin: '0px',
      threshold: 0.1
    };
    this.observer = new IntersectionObserver(this.callback, options);
    this.observer.observe(this.$bottomElement.current);
}
render() {
    return (
    <li className='img' ref={this.$bottomElement}>
    )
}
```
众所周知，`React 16.x`后推出了`useRef`来替代原有的`createRef`，用于追踪DOM节点。那让我们开始吧：

## 4. 原理

实现一个组件，可以显示具有15个元素的固定窗口大小的n个项目的列表：
即在任何时候，无限滚动n元素上也仅存在15个`DOM`节点。

![](https://user-gold-cdn.xitu.io/2019/9/17/16d3f4930c6793a3?w=600&h=288&f=gif&s=1174721)

* 采用`relative/absolute` 定位来确定滚动位置
* 追踪两个`ref`: `top/bottom`来决定向上/向下滚动的渲染与否
* 切割数据列表，保留最多15个DOM元素。

## 5. `useState`声明状态变量

我们开始编写组件`SlidingWindowScrollHook`:

```
const THRESHOLD = 15;
const SlidingWindowScrollHook = (props) =>  {
  const [start, setStart] = useState(0);
  const [end, setEnd] = useState(THRESHOLD);
  const [observer, setObserver] = useState(null);
  // 其它代码...
}
```
### 1. `useState的简单理解:`
```
const [属性, 操作属性的方法] = useState(默认值);
```
### 2. 变量解析

* `start`：当前渲染的列表第一个数据，默认为0
* `end`: 当前渲染的列表最后一个数据，默认为15
* `observer`: 当前观察的视图`ref`元素


## 6. `useRef`定义追踪的`DOM`元素
```
const $bottomElement = useRef();
const $topElement = useRef();
```
正常的无限向下滚动只需关注一个dom元素，但由于我们是固定15个`dom`元素渲染，需要判断向上或向下滚动。

## 7. 内部操作方法和和对应`useEffect`
**请配合注释食用：**
```
useEffect(() => {
    // 定义观察
    intiateScrollObserver();
    return () => {
      // 放弃观察
      resetObservation()
  }
},[start, end]) //仅在 [start, end] 更改时触发

// 定义观察
const intiateScrollObserver = () => {
    const options = {
      root: null,
      rootMargin: '0px',
      threshold: 0.1
    };
    const Observer = new IntersectionObserver(callback, options)
    // 分别观察开头和结尾的元素
    if ($topElement.current) {
      Observer.observe($topElement.current);
    }
    if ($bottomElement.current) {
      Observer.observe($bottomElement.current);
    }
    // 设初始值
    setObserver(Observer)    
}

// 交叉观察的具体回调，观察每个节点，并对实时头尾元素索引处理
const callback = (entries, observer) => {
    entries.forEach((entry, index) => {
      const listLength = props.list.length;
      // 向下滚动，刷新数据
      if (entry.isIntersecting && entry.target.id === "bottom") {
        const maxStartIndex = listLength - 1 - THRESHOLD;     // 当前头部的索引
        const maxEndIndex = listLength - 1;                   // 当前尾部的索引
        const newEnd = (end + 10) <= maxEndIndex ? end + 10 : maxEndIndex; // 下一轮增加尾部
        const newStart = (end - 5) <= maxStartIndex ? end - 5 : maxStartIndex; // 在上一轮的基础上计算头部
        setStart(newStart)
        setEnd(newEnd)
      }
      // 向上滚动，刷新数据
      if (entry.isIntersecting && entry.target.id === "top") {
        const newEnd = end === THRESHOLD ? THRESHOLD : (end - 10 > THRESHOLD ? end - 10 : THRESHOLD); // 向上滚动尾部元素索引不得小于15
        let newStart = start === 0 ? 0 : (start - 10 > 0 ? start - 10 : 0); // 头部元素索引最小值为0
        setStart(newStart)
        setEnd(newEnd)
        }
    });
}

// 停止滚动时放弃观察
const resetObservation = () => {
    observer && observer.unobserve($bottomElement.current); 
    observer && observer.unobserve($topElement.current);
}

// 渲染时，头尾ref处理
const getReference = (index, isLastIndex) => {
    if (index === 0)
      return $topElement;
    if (isLastIndex) 
      return $bottomElement;
    return null;
}
```

## 8. 渲染界面
```

  const {list, height} = props; // 数据，节点高度
  const updatedList = list.slice(start, end); // 数据切割
  
  const lastIndex = updatedList.length - 1;
  return (
    <ul style={{position: 'relative'}}>
      {updatedList.map((item, index) => {
        const top = (height * (index + start)) + 'px'; // 基于相对 & 绝对定位 计算
        const refVal = getReference(index, index === lastIndex); // map循环中赋予头尾ref
        const id = index === 0 ? 'top' : (index === lastIndex ? 'bottom' : ''); // 绑ID
        return (<li className="li-card" key={item.key} style={{top}} ref={refVal} id={id}>{item.value}</li>);
      })}
    </ul>
  );
```

## 9. 如何使用
`App.js`:
```
import React from 'react';
import './App.css';
import { SlidingWindowScrollHook } from "./SlidingWindowScrollHook";
import MY_ENDLESS_LIST from './Constants';

function App() {
  return (
    <div className="App">
     <h1>15个元素实现无限滚动</h1>
      <SlidingWindowScrollHook list={MY_ENDLESS_LIST} height={195}/>
    </div>
  );
}

export default App;
```
定义一下数据 `Constants.js`:
```
const MY_ENDLESS_LIST = [
  {
    key: 1,
    value: 'A'
  },
  {
    key: 2,
    value: 'B'
  },
  {
    key: 3,
    value: 'C'
  },
  // 中间就不贴了...
  {
    key: 45,
    value: 'AS'
  }
]
```
`SlidingWindowScrollHook.js`:
```
import React, { useState, useEffect, useRef } from "react";
const THRESHOLD = 15;

const SlidingWindowScrollHook = (props) =>  {
  const [start, setStart] = useState(0);
  const [end, setEnd] = useState(THRESHOLD);
  const [observer, setObserver] = useState(null);
  const $bottomElement = useRef();
  const $topElement = useRef();

  useEffect(() => {
    intiateScrollObserver();
    return () => {
      resetObservation()
  }
  // eslint-disable-next-line react-hooks/exhaustive-deps
  },[start, end])

  const intiateScrollObserver = () => {
    const options = {
      root: null,
      rootMargin: '0px',
      threshold: 0.1
    };
    const Observer = new IntersectionObserver(callback, options)
    if ($topElement.current) {
      Observer.observe($topElement.current);
    }
    if ($bottomElement.current) {
      Observer.observe($bottomElement.current);
    }
    setObserver(Observer)    
  }

  const callback = (entries, observer) => {
    entries.forEach((entry, index) => {
      const listLength = props.list.length;
      // Scroll Down
      if (entry.isIntersecting && entry.target.id === "bottom") {
        const maxStartIndex = listLength - 1 - THRESHOLD;     // Maximum index value `start` can take
        const maxEndIndex = listLength - 1;                   // Maximum index value `end` can take
        const newEnd = (end + 10) <= maxEndIndex ? end + 10 : maxEndIndex;
        const newStart = (end - 5) <= maxStartIndex ? end - 5 : maxStartIndex;
        setStart(newStart)
        setEnd(newEnd)
      }
      // Scroll up
      if (entry.isIntersecting && entry.target.id === "top") {
        const newEnd = end === THRESHOLD ? THRESHOLD : (end - 10 > THRESHOLD ? end - 10 : THRESHOLD);
        let newStart = start === 0 ? 0 : (start - 10 > 0 ? start - 10 : 0);
        setStart(newStart)
        setEnd(newEnd)
      }
      
    });
  }
  const resetObservation = () => {
    observer && observer.unobserve($bottomElement.current);
    observer && observer.unobserve($topElement.current);
  }


  const getReference = (index, isLastIndex) => {
    if (index === 0)
      return $topElement;
    if (isLastIndex) 
      return $bottomElement;
    return null;
  }

  const {list, height} = props;
  const updatedList = list.slice(start, end);
  const lastIndex = updatedList.length - 1;
  
  return (
    <ul style={{position: 'relative'}}>
      {updatedList.map((item, index) => {
        const top = (height * (index + start)) + 'px';
        const refVal = getReference(index, index === lastIndex);
        const id = index === 0 ? 'top' : (index === lastIndex ? 'bottom' : '');
        return (<li className="li-card" key={item.key} style={{top}} ref={refVal} id={id}>{item.value}</li>);
      })}
    </ul>
  );
}
export { SlidingWindowScrollHook };
```
以及少许样式：
```
.li-card {
  list-style: none;
  box-shadow: 2px 2px 9px 0px #bbb;
  padding: 70px 10px;
  margin-bottom: 20px;
  border-radius: 10px;
  position: absolute;
  left: 30px;
  width: 80%;
}
```
然后你就可以慢慢耍了。。。

![](https://user-gold-cdn.xitu.io/2019/9/18/16d4002ce94f29ad?w=1011&h=607&f=gif&s=1560233)

## 10. 兼容性处理
`IntersectionObserver`不兼容`Safari`?

莫慌，我们有`polyfill`版

![](https://user-gold-cdn.xitu.io/2019/9/18/16d4010e224d28f2?w=2714&h=1076&f=png&s=230326)
每周34万下载量呢，放心用
![](https://user-gold-cdn.xitu.io/2019/9/18/16d4006a9234af71?w=240&h=240&f=png&s=29821)


## ❤️ 看完三件事
如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章
* [那些你不经意间使用的设计模式(一) - 创建型模式](https://juejin.im/post/5d35d8c4518825360f16198e)
* [「数据可视化库王者」D3.js 极速上手到Vue应用
](https://juejin.im/post/5d1e074af265da1bca51f8ec)
* [「真®全栈之路」Web前端开发的后端指南](https://juejin.im/post/5cc02aacf265da039e1ff3fa)
* [「Vue实践」5分钟撸一个Vue CLI 插件](https://juejin.im/post/5cb59c4bf265da03a743e979)
* [「Vue实践」武装你的前端项目](https://juejin.im/post/5cab64ce5188251b19486041)
* [「中高级前端面试」JavaScript手写代码无敌秘籍](https://juejin.im/post/5c9c3989e51d454e3a3902b6)
* [「从源码中学习」面试官都不知道的Vue题目答案](https://juejin.im/post/5c959f74f265da610c068fa8)
* [「从源码中学习」Vue源码中的JS骚操作](https://juejin.im/post/5c73554cf265da2de33f2a32)
* [「Vue实践」项目升级vue-cli3的正确姿势](https://juejin.im/post/5c4a83e36fb9a049b13e91ba)
* [为何你始终理解不了JavaScript作用域链？](https://juejin.im/editor/posts/5c8efeb1e51d45614372addd)

![](https://user-gold-cdn.xitu.io/2019/8/5/16c5faffbefaea2e?w=2006&h=1014&f=png&s=672314)