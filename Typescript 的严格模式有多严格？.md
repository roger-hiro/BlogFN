## 前言

`"use strict"` 指令在 `JavaScript 1.8.5 (ECMAScript5)` 中新增。

至今，前端er们基本都默认开启严格模式敲代码。

那么，你知道`Typescript` 其实也有属于自己的严格模式吗？

## 1. `Typescript`严格模式规则

当`Typescript`严格模式设置为`on`时，它将使用` strict`族下的严格类型规则对项目中的所有文件进行代码验证。规则是：


| 规则名称                       | 解释                                      |
| ------------------------------ | ----------------------------------------- |
| `noImplicitAny`                | 不允许变量或函数参数具有隐式`any`类型。   |
| `noImplicitThis`               | 不允许`this`上下文隐式定义。              |
| `strictNullChecks`             | 不允许出现`null`或`undefined`的可能性。   |
| `strictPropertyInitialization` | 验证构造函数内部初始化前后已定义的属性。  |
| `strictBindCallApply`          | 对 `bind, call, apply` 更严格的类型检测。 |
| `strictFunctionTypes`          | 对函数参数进行严格逆变比较。              |


## 2. `noImplicitAny`

此规则不允许变量或函数参数具有隐式`any`类型。请看以下示例：

```
// Javascript/Typescript 非严格模式
function extractIds (list) {
  return list.map(member => member.id)
}
```
上述例子没有对`list`进行类型限制，`map`循环了`item`的形参`member`。
而在`Typescript`严格模式下，会出现以下报错：
```
// Typescript 严格模式
function extractIds (list) {
  //              ❌ ^^^^
  //                 Parameter 'list' implicitly
  //                 has an 'any' type. ts(7006)
  return list.map(member => member.id)
  //           ❌ ^^^^^^
  //              Parameter 'member' implicitly
  //              has an 'any' type. ts(7006)
}
```

正确写法应是：
```
// Typescript 严格模式
interface Member {
  id: number
  name: string
}

function extractIds (list: Member[]) {
  return list.map(member => member.id)
}
```

### 1.1 浏览器自带事件该如何处理？

浏览器自带事件，比如`e.preventDefault()`，是阻止浏览器默认行为的关键代码。

这在`Typescript` 严格模式下是会报错的：
```
// Typescript 严格模式
function onChangeCheckbox (e) {
  //                    ❌ ^
  //                       Parameter 'e' implicitly
  //                       has an 'any' type. ts(7006)
  e.preventDefault()
  const value = e.target.checked
  validateCheckbox(value)
}
```
![](https://user-gold-cdn.xitu.io/2019/10/31/16e21046ee1deb15?w=718&h=275&f=png&s=43787)
若需要正常使用这类`Web API`，就需要在全局定义扩展。比如：
```
// Typescript 严格模式
interface ChangeCheckboxEvent extends MouseEvent {
  target: HTMLInputElement
}

function onChangeCheckbox (e: ChangeCheckboxEvent) {
  e.preventDefault()
  const value = e.target.checked
  validateCheckbox(value)
}
```

### 1.2 第三方库也需定义好类型

请注意，如果导入了非`Typescript`库，这也会引发错误，因为导入的库的类型是`any`。
```
// Typescript 严格模式
import { Vector } from 'sylvester'
//                  ❌ ^^^^^^^^^^^
//                     Could not find a declaration file 
//                     for module 'sylvester'.
//                     'sylvester' implicitly has an 'any' type. 
//                     Try `npm install @types/sylvester` 
//                     if it exists or add a new declaration (.d.ts)
//                     file containing `declare module 'sylvester';`
//                     ts(7016)
```
这可能是项目重构`Typescript`版的一大麻烦，需要专门定义第三方库接口类型

## 3. `noImplicitThis`

此规则不允许`this`上下文隐式定义。请看以下示例：
```
// Javascript/Typescript 非严格模式
function uppercaseLabel () {
  return this.label.toUpperCase()
}

const config = {
  label: 'foo-config',
  uppercaseLabel
}

config.uppercaseLabel()
// FOO-CONFIG
```
在非严格模式下，`this`指向`config`对象。`this.label`只需检索`config.label`。

但是，**`this`在函数上进行引用可能是不明确的**：
```
// Typescript严格模式
function uppercaseLabel () {
  return this.label.toUpperCase()
  //  ❌ ^^^^
  //     'this' implicitly has type 'any' 
  //     because it does not have a type annotation. ts(2683)
}
```

如果单独执行`this.label.toUpperCase()`，则会因为`this`上下文`config`不再存在而报错，因为`label`未定义。

解决该问题的一种方法是避免`this`在没有上下文的情况下使用函数：
```
// Typescript严格模式
const config = {
  label: 'foo-config',
  uppercaseLabel () {
    return this.label.toUpperCase()
  }
}
```
更好的方法是编写接口，定义所有类型，而不是`Typescript`来推断：
```
// Typescript严格模式
interface MyConfig {
  label: string
  uppercaseLabel: (params: void) => string
}

const config: MyConfig = {
  label: 'foo-config',
  uppercaseLabel () {
    return this.label.toUpperCase()
  }
}
```

## 4. `strictNullChecks`

此规则不允许出现`null`或`undefined`的可能性。请看以下示例：
```
// Typescript 非严格模式
function getArticleById (articles: Article[], id: string) {
  const article = articles.find(article => article.id === id)
  return article.meta
}
```
`Typescript` 非严格模式下，这样写不会有任何问题。但严格模式会非给你搞出点幺蛾子：

**“你这样不行，万一`find`没有匹配到任何值呢？”：**
```
// Typescript严格模式
function getArticleById (articles: Article[], id: string) {
  const article = articles.find(article => article.id === id)
  return article.meta
  //  ❌ ^^^^^^^
  //     Object is possibly 'undefined'. ts(2532)
}
```
**“我星星你个星星！”**

于是你会将改成以下模样：
```
// Typescript严格模式
function getArticleById (articles: Article[], id: string) {
  const article = articles.find(article => article.id === id)
  if (typeof article === 'undefined') {
    throw new Error(`Could not find an article with id: ${id}.`)
  }

  return article.meta
}
```
**“真香！”**

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8hm33qf1xj308a083td0.jpg)


## 5. `strictPropertyInitialization`

此规则将验证构造函数内部初始化前后已定义的属性。

必须要确保每个实例的属性都有初始值，可以在构造函数里或者属性定义时赋值。

（`strictPropertyInitialization`，这臭长的命名像极了`React`源码里的众多任性属性）

请看以下示例：
```
// Typescript非严格模式
class User {
  username: string;
}

const user = new User();

const username = user.username.toLowerCase();
```
如果启用严格模式，类型检查器将进一步报错：

```
class User {
  username: string;
  //    ❌  ^^^^^^
  //     Property 'username' has no initializer
  //     and is not definitely assigned in the constructor
}

const user = new User();
/
const username = user.username.toLowerCase();
 //                 ❌         ^^^^^^^^^^^^
//          TypeError: Cannot read property 'toLowerCase' of undefined
```
解决方案有四种。

### 方案＃1：允许`undefined`

为`username`属性定义提供一个`undefined`类型：
```
class User {
  username: string | undefined;
}

const user = new User();
```
`username`属性可以为`string | undefined`类型，但这样写，**需要在使用时确保值为`string`类型**：
```
const username = typeof user.username === "string"
  ? user.username.toLowerCase()
  : "n/a";
```

这也太不`Typescript`了。

### 方案＃2：属性值显式初始化
这个方法有点笨，却挺有效：
```
class User {
  username = "n/a";
}

const user = new User();

// OK
const username = user.username.toLowerCase();
```

### 方案＃3：在构造函数中赋值

**最有用的解决方案是向`username`构造函数添加参数**，然后将其分配给`username`属性。

这样，无论何时`new User()`，都必须提供默认值作为参数：
```
class User {
  username: string;

  constructor(username: string) {
    this.username = username;
  }
}

const user = new User("mariusschulz");

// OK
const username = user.username.toLowerCase();
```
还可以通过`public`修饰符进一步简化：
```
class User {
  constructor(public username: string) {}
}

const user = new User("mariusschulz");

// OK
const username = user.username.toLowerCase();
```

### 方案＃4：显式赋值断言

在某些场景下，属性会被间接地初始化（使用辅助方法或依赖注入库）。

这种情况下，你可以在属性上使用**显式赋值断言**来帮助类型系统识别类型。
```
class User {
  username!: string;

  constructor(username: string) {
    this.initialize(username);
  }

  private initialize(username: string) {
    this.username = username;
  }
}

const user = new User("mariusschulz");

// OK
const username = user.username.toLowerCase();
```
通过向该`username`属性添加一个明确的赋值断言，我们告诉类型检查器：
`username`，即使它自己无法检测到该属性，也可以期望该属性被初始化。


## 6. `strictBindCallApply`

此规则将对 `bind, call, apply` 更严格地检测类型。

啥意思？请看以下示例：
```
// JavaScript
function sum (num1: number, num2: number) {
  return num1 + num2
}

sum.apply(null, [1, 2])
// 3
```

在你不记得参数类型时，非严格模式下不会校验参数类型和数量，运行代码时，`Typescript` 和环境（可能是浏览器）都不会引发错误：
```
// Typescript非严格模式
function sum (num1: number, num2: number) {
  return num1 + num2
}

sum.apply(null, [1, 2, 3])
// 还是...3?
```

而`Typescript`严格模式下，这是不被允许的：
```
// Typescript严格模式
function sum (num1: number, num2: number) {
  return num1 + num2
}

sum.apply(null, [1, 2, 3])
//           ❌ ^^^^^^^^^
//              Argument of type '[number, number, number]' is not 
//              assignable to parameter of type '[number, number]'.
//                Types of property 'length' are incompatible.
//                  Type '3' is not assignable to type '2'. ts(2345)
```
那怎么办？ **`“...”`扩展运算符和`reduce`老友来相救**：
```
// Typescript严格模式
function sum (...args: number[]) {
  return args.reduce<number>((total, num) => total + num, 0)
}

sum.apply(null, [1, 2, 3])
// 6
```

## 7. strictFunctionTypes

该规则将检查并限制函数类型参数是抗变（`contravariantly`）而非双变（`bivariantly`，即协变或抗变）的。

初看，内心OS： “这什么玩意儿？”，这里有篇介绍：
> [协变（covariance）和抗变（contravariance）是什么？](https://www.stephanboyer.com/post/132/what-are-covariance-and-contravariance)

协变和逆变维基上写的很复杂，但是总结起来原理其实就一个。
* **子类型可以隐性的转换为父类型**

说个最容易理解的例子，`int`和`float`两个类型的关系可以写成下面这样。
`int` ≦ `float` ：也就是说`int`是`float`的子类型。

这一更严格的检查应用于除方法或构造函数声明以外的所有函数类型。方法被专门排除在外是为了确保带泛型的类和接口（如 Array ）总体上仍然保持协变。

请看下面这个 `Animal` 是 `Dog` 和 `Cat` 的父类型的例子：
```
declare let f1: (x: Animal) => void;
declare let f2: (x: Dog) => void;
declare let f3: (x: Cat) => void;
f1 = f2;  // 启用 --strictFunctionTypes 时错误
f2 = f1;  // 正确
f2 = f3;  // 错误
```

1. 第一个赋值语句在默认的类型检查模式中是允许的，但是在严格函数类型模式下会被标记错误。 
2. 而严格函数类型模式将它标记为错误，因为它不能 被证明合理。
3. 任何一种模式中，第三个赋值都是错误的，因为它 永远不合理。


用另一种方式来描述这个例子则是，默认类型检查模式中 `T`在类型 `(x: T) => void`是 双变的，但在严格函数类型模式中 `T`是 抗变的：
```
interface Comparer<T> {
    compare: (a: T, b: T) => number;
}

declare let animalComparer: Comparer<Animal>;
declare let dogComparer: Comparer<Dog>;

animalComparer = dogComparer;  // 错误
dogComparer = animalComparer;  // 正确
```

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8hm33a0e0j303k02n74g.jpg)

**写到此处，逼死了一个菜鸡前端。**


## 总结&参考

参考文章：
> 1. [How strict is Typescript’s strict mode?](https://medium.com/swlh/how-strict-is-typescripts-strict-mode-f36a4d1a948a)
> 2. [应该怎么理解编程语言中的协变逆变？](https://www.zhihu.com/question/38861374)
> 3. [TypeScript 严格函数类型](https://www.tslang.cn/docs/release-notes/typescript-2.6.html)

在面试的过程中，常被问到**为什么`Typescript`比`JavaScript`好用？**

从这些严格模式规则，你就可以一窥当中的奥秘，**今日开严格，他日Bug秒甩锅**，噢耶。



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