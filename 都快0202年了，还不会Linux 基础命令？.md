## 前言
程序员：“我要跑路了，告诉我命令行是“`rm -rf /*`”的那个人你小心点。”

**“`rm -rf`” 引发的血案都在菜鸟程序员中经常出现，初初入行的前后端们基础不扎实。**
![](https://user-gold-cdn.xitu.io/2019/11/19/16e82903ad18c22e?w=510&h=389&f=png&s=139565)

容易出现没有图形用户界面 (`GUI`)就无从下手，连部署应用都不会的尴尬局面。

**窃以为，熟悉掌握`Linux`下的`Vim`和常用的命令是每个程序员的必修课。**

而且，连微软都拥抱`Linux terminal`，推出了`Windows Terminal`。你还有 什么理由不学？


![](https://user-gold-cdn.xitu.io/2019/11/19/16e81d75e3282abb?w=1024&h=668&f=png&s=919913)


## 1. `grep`：查找文件中的关键字

```
$ grep "string" [选项] file
```
使用`grep`命令查找文件中的所有`React`关键字:

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ecabcc8568c8?w=890&h=334&f=png&s=158856)


* `-i`选项可以在文件中不区分大小写地搜索字符串。它匹配"`REACT`"，"`REact`"和"`react`"等词。
    ```
    $ grep -i "REact" file
    ```
* `-c (count)`选项，可以找到给定字符串/模式匹配的行数
    ```
    $ grep -c "react" index.js
    ```
    

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ed01aa5f82d8?w=782&h=190&f=png&s=48454)

更多的选项可以查看下图：

![](https://user-gold-cdn.xitu.io/2019/11/19/16e8298dfe48b851?w=2014&h=1260&f=jpeg&s=452134)

## 2. `ls`：列出当前路径中的文件和目录。
```
$ ls
```
`ls`列出当前路径中的文件和目录。

* 如果为文件，则显示成蓝色。
* 如果为文件夹，则显示成灰色

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ed2ed7ed332e?w=1124&h=244&f=png&s=89922)


## 3. `pwd`: 显示工作目录
```
$ pwd
```

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ed48dcc7a326?w=874&h=178&f=png&s=28544)

## 4. `cat`：观看文件的内容

```
$ cat somefile.js
```

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ed5f5de5be37?w=1072&h=700&f=png&s=82606)
cat主要有三大功能：

1. 一次显示整个文件。
  ```
 $ cat filename
  ```
 ```
2. 从键盘创建一个文件。
 ```
  $ cat > filename
  ```
  **只能创建新文件,不能编辑已有文件。**
3. 将几个文件合并为一个文件。
  ```
   $cat file1 file2 > file
   ```
   
以下例子，将`index.js`拷贝一份为`index2.js`
![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ed95ad94a766?w=1148&h=778&f=png&s=128196)

## 5. `echo`：字符串的输出

   ```
$ echo "some text"
```

这是一个内置命令，主要用于Shell脚本和批处理文件中，以将状态文本输出到屏幕或文件。

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ee2bc9743e5a?w=1044&h=180&f=png&s=31414)

## 6. ` touch`：创建文件
```
$ touch somefile
```
`touch`命令用于创建没有任何内容的文件。

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ee5d32d83e86?w=764&h=200&f=png&s=35838)
请注意，在上图中，我们用`touch`创建文件和`cat`查看文件内部的。由于新创建的`index2.js`文件为空，因此`cat`不返回任何内容。

以下是`cat`和`touch`之间的主要区别：

* `cat`，用于创建包含内容的文件。
* `touch`，创建一个没有任何内容的空文件。

## 7. `mkdir`：创建一个新的空目录

```
$ mkdir some-directory
```
`mkdir`在当前路径中创建一个新的空目录

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7eec4a7d26682?w=1092&h=364&f=png&s=158891)


## 8.`rm`：删除文件/目录
```
$ rm [选项] someFile
```
`rm`命令用于删除一个文件或者目录。

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7eeeec2a21a48?w=1074&h=496&f=png&s=209748)

选项：
* `-i` 删除前逐一询问确认。
* `-f` 即使原档案属性设为唯读，亦直接删除，无需逐一确认。
* `-r` 将目录及以下之档案亦逐一删除。

### 8.1 `rmdir`：删除空目录
```
$ rmdir some-directory
```

如果目录中没有内容，该命令将删除目录。否则返回提示`xxx not empty`：

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ef3888050e11?w=940&h=418&f=png&s=132056)

## 9. `tail`：查看文档的内容
```
$ tail [选项] somefile
```
**默认显示文档的最后 10 行**

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ef6dfc96f9cf?w=1402&h=892&f=png&s=321982)

几个常用的参数:

* `-f`，循环读取。
```
  tail -f notes.log
  ```
  此命令显示 `notes.log` 文件的最后 10 行。当将某些行添加至 `notes.log` 文件时，`tail` 命令会继续显示这些行。 显示一直继续，直到您按下（`Ctrl-C`）组合键停止显示。
* `+`，从xx行到结尾
  ```
  tail +20 notes.log
  ```
  显示文件 `notes.log` 的内容，从第 20 行至文件末尾。
* `-c `，最后xx行。
  ```
   tail -c 10 notes.log
   ```
   显示文件 `notes.log`的最后 10 个字符:
   
`tail`命令在查看崩溃报告或以前的历史记录日志时很有用:
   ```
# tail /var/log/messages
Mar 20 12:42:22 hameda1d1c dhclient[4334]: DHCPREQUEST on eth0 to 255.255.255.255 port 67 (xid=0x280436dd)
Mar 20 12:42:24 hameda1d1c avahi-daemon[2027]: Registering new address record for fe80::4639:c4ff:fe53:4908 on eth0.*.
Mar 20 12:42:28 hameda1d1c dhclient[4334]: DHCPREQUEST on eth0 to 255.255.255.255 port 67 (xid=0x280436dd)
Mar 20 12:42:28 hameda1d1c dhclient[4334]: DHCPACK from 10.76.198.1 (xid=0x280436dd)
Mar 20 12:42:30 hameda1d1c avahi-daemon[2027]: Joining mDNS multicast group on interface eth0.IPv4 with address 10.76.199.87.
Mar 20 12:42:30 hameda1d1c avahi-daemon[2027]: New relevant interface eth0.IPv4 for mDNS.
Mar 20 12:42:30 hameda1d1c avahi-daemon[2027]: Registering new address record for 10.76.199.87 on eth0.IPv4.
Mar 20 12:42:30 hameda1d1c NET[4385]: /sbin/dhclient-script : updated /etc/resolv.conf
Mar 20 12:42:30 hameda1d1c dhclient[4334]: bound to 10.76.199.87 -- renewal in 74685 seconds.
Mar 20 12:45:39 hameda1d1c kernel: usb 3-7: USB disconnect, device number 2
```


## 10. `find`:搜索文件
```
$ find path -name filename
```
`find`命令可以快速查找文件或目录。当你正在处理具有数百个文件和多个目录的大型项目时，此功能很有用。

查找所有名为`index.js`的文件:
![](https://user-gold-cdn.xitu.io/2019/11/19/16e7f7c5f2ede87a?w=848&h=900&f=png&s=347652)

查找指定类型的文件：
```
$ find . -name "*.js"
```

![](https://user-gold-cdn.xitu.io/2019/11/19/16e7f7e4685b2130?w=996&h=556&f=png&s=213757)


## 11. `mv`：移动文件
```
$ mv somefile /to/some/other/path
```
该mv命令将文件或目录从一个位置移动到另一个位置。

**支持移动单个文件，多个文件和目录。**

![](https://user-gold-cdn.xitu.io/2019/11/19/16e7f7f971974e8d?w=1180&h=608&f=png&s=278393)
## 12. `wget`：下载文件的工具
```
$ wget someurl
```

`Wget`是一个免费软件包，用于使用`HTTP，HTTPS，FTP和FTPS`（最广泛使用的`Internet`协议）检索文件。

这是一个非交互式的命令行工具，因此可以很容易地从脚本，CRON作业，不支持`X-Windows`的终端等中调用它。


![](https://user-gold-cdn.xitu.io/2019/11/19/16e7f4f47f194606?w=1440&h=1008&f=png&s=635228)

`Wget`具有许多使检索大型文件或镜像整个Web或FTP站点变得容易的功能，包括：

* 可以使用`REST`和`RANGE`恢复中止的下载。
* 可以使用文件名通配符并递归镜像目录
* 基于NLS的消息文件，适用于多种语言
* 可将下载的文档中的绝对链接转换为相对链接，以便下载的文档可以在本地链接。
* 在大多数类似`UNIX`的操作系统以及`Microsoft Windows`上运行
* 支持`HTTP`代理,`cookie` 和持久的`HTTP`连接.
* 无人值守/后台操作。


## 13. `tree`：以树状图列出目录的内容

常在写文档时需要列一下文件目录结构，这个时候`tree`命令就能帮个忙了。某些`Linux`、`macOS`上没有`tree`命令，需要执行安装：
1. 先确保安装了`Homebrew`，若没有则执行：
```
   /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```
2. 安装`tree`命令
   ```
  brew install tree
  ```

效果：
  ```
(base) xxx$ tree
.
├── djangoStudy
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

1 directory, 5 files
```

## 14. `|`：管道命令

通常情况下，我们在终端只能执行一条命令，然后按下回车执行，那么如何执行多条命令呢？


* 顺序执行多条命令：`command1;command2;command3;`
  简单的顺序指令可以通过 `;`来实现。
* 有条件的执行多条命令：`which command1 && command2 || command3`
  * `&&` : 如果前一条命令执行成功则执行下一条命令，和`JavaScript`中用法一致
  * || :与`&&`命令相反，执行不成功时执行下一个。
* `$?`: 存储上一次命令的返回结果
```
// 栗子：
$ which git>/dev/null && git --help  // 如果存在git命令，执行git --help命令
$ echo $? 
```
而管道命令则可以衔接各种命令的输出输入，使得连锁操作变得简单。
> 管道是一种通信机制，通常用于进程间的通信（也可通过socket进行网络通信），它表现出来的形式将前面每一个进程的输出（stdout）直接作为下一个进程的输入（stdin）

![](https://user-gold-cdn.xitu.io/2019/11/19/16e820845bbd0e6f?w=698&h=203&f=png&s=14317)
```
 $ 指令1 | 指令2 | …
```
管道命令的注意事项：
* 只能处理前一条指令的正确输出，不能处理错误输出；
* 后一条指令，必须能够接收标准输入流命令才能执行。

例子：
1、分页显示 `/etc` 目录 中内容的详细信息
```
$ ls -l /etc | more
```

2、将一个字符串输入到一个文件中
```
$ echo "Hello World" | cat > hello.txt
```

## 后记 & 引用

> * [Here Are 11 Console Commands Every Developer Should Know](https://medium.com/better-programming/here-are-11-console-commands-every-developer-should-know-54e348ef22fa)
> * [Linux管道命令（pipe）](https://www.jianshu.com/p/9c0c2b57cb73)
> * [MacOS上使用tree命令](https://www.jianshu.com/p/f540e8b6e53f)
附赠一张强大无比的`Linux`命令表

![](https://user-gold-cdn.xitu.io/2019/11/19/16e829fca12cbc07?w=1680&h=2980&f=png&s=913641)
## ❤️ 看完三件事
如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章
* [Chrome Devtools 高级调试指南（新）](https://juejin.im/post/5d9eea84e51d4577eb5d8510)
* [JavaScript 工具函数大全（新）](https://juejin.im/post/5da1a04ae51d45783d6122bf)
* [「React Hooks」120行代码实现一个交互完整的拖拽上传组件](https://juejin.im/post/5d674313e51d4561c94b1000)
* [「React Hooks」160行代码实现动态炫酷的可视化图表 - 排行榜](https://juejin.im/post/5d565015f265da03eb13c575)
![](https://user-gold-cdn.xitu.io/2019/8/5/16c5faffbefaea2e?w=2006&h=1014&f=png&s=672314)

也可以来我的`GitHub`博客里拿所有文章的源文件：

**前端劝退指南**：https://github.com/roger-hiro/BlogFN
```

```

## 后记 & 引用

> * [Here Are 11 Console Commands Every Developer Should Know](https://medium.com/better-programming/here-are-11-console-commands-every-developer-should-know-54e348ef22fa)
> * [Linux管道命令（pipe）](https://www.jianshu.com/p/9c0c2b57cb73)
> * [MacOS上使用tree命令](https://www.jianshu.com/p/f540e8b6e53f)
附赠一张强大无比的`Linux`命令表

![](https://user-gold-cdn.xitu.io/2019/11/19/16e829fca12cbc07?w=1680&h=2980&f=png&s=913641)
## ❤️ 看完三件事
如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：

1. 点赞，让更多的人也能看到这篇内容（收藏不点赞，都是耍流氓 -_-）
2. 关注公众号「前端劝退师」，不定期分享原创知识。
3. 也看看其它文章
* [Chrome Devtools 高级调试指南（新）](https://juejin.im/post/5d9eea84e51d4577eb5d8510)
* [JavaScript 工具函数大全（新）](https://juejin.im/post/5da1a04ae51d45783d6122bf)
* [「React Hooks」120行代码实现一个交互完整的拖拽上传组件](https://juejin.im/post/5d674313e51d4561c94b1000)
* [「React Hooks」160行代码实现动态炫酷的可视化图表 - 排行榜](https://juejin.im/post/5d565015f265da03eb13c575)
![](https://user-gold-cdn.xitu.io/2019/8/5/16c5faffbefaea2e?w=2006&h=1014&f=png&s=672314)

也可以来我的`GitHub`博客里拿所有文章的源文件：

**前端劝退指南**：https://github.com/roger-hiro/BlogFN
```