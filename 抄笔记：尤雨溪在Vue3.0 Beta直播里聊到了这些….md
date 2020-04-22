## 前言

在 4 月 21 日晚，Vue 作者尤雨溪在哔哩哔哩直播分享了`Vue.js 3.0 Beta`最新进展。
以下是直播内容整理
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zd36qh6j31hc0u0npd.jpg)

## 1. 全新文档`RFCs`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zqzd824j31i70u0gpm.jpg)

`Vue.js 3.0 Beta`发布后的工作重点是保证稳定性和推进各类库集成

所有的进度和文档都将在全新`RFCs`文档可以看到。

## 2. 六大亮点

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zd5pgiij31h80u00wy.jpg)

- `Performance`：性能更比`Vue 2.0`强。
- `Tree shaking support`：可以将无用模块“剪辑”，仅打包需要的。
- `Composition API`：组合`API`
- `Fragment, Teleport, Suspense`：“碎片”，`Teleport`即`Protal传送门`，“悬念”
- `Better TypeScript support`：更优秀的 Ts 支持
- `Custom Renderer API`：暴露了自定义渲染`API`

下面将按顺序分别描述。

## 3.`Performance`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zd63pq4j315a0iggp1.jpg)

1. 重写了虚拟`Dom`的实现（且保证了兼容性，脱离模版的渲染需求旺盛）。
2. 编译模板的优化。
3. 更高效的组件初始化。
4. `update`性能提高 1.3~2 倍。
5. `SSR`速度提高了 2~3 倍。

下面是各项性能对比

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zdapyzwj31hk0u04dz.jpg)

### 要点 1：编译模板的优化

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge22bv37dhj31k60ladks.jpg)

假设要编译以下代码

```
<div>
  <span/>
  <span>{{ msg }}</span>
</div>
```

将会被编译成以下模样：

```
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", null, "static"),
    _createVNode("span", null, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}

// Check the console for the AST
```

注意看第二个`_createVNode`结尾的“1”：

Vue 在运行时会生成`number`（大于 0）值的`PatchFlag`，用作标记。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge22byzcwaj31k80s011q.jpg)
仅带有`PatchFlag`标记的节点会被真正追踪，且**无论层级嵌套多深，它的动态节点都直接与`Block`根节点绑定，无需再去遍历静态节点**

再看以下例子：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge22c2abk2j31kg0o4n30.jpg)

```
<div>
  <span>static</span>
  <span :id="hello" class="bar">{{ msg }}   </span>
</div>
```

会被编译成：

```
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", null, "static"),
    _createVNode("span", {
      id: _ctx.hello,
      class: "bar"
    }, _toDisplayString(_ctx.msg), 9 /* TEXT, PROPS */, ["id"])
  ]))
}
```

`PatchFlag` 变成了`9 /* TEXT, PROPS */, ["id"]`

它会告知我们不光有`TEXT`变化，还有`PROPS`变化（id）

这样既跳出了`virtual dom`性能的瓶颈，又保留了可以手写`render`的灵活性。
等于是：既有`react`的灵活性，又有基于模板的性能保证。

### 要点 2: 事件监听缓存：`cacheHandlers`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge22c6ayicj31kg0k8q7w.jpg)
假设我们要绑定一个事件：

```
<div>
  <span @click="onClick">
    {{msg}}
  </span>
</div>
```

关闭`cacheHandlers`后：

```
import { toDisplayString as _toDisplayString, createVNode as _createVNode, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", { onClick: _ctx.onClick }, _toDisplayString(_ctx.msg), 9 /* TEXT, PROPS */, ["onClick"])
  ]))
}
```

`onClick`会被视为`PROPS`动态绑定，后续替换点击事件时需要进行更新。

开启`cacheHandlers`后：

```
import { toDisplayString as _toDisplayString, createVNode as _createVNode, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", {
      onClick: _cache[1] || (_cache[1] = $event => (_ctx.onClick($event)))
    }, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}
```

`cache[1]`，会自动生成并缓存一个内联函数，“神奇”的变为一个静态节点。
Ps：相当于`React`中`useCallback`自动化。

并且支持手写内联函数：

```
<div>
  <span @click="()=>foo()">
    {{msg}}
  </span>
</div>
```
### 补充：`PatchFlags`枚举定义

而通过查询`Ts`枚举定义，我们可以看到分别定义了以下的追踪标记：
![](https://imgkr.cn-bj.ufileos.com/bf97144a-ef76-4a36-b756-388af5479fce.png)
感兴趣的可以看源码：`packages/shared/src/patchFlags.ts`
## 4. `Tree shaking support`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zsevggvj31he0u0afk.jpg)

- 可以将无用模块“剪辑”，仅打包需要的（比如`v-model,<transition>`，用不到就不会打包）。
- 一个简单“`HelloWorld`”大小仅为：13.5kb
  - 11.75kb，仅`Composition API`。
- 包含运行时完整功能：22.5kb
  - 拥有更多的功能，却比`Vue 2`更迷你。

很多时候，我们并不需要 `vue`提供的所有功能，在 `vue 2` 并没有方式排除掉，但是 3.0 都可能做成了按需引入。

## 5. `Composition API`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zddq3e4j31h80u0gqs.jpg)
与`React Hooks` 类似的东西，实现方式不同。

- 可与现有的 `Options API`一起使用
- 灵活的逻辑组合与复用
- `vue 3`的响应式模块可以和其他框架搭配使用

混入(`mixin`) 将不再作为推荐使用， `Composition API`可以实现更灵活且无副作用的复用代码。

感兴趣的可以查看：https://composition-api.vuejs.org/#summary
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zt3s5sqj31po0u0e81.jpg)

`Composition API`包含了六个主要`API`

可以到这里查看：https://composition-api.vuejs.org/api.html#setup

Ps：其它的均为常见的工具函数，可先忽略不看。

## 6. `Fragment`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zdhu123j31he0u0whx.jpg)

`Fragment`翻译为：“碎片”

- 不再限于模板中的单个根节点
- `render` 函数也可以返回数组了，类似实现了 `React.Fragments` 的功能 。
- '`Just works`'

### 6.1 `<Teleport>`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zdlt1nhj31hb0u0gp6.jpg)

- 以前称为`<Portal>`，译作传送门。
- 更多细节将由@Linusborg 分享

`<Teleport>`原先是对标 `React Portal`（增加多个新功能，更强）

**但因为`Chrome`有个提案，会增加一个名为`Portal`的原生`element`，为避免命名冲突，改为`Teleport`**

### 6.2 `<Suspense>`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zdpfkktj31he0u00xj.jpg)

`Suspense`翻译为：“悬念”

- 可在嵌套层级中等待嵌套的异步依赖项
- 支持`async setup()`
- 支持异步组件

虽然`React 16`引入了`Suspense`，但直至现在都不太能用。如何将其与异步数据结合，还没完整设计出来。

Vue 3 的`<Suspense>`更加轻量：

**仅 5%应用能感知运行时的调度差异，综合考虑下，Vue3 的`<Suspense>` 没和 React 一样做运行调度处理**

## 7. 更好的`TypeScript`支持

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zdt5e3sj31he0u046k.jpg)

- `Vue 3`是用`TypeScript`编写的库，可以享受到自动的类型定义提示
- `JavaScript`和`TypeScript`中的 API 是相同的。
  - 事实上，代码也基本相同
- 支持`TSX`
- `class`组件还会继续支持，但是需要引入`vue-class-component@next`，该模块目前还处在 alpha 阶段。

还有`Vue 3 + TypeScript` 插件正在开发，有类型检查，自动补全等功能。目前进展可喜。
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zdxzy74j31hc0u010p.jpg)

## 8. `Custom Renderer API`：自定义渲染器 API

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zdz8r6hj31he0u0q9f.jpg)

- 正在进行`NativeScript Vue`集成
- 用户可以尝试`WebGL`自定义渲染器，与普通 Vue 应用程序一起使用（`Vugel`）。

意味着以后可以通过 `vue`， `Dom` 编程的方式来进行 `webgl` 编程 。感兴趣可以看这里：[Getting started vugel](https://vugel.planning.nl/#application)

## 9. 剩余工作

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1ze8h3fgj31ha0u0gpv.jpg)

### 9.1 `Docs & Migration Guides`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zeab0qdj31hm0u045x.jpg)

- 新的文档编写交由`@NataliaTepluhina, @sdras, @bencodezen & @phanan` 负责
- `@sdras` 正在做自动升级迁移工具
- `@sodatea` 已经开始研究`CodeMods`

### 9.2 `Router`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zeejh3sj31hl0u0whv.jpg)

- 下一代 `Router`：`vue-router@next`已在`alpha`阶段，感谢`@posva`

有部分的`API`变动，可到`RFC`上看。

### 9.3 `Vuex`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zej91d2j31hg0u0n31.jpg)`

- 下一代`Vuex`：，`vuex@next`（与`Vue 3 compat`相同的 API），已在`alpha`阶段，感谢`@KiaKing`。
- 团队正在为下一次迭代试验`Vuex API`的简化

目前以兼容`Vue 3`为主，基本上没有`API`变动，莫慌。

### 9.4 `CLI`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zemntztj31h80u0wja.jpg)

- `CLI`插件：`vue-cli-plugin-vue-next`by `@sodatea`
- （wip）`CodeMods`支持升级`Vue 2`应用

### 9.5 新工具：`vite`（法语 “快”）

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zeqxdv0j31h20u0n12.jpg)
地址：https://github.com/vuejs/vite

一个简易的`http`服务器，无需`webpack`编译打包，根据请求的`Vue`文件，直接发回渲染，且支持热更新（非常快）

### 9.6 `vue-test-utils`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zeu7cqcj31hg0u00xc.jpg)

- 下一代`test-utils`:`test-utils@next`
  - by `@lmiller1990, @dobromir-hristov, @afontcu & @JessicaSachs`

### 9.7 `DevTools`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zexzr4xj31hg0u0gpx.jpg)

- 早期的原型已经由@Akryum 完成，当我们到`beta`时，将完全集成。

目前需要花更多精力去完善。

### 9.8 `IDE Support (Vetur)`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zf1owoqj31hm0u0wjz.jpg)

- `@znck`目前正在试验模板的类型检查
- `@octref`将在 5 月为`Vue 3`进行`Vetur`集成

### 9.9 `Nuxt`

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zf302shj31ha0u0dkc.jpg)

目前`Nuxt`的整合工作也正在进行中，内部团队已经跑起来了。还需要时间磨合

## 10 `Vue 2.x`还有 2.7 版本

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zf6uj2oj31ho0u0agc.jpg)

- 将有最后一个小版本（2.7）
- 从`Vue 3`向后移植兼容的改进(不损坏兼容性前提下)
- 加上在`Vue 3`中删除的功能的弃用警告
- `LTS1` 18 个月。

最后建议：`Vue 3`虽好，如果你的项目很稳定，且对新功能无过多的要求或者迁移成本过高，则不建议升级。

## 结束

花了一宿反复回放整理出来的，如有错误，尽情谅解。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zij5j87j31hj0u0q4u.jpg)

附：**直播中用到的渲染模板查看工具地址**：https://vue-next-template-explorer.netlify.app/
![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1zilibhuj31kg0oc0xi.jpg)

## ❤️ 看完三件事

如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -\_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ge1ztq8xd2j30s00f6gpa.jpg)

劝退师个人微信：**huab119**

也可以来我的`GitHub`博客里拿所有文章的源文件：

**前端劝退指南**：https://github.com/roger-hiro/BlogFN
一起玩耍呀。~