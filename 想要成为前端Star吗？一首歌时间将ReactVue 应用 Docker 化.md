## 前言

以前一直有疑问困扰着我：人人都在吹的`Docker`容器化，与前端有何关系？

然而在近两年的编程生涯，在每一次产品迭代中，渐渐体会到了容器化其魅力所在。

应用部署从刀耕火种，到`DevOps`崛起，原来不止前端在迅捷发展。接下来，我将用一首歌的时间，带大家真实的体验一番`Docker`容器化。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe9aea1ea64a4f589f1e3a73b90c6e8e~tplv-k3u1fbpfcp-zoom-1.image)


## 1. 朴素的`Dockerfile`

首先准备一个有标准运行指令的`Web`应用，用脚手架`creat-react-app`或`Vue CLI`等生成的即可。



以下的`Dockerfile`不参杂其它依赖，争取做到都能看懂：
```
# 指定Node版本
FROM node:12.18.3

# 容器中应用程序的路径。将Web目录作为工作目录
WORKDIR /web

# 将package.json 复制到 Docker 环境
COPY ./package.json /web/package.json

# 安装依赖
RUN yarn

# 将代码复制到Docker容器中的Web目录 
COPY . /web/

# 暴露容器内部访问端口，根据项目变动
EXPOSE 8080

## 如果是Vue CLi，则换成 yarn serve
CMD ["npm", "start"]
```

是的，开发环境在`Docker` 部署，关键配置就那么几行。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/722942f47dcd4e6c85bc452f40de5a77~tplv-k3u1fbpfcp-zoom-1.image)

此外，还需要添加一个`.dockerignore`文件，加快构建过程的速度
```
node_modules/**/*
build/**/*
.DS_Store
```

## 2. 为应用构建`Docker`镜像

首先确认你的`Dcoker` 正在运行。
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f4d6b6004f54837adaf177f6c4ba38c~tplv-k3u1fbpfcp-zoom-1.image)

运行以下命令来构建`Docker`映像。`react-docker` 可以替换为你要为镜像命名的任何值。

```
docker build -t react-docker .

```
其中`-t` 为打标签的意思，执行完后将会看到：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d84ce35f6f446de8e1906296e08cfa2~tplv-k3u1fbpfcp-zoom-1.image)

```
Successfully built 137c69857dd0
Successfully tagged react-docker:latest
```

您的镜像已经嗷嗷待发。

## 3. 运行`Docker` + `React/Vue App`

现在，使用以下`docker run`命令, 通过`Docker`在端口`3000`上运行`React`应用。

```
docker run -p 3000:3000 react-docker
```
其中：前一个3000对应本机`http://localhost:3000/`，第二个3000则是`Docker`容器端口。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/970a2f96130749138988d742a9d84e28~tplv-k3u1fbpfcp-zoom-1.image)

可以通过`Dcoker ps`查看容器信息
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d23d9a1cff9e4cf08eb341f95241418a~tplv-k3u1fbpfcp-zoom-1.image)

在`Docker`的`Dashboard`中也可以看到：
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c7ccae3c64d4e4d9baddec393c0983e~tplv-k3u1fbpfcp-zoom-1.image)

此时打开`http://localhost:3000/`就会看到熟悉又亲切的画面

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d49d808ef4547a3b2250182cf95434b~tplv-k3u1fbpfcp-zoom-1.image)

到这里，你的一首歌的时间之`Docker`之旅就结束了。接下来的将是更标准化的流程，劝退劝退！

## 4. 使用`Docker Compose`标准化流程

将`docker-compose.yml`文件添加到项目根目录：

```
version: '3.7'

services:

  sample:
    container_name: sample
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - '.:/app'
      - '/app/node_modules'
    ports:
      - 3000:3000
    environment:
      - CHOKIDAR_USEPOLLING=true
```

有了该文件，就不需要分步执行了，直接：
```
docker-compose up -d --build
```
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42781692c3444fb79b73223076120f5c~tplv-k3u1fbpfcp-zoom-1.image)
就能看到一样构建了：
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a586e65efb73416089f8441892e27b43~tplv-k3u1fbpfcp-zoom-1.image)


## 5. 生产环境下的`Dockerfile`

生产环境下需要`nginx`配置，在根目录先创建`nginx.config`
```
server {
    listen       ${PORT:-80};
    server_name  _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $$uri /index.html;
    }
}
```
让我们创建一个单独的`Dockerfile`，用于生产环境，称为`Dockerfile.prod`：
```
FROM node:12.18.3 AS builder

WORKDIR /app

ENV PATH /app/node_modules/.bin:$PATH
COPY package.json ./
COPY package-lock.json ./

# 前端项目构建命令 — npm ci 或 npm install 
# http://www.gaoxiukun.com/wp/archives/509

RUN npm ci
# React 应用需要react-script
RUN npm install react-scripts@3.4.1 -g

COPY . ./
RUN npm run build

# 安装nginx
FROM nginx:1.17-alpine
RUN apk --no-cache add curl
RUN curl -L https://github.com/a8m/envsubst/releases/download/v1.1.0/envsubst-`uname -s`-`uname -m` -o envsubst && \
    chmod +x envsubst && \
    mv envsubst /usr/local/bin
COPY ./nginx.config /etc/nginx/nginx.template
CMD ["/bin/sh", "-c", "envsubst < /etc/nginx/nginx.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"]

COPY --from=builder /app/build /usr/share/nginx/html
```

因为`Dockerfile.prod`不是默认的执行文件，所以需要构建并标记：
```
docker build -f Dockerfile.prod -t sample:prod .
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5de975c8ba74b578352224f6d02f996~tplv-k3u1fbpfcp-zoom-1.image)


接下来执行`docker run`

```
docker run -it --rm -p 3000:80 sample:prod
```
* `-i`: 以交互模式运行容器。
* `-t`: 为容器重新分配一个伪输入终端，通常与 `-i` 同时使用。
* `--rm`:在容器退出时自动清理容器内部的文件系统，不懂可忽略
* `-p`: 指定端口。

成功运行：
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c19ccd33e1b3405fbe684b507ce2c109~tplv-k3u1fbpfcp-zoom-1.image)

在浏览器中导航到`http://localhost:3000` 以查看该应用程序。

接下来使用新的`Docker Compose`文件以及`docker-compose.prod.yml`进行测试：

```
version: '3.7'

services:
  sample-prod:
    container_name: sample-prod
    build:
      context: .
      dockerfile: Dockerfile.prod
    ports:
      - '3000:80'
```
启动容器：
```
docker-compose -f docker-compose.prod.yml up -d --build
```
在浏览器中再次进行校验。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93211ad9f83d43149c01071410c7a67e~tplv-k3u1fbpfcp-zoom-1.image)


## ❤️ 结语
在以往，我对`Docker`容器化的概念，仅停留在了解。而真正实操中，也是被一群指令，配置给吓到劝退。

本文弱化了命令行参数，希望能让广大萌新们能先看懂，再去演练一番，举一反三，不再怕`Docker`，然后再去学习`k8s`相关。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35da081230aa4fe6996063c8f6d6790d~tplv-k3u1fbpfcp-zoom-1.image)
`Docker` 在接下来的几年里，会逐渐成为开发的标配，希望大家能放下对运维领域的偏见，多多学习这些行业内的新标准与概念。


如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -\_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章



![](https://tva1.sinaimg.cn/large/00831rSTgy1gcy5n9t0lgj30ga08oq5z.jpg)

劝退师个人微信：**huab119**，或公众号留言，我加你们




也可以来我的`GitHub`博客里拿所有文章的源文件：

**前端劝退指南**：https://github.com/roger-hiro/BlogFN
一起玩耍呀。~