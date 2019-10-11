## 前言
本文暂未涉及`Performance`面板的内容。
后续会单独出一篇，以下是目录：
1. 常用命令和调试
2. 黑盒脚本：`Blackbox Script`
3. 控制台内置指令
4. 远程调试`WebView`


## 1. Chrome Devtools 的用处

* 前端开发：开发预览、远程调试、性能调优、`bug`跟踪、断点调试等
* 后端开发：网络抓包、开发调试Response
* 测试：服务端`API`数据是否正确、审查页面元素样式及布局、页面加载性能分析、自动化测试
* 其他：安装扩展插件，如`AdBlock、Gliffy、Axure`等

## 2. 菜单面板拆解

![](https://user-gold-cdn.xitu.io/2019/10/10/16db4c8b9f9e0039?w=1334&h=606&f=png&s=372408)

* `Elements` - 页面dom元素
* `Console` - 控制台
* `Sources` - 页面静态资源
* `Network` - 网络
* `Performance` - 设备加载性能分析
* `Application` - 应用信息，`PWA/Storage/Cache/Frames`
* `Security` - 安全分析
* `Audits` - 审计，自动化测试工具


## 3. 常用命令和调试

### 1. 呼出快捷指令面板：`cmd + shift + p`
在`Devtools`打开的情况下，键入`cmd + shift + p`将其激活，然后开始在栏中键入要查找的命令或输入`"?"`号以查看所有可用命令。

![](https://user-gold-cdn.xitu.io/2019/10/10/16db4e8a7474c77f?w=542&h=171&f=png&s=14882)

* `...`: 打开文件
* `:`: 前往文件
* `@`：前往标识符（函数，类名等）
* `!`: 运行脚本文件
* `>`: 打开某菜单功能

![](https://user-gold-cdn.xitu.io/2019/10/10/16db4f67a43c6570?w=800&h=277&f=gif&s=216716)

#### 1.性能监视器：`> performance monitor` 
![](https://user-gold-cdn.xitu.io/2019/10/10/16db4fb29642e1e3?w=800&h=277&f=gif&s=1986204)
将显示性能监视器以及相关信息，例如CPU，JS堆大小和DOM节点。

#### 2.FPS实时监控性能：`> FPS`选择第一项 

![](https://user-gold-cdn.xitu.io/2019/10/10/16db50b5d0815563?w=800&h=450&f=gif&s=738130)

#### 3.截图单个元素：`> screen` 选择`Capture node screenhot`
![](https://user-gold-cdn.xitu.io/2019/10/11/16db930f51b63ffe?w=258&h=30&f=png&s=4848)
![](https://user-gold-cdn.xitu.io/2019/10/11/16db9307c60c3515?w=1200&h=568&f=gif&s=2118445)
### 2. `DOM`断点调试
当你要调试特定元素的DOM中的更改时，可以使用此选项。这些是DOM更改断点的类型：
![](https://user-gold-cdn.xitu.io/2019/10/10/16db50f8ed98bae0?w=523&h=350&f=png&s=199466)

* `Subtree modifications`: 子节点删除或添加时
* `Attributes modifications`: 属性修改时
* `Node Removal`: 节点删除时


![](https://user-gold-cdn.xitu.io/2019/10/10/16db5195a66fdb31?w=800&h=354&f=gif&s=419635)

如上图：**监听`form`标签，在`input`框获得焦点时，触发断点调试**

### 3. 黑盒脚本：`Blackbox Script`
剔除多余脚本断点。

例如第三方（`Javascript`框架和库，广告等的堆栈跟踪）。

为避免这种情况并集中精力处理核心代码，在`Sources`或网络选项卡中打开文件，右键单击并选择`Blackbox Script`
![](https://user-gold-cdn.xitu.io/2019/10/10/16db534dc43153fb?w=800&h=277&f=gif&s=1159439)。

### 4. 事件监听器：`Event Listener Breakpoints`
1. 点击`Sources`面板
2. 展开`Event Listener Breakpoints`
3. 选择监听事件类别，触发事件启用断点

![](https://user-gold-cdn.xitu.io/2019/10/10/16db655b352efa5c?w=1200&h=519&f=gif&s=290416)
如上图：**监听了键盘输入事件，就会跳到断点处。**

### 5. 本地覆盖：`local overrides`

使用我们自己的本地资源覆盖网页所使用的资源。

类似的，使用`DevTools`的工作区设置持久化，将本地的文件夹映射到网络，在`chrome`开发者功能里面对css 样式的修改，都会直接改动本地文件，页面重新加载，使用的资源也是本地资源，达到持久化的效果。


* 创建一个文件夹以在本地添加替代内容；
* 打开`Sources > Overrides > Enable local Overrides`，选择本地文件夹
![](https://user-gold-cdn.xitu.io/2019/10/11/16db67e0156cf5cc)
* 打开`Elements`，编辑样式，自动生成本地文件。
* 返回`Sources`，检查文件，编辑更改。
![](https://user-gold-cdn.xitu.io/2019/10/11/16db688da68d1556?w=1206&h=521&f=gif&s=1146374)

### 6. 扩展：`local overrides`模拟`Mock`数据
> [来自：chrome 开发者工具 - local overrides](https://segmentfault.com/a/1190000016612065)

对于返回json 数据的接口，可以利用该功能，简单模拟返回数据。

比如：

* `api` 为: `http://www.xxx.com/api/v1/list`

* 在根目录下，新建文件` www.xxx.com/api/v1/list`，`list` 文件中的内容，与正常接口返回格式相同。

![](https://user-gold-cdn.xitu.io/2019/10/11/16db6797869a78d9?w=800&h=133&f=png&s=39225)

对象或者数组类型，从而覆盖掉原接口请求。

## 4. 控制台内置指令
可以执行常见任务的功能，例如选择`DOM`元素，触发事件，监视事件，在`DOM`中添加和删除元素等。

这像是`Chrome`自身实现的`jquery`加强版。

### 1. `$(selector, [startNode])`：单选择器
`document.querySelector`的简写
语法：
```
$('a').href;
$('[test-id="logo-img"]').src;
$('#movie_player').click();
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db6a64e3043b0d?w=600&h=316&f=gif&s=2233622)
控制台还会预先查询对应的标签，十分贴心。
还可以触发事件，如暂停播放：
![](https://user-gold-cdn.xitu.io/2019/10/11/16db6a848bda0fdd?w=600&h=319&f=gif&s=2525736)

此函数还支持第二个参数startNode，该参数指定从中搜索元素的“元素”或Node。此参数的默认值为document
### 2. `$（选择器，[startNode]）`：全选择器
`document.querySelectorAll`的简写，返回一个数组标签元素
语法：
```
$$('.button')
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db6ac2ccee76e7?w=600&h=351&f=gif&s=642918)
可以用过循环实现切换全选
![](https://user-gold-cdn.xitu.io/2019/10/11/16db6ae2acd51188?w=600&h=348&f=gif&s=792930)
或者打印属性

![](https://user-gold-cdn.xitu.io/2019/10/11/16db6b39a67edde8?w=2240&h=1500&f=png&s=468506)
此函数还支持第二个参数startNode，该参数指定从中搜索元素的“元素”或Node。此参数的默认值为document
用法：
```
var images = $$('img', document.querySelector('.devsite-header-background'));
   for (each in images) {
       console.log(images[each].src);
}
```


![](https://user-gold-cdn.xitu.io/2019/10/11/16db6b4ca2ce7fd9?w=2240&h=940&f=png&s=283050)

### 3. `$x(path, [startNode])`：`xpath`选择器
`$x(path) `返回与给定`xpath`表达式匹配的DOM元素数组。

例如，以下代码返回`<p>`页面上的所有元素：
```
$x("//p")
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db6b72a99ea2cf?w=2708&h=954&f=png&s=242469)
以下代码返回`<p>`包含`<a>`元素的所有元素：
```
$x("//p[a]")
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db6b7d1025abb3?w=2708&h=850&f=png&s=184012)

**`xpath`多用于爬虫抓取，前端的同学可能不熟悉。**

### 4. `getEventListeners（object）`：获取指定对象的绑定事件

`getEventListeners(object)`返回在指定对象上注册的事件侦听器。返回值是一个对象，其中包含每个已注册事件类型（例如，`click`或`keydown`）的数组。每个数组的成员是描述为每种类型注册的侦听器的对象。
用法:
```
getEventListeners(document);
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db6c360ec9c0db?w=1304&h=608&f=png&s=130854)
相对于到监听面板里查事件，这个`API`便捷多了。


## 5. 花式`console`

![](https://user-gold-cdn.xitu.io/2019/10/11/16db8d89286ce068?w=2400&h=998&f=png&s=146518)
除了不同等级的`warn`和`error`打印外
![](https://user-gold-cdn.xitu.io/2019/10/11/16db8ab4e065c1e5?w=244&h=119&f=png&s=10660)
还有其它非常实用的打印。

### 1. 变量打印：`%s`、`%o`、`%d`、和`%c`
```
const text = "文本1"
console.log(`打印${text}`)
```
除了标准的`ES6`语法，实际上还支持四种字符串输出。
分别是：
```
console.log("打印 %s", text)
```

* `%s`：字符串
* `%o`：对象
* `%d`：数字或小数

还有比较特殊的`%c`，可用于改写输出样式。
```
console.log('%c 文本1', 'font-size:50px; background: ; text-shadow: 10px 10px 10px blue')
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db8b60a77369e5?w=676&h=93&f=png&s=23316)
当然，你也可以结合其它一起用(注意占位的顺序)。
```
const text = "文本1"
console.log('%c %s', 'font-size:50px; background: ; text-shadow: 10px 10px 10px blue', text)
```

![](https://user-gold-cdn.xitu.io/2019/10/11/16db8b8fa07af326?w=724&h=104&f=png&s=24976)

你还可以这么玩：
```
console.log('%c Auth ', 
            'color: white; background-color: #2274A5', 
            'Login page rendered');
console.log('%c GraphQL ', 
            'color: white; background-color: #95B46A', 
            'Get user details');
console.log('%c Error ', 
            'color: white; background-color: #D33F49', 
            'Error getting user details');
```

![](https://user-gold-cdn.xitu.io/2019/10/11/16db8e31a49f2ff3?w=637&h=161&f=png&s=25913)

### 2. 打印对象的小技巧
当我们需要打印多个对象时，经常一个个输出。且看不到对象名称，不利于阅读：

![](https://user-gold-cdn.xitu.io/2019/10/11/16db8bd81861ee5c?w=553&h=247&f=png&s=32257)

以前我的做法是这么打印：
```
console.log('hello', hello);
console.log('world', world);
```
这显然有点笨拙繁琐。其实，输出也支持对象解构：
```
console.log({hello, world});
```

![](https://user-gold-cdn.xitu.io/2019/10/11/16db8c0009fca11b?w=551&h=419&f=gif&s=63881)

### 3. 布尔断言打印：`console.assert()`
当你需要在特定条件判断时打印日志，这将非常有用。

* 如果断言为false，则将一个错误消息写入控制台。
* 如果断言是true，没有任何反应。

语法
```
console.assert（assertion，obj）
```
用法
```
const value = 1001
console.assert(value===1000,"value is not 1000")
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db8cd6a806f7fb?w=545&h=115&f=png&s=16182)

### 4. 给`console`编组：`console.group()`
当你需要将详细信息分组或嵌套在一起以便能够轻松阅读日志时，可以使用此功能。
```
console.group('用户列表');
console.log('name: 张三');
console.log('job: 🐶前端');
// 内层
console.group('地址');
console.log('Street: 123 街');
console.log('City: 北京');
console.log('State: 在职');
console.groupEnd(); // 结束内层
console.groupEnd(); // 结束外层
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db8e3c98c12c36?w=539&h=167&f=png&s=23016)

### 5. 测试执行效率：`console.time()`
没有`Performance API` 精准，但胜在使用简便。
```
let i = 0;
console.time("While loop");
while (i < 1000000) {
  i++;
}
console.timeEnd("While loop");
console.time("For loop");
for (i = 0; i < 1000000; i++) {
  // For Loop
}
console.timeEnd("For loop");
```

![](https://user-gold-cdn.xitu.io/2019/10/11/16db8e6fee80fee6?w=1278&h=310&f=png&s=55596)

### 6. 输出表格：`console.table()`
这个适用于打印数组对象。。。
```
let languages = [
    { name: "JavaScript", fileExtension: ".js" },
    { name: "TypeScript", fileExtension: ".ts" },
    { name: "CoffeeScript", fileExtension: ".coffee" }
];
console.table(languages);
```

![](https://user-gold-cdn.xitu.io/2019/10/11/16db8ed936871244?w=537&h=158&f=png&s=16643)

### 7. 打印`DOM`对象节点：`console.dir()`
打印出该对象的所有属性和属性值.
`console.dir()`和`console.log()`的作用区别并不明显。若用于打印字符串，则输出一摸一样。
```
console.log（"Why，hello!"）；
console.dir（"Why，hello!"）;
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db98c0e64a88d8?w=359&h=158&f=png&s=16682)
在输出对象时也仅是显示不同（`log`识别为字符串输出，`dir`直接打印对象。）。
![](https://user-gold-cdn.xitu.io/2019/10/11/16db98a92f8b4c84?w=622&h=212&f=png&s=27976)

唯一显著区别在于打印`dom`对象上：
```
console.log(document)
console.dir(document)
```
![](https://user-gold-cdn.xitu.io/2019/10/11/16db99148b525dea?w=533&h=540&f=png&s=95565)
一个打印出纯标签，另一个则是输出`DOM`树对象。

## 6. 远程调试`WebView`

使用`Chrome`开发者工具在原生`Android`应用中调试`WebView`。

1. 配置`WebViews`进行调试。

    在 `WebView`类上调用静态方法`setWebContentsDebuggingEnabled`。
    ```
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        WebView.setWebContentsDebuggingEnabled(true);
    }
    ```
2. 手机打开`usb`调试，插上电脑。
3. 在`Chrome`地址栏输入：`Chrome://inspect`

![](https://user-gold-cdn.xitu.io/2019/10/11/16db9f2b540ab143?w=627&h=511&f=png&s=58609)
    正常的话在`App`中打开`WebView`时，`chrome`中会监听到并显示你的页面。
4. 点击页面下的`inspect`，就可以实时看到手机上`WebView`页面的显示状态了。（第一次使用可能会白屏，这是因为需要去`https://chrome-devtools-frontend.appspot.com`那边下载文件）
![](https://user-gold-cdn.xitu.io/2019/10/11/16db9f0896eae1c8?w=1365&h=728&f=png&s=541868)

除了`inspect`标签，还有 **`Focus tab`**:
* 它会自动触发`Android`设备上的相同操作

**其他应用里的`WebView`也可以，例如这是某个应用里的游戏，用的也是网页：**

![](https://user-gold-cdn.xitu.io/2019/10/11/16db9e59da8e4db9?w=540&h=960&f=png&s=128262)

## 7.调试`Node.js`
具体可以查看阿里云社区的这篇：
> [Node.js应用程序故障排除手册-正确启用Chrome DevTools](https://www.alibabacloud.com/blog/node-js-application-troubleshooting-manual---correctly-enabling-chrome-devtools_594964)

![](https://user-gold-cdn.xitu.io/2019/10/11/16dba541b281c499?w=911&h=461&f=png&s=64801)
Ps: 属于我的知识盲区，就不搬运造次了。
## 参考资料
> 1. [Practical Chrome Devtools — Common commands & Debugging](https://medium.com/@willmendesneto/practical-chrome-devtools-common-commands-debugging-891636b5fbf1)
> 2. [Mobile web specialist — Remote Debugging](https://medium.com/@ahmzyjazzy/mobile-web-specialist-remote-debugging-bf0d7b3b0dde)
> 3. [Console Utilities API Reference](https://developers.google.com/web/tools/chrome-devtools/console/utilities)
> 4. [Console API Reference](https://developers.google.com/web/tools/chrome-devtools/console/api?hl=zh-cn)

## ❤️ 看完三件事
如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章
* [120行代码实现一个交互完整的拖拽上传组件](https://juejin.im/post/5d674313e51d4561c94b1000)
* [160行代码实现动态炫酷的可视化图表 - 排行榜](https://juejin.im/post/5d565015f265da03eb13c575)
* [「数据可视化库王者」D3.js 极速上手到Vue应用](https://juejin.im/post/5d1e074af265da1bca51f8ec)
* [「真®全栈之路」Web前端开发的后端指南](https://juejin.im/post/5cc02aacf265da039e1ff3fa)
* [「Vue实践」5分钟撸一个Vue CLI 插件](https://juejin.im/post/5cb59c4bf265da03a743e979)
* [「Vue实践」武装你的前端项目](https://juejin.im/post/5cab64ce5188251b19486041)
* [「中高级前端面试」JavaScript手写代码无敌秘籍](https://juejin.im/post/5c9c3989e51d454e3a3902b6)
* [「从源码中学习」面试官都不知道的Vue题目答案](https://juejin.im/post/5c959f74f265da610c068fa8)
* [「从源码中学习」Vue源码中的JS骚操作](https://juejin.im/post/5c73554cf265da2de33f2a32)
* [「Vue实践」项目升级vue-cli3的正确姿势](https://juejin.im/post/5c4a83e36fb9a049b13e91ba)
* [为何你始终理解不了JavaScript作用域链？](https://juejin.im/editor/posts/5c8efeb1e51d45614372addd)

![](https://user-gold-cdn.xitu.io/2019/8/5/16c5faffbefaea2e?w=2006&h=1014&f=png&s=672314)