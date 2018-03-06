title: Atom activate-power-mode插件安装
date: 2015-12-21 17:48:00
categories: 工具
tags: 插件
---

两周前在微信公众号里看到关于对Atom编辑器中酷炫效果的展示，觉得很有意思，于是去尝试了一下这个酷炫的插件。

<!--more-->

Atom
-----
`Atom`是Github开源的文本编辑器，这个编辑器完全是使用Web技术构建的(基于Node-Webkit)。Atom的强大可以与大名鼎鼎的Sublime Text相媲美。两者界面相差也不是很多，易于上手。

以下是一位知乎大神对其本质的解析：
>Sublime 是原生界面，脚本用的是 python；Atom 应该是基于 Chromium Embedded Framework，基本上就是个 web app，源码都是 CoffeeScript 写的，连界面都可以用 CSS 来自定义。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[知乎--尤雨溪](http://www.zhihu.com/question/22867204/answer/22944645)


Atom安装
----
目前[Atom官网](https://atom.io/)已经有了1.0版本.可以从它的官网上进行下载。

![官网首页](http://ww2.sinaimg.cn/large/8fa46249gw1ez86eesslqj211e0h175h.jpg)

从网站图片就可以看出这款编辑器的精美，点击中间的红色按钮就下载就可以了。安装完成后启动界面如下图所示：
![启动界面](http://ww2.sinaimg.cn/large/8fa46249gw1ez86bgo9bkj20b40b4759.jpg)

下载安装完成后，难处主要实在安装`activate-power-mode`的过程。这个过程很曲折。<br>


插件下载
---
首先要从官网下载插件：
[activate-power-mode](https://atom.io/packages/activate-power-mode)
![插件页面](http://ww4.sinaimg.cn/large/8fa46249gw1ez7hhmbxqpj20po0gyjsj.jpg)
进入页面后点击`versions`，会进入一个显示各个历史安装包的页面可以下载。选择适合的包进行下载。

具体的下载页面如下所示：
![下载页面](http://ww2.sinaimg.cn/large/8fa46249gw1ez7hjqbj65j20ss0fgmxp.jpg)


插件安装
---
1.在安装好Atom后右键点击，打开文件位置。可以看到文件安装在`C:\Users\Administrator\AppData\Local\atom\app-1.3.2`下。
 <blockquote>**注意**：（如果是安装的话，忽略这步，如果是解压的话，需要配置环境变量到path中，路径为 ：`~\atom\app-1.3.2\resources\app\apm\bin`,和JAVA_HOME类似）。</blockquote>
2.将下载的zip包解压，放到`C:\Users\Administrator\.atom\packages`下。
以管理员身份运行windowsDos命令窗口，进入到~/packages/activate-power-mode下，运行命令：
```bash
$ apm install
```
```bash
$ apm install activate-power-mode
```

3.当两个命令都出现done的提示的时候，证明安装成功了。(借用CSDN的图)
![安装成功](http://ww2.sinaimg.cn/large/8fa46249gw1ez86zcicw5j20io0c40tl.jpg)

在安装过程中可能会出现如下错误：
![error](http://ww2.sinaimg.cn/large/8fa46249gw1ez86surp6ij20i80dhdgu.jpg)
网上搜索后，说是缺少依赖组件产生的。需要运行如下命令：
**ps：运行这条命需要安装Git客户端**。
```bash
npm install easyimage
```

![运行相关命令后](http://ww3.sinaimg.cn/large/8fa46249gw1ez8710q5acj20jl075q3g.jpg)

**奇怪的是：我运行后还是会出现错误，但是我打开Atom编辑器已经可以使用了，希望哪位大神，弄清原理后说明一下。十分感谢！**


安装完之后重启，打开Atom，在新建文档时，不会出现效果。需要手动加载插件（通过快捷键ctrl+alt+o​打开），就可以看到酷炫效果了！
最终的效果：
![最终效果](http://ww2.sinaimg.cn/large/8fa46249gw1ez7hl6eeiig20e607m4qq.gif)


小技巧：去除屏幕抖动效果——shake。
可能会有人觉得这个插件抖动的时候看起来很不舒服，晃眼。
其实去掉这个效果很简单，在文件夹下
```
~\.atom\packages\activate-power-mode\lib
```
找到`activate-power-mode.coffee`文件，打开->找到如下图所示文件位置。将文件中的`screenShake.enabled`改为`screenShake.disabled`即可。

![去除效果文字行](http://ww2.sinaimg.cn/large/8fa46249gw1ezaxih9nnoj20cy02dmx5.jpg)
<p>是不是看起来舒服多了呢？快开始你的编码旅程吧！</p>
部分内容转载自：<br/>
1.[Atom与markdown](http://www.jianshu.com/p/ad3e737e5dc2)
2.[Github Atom 安装activate power mode炫酷插件超详细教程](http://blog.sina.com.cn/s/blog_8fdfb1d90102w5h4.html)
