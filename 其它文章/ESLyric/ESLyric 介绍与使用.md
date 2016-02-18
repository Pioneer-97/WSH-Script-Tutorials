# ESLyric 介绍与使用

标签（空格分隔）： foobar2000 lyric JScript

---

ESLyric 是由 **ttsping** 写的一个 foobar2000 歌词组件。大概是 2014 年第一次发布的（吧？）。此插件与[歌词秀3]()类似，可以内嵌如 foobar2000 的界面作为显示歌词之用，另外还提供了桌面歌词功能，其它种种功能后面再说。

## 历史

都知道 foobar2000 是个不错的播放器，然而它没有官方的歌词显示，于是各路大神纷纷出马，写了不少的歌词插件，国外最新比较流行的是歌词秀3（也是老插件了）。对国内用户来说，歌词插件不仅是能显示歌词，还得能自己下载歌词，所以得有国内的网站歌词源。歌词秀3显然不可能默认带国内源。

于是 ttsping 出马，写了个歌词秀3的扩展插件（什么名字忘了），主要就是添加国内歌词源，顺便还加上了桌面歌词功能。这个插件流传了挺长一段时间。

后来 ttsping 蛰伏一段时间后，在带来 WSH Panel Mod Plus 的同时，又将歌词秀3扩展插件的所有功能独立出歌词秀3，不再作为它的附属而是平等地位的一个插件，也就是 foo_eslyric.dll。这个插件 2013 年第一次在贴吧发布（一晃眼已经是 2016 年，时间过得真快）。

这个插件不带歌词面板的功能，不能内嵌入 foobar2000 的界面，它的主要功能还是提供歌词源和桌面歌词。不过这时歌词源已经不是硬编入插件代码，而是由用户编写 JScript 代码来获取了——这可能是我见过的最好的歌词软件功能，因为这理论上可以使这个插件永不失效，只要还能运行，就可以下载歌词，不会因为网站更新等等问题再也抓不到歌词；而且可以不分地域，任何国家都可以获取当地网站的歌词。当然还有些其它人性化的功能，在此不再赘述。

> foo_eslyric 差不多这个样子 

> ![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc3/Pic_20160214001.png)

之后，在 ttsping 准备毕业论文的那段时间，默默憋大招的一段时间之后，大概是 2014 年末，foo\_uie\_eslyric.dll 横空出世，也就是现在说的 eslyric。

## 功能

- 歌词面板，可以内嵌入 foobar2000 界面，显示歌词；
- 桌面歌词显示；
- 强大的歌词源机制，可以自己写 JScript 脚本来获取歌词；
- 歌词解析功能，可以解析特殊格式的歌词（仍需用户自己写 Jscript 脚本），比如酷狗歌词格式 krc. （酷狗歌词已经被 Asion 搞定）
- 歌词搜索过滤，可以过滤符合特定要求的歌词；
- 自定义歌词搜索参数（较少用）
- 歌词编辑功能
- 接收从 WSH 面板发来的通知，可以与 WSH 面板交互（单向）

## 使用方法

安装参考[这篇](https://github.com/elia-is-me/WSH-Script-Tutorials/blob/master/%E5%85%B6%E5%AE%83%E6%96%87%E7%AB%A0/foobar2000%20%E7%BB%84%E4%BB%B6%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.md)。

### 添加歌词源

歌词源是一个一个的 `*.js` 脚本，在导入界面将其导入即可。

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc3/Pic_20160214002.png)

AsionWu 的汉化版 foobar2000 已经自带了最新的歌词源，安装时，勾选上 ESLyric 配置文件即可。天天动听的歌词源文件在[这里](https://github.com/elia-is-me/Foobar2000-WSH-Scripts/blob/master/ESLyric/ttpod.js)也可以下载。

### 添加多个本地文件夹

打开 `设置 > 工具 > ESLyric : 搜索` 的设置页面（就是 eslyric 的设置界面，搜索标签页）。`歌词来源` 列表里有个 `本地文件` 这一项。在上面双击，可以打开一个设置窗口（隐藏较深...）

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc3/Pic_20160218001.png)

你们都知道该怎么办。

### 歌词保存设置

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc3/Pic_20160214003.png)

### 歌词搜索过滤

在设置界面 `搜索 > 高级` 打开高级搜索设置对话框，勾选 `启用歌词搜索过滤`，点开设置，可以编辑过滤脚本，这里是一个示例：

```
// return:  true - 过滤启用，不再搜索此歌曲歌词
//          false - 不过滤
// tf: 当前正在播放歌曲的 TitleFormat 对象
function is_item_filtered(tf)
{
    // 如果 ESLYRIC 字段的值是 "no-lyric"，则该曲不再搜索歌词
    if (tf.Eval("$meta(ESLYRICS)").toLowerCase() == "no-lyric") {
        return true;
    }
	return false;
}
```

接口可以参考编辑器左下角的 `工具 > 文档`，也可以查看[这里](https://github.com/elia-is-me/WSH-Script-Tutorials/blob/master/%E5%85%B6%E5%AE%83%E6%96%87%E7%AB%A0/ESLyric/ESLyric%20Interfaces.txt)。

### 歌词面板

根据自己的需求设置就行了，也没有难的。

### 与 WSH Panel Mod（Plus）的互动

(待完成)

### 歌词源脚本编写

(待完成)







