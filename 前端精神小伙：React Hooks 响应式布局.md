## 前言

现在稍微大型的站点都会采用`H5/PC`端 并行，通过`nignx`获取浏览器的`UA`信息来切换站点。

但这对于一些企业站点或人手不足的小型项目来说，就很难实现。

通过`CSS`媒体查询实现响应式布局，是主流方式。

但是，有时在React程序中，需要根据屏幕大小有条件地渲染不同的组件（写媒体查询太麻烦了，还不如另写组件），其实使用`React Hooks`，可以更灵活实现。

![](https://user-gold-cdn.xitu.io/2020/3/13/170d37f461e1e35a?w=500&h=270&f=png&s=82573)

本文的实现来自：
> [Developing responsive layouts with React Hooks](https://blog.logrocket.com/developing-responsive-layouts-with-react-hooks/)


## 1. 方案一：`innerWidth`

一个很简单粗略的方案，是个前端都知道：
```
const MyComponent = () => {
  // 当前窗口宽度
  const width = window.innerWidth;
  // 邻介值
  const breakpoint = 620;
  // 宽度小于620时渲染手机组件，反之桌面组件
  return width < breakpoint ? <MobileComponent /> : <DesktopComponent />;
}
```
这个简单的解决方案肯定会起作用。根据用户设备的窗口宽度，我们可以呈现桌面视图或手机视图。

但是，当调整窗口大小时，未解决宽度值的更新问题，可能会渲染错误的组件。

![](https://user-gold-cdn.xitu.io/2020/3/13/170d356306ea9d53?w=200&h=200&f=png&s=1702109)

## 2. 方案二：`Hooks`+`resize`

说着也简单，监听`resize`事件时，触发`useEffect`改变数据。
```
const MyComponent = () => {
  const [width, setWidth] = React.useState(window.innerWidth);
  const breakpoint = 620;

  React.useEffect(() => {
    window.addEventListener("resize", () => setWidth(window.innerWidth));
  }, []);

  return width < breakpoint ? <MobileComponent /> : <DesktopComponent />;
}
```
但精通`Hooks`的你，一定知道这里存在内存性能消耗问题：`resize`事件没移除！

优化版本：
```
const useViewport = () => {
  const [width, setWidth] = React.useState(window.innerWidth);

  React.useEffect(() => {
    const handleWindowResize = () => setWidth(window.innerWidth);
    window.addEventListener("resize", handleWindowResize);
    return () => window.removeEventListener("resize", handleWindowResize);
  }, []);

  return { width };
}
```

![](https://user-gold-cdn.xitu.io/2020/3/13/170d3568ba09d877?w=689&h=548&f=png&s=277703)

## 3. 方案三：构建`useViewport`

自定义`React Hooks`,可以将组件/函数最大程度的复用。构建一个也很简单：

```
const useViewport = () => {
  const [width, setWidth] = React.useState(window.innerWidth);

  React.useEffect(() => {
    const handleWindowResize = () => setWidth(window.innerWidth);
    window.addEventListener("resize", handleWindowResize);
    return () => window.removeEventListener("resize", handleWindowResize);
  }, []);

  return { width };
}
```

精简后的组件代码：
```
const MyComponent = () => {
  const { width } = useViewport();
  const breakpoint = 620;

  return width < breakpoint ? <MobileComponent /> : <DesktopComponent />;
}
```
![](https://user-gold-cdn.xitu.io/2020/3/13/170d35cbee3ef95e?w=50&h=50&f=png&s=4168)


但是这里还有另一个性能问题：

**响应式布局影响的是多个组件，如果在多处使用`useViewport`，这将浪费性能。**


![](https://user-gold-cdn.xitu.io/2020/3/13/170d36293a23fdb8?w=61&h=71&f=png&s=9489)

这时就需要另一个`React`亲儿子：`React Context(上下文)` 来帮忙。

## 4.终极方案：`Hooks`+`Context`

我们将创建一个新的文件`viewportContext`，在其中可以存储当前视口大小的状态以及计算逻辑。

```
const viewportContext = React.createContext({});

const ViewportProvider = ({ children }) => {
  // 顺带监听下高度，备用
  const [width, setWidth] = React.useState(window.innerWidth);
  const [height, setHeight] = React.useState(window.innerHeight);

  const handleWindowResize = () => {
    setWidth(window.innerWidth);
    setHeight(window.innerHeight);
  }

  React.useEffect(() => {
    window.addEventListener("resize", handleWindowResize);
    return () => window.removeEventListener("resize", handleWindowResize);
  }, []);

  return (
    <viewportContext.Provider value={{ width, height }}>
      {children}
    </viewportContext.Provider>
  );
};

const useViewport = () => {
  const { width, height } = React.useContext(viewportContext);
  return { width, height };
}
```

紧接着，你需要在`React`根节点，确保已经包裹住了`App`：
```
const App = () => {
  return (
    <ViewportProvider>
      <AppComponent />
    </ViewportProvider>
  );
}
```
在往后的每次`useViewport()`，其实都只是共享`Hooks`。

```
const MyComponent = () => {
  const { width } = useViewport();
  const breakpoint = 620;

  return width < breakpoint ? <MobileComponent /> : <DesktopComponent />;
}
```
![](https://user-gold-cdn.xitu.io/2020/3/13/170d3740e54570b6?w=427&h=413&f=gif&s=81696)


## 后记

`github`上面的响应式布局`hooks`，都是大同小异的实现方式。

本文除了介绍`React Hooks`的响应式布局实现，还介绍了如何自定义`hooks`与使用`Context`上下文，来复用，以达到性能最佳优化。

## ❤️ 看完三件事
如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章

![](https://user-gold-cdn.xitu.io/2019/11/20/16e86b1de975589f?w=586&h=312&f=png&s=122619)

劝退师个人微信：**huab119**

也可以来我的`GitHub`博客里拿所有文章的源文件：

**前端劝退指南**：https://github.com/roger-hiro/BlogFN
一起玩耍呀。~