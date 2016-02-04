WSH 面板系列教程（一）

标签（空格分隔）： WSH JScript

---

## 前言

WSH 面板是 foobar2000 的一个面板组件，它提供了自定义 foobar2000 的面板样式的功能。WSH 面板使用 JavaScript 作为脚本语言，并且提供了许多 foobar2000 的底层接口，所以它可以实现许许多多复杂的、奇葩的功能。也正因为有了它，所以 foobar2000 变得很复杂，非常复杂。

> 人们常说 foobar2000 太复杂了难用，其实 foobar2000
> 本不是个非常复杂的玩意，它就是个播放器，功能有限，界面简陋……只不过后来出现个叫 musicmusic 的人，他带来了
> ColumnUI，foobar2000 于是稍微变的有那么复杂了一点，顺便还带坏了 peter 把原来的界面弄得也复杂了一点；后来又出现个叫
> ssenna 的人，他带来了 PanelStackSplitter（面板栈分离器）和 ELPlaylist（以及ESPlaylist），这下
> foobar2000 就称不上简单了；后来不知道谁弄了个 WSH Panel，坑了以后 Wang T.P.接手弄了个 WSH Panel
> Mod……从此 foobar2000 再也洗不清『复杂』的恶名了……（此中省略 Panel UI 不谈）

这里先介绍一下 WSH 面板的谱系：WSH 面板最初是 WSH Panel，由于太古老，本人也不知它的作者是谁，只知道这玩意容易崩溃且接口有限，能做的东西很少，后来 Wang T.P.（某个中国的大牛）接手开发，做了个 WSH Panel Mod，解决了容易崩溃的问题，并且新增了许多接口，这下这东西的名声开始显赫了。Marc2003 在官方论坛开了个专门讨论WSH脚本的帖子，这个帖子飙到 179 页接近 4500 楼，各种各样的大神开始给 WSH 面板写脚本，从开始的简单到后面的越来越复杂……

光阴流逝，WSH Panel Mod 版本号到 1.5.6 后开发就停止了，WSH Panel Mod 1.5.6也就成为 WSH 面板最经典的一个版本。之后就是其它接手的人做的 WSH Panel Mod 的修改版本了。

Wang T.P.在开发 WSH Panel Mod 的时候是遵守 foobar2000 的开发协议的，foobar2000 规定了有一些东西 foobar2000 组件的开发者不能碰，但对于我们用户来说，当然是限制越少越好了，于是一些人就基于Wang T.P.的工作加了一些其它的功能，具体不说了。早期是 WSH Panel Mod Mod 和 WSH Panel Mod SP 这两个。

后来，ttsping 带着 WSH Panel Mod Plus 粉墨登场（此处有音乐响起）。WSH Panel Mod Plus 可谓 WSH 面板的集大成者及发展者，它基本包含了之前（及之后）各种 Mod 版本的功能，并且新增了许许多多接口，功能更加强大，目前在中国新流传的界面基本都用这个。另外截至目前（2016-2-4）ttsping 仍然活跃着（不像 WangTP 已经隐退），所以理论上 WSH Panel Mod Plus 还在维护，修 bug 啦，新增接口啦都是有可能的。

由于 WSH Panel Mod Plus 没能在国外引起大的反响（主要是官网论坛不能宣传——违反开发协议了），而国际友人对 WSH Panel Mod 也是有新的需求的，于是 Marc2003（前面提到过）也开了个新分支，他做的主要是删去原来的一些重复的接口，后来又新加了些新接口，结果是 WSH Panel Mod的版本号又开到了1.5.10. 新加接口当然没问题，不过删减接口就有麻烦了——不兼容之前的脚本，于是Marc就更新到 1.5.10 停，并且新开了个分支叫 Jscript Panel，名字变了，东西仍然是原来的东西（当然还在开发中，有增减接口的事情时而有之），意思很明确：我这东西不叫 WSH Panel *** 啦，所以兼容不了以前的 WSH 脚本我也不负责啦！

于是我们知道，前前后后出现了的WSH面板插件有：WSH Panel、WSH Panel Mod 1.5.6、WSH Panel Mod Mod、WSH Panel Mod SP、WSH Panel Mod Plus及Jscript Panel（包括 WSH Panel Mod 1.5.7-1.5.10）。推荐当然是推荐 WSH Panel Mod Plus 和 Jscript Panel 的最新版本，因为功能更多。写脚本的时候稍微注意下别用Jscript Panel 删掉的接口，这样兼容性会更好点。

本篇教程当然是以 WSH Panel Mod Plus（版本1.5.7.1）（以后简称WSHMP）为基准，理由有 N：

- 功能强大，基本囊括了上面提到名字的所有的分支的功能；
- 兼容性好，兼容所有 WSH Panel Mod 1.5.6+ 及 Jscript Panel 的脚本；
- 有中文版，没语言障碍
- 编辑器功能也更强大（可以折叠及显示控制台）；
- 国内用户多，脚本作者也多

尽管前言写了这么长，但本篇教程并没打算写多少内容。因为 WSHMP 的脚本编程很复杂，所以打算写一个系列，做一个简单的介绍，能让大家了解 WSHMP，使得在使用 foobar2000 的时候如果有需求可以简单修改脚本，也可以自己写一些简单脚本即可。如果打算写很复杂的东西的话，除了要对 Javascript 有更深的理解外，只剩参考其它的复杂脚本怎么写的这条路了。

## 事前准备

WSH Panel Mod Plus 是第三方插件，理所当然的要你自己下载安装。
- [下载地址]()
- [插件安装教程（参考）]()

安装好之后，要编辑布局，新建一个 WSHMP 面板显示在你的界面上，DUI 编辑布局很简单：
![执行菜单命令](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160204004.png)

![右键单击](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160204005.png)

![选择](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160204006.png)

当看到这个界面时，准备工作完毕：
![初始界面](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160204007.png)

然后鼠标单击，打开编辑器，删掉里面的内容，就可以开始编辑自己的脚本了。
![脚本编辑器](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160204008.png)

默认的字体不太好看，你可以换成自己喜欢的字体，在`首选项 > WSH 面板加强版 > （右边）style.default`里，双击，修改字体名称和字号。
![脚本编辑器](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160204009.png)

在下载的插件包里，有 doc 和 samples 两个文件夹，里面分别是**接口文档**和**实例**，这两个东西很重要，因为只要踏入 WSH 脚本的坑这两个文件夹里的东西以后是要常用到的。

> 多余的话：凡开始编程的人一般都会遇到选择哪个编辑器的问题，其实 WSHMP 自带的就是很好的编辑器，带提示，自动缩进，高亮等等，还可以折叠，可以显示控制台（很有用）。其它的话，notepad++，evernote，sublime3 等等都是挺好的编辑器，如果是编程新手的话只要注意别被拐骗到 vim 或者 emacs 的坑里的话基本不会有问题。

本篇教程到这里，可能废话居多，不过没办法本人就这么个样子...下篇开始脚本篇。

。






