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

- [Electron][electron]
- [NW.js][nw.js]
- [Vuido][vuido]
- [Proton-Natvie][proton-natvie]
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

开发一个 Electron 应用本质上就是开发一个 Web + Node.js 应用，之前使用的单元测试方案，项目构建方案等都可以继续使用。

入门时可以借助 [Fiddle][fiddle] 来暂时跳过环境配置、脚手架选择等问题，快速的进入开发。

### 如何打包

现阶段较为流行的 Electron 打包工具有 [Electron-packager][electron-packager] 和 [Electron-builder][electron-builder]。

后者的封装程度更高，所需要做的事情会少很多，但是如果一旦出现问题会更难定位。选择那个就是取舍问题了。

## Typescript

![img](https://github.com/michaelcai/blog/blob/master/source/_posts/Building-Funny-Cross-Platform-Desktop-Apps-with-Electron/9.png?raw=true)

**"Typescript is a typed superset of Javascript that compiles to plan Javascript."**

### 什么是 Typescript

Typescript 是 Javascript 的超集，除了静态类型检查之外，Typescript 还支持部分 Javascript 后续版本的功能（ES6、ES7、ES2018、ESNEXT 等）。

### Typescript 的优缺点是什么

#### 优点

- 静态类型检查：Typescript 的静态类型检查能给原来的项目带来很大的工程化提升，与 Javascript 需要在运行时才能发现某些语法错误不同， Typescript 能在你编写代码时就能发现相关问题。
- 无缝兼容 Javascript: Typescript 的语法除了类型之外与原生的 Javascript 几乎没差别
- 代码提示： 由静态类型检查带来的与原有相比提升巨大的代码提示，对开发的效率提升非常大
- 重构： 静态类型检查加上单元测试在需要重构代码时，静态类型检查能带来非常大的帮助，前者在编写代码时就能提前发现问题。
- 文档： 而静态类型检查加上单元测试能很大程度的替代文档和注释，并且更加的直观易维护

#### 缺点

Typescript 本质上还是一门预编译语言，现阶段来说无法直接的跑在浏览器上，需要预编译成 Javascript 后才能运行。

当然，如果你需要在本地环境下运行，那么可以借用 [Deno][deno] 和 [Ts-Node][ts-node] 来运行相关的代码，不过前者现阶段只是一个玩具，后者其实本质上等于一个预编译器加 Node.js 运行时环境。

所以导致除非项目有一定大小，不然的话，如果只是编写几个活动页，光是项目构建消耗的时间都比写代码时间长，得不偿失。

另外，如果只用支持 Typescript 的库话那么一切都很美好，但一旦你的框架或者所使用的库对 Typescript 不友好的话，为了解决这种问题，要么当成普通 Javascript 用，要么花大量的时间解决类型问题。

生态也是一大问题，前端很多的工具库（比如围绕 Babel 的一系列 Parser）往往依赖性非常强，不鲁棒，强耦合于 AST 格式，非常脆，一碰就坏，脱离不了 Babel，要在 Typescript 中使用这样的库往往是件麻烦事。

### Typescript 使用

Typescript 的开发体验其实跟 Javascript 开发区别不是非常大，入门不难，影响开发的就跟上面所说的优缺点差不多。要享受类型检查带来的美好就必须要花时间编写类型，还必须要花时间解决工具链之间的坑。

当然你也可以用 Typescript 来实现更易理解的依赖注入、控制反转、反射、注解等。

## Funny

下面介绍一下用途有限但是很十分有趣的可以通过 Electron 做跨平台应用，或者已经使用了 Electron 的应用。

### 前置知识

在介绍应用之前需要简单的介绍一些除了 Electron 之外用到的技术或者工具。

#### LLVM （Low Level Virtual Machine）

LLVM 是一个自由软件项目，它是一种编译器基础设施，以 C++ 写成。它是为了任意一种编程语言而写成的程序，利用虚拟技术创造出编译时期、链接时期、运行时期以及“闲置时期”的最优化。

它最早以 C/C++ 为实现对象，而当前它已支持包括 ActionScript、Ada、D 语言、Fortran、GLSL、Haskell、Java 字节码、Objective-C、Swift、Python、Ruby、Rust、Scala以及 C# 等语言。

现今 LLVM 已单纯成为一个品牌，适用于LLVM下的所有项目，包含 LLVM 中介码（LLVM IR）、LLVM 调试工具、LLVM C++ 标准库等。

#### [Emscripten][emscripten]

Emscripten 是一个 LLVM to Javascript 编译器，可以将由 C/C++ 编写的 LLVM 编译成 Javacript 然后运行在浏览器上或者任何 Javascript 能运行的地方。

之前的 Emscripten 编译器最后导出的格式是 ASM.js，后期 Webassembly 的出现，因为 Webassembly 更强的性能和可能性， Webassembly 逐渐成为 Emscripten 导出的主要格式。

#### [ASM.js][asm.js]

ASM.js 是一个中间语言，包括一个 JavaScript 的严格子集，其中的代码采用具有手动内存管理的静态类型语言（即无 GC）编写，代码使用一个源代码至源代码编译器（例如基于LLVM的Emscripten）编译。

通过将语言特性限制在适合提前优化和其他性能改进的范围内，性能得到了提高。

#### [WebAssembly][webassembly-guide]

WebAssembly 或者 wasm 是一个可移植、体积小、加载快并且兼容 Web 的全新格式。

和 Javascript 需要解释执行不同的是，WebAssembly 字节码和底层机器码很相似可快速装载运行，因此性能相对于 JS 解释执行大大提升。

也就是说 WebAssembly 并不是一门编程语言，而是一份字节码标准，需要用高级编程语言编译出字节码放到 WebAssembly 虚拟机中才能运行。

### Funny Application

#### [DOS Game][dos-game]

[DOS Game][dos-game] 是一个集合了所有能在浏览器上运行的 DOS 中文游戏的项目，依赖了 [EM-DOSBox][em-dosbox] 和 [Emularity][emularity]。

Emularity 是一个浏览器内模拟器的加载器，用于简化模拟器的使用。

EM-DOSbox 是一款基于 DOSBox 模拟器通过 Emscripten 编译后移植的模拟器，专门为了运行老游戏而生。

#### [Windows95][windows95]

Windows95 是一个把 Windows95 系统移植到浏览器上，然后用 Electron 套了一层壳运行在各个平台的项目。

除了 Electron 之外，项目依赖了一个名为 [V86][v86] 的项目，V86 是一个使用 Javascript 编写的 x86 虚拟机，模拟了 x86 兼容的 CPU 和相关硬件，使得 x86 的系统能够运行在 Web 和 Node.js 环境。

V86 现在没有使用 WASM 编写，因为 WASM 仍旧处于开发阶段，还不是十分稳定。

当然，这是个玩具，本质上用途有限。

[dos-game]: https://github.com/rwv/chinese-dos-games-web
[em-dosbox]: https://github.com/dreamlayers/em-dosbox
[emularity]: https://github.com/db48x/emularity
[windows95]: https://github.com/felixrieseberg/windows95
[v86]: https://github.com/copy/v86
[webassembly-guide]: https://webassembly.org/getting-started/developers-guide/
[webassembly-design]: https://github.com/WebAssembly/design
[webassembly-mdn]: https://developer.mozilla.org/zh-CN/docs/WebAssembly
[webassembly-vim]: https://github.com/rhysd/vim.wasm
[webassembly-tanks]: https://webassembly.org/demo/Tanks/
[wasmboy]: https://github.com/torch2424/wasmBoy
[electron: the bad parts]: https://hackernoon.com/electron-the-bad-parts-2b710c491547
[electron]: https://github.com/electron/electron
[nw.js]: https://github.com/nwjs/nw.js
[vuido]: https://github.com/mimecorg/vuido
[proton-natvie]: https://github.com/kusti8/proton-native
[electron-packager]: https://github.com/electron-userland/electron-packager
[electron-builder]: https://github.com/electron-userland/electron-builder
[fiddle]: https://github.com/electron/fiddle
[deno]: https://github.com/denoland/deno
[ts-node]: https://github.com/TypeStrong/ts-node
[asm.js]: http://asmjs.org/
[emscripten]:https://github.com/kripken/emscripten