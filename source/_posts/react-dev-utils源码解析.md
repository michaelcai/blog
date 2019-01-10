---
title: react-dev-utils源码解析（一）
date: 2018-08-06 19:22:01
tags: 
- 源码
- Javacript
- React
categories:
- 源码
---

# 简介

react-dev-utils 是 Facebook 开发的 Creact-React-App 里的一个工具库，里面包含了一系列很有用的工具，对于学习库的编写和学习 Webpack 的使用很有好处。

由于文件较多，会分几篇进行解析。

# 解析

## browsersHelper.js

[browsersHelper.js][browsersHelper.js] 里提供了 defaultBrowsers 对象和 checkBrowsers， printBrowsers 两个函数。

### defaultBrowsers

defaultBrowsers 对象里分别定义了开发环境和生产环境下所支持的浏览器。

### checkBrowsers

checkBrowsers 检查了 package.json 里的浏览器设定

### printBrowsers

printBrowsers 打印支持的浏览器列表

## checkRequiredFiles.js

[checkRequiredFiles.js][checkRequiredFiles.js] 只提供了一个 checkRequiredFiles 方法。

对传入的文件路径列表进行遍历，通过 fs.accessSync 判断文件是否存在，如果都存在则返回 true，否则打印错误并返回 false。

## clearConsole.js

[clearConsole.js][clearConsole.js] 只提供了一个 clearConsole 方法。

通过 ANSI 转义序列输入相应清屏命令。可以参考[ANSI转义序列](https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97)

## crossSpawn.js

直接引用了 cross-spawn 库，用于跨平台。

## formatter.js

[formatter.js][formatter.js] 是 Creact-React-App 自用的一个 ESlint formatter 库，对 Lint 的信息进行处理，比如只打印错误之类的。

[browsersHelper.js]: https://github.com/facebook/create-react-app/blob/next/packages/react-dev-utils/browsersHelper.js
[checkRequiredFiles.js]: https://github.com/facebook/create-react-app/blob/next/packages/react-dev-utils/checkRequiredFiles.js
[clearConsole.js]: https://github.com/facebook/create-react-app/blob/next/packages/react-dev-utils/clearConsole.js
[errorOverlayMiddleware.js]: https://github.com/facebook/create-react-app/blob/next/packages/react-dev-utils/errorOverlayMiddleware.js
[formatter.js]: https://github.com/facebook/create-react-app/blob/next/packages/react-dev-utils/formatter.js