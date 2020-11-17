---
title: 你不知道浏览器debug方法
date: 2020-11-13 11:12:57
img: /medias/featureimages/6.jpg
top: true
cover: true
coverImg: /medias/featureimages6.jpg
toc: true
mathjax: false
summary: 因为工作中，经常涉及到前端后台数据交互，js测试，性能优化，前端js操作dom，ajax接口调试等都会比较麻烦，所以会前端调试非常重要，尤其对于工程优化方面能给我们一个简单直接的分析工具。
categories: 前端
tags:
- debug
- chrome
---

### 引言

写本篇博客的由来是上次舜阳的性能调试引申出来的，本人顺带挖掘了一下除了常用的sorce debugger和console，我们的Chrome浏览器都还能帮我们在平时工作中提供那些好用的调试模式呢。如下图chrome调试模式有好多的内容，下面就一一列举一下。

![](https://ftp.bmp.ovh/imgs/2020/11/b798ddc332d26ebd.jpg)

#### elements

1.用来查看，修改页面上的元素；（包括DOM标签，以及css样式的查看，修改，还有相关盒模型的图形信息等）（我们常用的略过）
2.查看dom上绑定的事件。
3.拷贝都没节点。
4.查看样式所在的文件路径。

![](https://ftp.bmp.ovh/imgs/2020/11/99cef2dabe985735.jpg)

    ———handler是处理函数, 右键可以看到这个函数定义的位置, 一般 js 库绑定事件会包一层, 所以这里很难找到对应handler

    ———isAtribute 表明事件是否通过 html 属性(类似onClick)形式绑定的

    ———useCapture 是 addEventListener 的第三个参数, 说明事件是以 冒泡 还是 捕获 顺序执行
#### console

1.用来吐出代码中console的日志有（log、gruop、error等方法，具体可自行查找）
2.除了1我们还可以在里面执行一些测试代码和方法。
3.也可以吐出一些引用，查找方法对应的执行前后循序和位置。

![](https://ftp.bmp.ovh/imgs/2020/11/980bc3e8e3887079.jpg)

#### Sources
这个页面内我们可以找到当前浏览器页面中的js源文件，方便我们查看和调试。在正式发布的网站中我们看到的都是压缩过的代码，但是在调试的是后这里是本地的资源文件。

我们可以点击下面的{}大括号按钮将代码转成可读格式，下图是两个不同的代码，由于线上的umi打包后太大了我就先截取了一个本地的图。

![](https://ftp.bmp.ovh/imgs/2020/11/0ecb959850f40374.jpg)

![](https://ftp.bmp.ovh/imgs/2020/11/ae6885aabe64e56d.jpg)

- Sinppets:

可以执行代码片段：
当我们想知道代码中的某个方法怎么是用或者是需要测试一个新的函数是否在这里可以调用时，会打开控制台有针对性的写一些调试代码，或者想测试一下刚刚写的方法是否会出现期待的样子，但是控制台一打回车本想换行但是却执行刚写的半截代码.

所以推荐使用Sources下面的左侧的Sinppets代码片段按钮这时候点击创建一个新的片段文件，写完测试代码后把鼠标放在新建文件上run，再结合控制台查看相关信息

新建了一个名叫：a.js的片段代码，在你的项目环境页面内，该片段可执行项目内的方法；

![](https://ftp.bmp.ovh/imgs/2020/11/59f7c6ef5e2f6e82.jpg)

- Content scripts:
Chrome 的一种扩展程序，它是按照扩展的ID来组织的，这些文件也是嵌入在页面中的资源，这类文件可以读写和操作我们的资源，需要调试这些扩展文件，则可以在这个目录下打开相关文件调试，我们的项目里目前引入里jquery的相关方法。

![](https://ftp.bmp.ovh/imgs/2020/11/0044dc512a86138a.jpg)

#### Network

可以看到所有的资源请求，包括网络请求，图片资源，html,css，js文件等请求，可以根据需求筛选请求项，一般多用于网络请求的查看和分析，分析后端接口是否正确传输，获取的数据是否准确，请求头，请求参数的查看等。
1.可勾选单独的xhr，js，css等。
2.可以在preview中查看json格式化的接口返回。

这里重点提一下，在邮件copy中有一系列的选下，这里可以把这个请求的所有信息都copy出来，便于我们给后获发送。（主要用curl）
![](https://ftp.bmp.ovh/imgs/2020/11/07d6f18ff7553c44.jpg)

#### Performance

时间表可以记录和运行分析应用程序所有的活动，为了使的记录页面的交互，打开时间轴面板，然后按开始录制录制按钮（），或者通过键入键盘快捷键Cmd的 +E（Mac）或按Ctrl +E（Windows / Linux版）。这个记录按钮会从灰色变成红色，而Timeline将开始从你的页面获取时间线（timeline）。在你的应用中完成一些操作，记录到一些数据之后，再一次点击按钮来停止记录。这个分析会有点慢。

![](https://ftp.bmp.ovh/imgs/2020/11/8a4a398265e0e37d.jpg)

第一个框里是概述，这里可以大致看到页面的性能和执行效率。

第二个框里是event，即是事件监控。这是CPU的堆栈跟踪的可视化，绿色表示媒体时间，红色表示负载事件，蓝色表示DOM事件。

第三个框里表示存储。

第四个框里是详细信息，这里会显示事件的详细信息。

上图可见脚本15.68s，渲染时间2.7s，绘制时间0.46s说明我们脚本占用的比较大的资源。通过这个可以分析如果系统反应慢主要症状点在那个过程，通过如下分析可以相应的给出一些策略。

![](https://ftp.bmp.ovh/imgs/2020/11/00049d68d16f1bda.jpg)

在call tree中可以看到整个过程各个环节函数和方法调用情况

![](https://ftp.bmp.ovh/imgs/2020/11/d9ab72aa46ff6e0a.jpg)

#### Memory
会列出所有的资源占用空间。

![](https://vkceyugu.cdn.bspapp.com/VKCEYUGU-imgbed/b20d67d4-91bd-449f-99c2-23ee9fef8601.jpg)

#### Security
可以告诉你这个网站的安全性，查看有效的证书等。

#### Application
会列出所有的资源，以及HTML5的Database和LocalStore等，你可以对存储的内容编辑和删除 不做过多介绍
[![](https://vkceyugu.cdn.bspapp.com/VKCEYUGU-imgbed/6a0eef45-e921-4543-8d13-09c8749a5aae.jpg)](https://imgbed.cn)