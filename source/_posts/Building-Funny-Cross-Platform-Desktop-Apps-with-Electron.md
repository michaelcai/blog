---
title: Building Funny Cross-Platform Desktop Apps with Electron
date: 2018-12-05 16:36:01
tags:
---

# Building Funny Cross-Platform Desktop Apps with Electron

## Why

### 为什么要跨平台

**“我们需要一个这样的应用”**

通常来说，只有在同时满足以下几点条件的时候，我们才需要跨平台：

1. 多平台原生应用。
2. **钱**不够。
3. **人**不够。
4. 应用的**重要性**不够。

如果不需要多平台，那自然也不需要跨平台。如果这个应用重要性非常高，值得投入大量的人力物力，每一个平台一个单独的开发团队，自然也不需要跨平台。但现实情况往往与此相反。

### 跨平台的难点是什么

**“Write once, run anywhere”**

> “一次编写，到处运行”是 Sun 公司（已被甲骨文收购）在推广 Java 语言的跨平台特性时的口号，这意味着 Java 可以在任何设备上开发，然后编译成一段标准的字节码，就可以在任何安装有 Java 虚拟机（JVM）的设备上运行。但理想是丰满的，现实是骨感的，由于 JVM 在不同平台上的实现不尽相同，所以一种应用可能需要在许多平台上进行测试才能确保正确性和稳定性，这造就了一个程序员间的笑话：“一次编译，到处调试”（“Write Once, Debug Everywhere”）。

跨平台的难点主要有以下几点：

- 平台之间差异大
- 平台持续更新迭代
- 逻辑复杂，代码量大

所有的跨平台的技术方案都大同小异，不外乎是在每一个平台上写一个**运行时环境**，通过这个运行时来掩盖各个平台之间的差异，但是和 JVM 所遇到的问题一样，平台之间的差异太大，运行时并不能完全的掩盖住平台之间的差异，而平台自身的更新迭代更加加剧了这个问题。

### 为什么选 Electron

事实上来说现在流行的跨平台桌面应用开发框架不外乎以下几种：

- [Electron][Electron]
- [NW.js][NW.js]
- [Vuido][Vuido]
- [Proton-Natvie][Proton-Natvie]
- Qt5

Electron 和 NW.js 实现很类似，各有优劣，但 Electron 的生态要更强，这是很重要的一点。而后面的 Vuido 和 Proton-Native 实现类似，和 Electron 选择 Chromium 作为渲染层不同，这两个选择了和 React-Native 相似的方式，自己实现了一个渲染线程来抛弃 Chromium 的包袱，但与此同时也抛弃了 Chromium 的一些优点，渲染层的效果完全依赖于作者的实现，和 Chromium 这种自然没法比，而且生态现在来看几乎没有。而 Qt5 需要结合 C++ 或者 Python 之类的使用，如果没学过相关的语言，会增加额外的成本。

综合来说， Electron 的生态最好，学习成本也较低。生态是一个很重要的问题，生态决定了一个框架能不能活下去，活不下去的框架理念再好都没意义。

但 Electron 不是没有缺点，体积太大、占用内存太高、冗余的运行时消耗、无代码保护等问题的存在都说明它不是一个完美的解决方案，不过现阶段而言，它是最好的解决方案之一。

## What

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/1.jpg?raw=true)

**"Electron is a library you can use to build desktop applications with JavaScript, HTML and CSS. "**

### Electron 的底层原理

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/2.png?raw=true)

Electron 由以下三部分组成：

- Chromium 作为渲染层
- Node.js 作为文件层和网络层
- 三个系统的本地 API

Chromium 作为渲染层无缝接入 HTML + CSS + Javascript 生态，庞大的浏览器生态的支持能省掉不少的烦恼，当然带来的副作用也不少，这里就不一一叙述。

同样的，使用现成的 Node.js 作为文件和网络层也能省掉不少的麻烦。前两者由于都已有开源社区维护，所以三个平台之间的兼容不需要额外的劳动，唯一需要单独做兼容的就是三个系统的本地 API。

### Electron 应用的组成

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/3.png?raw=true)

Electron 的应用有两个主要的线程：主线程和渲染线程，主线程隐于幕后掌控全局，每一个渲染线程之间都是独立的，互相之间需要通过主线程通信。

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/6.png?raw=true)

两者间组成就像一个浏览器一样，渲染线程就像浏览器的每一个 Tab, 主线程就是浏览器自身。

#### 主线程

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/4.png?raw=true)

主线程的文件名通常是 **main.js** 或 **app.js**，是一个 Electron 应用的唯一入口。主线程负责控制了整个程序的生命周期，在这里编写程序打开、关闭、异常处理等逻辑。

除此之外，还负责调用本地资源，比如调用本地的对话框、文件框等，还有渲染进程之间的通信都是通过主线程进行的。

#### 渲染线程

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/5.png?raw=true)

渲染线程跟普通的浏览器页面基本没有区别，可以跟写普通的页面一样的使用一个渲染线程。除此之外，相比于普通的浏览器页面，你可以在 Electron 渲染线程里自由的调用 Node.js 的功能。

#### 线程间通信

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/7.png?raw=true)

主线程和渲染线程之间通过发送 IPC 消息通信，而渲染线程和渲染线程之间需要通过主线程作为中转点通信。

## How To

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/8.png?raw=true)

> Developing with Electron is like building web pages that you can seamlessly use Node in—or building a Node app in which you can build an interface with HTML and CSS.

### 如何开发

借助于 Chromium， Electron 的开发就像开发一个能够无缝使用 Node.js 的网页一样。除了要克服因为平台差异带来的坑之外， Electron 的开发对于前端开发人员来说应该是无压力入门的。

开发一个 Electron 应用本质上就是开发一个 Web + Node.js  应用，之前使用的单元测试方案，项目构建方案等都可以继续使用。

入门时可以借助 [Fiddle][Fiddle] 来暂时跳过环境配置、脚手架选择等问题，快速的进入开发。

### 如何打包

现阶段较为流行的 Electron 打包工具有 [Electron-packager][Electron-packager] 和 [Electron-builder][Electron-builder]。

后者的封装程度更高，所需要做的事情会少很多，但是如果一旦出现问题会更难定位。选择那个就是取舍问题了。

## Typescript

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/9.png?raw=true)

**"Typescript is a typed superset of Javascript that compiles to plan Javascript."**

### 什么是 Typescript

Typescript 是 Javascript 的超集，除了静态类型检查之外，Typescript 还支持部分 Javascript 后续版本的功能。

### Typescript 的优缺点是什么

### Typescript 开发

## Funny

[dos-game]: https://github.com/rwv/chinese-dos-games-web
[em-dosbox]: https://github.com/dreamlayers/em-dosbox
[emularity]: https://github.com/db48x/emularity
[v86]: https://github.com/copy/v86
[webassembly-guide]: https://webassembly.org/getting-started/developers-guide/
[webassembly-design]: https://github.com/WebAssembly/design
[webassembly-mdn]: https://developer.mozilla.org/zh-CN/docs/WebAssembly
[webassembly-vim]: https://github.com/rhysd/vim.wasm
[webassembly-tanks]:https://webassembly.org/demo/Tanks/
[WasmBoy]: https://github.com/torch2424/wasmBoy
[Electron: The Bad Parts]: https://hackernoon.com/electron-the-bad-parts-2b710c491547
[Electron]: https://github.com/electron/electron
[NW.js]: https://github.com/nwjs/nw.js
[Vuido]: https://github.com/mimecorg/vuido
[Proton-Natvie]: https://github.com/kusti8/proton-native
[Electron-packager]: https://github.com/electron-userland/electron-packager
[Electron-builder]: https://github.com/electron-userland/electron-builder
[Fiddle]: https://github.com/electron/fiddle