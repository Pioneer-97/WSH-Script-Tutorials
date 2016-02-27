# foobar2000 组件安装教程

标签（空格分隔）： foobar2000 

---

## 前言

foobar2000 由于其软件架构特点以及开放的姿态，使得第三方很容易开发组件（component）来拓展它的功能。由于在官网下载的默认安装文件只带了少量几个默认的组件，满足不了使用的需求，例如：默认不带 ape，tta，tak 等音频文件格式的解码器，很多无损压缩格式音乐没法播放。所以自己下载安装组件是必备的基本技能。

> foobar2000 的中文汉化版（Asion 汉化）为了方便使用，集成了无损压缩文件解码器以及一些其它有用的插件，安装时选上即可，不喜欢折腾的建议使用汉化版。

这里组件指的是 foobar2000 标准组件（\*.dll 文件），而非 vst 插件等其它插件，姑且把组件分为两类：
* 官方组件： 英文版安装包自带，安装时可选择；
* 第三方组件：非官方自带的组件

除了 `foo_input_std.dll` 和 `foo_ui_std.dll` 这两个组件是必须的外，其它的所有组件都**非必需**的，可以随需要增删。第三方组件可以去[官网](http://www.foobar2000.org/components)、[官方论坛](http://www.hydrogenaud.io/forums/index.php?act=SF&s=&f=28)或者[官方 wiki](http://wiki.hydrogenaud.io/index.php?title=Foobar2000:Components) 去找，也可以去贴吧等地逛逛。

## 下载

还是要强调一下，这里说的是 **foobar2000 component**，不是中文网上通常说的 vst 插件。

下载好的组件包一般是 `xxx.zip` 或 `xxx.fb2k-component` 格式的文件，也有用 7z 打包的。前两种都是 zip 压缩（只要把 fb2k-component 改成 zip 文件就变成了 zip： 包）。标准状况下压缩包里的内容结构应该是

- xxx.zip
    - yyy.dll
    - README.txt (可能没有)
    - LICENCE.txt (可能没有)
    - (其它杂七杂八）

除少数外一般只有一个 `xxx.dll` 文件.一定要注意压缩包结构不能是:

- xxx.zip
    - yy folder (文件夹）
        - zzz.dll
        - ...
        
否则要解压缩，提取那个 dll 文件。

## 安装

### 方法一（推荐）

- 打开 foobar2000 的菜单 `文件 > 首选项(file >preferences)` 的`组件(components)`一栏
- 把组件文件拖上去，再点击`应用`，重启后组件安装成功

> 如果你下载的组件是 `.zip` 文件或者 `.fb2k-component` 文件，就可以这么安装。如果是 `*.7z` 或者 `*.rar` 打包的文件，那么就先解压缩，把里面的 `.dll` 文件拖动到下图的界面上去，一样可以正确安装。

图示

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc2/install_components.png)


之所以推荐这么安装，是为了便于管理，你可以随时通过组件管理界面卸载不需要的组件；如果是官方组件的话还可以自动更新。

### 方法二

- 把扩展名 `.fb2k-component` 改成 `.zip`，用压缩软件解压缩，
- 把得到的 `.dll` 文件放到 foobar 安装文件夹里的 component 文件夹里，
- 重启 foobar 即可。

一般一个组件包里只有一个 `.dll` 文件，也有例外比如 UIHacks 里面几个文件都要复制到 component 文件夹里，tak 解码器组件也是有两个 `.dll` 文件。现在的懒人包一般都是这么处理的：所有组件都放在 component 文件夹里。**本人不推荐你这么做，理由基于且不仅限于以下几点**：

1. 管理不便，第一种方式安装的组件，如果是在官网下载的组件的话，可以直接通过`升级`按钮升级到最新版，可以方便的移除，而不用到 component 文件夹里去找到组件再删除。

2. 对组件发布者的尊重，有的组件包里有 license.txt 文件，如果你要在网络上分享你的配置的话，把这些 license.txt 文件附带上是对组件作者的尊重（虽然不附带也不会有什么后果）。

3. 防止丢失重要的帮助内容，有的组件里带有帮助文档的比如 wsh panel mod 或 random pool 等，也许发布人有这些帮助文件，但从网络上下载他人发布的懒人包的人就无从寻找了，比如吧里分享的懒人包里很少能看到 wsh panel mod 作者提供的接口和实例文档的。

4. 便于分享，如果你懂得如何把所有的配置文件都放到配置目录里的话，那么你向其它人分享你的配置，只需要分享配置目录就可以了。

##  插件的发布地址

首先是官网的[组件下载页](http://www.foobar2000.org/components)。这里基本包括了 foobar2000 的几乎所有解码器组件，以及一些非常流行的功能扩展组件，比如: Lyric Show Panel 3, Playlist Attributes, Playback Statistics等等。

[Column UI 官方网站](http://yuo.be/columns.php)。

foobar2000 吧 **ttsping** 至少发布过至少四个组件，分别是：

- [eslyric 桌面歌词](http://tieba.baidu.com/p/2370754361).
- [歌词秀扩展](http://tieba.baidu.com/p/2740822021).
- [WSH Panel Mod Plus（扩展了 WSH Panel Mod） 的功能）](https://github.com/ttsping/foo_uie_wsh_panel_mod_plus/releases).
- [foo_fix](http://tieba.baidu.com/p/3332882408).


**ssenna** 发布了不少组件，包括 ELPlaylist，EsPlaylist，面板栈分离器，album tree 等几个非常流行的组件，[发布网址](http://foo2k.chottu.net/).（可惜很长时间没更新了，据说硬盘坏了源码全失…如果你喜欢做 Column UI 皮肤的话，他写的组件几乎是必不可少的，可以说是 CUI 上最流行的几个组件了。）

**AsionWu** 汉化了许多插件，[下载地址](http://pan.baidu.com/s/1qWDtlF2)。

[日文维基网站](http://foobar2000.xrea.jp/index.php?FrontPage)，上面有很多 foobar2000 的更新信息，不仅仅包括 fb2k 主程序，还有插件，WSH 脚本等等。

>### 扩展阅读

> - [How to install a component](http://wiki.hydrogenaud.io/index.php?title=Foobar2000:How_to_install_a_component).
