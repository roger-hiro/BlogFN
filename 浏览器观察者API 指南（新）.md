## 前言

前段时间在研究前端异常监控/埋点平台的实现。

在思考方案时，想到了浏览器自带的观察者以及页面生命周期API 。

于是在翻查资料时意外发现，原来现代浏览器支持多达四种不同类型的观察者：

* `Intersection Observer`，交叉观察者。
* `Mutation Observer`，变动观察者。
* `Resize Observer`，视图观察者。
* `Performance Observer`，性能观察者
![](https://user-gold-cdn.xitu.io/2019/10/24/16df977e0b2680a0?w=300&h=300&f=png&s=154461)

|      | IntersectionObserver                                         | MutationObserver                                             | ResizeObserver                             | PerformanceObserver                                          |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------ | ------------------------------------------------------------ |
| 用途 | 观察一个元素是否在视窗可见                                   | 观察DOM中的变化                                              | 观察视口大小的变化                         | 监测性能度量事件                                             |
| 方法 | observe() <br>disconnect() <br> takeRecords()                | observe() <br>disconnect() <br>takeRecords() <br>unobserve() | observe() <br>disconnect() <br>unobserve() | observe() <br>disconnect() <br>takeRecords()                 |
| 取代 | Dom Mutation events                                          | getBoundingClientRect() 返回元素的大小及其相对于可视窗口的位置  <br> <br> Scroll 和 Resize 事件 | Resize 事件                                | Performance 接口                                             |
| 用途 | 1. 无限滚动 <br>2. 图片懒加载<br> 3. 兴趣埋点 <br> 4. 控制动画/视频执行（性能优化） | 1. 更高性能的数据绑定及响应<br> 2. 实现视觉差滚动  <br> 3. 图片预加载 <br> 4. 实现富文本编辑器 | 1. 更智能的响应式布局（取代@media） <br>2. 响应式组件       | 1. 更细颗粒的性能监控 <br>2. 分析性能对业务的影响（交互快/慢是否会影响销量） |
## 1. `IntersectionObserver`：交叉观察者

> `IntersectionObserver`接口，提供了一种异步观察目标元素与其祖先元素或顶级文档视窗(`viewport`)交叉状态的方法，祖先元素与视窗(`viewport)`被称为根(`root`)

### 1. 出现的意义

想要计算Web页面的元素的位置，非常依赖于`DOM`状态的显式查询。但这些查询是同步的，会导致昂贵的样式计算开销（重绘和回流），且不停轮询会导致大量的性能浪费。

![](https://user-gold-cdn.xitu.io/2019/10/22/16df34ffa206ff4f?w=710&h=379&f=png&s=15142)
于是便发展了以下的几种方案：

* 构建DOM和数据的自定义预加载和延迟加载。
* 实现了数据绑定的高性能滚动列表，该列表加载和呈现数据集的子集。
* 通过`scroll`等事件或通过插件的形式，计算真实元素可见性。

而它们都有几项共同特点：
1. 基本实现形式都是查询各个元素相对与某些元素（全局视口）的“被动查询”。
2. 信息可以异步传递（例如从另一个线程传递），且没有统一捕获错误的处理。
3. `web`平台支持匮乏，各有各家的处理。需要开发人员消耗大量精力兼容。

### 2. `IntersectionObserver`的优势

`Intersection Observer API`通过为开发人员提供一种新方法来异步查询元素相对于其他元素或全局视口的位置，从而解决了上述问题:


*  **异步处理**消除了昂贵的`DOM`和样式查询，连续轮询以及使用自定义插件的需求。
*  通过消除对这些方法的需求，可以使应用程序显着降低`CPU`，`GPU`和资源成本。


### 3. `IntersectionObserver`基本使用

使用`IntersectionObserver API`主要需要三个步骤：

1. 创建观察者
2. 定义回调事件
3. 定义要观察的目标对象

#### 1.**创建观察者**
```
const options = {
    root: document.querySelector('.scrollContainer'),
    rootMargin: '0px',
    threshold: [0.3, 0.5, 0.8, 1] }
    
const observer = new IntersectionObserver(handler, options)
```
这几个参数用大白话解释就是：

1.  `root`：指定一个根元素
2.  `rootMargin`：使用类似于设置CSS边距的语法来指定根边距（根元素的观察影响范围）
    ![](https://user-gold-cdn.xitu.io/2019/10/22/16df3610c9bf7742?w=710&h=556&f=png&s=23924)
3.  `threshold`：阈值，可以为数组。`[0.3]`意味着，当目标元素在根元素指定的元素内可见30%时，调用处理函数。

#### 2. **定义回调事件**

当目标元素与根元素通过阈值相交时，就会触发回调函数。
```
function handler (entries, observer) { 
    entries.forEach(entry => { 
    // 每个成员都是一个IntersectionObserverEntry对象。
    // 举例来说，如果同时有两个被观察的对象的可见性发生变化，entries数组就会有两个成员。
    // entry.boundingClientRect 
    // entry.intersectionRatio 
    // entry.intersectionRect 
    // entry.isIntersecting 
    // entry.rootBounds 
    // entry.target 
    // entry.time 
    }); 
}
```


* time 时间戳
* rootBounds 根元素的位置
* boundingClientRect 目标元素的位置信息
* intersectionRect 交叉部分的位置信息
* intersectionRatio 目标元素的可见比例，看下图示
* target。


![](https://user-gold-cdn.xitu.io/2019/10/22/16df3f6a6b04daeb?w=886&h=876&f=png&s=86284)


#### 3. **定义要观察的目标对象**

任何目标元素都可以通过调用`.observer(target)`方法来观察。
```
const target = document.querySelector(“.targetBox”); 
observer.observe(target);
```

此外，还有两个方法：

停止对某目标的监听
```
observer.unobserve(target)
```
终止对所有目标的监听
```
observer.disconnect()
```

### 4. 例子1：图片懒加载

`HTML`:

```
<img src="placeholder.png" data-src="img-1.jpg">
<img src="placeholder.png" data-src="img-2.jpg">
<img src="placeholder.png" data-src="img-3.jpg">
<!-- more images -->
```

脚本：
```
let observer = new IntersectionObserver(
(entries, observer) => { 
entries.forEach(entry => {
    /* 替换属性 */
    entry.target.src = entry.target.dataset.src;
    observer.unobserve(entry.target);
  });
}, 
{rootMargin: "0px 0px -200px 0px"});

document.querySelectorAll('img').forEach(img => { observer.observe(img) });
```
上述例子表示 仅在到达视口距离底部200px视加载图片。

### 5. 例子2：兴趣埋点

关于兴趣埋点，一个比较通用的方案是：
> [来自：《超好用的API之IntersectionObserver》](https://juejin.im/post/5d11ced1f265da1b7004b6f7#heading-8)

```
const boxList = [...document.querySelectorAll('.box')]

var io = new IntersectionObserver((entries) =>{
  entries.forEach(item => {
    // intersectionRatio === 1说明该元素完全暴露出来，符合业务需求
    if (item.intersectionRatio === 1) {
      // 。。。 埋点曝光代码
      io.unobserve(item.target)
    }
  })
}, {
  root: null,
  threshold: 1, // 阀值设为1，当只有比例达到1时才触发回调函数
})

// observe遍历监听所有box节点
boxList.forEach(box => io.observe(box))
```

至于怎样评断用户是否感兴趣，记录方式就见仁见智了：

* 位于屏幕中间，并停留时长大于2秒，计数一次。
* 区域悬停，触发定时器记录时间。
* `PC`端记录鼠标点击次数/悬停时间，移动端记录`touch`事件

这里就不展开写了（我懒）。

### 6. 控制动画/视频 执行
这里提供控制视频的版本

`HTML`:

```
<video src="OSRO-animation.mp4" controls=""></video>
```

`js`:

```
let video = document.querySelector('video');
let isPaused = false; /* Flag for auto-paused video */
let observer = new IntersectionObserver((entries, observer) => { 
  entries.forEach(entry => {
    if(entry.intersectionRatio!=1  && !video.paused){
      video.pause(); isPaused = true;
    }
    else if(isPaused) {video.play(); isPaused=false}
  });
}, {threshold: 1});
observer.observe(video);
```
效果：


![](https://user-gold-cdn.xitu.io/2019/10/23/16df6bce3adeb937?w=504&h=721&f=gif&s=520369)

## 2. `Mutation Observer`：变动观察者

> 接口提供了监视对`DOM`树所做更改的能力。它被设计为旧的`MutationEvents`功能的替代品，该功能是`DOM3 Events`规范的一部分。

### 1. 出现的意义

![](https://user-gold-cdn.xitu.io/2019/10/23/16df775ba854199d?w=800&h=239&f=png&s=43279)
归根究底，是`MutationEvents`的功能不尽人意：
1. 在`MDN`中也写到了，是被`DOM Event`承认在API上有缺陷，反对使用。
2. 核心缺陷是：性能问题和跨浏览器支持。
3. 为`DOM`添加 `mutation` 监听器极度降低进一步修改`DOM`文档的性能（慢1.5 - 7倍），此外, 移除监听器不会逆转的损害。

> [来自：《监听DOM加载完成及改变——MutationObserver应用》](https://juejin.im/post/5d6dd5f3f265da03c23eeff9)

`MutationEvents`的原理：通过绑定事件监听`DOM`

乍一看到感觉很正常，那列一下相关监听的事件：
```
DOMAttributeNameChanged
DOMCharacterDataModified
DOMElementNameChanged
DOMNodeInserted
DOMNodeInsertedIntoDocument
DOMNodeRemoved
DOMNodeRemovedFromDocument
DOMSubtreeModified
```
甭记，这么多事件，各内核各版本浏览器想兼容怕是要天荒地老。

### 2. `MutationObserver`的优势

而`Mutation Observer`的优势在于：

* `MutationEvents`事件是同步触发，也就是说，`DOM` 的变动立刻会触发相应的事件；
* `Mutation Observer` 则是异步触发，`DOM` 的变动并不会马上触发，而是要等到当前所有 `DOM` 操作都结束才触发。
* 可以通过配置项，监听目标`DOM`下子元素的变更记录

简单讲：`异步万岁`!

### 3. `MutationObserver`基本使用

使用`MutationObserver API`主要需要三个步骤：

1. 创建观察者
2. 定义回调函数
3. 定义要观察的目标对象

#### 1. 创建观察者
```
let observer = new MutationObserver(callback);

```
#### 2. 定义回调函数

上面代码中的回调函数，会在每次 DOM 变动后调用。该回调函数接受两个参数，第一个是变动数组，第二个是观察器实例，下面是一个例子：
```
function callback (mutations, observer) {
  mutations.forEach(function(mutation) {
    console.log(mutation);
  });
});
```
其中每个`mutation`都对应一个`MutationRecord`对象，记录着`DOM`每次发生变化的变动记录

`MutationRecord`对象包含了DOM的相关信息，有如下属性：
| 属性 | 意义 |
| ---- | ---- |
| `type`| 观察的变动类型（`attribute`、`characterData`或者`childList`）| 
|  `target`| 发生变动的`DOM`节点 | 
|  `addedNodes`| 新增的`DOM`节点 | 
|  `removedNodes`| 删除的`DOM`节点 | 
|  `previousSibling`| 前一个同级节点，如果没有则返回`null` | 
|  `nextSibling`| 下一个同级节点，如果没有则返回`null` | 
|  `attributeName`| 发生变动的属性。如果设置了`attributeFilter`，则只返回预先指定的属性 | 
| `oldValue`| 变动前的值。这个属性只对`attribute`和`characterData`变动有效，如果发生`childList`变动，则返回`null`| 


#### 3. 定义要观察的目标对象

```
MutationObserver.observe(dom, options)
```
启动监听，接收两个参数。

* 第一参数：被观察的`DOM`节点。
* 第二参数：配置需要观察的变动项`options`。
```
mutationObserver.observe(content, {
    attributes: true, // Boolean - 观察目标属性的改变
    characterData: true, // Boolean - 观察目标数据的改变(改变前的数据/值)
    childList: true, // Boolean - 观察目标子节点的变化，比如添加或者删除目标子节点，不包括修改子节点以及子节点后代的变化
    subtree: true, // Boolean - 目标以及目标的后代改变都会观察
    attributeOldValue: true, // Boolean - 表示需要记录改变前的目标属性值
    characterDataOldValue: true, // Boolean - 设置了characterDataOldValue可以省略characterData设置
    // attributeFilter: ['src', 'class'] // Array - 观察指定属性
});
```
 优先级 ：

1. `attributeFilter/attributeOldValue` > `attributes`
2. `characterDataOldValue` > `characterData`
3. `attributes/characterData/childList`（或更高级特定项）至少有一项为true；
4. 特定项存在, 对应选项可以忽略或必须为`true`

此外，还有两个方法：

停止观察。调用后不再触发观察器，解除订阅
```
MutationObserver.disconnect()
```
清除变动记录。即不再处理未处理的变动。该方法返回变动记录的数组，注意，该方法立即生效。
```
MutationObserver.takeRecords()
```

### 4. 例子1：`MutationObserver`监听文本变化

基本使用是：
```
const target = document.getElementById('target-id')

const observer = new MutationObserver(records => {
  // 输入变更记录
})

// 开始观察
observer.observe(target, {
  characterData: true
})
```
这里可以有几种处理。


* 聊天的气泡框彩蛋，检测文本中的指定字符串/表情包，触发类似微信聊天的表情落下动画。
* 输入框的热点话题搜索，当输入“`#`”号时，启动搜索框预检文本或高亮话题。

有个`Vue`的小型插件就是这么实现的：
> [来自：《vue-hashtag-textarea》](https://github.com/mitsuyacider/vue-hashtag-textarea)

![](https://user-gold-cdn.xitu.io/2019/10/23/16df7e875c46f836?w=1718&h=962&f=gif&s=411083)


### 5. 例子2: 色块小游戏脚本

这个实现也是秀得飞起：
> [Hacking the color picker game — MutationObserver](https://benhuang.info/2019/02/20/hacking-the-color-picker-game-mutationobserver/)

![](https://user-gold-cdn.xitu.io/2019/10/23/16df7f27f7224655?w=1024&h=508&f=png&s=90098)

游戏的逻辑很简单，当中间的色块颜色改变时，在时间限制内于底下的选项选择跟它颜色一样的选项就得分。难的点在于越后面的关卡选项越多，而且选项颜色也越相近，例如：


![](https://user-gold-cdn.xitu.io/2019/10/23/16df7f323c1b016c?w=1024&h=504&f=png&s=108051)

其实原理非常简单，就是观察色块的`backgroundColor`（属性变化`attributes`)，然后触发点击事件`e.click()`。
```
var targetNode = document.querySelector('#kolor-kolor');
var config = { attributes: true };
var callback = function(mutationsList, observer) {
    if (mutationsList[0].type == 'attributes') {
        console.log('attribute change!');
        let ans = document.querySelector('#kolor-kolor').style.backgroundColor;
        document.querySelectorAll('#kolor-options a').forEach( (e) => {
            if (e.style.backgroundColor == ans) {
                e.text = 'Ans!';
                e.click()
            }
        })
    }
};

var observer = new MutationObserver(callback);
observer.observe(targetNode, config);
```

![](https://user-gold-cdn.xitu.io/2019/10/23/16df7f942b6c5432?w=1024&h=769&f=png&s=249432)

## 3. `ResizeObserver`，视图观察者
`ResizeObserver API`是一个新的`JavaScript API`，与`IntersectionObserver API`非常相似，它们都允许我们去监听某个元素的变化。

### 1. 出现的意义


* 开发过程当中经常遇到的一个问题就是如何监听一个 `div` 的尺寸变化。
* 但众所周知，为了监听 `div` 的尺寸变化，都将侦听器附加到 `window` 中的 `resize` 事件。
* 但这很容易导致性能问题，因为大量的触发事件。
* 换句话说，使用
`window.resize` 通常是浪费的，因为它告诉我们每个视窗大小的变化，而不仅仅是当一个元素的大小发生变化。

* **而且`resize`事件会在一秒内触发将近60次，很容易在改变窗口大小时导致性能问题**

比如说，你要调整一个元素的大小，那就需要在 `resize` 的回调函数 `callback()` 中调用 `getBoundingClientRect` 或 `getComputerStyle`。不过你要是不小心处理所有的读和写操作，就会导致布局混乱。比如下面这个小示例：

![](https://user-gold-cdn.xitu.io/2019/10/23/16df805c03bd6059?w=960&h=540&f=gif&s=489696)

### 2. `ResizeObserver`的优势

`ResizeObserver API` 的核心优势有两点：
* 细颗粒度的`DOM`元素观察，而不是`window`
* 没有额外的性能开销，只会在绘制前或布局后触发调用

### 3. `ResizeObserver`基本使用

使用`ResizeObserver API`同样也是三个步骤：

1. 创建观察者
2. 定义回调函数
3. 定义要观察的目标对象


#### 1. 创建观察者
```
let observer = new ResizeObserver(callback);

```
#### 2. 定义回调函数

```
const callback = entries => {
    entries.forEach(entry => {
        
    })
}
```
每一个`entry`都是一个对象，包含两个属性`contentRect`和`target`
![](https://user-gold-cdn.xitu.io/2019/10/23/16df8340db8bb93f?w=334&h=228&f=png&s=26592)

`contentRect`都是一些位置信息：
| 属性 | 作用 |
| ---- | ---- |
|  `bottom`|  `top + height`的值 | 
|  `height`| 元素本身的高度，不包含`padding`，`border`值 | 
|  `left`| `padding-left`的值 | 
|  `right`| `left + width`的值 | 
|  `top`| `padidng-top`的值 | 
| `width`| 元素本身的宽度，不包含`padding`，`border`值 | 
|  `x`| 大小与`top`相同 | 
|  `y`| 大小与`left`相同 | 


#### 3. 定义要观察的目标对象

```
observer.observe(document.body)
```

![](https://user-gold-cdn.xitu.io/2019/10/23/16df84167c1a9737?w=858&h=323&f=gif&s=1081983)

`unobserve`方法：取消单节点观察
```
observer.unobserve(document.body)
```
`disconnect`方法：取消所有节点观察
```
observer.disconnect(document.body)
```
### 4. 例子1：缩放渐变背景
`html`：
```
<div class="box">
    <h3 class="info"></h3>
</div>
<div class="box small">
    <h3 class="info"></h3>
</div>
```

添加点样式：
```
body {
    width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 2vw;
    box-sizing: border-box;
}
.box {
    text-align: center;
    height: 20vh;
    border-radius: 8px;
    box-shadow: 0 0 4px rgba(0,0,0,.25);
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 1vw
}
.box h3 {
    color: #fff;
    margin: 0;
    font-size: 5vmin;
    text-shadow: 0 0 10px rgba(0,0,0,0.4);
}
.box.small {
    max-width: 550px;
    margin: 1rem auto;
}
```

`JavaScript`代码：
```
const boxes = document.querySelectorAll('.box');
let callbackFired = 0;
const myObserver = new ResizeObserver(entries => {
    for (let entry of entries) {
        callbackFired++
        const infoEl = entry.target.querySelector('.info');
        const width = Math.floor(entry.contentRect.width);
        const height = Math.floor(entry.contentRect.height);
        const angle = Math.floor(width / 360 * 100);
        const gradient = `linear-gradient(${ angle }deg, rgba(0,143,104,1) 50%, rgba(250,224,66,1) 50%)`;
        entry.target.style.background = gradient;
        infoEl.innerText = `
        I'm ${ width }px and ${ height }px tall
        Callback fired: ${callbackFired}
        `;
    }
});
boxes.forEach(box => {
    myObserver.observe(box);
});
```

当你拖动浏览器窗口，改变其大小时，看到的效果如下：
![](https://user-gold-cdn.xitu.io/2019/10/23/16df844ce8f85e45?w=738&h=417&f=gif&s=521271)

### 5. 例子2：响应式`Vue`组件


![](https://user-gold-cdn.xitu.io/2019/10/23/16df85acd889db3e?w=1360&h=504&f=png&s=111457)

* 假设你要创建一个postItem组件，在大屏上是这样的显示效果

![](https://user-gold-cdn.xitu.io/2019/10/23/16df8639cc511681?w=800&h=512&f=png&s=200064)
* 在手机上需要这样的效果：
![](https://user-gold-cdn.xitu.io/2019/10/23/16df8640f1cc6ade?w=1078&h=1602&f=png&s=914860)

简单的`@media`就可以实现:
```
@media only screen and (max-width: 576px) {
  .post__item {
    flex-direction: column;
  }
  
  .post__image {
    flex: 0 auto;
    height: auto;
  }
}
```

* 但这就很容易出现 当你在超过预期的屏幕（过大）查看页面时，会出现以下的布局：

![](https://user-gold-cdn.xitu.io/2019/10/23/16df85d633a4da85?w=2524&h=1596&f=png&s=1038062)

**`@media`查询的最大问题是：**

* 组件响应度取决于屏幕尺寸，而不是响应自身的尺寸。


以下是指令版实现：


![](https://user-gold-cdn.xitu.io/2019/10/23/16df87b077e12408?w=762&h=624&f=png&s=104919)
使用：

![](https://user-gold-cdn.xitu.io/2019/10/23/16df8785890f52e3?w=762&h=437&f=png&s=69839)

效果：

![](https://user-gold-cdn.xitu.io/2019/10/23/16df86e5671e9e59?w=2514&h=1600&f=png&s=1773292)

这是`vue-responsive-components`库的具体实现代码，还有组件形式的实现，感兴趣的可以去看看。


## 4. `PerformanceObserver`：性能观察者

**这是一个浏览器和`Node.js` 里都存在的API，采用相同`W3C`的`Performance Timeline`规范**


* 在浏览器中，我们可以使用 window 对象取得`window.performance`和 `window.PerformanceObserver` 。
* 而在 `Node.js` 程序中需要`perf_hooks` 取得性能对象，如下：
  ```
  const { PerformanceObserver, performance } = require('perf_hooks');
  ```

### 1. 出现的意义

首先来看`Performance` 接口：

* 可以获取到当前页面中与性能相关的信息。它是 `High Resolution Time API` 的一部分，同时也融合了 `Performance Timeline API`、`Navigation Timing AP`、 `User Timing API` 和 `Resource Timing API`。


* `Performance API` 是大家熟悉的一个接口，他记录着几种性能指数的庞大对象集合。

![](https://user-gold-cdn.xitu.io/2019/10/23/16df910b6d854744?w=1146&h=438&f=png&s=285484)

1. 若想获得某项页面加载性能记录，就需要调用`performance.getEntries`或者`performance.getEntriesByName`来获得。
2. 而获得执行效率，也只能通过`performance.now`来计算。

![](https://user-gold-cdn.xitu.io/2019/10/23/16df918d6a33e2b9?w=912&h=555&f=png&s=214147)

为了解决上述的问题，在`Performance Timeline Level 2`中，除了扩展了`Performance`的基本定义以外，还增加了`PerformanceObserver`接口。

### 2. `PerformanceObserver`的优势

`PerformanceObserver`是浏览器内部对`Performance`实现的观察者模式，也是现代浏览器支持的几个 `Observer` 之一。

> [来自：《你了解 Performance Timeline Level 2 吗？》](https://juejin.im/post/5cb19aece51d456e4037727e#heading-3)

它解决了以下3点问题：

* 避免不知道性能事件啥时候会发生，需要重复轮训`timeline`获取记录。
* 避免产生重复的逻辑去获取不同的性能数据指标
* 避免其他资源需要操作浏览器性能缓冲区时产生竞态关系。

`W3C`官网文档鼓励开发人员尽可能使用`PerformanceObserver`，而不是通过`Performance`获取性能参数及指标。

### 3. `PerformanceObserver`的使用

使用`PerformanceObserver API`主要需要三个步骤：

1. 创建观察者
2. 定义回调函数事件
3. 定义要观察的目标对象

#### 1. 创建观察者
```
let observer = new PerformanceObserver(callback); 
```

#### 2. 定义回调函数事件

```
const callback = (list, observer) => {
   const entries = list.getEntries();
   entries.forEach((entry) => {
    console.log(“Name: “ + entry.name + “, Type: “ + entry.entryType + “, Start: “ + entry.startTime + “, Duration: “ + entry.duration + “\n”); });
}
```
其中每一个`list`都是一个完整的`PerformanceObserverEntryList`对象：
![](https://user-gold-cdn.xitu.io/2019/10/23/16df9412061956d0?w=1130&h=310&f=png&s=93030)
包含三个方法`getEntries`、`getEntriesByType`、`getEntriesByName`：
| 方法 | 作用   |
| ----- | --------- |
| getEntries()	| 返回一个列表，该列表包含一些用于承载各种性能数据的对象，不做任何过滤 |
| getEntriesByType() | 	返回一个列表，该列表包含一些用于承载各种性能数据的对象，按类型过滤 |
| getEntriesByName() |	返回一个列表，，该列表包含一些用于承载各种性能数据的对象，按名称过滤 |
#### 3. 定义要观察的目标对象
```
observer.observe({entryTypes: ["entryTypes"]});
```
`observer.observe(...)`方法接受可以观察到的有效的入口类型。这些输入类型可能属于各种性能API，比如`User tming`或`Navigation Timing API`。有效的`entryType`值：


| 属性 | 别名 | 类型 | 描述
| ---- | ---- | ---- | ---- |
|frame， navigation	| PerformanceFrameTiming， PerformanceNavigationTiming | URL| 文件的地址。|
|resource|	PerformanceResourceTiming |	URL	| 所请求资源的解析URL。| 即使重定向请求，此值也不会更改。|
|mark|	PerformanceMark	| DOMString |	通过调用创建标记时使用的名称performance.mark()。|
|measure |	PerformanceMeasure |	DOMString |	通过调用创建度量时使用的名称performance.measure()。|
|paint |	PerformancePaintTiming	| DOMString |无论是'first-paint'或'first-contentful-paint'。|
|longtask |	PerformanceLongTaskTiming |	DOMString |报告长任务的实例|

### 4. 例子1：静态资源监控

> [来自：《资源监控》](http://www.zhanglongdream.com/2018/12/27/js/JavaScript/%E8%B5%84%E6%BA%90%E7%9B%91%E6%8E%A7/)
```
function filterTime(a, b) {
  return (a > 0 && b > 0 && (a - b) >= 0) ? (a - b) : undefined;
}

let resolvePerformanceTiming = (timing) => {
  let o = {
    initiatorType: timing.initiatorType,
    name: timing.name,
    duration: parseInt(timing.duration),
    redirect: filterTime(timing.redirectEnd, timing.redirectStart), // 重定向
    dns: filterTime(timing.domainLookupEnd, timing.domainLookupStart), // DNS解析
    connect: filterTime(timing.connectEnd, timing.connectStart), // TCP建连
    network: filterTime(timing.connectEnd, timing.startTime), // 网络总耗时

    send: filterTime(timing.responseStart, timing.requestStart), // 发送开始到接受第一个返回
    receive: filterTime(timing.responseEnd, timing.responseStart), // 接收总时间
    request: filterTime(timing.responseEnd, timing.requestStart), // 总时间

    ttfb: filterTime(timing.responseStart, timing.requestStart), // 首字节时间
  };

  return o;
};

let resolveEntries = (entries) => entries.map(item => resolvePerformanceTiming(item));

let resources = {
  init: (cb) => {
    let performance = window.performance || window.mozPerformance || window.msPerformance || window.webkitPerformance;
    if (!performance || !performance.getEntries) {
      return void 0;
    }

    if (window.PerformanceObserver) {
      let observer = new window.PerformanceObserver((list) => {
        try {
          let entries = list.getEntries();
          cb(resolveEntries(entries));
        } catch (e) {
          console.error(e);
        }
      });
      observer.observe({
        entryTypes: ['resource']
      })
    } else {
        window.addEventListener('load', () => {
        let entries = performance.getEntriesByType('resource');
        cb(resolveEntries(entries));
      });
    }
  },
};
```


## 参考文章&总结

参考文章有点多：
> * [资源监控](http://www.zhanglongdream.com/2018/12/27/js/JavaScript/%E8%B5%84%E6%BA%90%E7%9B%91%E6%8E%A7/)
> * [Media Queries Based on Element Width with MutationObserver](https://codeburst.io/media-queries-based-on-element-width-with-mutationobserver-cf2eff172787)
> * [以用户为中心的性能指标](https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics?hl=zh-cn)
> * [A Few Functional Uses for Intersection Observer to Know When an Element is in View](https://css-tricks.com/a-few-functional-uses-for-intersection-observer-to-know-when-an-element-is-in-view/)
> * [Getting To Know The MutationObserver API](https://css-tricks.com/getting-to-know-the-mutationobserver-api/)
> * [Different Types Of Observers Supported By Modern Browsers](https://www.zeolearn.com/magazine/different-types-of-observers-supported-by-modern-browsers)
> * [THE RESIZE OBSERVER EXPLAINED](https://pawelgrzybek.com/the-resize-observer-explained/)
> * [A Look at the Resize Observer JavaScript API](https://alligator.io/js/resize-observer/)
这四个观察者，都非常适合集成到监控系统。

且都有对应的`Polyfills`版实现。

网上的总结和文档都深浅不一，如果哪里有错误，欢迎指正。

![](https://user-gold-cdn.xitu.io/2019/10/24/16df9766165811f9?w=198&h=189&f=png&s=34671)

## ❤️ 看完三件事
如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章
* [Chrome Devtools 高级调试指南（新）](https://juejin.im/post/5d9eea84e51d4577eb5d8510)
* [90行代码，15个元素实现无限滚动](https://juejin.im/post/5d7f80796fb9a06b24434d4e)
* [「React Hooks」120行代码实现一个交互完整的拖拽上传组件](https://juejin.im/post/5d674313e51d4561c94b1000)
* [「React Hooks」160行代码实现动态炫酷的可视化图表 - 排行榜](https://juejin.im/post/5d565015f265da03eb13c575)
![](https://user-gold-cdn.xitu.io/2019/8/5/16c5faffbefaea2e?w=2006&h=1014&f=png&s=672314)

也可以来我的`GitHub`博客里拿所有文章的源文件：

**前端劝退指南**：https://github.com/roger-hiro/BlogFN
