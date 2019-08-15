## 一. 原以为升级vue-cli3的路线是这样的：

* 创建vue-cli3项目，按原有项目的配置选好各项配置
![选好各项配置](https://user-gold-cdn.xitu.io/2019/1/25/168832663fef4c33?w=1492&h=818&f=png&s=150555)
* 迁移目录
 ```
src->src
static->public
 ```
* 对比新旧 `package.json`，然后`yarn install`，完毕。
![](https://user-gold-cdn.xitu.io/2019/1/25/16883c1f3454502a?w=440&h=440&f=png&s=108845)


### 然鹅... 运行项目，报错`You are using the runtime-only build of Vue......`：

![报错](https://user-gold-cdn.xitu.io/2019/1/25/16883346f5f31786?w=1384&h=898&f=png&s=775662)
![](https://user-gold-cdn.xitu.io/2019/1/25/1688332b6f378f85?w=86&h=76&f=png&s=5808)

然后去查了下旧项目的相关字眼文件：

![](https://user-gold-cdn.xitu.io/2019/1/25/1688333b9883f139?w=4500&h=2552&f=png&s=2894971)

噢，原来是vue-cli3的webpack相关文件都得自己写。于是乎根据官网的指引，在根目录创建了`vue.config.js`

此时粗略配置：
```
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .loader('vue-loader')
      .tap(options => {
        options.compilerOptions.preserveWhitespace = false
        return options
      })
    config.resolve.alias
      .set('vue$', 'vue/dist/vue.esm.js')
      .set('@', resolve('src'))
  }
```
## 二. 此时勉强能跑起来，但后续遇到了这些坑：

### `#1` public 静态资源不加载
    ```
     const CopyWebpackPlugin = require('copy-webpack-plugin')
     // ....
     // 确保静态资源
     config.resolve.extensions = ['.js', '.vue', '.json', '.css']
     config.plugins.push(
      new CopyWebpackPlugin([{ from: 'public/', to: 'public' }]),
    )
    ```

### `#2` Chrome 查看样式时无法找到源文件
![](https://user-gold-cdn.xitu.io/2019/1/25/16883671ac2a98aa?w=796&h=882&f=png&s=421128)
 原因：`vue-cli3` 里默认关闭 `sourceMap`，样式都会被打包到首页。
 解决: 需要自己配置打开
```
  // 让样式找到源
  css: {
    sourceMap: true
  },
```

### `#3` 生产环境的`debuger`和`console`无法通过 `uglifyjs-webpack-plugin` 和         `uglify-es` 剔除

原因：不支持`es6`, 需要配置`babel`(`uglify-es`按配置填会显示不存在选项)

解决：插件[terser](https://github.com/terser-js/terser#output-options)

    ```
    const TerserPlugin = require('terser-webpack-plugin')
    if (process.env.NODE_ENV === 'production') {
      // 为生产环境修改配置...
      new TerserPlugin({
        cache: true,
        parallel: true,
        sourceMap: true, // Must be set to true if using source-maps in production
        terserOptions: {
          compress: {
            drop_console: true,
            drop_debugger: true
          }
        }
      })
    } else {
      // 为开发环境修改配置...
    }
    ```

### `#4` 无法在`config`目录下配置不同环境的`API_URL`，用于跨域请求

原因：[vue-cli3 中需要遵循变量规则，使用`VUE_APP`前缀](https://stackoverflow.com/questions/51764070/configure-environment-specific-variables-using-vue-cli)

官方规则：[在客户端侧代码中使用环境变量](https://cli.vuejs.org/zh/guide/mode-and-env.html#在客户端侧代码中使用环境变量)

解决：于是你需要创建如下几个文件：

![](https://user-gold-cdn.xitu.io/2019/1/25/16883a6f636e0b42?w=1244&h=968&f=png&s=178193)
> `.local`也可以加在指定模式的环境文件上，比如 `.env.development.local`将会在 `development` 模式下被载入，且被 git 忽略。

文件内容：
```
// env.development.local
NODE_ENV = development
VUE_APP_URL = http://xxx.x.xxx/
```

### `#5` vue-cli代理转发控制台反复打印`"WebSocket connection to'ws://localhost..."`

![](https://user-gold-cdn.xitu.io/2019/1/25/16883b1a471e21c4?w=1374&h=746&f=png&s=795181)

解决方法：

`vue.config.js`中配置`devServer.proxy`的`ws`为`false`

结合上述两步，相对应的`vue.config.js`，需要这么写：
```
const env = process.env.NODE_ENV
let target = process.env.VUE_APP_URL

const devProxy = ['/api', '/']  // 代理
// 生成代理配置对象
let proxyObj = {};
devProxy.forEach((value, index) => {
  proxyObj[value] = {
    ws: false,
    target: target,
    // 开启代理：在本地会创建一个虚拟服务端，然后发送请求的数据，并同时接收请求的数据，这样服务端和服务端进行数据的交互就不会有跨域问题
    changeOrigin: true,
    pathRewrite: {
      [`^${value}`]: value
    }
  };
})
// ....
devServer: {
    open: true,
    host: 'localhost',
    port: 8080,
    proxy: proxyObj
  }
```

### 最后贴上我的`vue.config.js`：
```
const CopyWebpackPlugin = require('copy-webpack-plugin')
const TerserPlugin = require('terser-webpack-plugin')

const path = require('path')
const env = process.env.NODE_ENV
let target = process.env.VUE_APP_URL

const devProxy = ['/api', '/']  // 代理

// 生成代理配置对象
let proxyObj = {};
devProxy.forEach((value, index) => {
  proxyObj[value] = {
    ws: false,
    target: target,
    // 开启代理：在本地会创建一个虚拟服务端，然后发送请求的数据，并同时接收请求的数据，这样服务端和服务端进行数据的交互就不会有跨域问题
    changeOrigin: true,
    pathRewrite: {
      [`^${value}`]: value
    }
  };
})

function resolve (dir) {
  return path.join(__dirname, dir)
}

module.exports = {
  publicPath: '/',
  // 让样式找到源
  css: {
    sourceMap: true
  },
  configureWebpack: config => {
    // 确保静态资源
    config.resolve.extensions = ['.js', '.vue', '.json', '.css']
    config.plugins.push(
      new CopyWebpackPlugin([{ from: 'public/', to: 'public' }]),
    )
    if (process.env.NODE_ENV === 'production') {
      // 为生产环境修改配置...
      new TerserPlugin({
        cache: true,
        parallel: true,
        sourceMap: true, // Must be set to true if using source-maps in production
        terserOptions: {
          compress: {
            drop_console: true,
            drop_debugger: true
          }
        }
      })
    } else {
      // 为开发环境修改配置...
    }

  },
  chainWebpack: config => {
    config.module
      .rule('vue')
      .use('vue-loader')
      .loader('vue-loader')
      .tap(options => {
        options.compilerOptions.preserveWhitespace = false
        return options
      })
    config.resolve.alias
      .set('vue$', 'vue/dist/vue.esm.js')
      .set('@', resolve('src'))
  },
  devServer: {
    open: true,
    host: 'localhost',
    port: 8080,
    proxy: proxyObj
  }
}
```

## 三. Eslint相关报错及配置

![](https://user-gold-cdn.xitu.io/2019/1/25/16883d48702ec988?w=1176&h=477&f=png&s=61425)
```
module.exports = {
  root: true,
  env: {
    node: true
  },
  'extends': [
    'plugin:vue/essential',
    '@vue/standard'
  ],
  rules: {
    'generator-star-spacing': 'off',
    'object-curly-spacing': 'off',
    // 最常出现的错误
    'no-unused-vars': 'off',
    // 最常出现的错误
    "vue/no-use-v-if-with-v-for": ["error", {
      "allowUsingIterationVar": true
    }],
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off'
  },
  parserOptions: {
    parser: 'babel-eslint'
  }
}
```

### 最后的最后，跑个项目


`yarn serve`
![](https://user-gold-cdn.xitu.io/2019/1/25/16883c41699c2b80?w=3592&h=1124&f=png&s=1112582)
`yarn build`
![](https://user-gold-cdn.xitu.io/2019/1/25/16883c445508a579?w=3324&h=548&f=png&s=954824)
![](https://user-gold-cdn.xitu.io/2019/1/25/16883c6a7e1418eb?w=459&h=448&f=png&s=362850)


### 作者文章总集

* [「从源码中学习」彻底理解Vue选项Props](https://juejin.im/post/5c88e669f265da2d8f47792a)
* [「Vue实践」项目升级vue-cli3的正确姿势](https://juejin.im/post/5c4a83e36fb9a049b13e91ba)
* [「从源码中学习」Vue源码中的JS骚操作](https://juejin.im/post/5c73554cf265da2de33f2a32)
* [为何你始终理解不了JavaScript作用域链？](https://juejin.im/editor/posts/5c8efeb1e51d45614372addd)