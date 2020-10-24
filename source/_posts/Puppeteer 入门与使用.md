---
title: Puppeteer 入门与使用
date: 2019-11-12 10:39:35
tags: [Node, Puppeteer, 无头浏览器]
categories: Node
---
> by zhaowei 2019/12/19
![](https://pptr.dev/images/pptr.png)

> 图片来源：https://pptr.dev/images/pptr.png

[TOC]

# 前言

在学习Puppeteer之前，我们先来了解下什么是Headless Chrome，Headless Chrome能做些什么

Headless是Chrome在17年自行开发的特性，支持以下功能：

1. 在无界面的环境中运行 Chrome
2. 通过命令行或者程序语言操作 Chrome
3. 无需人的干预，运行更稳定
4. 在启动 Chrome 时添加参数 --headless，便可以 headless 模式启动 Chrome

```bash
alias chrome="/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome"

# Mac OS X 命令别名
chrome --headless --remote-debugging-port=9222 --disable-gpu

# 开启远程调试
chrome --headless --disable-gpu --dump-dom https://www.baidu.com

# 获取页面 DOM
chrome --headless --disable-gpu --screenshot https://www.baidu.com
```


# 正文

基于Chrome Handless特性，Chrome推出了Puppeteer node包，是对Chrome的无界面版本以及对其进行操作的js接口封装，通过调用Chrome DevTools开放的接口与Chrome通信，使得我们可以非常方便的操作Chrome。

## Puppeteer 简介

Puppeteer 是一个node库，他提供了一组用来操纵Chrome的API, 通俗来说就是一个 headless chrome浏览器 (当然你也可以配置成有UI的，默认是没有的)。既然是浏览器，那么我们手工可以在浏览器上做的事情 Puppeteer 都能胜任, 另外，Puppeteer 翻译成中文是”木偶”意思，所以听名字就知道，操纵起来很方便，你可以很方便的操纵她去实现：

1. 生成网页截图或者 PDF
2. 高级爬虫，可以爬取大量异步渲染内容的网页
3. 模拟键盘输入、表单自动提交、登录网页等，实现 UI 自动化测试
4. 捕获站点的时间线，以便追踪你的网站，帮助分析网站性能问题

Chrome Headless 特性，意思是在无界面的环境下运行Chrome，可以通过命令行或者程序语言操作Chrome

## Puppeteer 安装

在安装Puppeteer包时，可能会遇到chromium安装失败的情况（原因你懂的），遇到这种情况，我们可以自行下载 chromium 浏览器进行安装

```bash
cnpm install puppeteer-chromium-resolver
```

成功后会显示安装路径，记住这个安装路径，在后面会用到
![](https://global.uban360.com/sfs/file?digest=fidf0a263b4410182e9966687795f732c39&fileType=2)

浏览器安装成功之后，继续安装 puppeteer 时可以通过set PUPPETEER_SKIP_CHROMIUM_DOWNLOAD 跳过下载过程

```bash
npm set PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true
npm install puppeteer
```

不过更推荐安装 puppeteer-core，不会下载 chromium，而且包体积会更小

```bash
npm install puppeteer-core
```

## 玩转 Chrome Handless

puppeteer安装完成之后，我们就可以利用puppeteer 操作 Chrome，详细api可以查看操作文档：[Puppeteer](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.1&show=api-class-puppeteer)

首先打开浏览器并跳转到指定页面，需要用到上面 chromium 的安装路径

```js
const puppeteer = require('puppeteer-core');

const launch_options = {
  // 启动模式： true 无头模式
  headless: false,
  // 可运行 Chromium 或 Chrome 可执行文件的路径，这里就是上面 Chromium 的安装路径
  executablePath:
    "安装路径"
};

(async () => {
  // 开启浏览器
  const browser = await puppeteer.launch(launch_options)
  // 开启一个tab页
  const page = await browser.newPage()
  // 跳转到百度首页
  await page.goto('http:/www.baidu.com')
  // 关闭页面
  await page.close()
  // 关闭浏览器
  await browser.close()
})();
```

我们这里为了查看效果配置了headless为false，表示不使用无头模式，可以用来调试项目，其他关于 browser和page的配置可以查看 [文档](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.1&show=api-class-browser)。

运行之后会打开浏览器并新建tab页，跳转到百度首页，之后会自动关掉

![](https://global.uban360.com/sfs/file?digest=fid5135c500200de8d7661ba6d8fe0acb62&fileType=2)

浏览器运行起来之后我们就可以来做下面这些事情了

### 生成网页截图或者PDF

网页截图在Chrome浏览器中是可以调用命令导出png图片的，f12打开开发面板，ctrl + shift + p打开命令行窗口，输入 screenshot 关键字搜索命令，就可以截取当前页面了

![](https://global.uban360.com/sfs/file?digest=fid05095d8137131f6e5e4ca165bc5e1577&fileType=2)

而通过代码也是很方便的截图

```js
 // 截图
  await page.screenshot({
    path: './baidu.png'
  })
```
运行之后会在当前目录下生成 baidu.png，当然他还支持有页面任意范围的截图，其他详细配置可以查看 [文档](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.1&show=api-pagescreenshotoptions)

而puppeteer导出pdf也非常简单，一行代码搞定

```js
// pdf
await page.pdf({
   path: './baidu.pdf'
})
```

在导出pdf时，可以通过设置css的媒体模式为 screen 优化导出效果，具体可以查看 [文档]()

```js
await Global.page.emulateMediaType('screen'); // 'screen', 'print' 和 null
```

运行完毕当前目录下就会生成 pdf 文件，但要注意的是 puppeteer 导出pdf必须是在无头模式下，即 headless 必须为 true，否则会报错。关于其他导出pdf的配置可以查看 [文档](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.1&show=api-pagepdfoptions)

当然如果是需要导出doc或execl等文件，可以运行 js 脚本进行下载，可以通过设置指定下载目录，保存我们的文件

```js
// 指定文件下载路径
	await Global.page._client.send('Page.setDownloadBehavior', {
		behavior: 'allow',
		downloadPath: "下载路径", // 建议使用绝对路径
	})
```

### 高级爬虫，可以爬取大量异步渲染内容的网页

利用puppeteer可以在不打开浏览器（即无头模式下）对网站中的页面爬虫爬取数据，更多的是拉取页面上的图片、媒体的等各种资源，网上案例很多，可自行查阅。

我们目前项目里面用到的是插入markdown渲染之后的html代码，并插入执行js脚本，主要用到下面几个方法：

插入html文档

```js
// 设置html content
await Global.page.setContent(html, {
	waitUntil: 'domcontentloaded'
})

// 插入以来的css代码段或者文件
await Global.page.addStyleTag({
	path: path.resolve(__dirname, 'assets/markdown.css'),
});
```

向指定dom插入html片段
```js
// 指定位置插入插入 html 片段
await Global.page.$eval('#content', (dom, html) => {
	dom.innerHTML = html
}, html);
```

插入js外部库
```js
await Global.page.addScriptTag({
	path: path.resolve(__dirname, 'lib/waterMask.js'),
});
```

执行js脚本
```js
await Global.page.evaluate((text) => {
	if (window.waterMark) {
		window.waterMark({
			text
		});
	}
}, watermark)
```

需要注意的是 page.evaluate* 等方法都是可以传递外部参数的，插入的js脚本是运行在浏览器端，在访问一些库如jquery等时可能需要显示使用 window 命名空间，否则node可能会校验变量为定义。如果内部返回的是Promise，node端也是可以拿到数据的

### 模拟键盘输入、表单自动提交、登录网页等，实现 UI 自动化测试

模拟键盘输入、表单自动提交、登录网页等，实现 UI 自动化测试说白了就是定位到页面元素，触发dom事件，使用 puppeteer 中 page 的下面方法可以很轻松的做到，具体可自行查阅

```js
page.$(selector)
page.$$(selector)
page.$$eval(selector, pageFunction[, ...args])
page.$eval(selector, pageFunction[, ...args])
page.$x(expression)
```

表单自动提交等案例如下：

```js
// 自动填充表单数据
const name = 'mayanjun@haowumc.com';
await page.type('.text.pristine.untouched', name, {delay: 0});
const pwd = '密码';
await page.type('.text.pristine.untouched', pwd, {delay: 1});

// 触发表单提交
const inputElement = await page.$('input[type=submit]');
await inputElement.click();

// 等待浏览器跳转
await page.waitForNavigation();

console.log(page.url());
await page.goto(url, {
	waitUntil: 'networkidle2'  // 网络空闲说明已加载完毕
});
```

关于 page.$*的各种api，用法jquery差不多，有兴趣可以查看 [文档](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.1&show=api-pageselector)

###	 捕获站点的时间线，以便追踪你的网站，帮助分析网站性能问题

chrome自身带有监控工具，可以查看收集到各种资源请求的结果、耗时操作以及错误警告等信息，通过 puppeteer 的 Events api可以监听到页面中的很多信息，很方便的帮助我们追踪网站、帮助分析网站性能问题，也有很多前端团队基于 puppeteer 做自己的监控工具，更多信息可以查看 [文档](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.1&show=api-class-page)。

![](https://global.uban360.com/sfs/file?digest=fid71e95eafffa3f73d4224fb76b19dfc83&fileType=2)

# 总结

chrome浏览器功能很强大，基于 chrome 的 Headless 特性在node端也可以模拟实现前端页面中的各种神奇操作，达到我们的预期效果，但同时Headless Chrome 会占用大量的资源，特别是跑其他应用的服务器上时，无头浏览器的行为难以预测，目前线上应用更多的都是使用 docker 来管理 Chrome，构建定制镜像，在容器中运行，确保 Chrome 的稳定状态。

# 参考文章

-   [1] [爬虫利器 Puppeteer 的一些最佳实践](https://zhuanlan.zhihu.com/p/66296309)
-   [2] [puppeteer 生成pdf，绝对解决你的需求](https://blog.csdn.net/zhai_865327/article/details/104792646)
-   [3] [结合项目来谈谈 Puppeteer](https://zhuanlan.zhihu.com/p/76237595)
-   [4] [puppeteer 文档](https://zhaoqize.github.io/puppeteer-api-zh_CN/#?product=Puppeteer&version=v5.3.1&show=outline)
