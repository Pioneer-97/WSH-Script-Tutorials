# WSH 面板系列教程（二）

标签（空格分隔）： jscript foobar2000 wsh

---

WSHMP 的脚本语言是 Javascript，不过不是大家可能通常知道的那个 Javascript——在网页前端或者后端非常流行的那个——我们要写的是有 Javascript 语法的一种脚本语言，更准确点叫 ECMAScript。所以如果还不会 Javascript 的就不要抱着厚教材学怎么写 JS 了，参考下 [W3cschool](http://www.w3school.com.cn/js/index.asp)（只需要参考 JS 教程 和 JS 对象两部分就行，DOM 等其它的不需要看）。

插件包里的文档，接口在 `interface.txt` 里，回调函数（事件函数）在 `callbacks.txt` 里。

## 第一个例子

WSHMP 是界面插件，是一块面板。对它进行脚本编程，是为了：

- 让它显示你想它显示的东西；
- 你可以通过鼠标与这块面板交互（比如按按钮啊什么的）；
- 能操作快捷键，或者输入文字就更高级了（很少用）；

既然是初步，我们先来看第一条，先画点什么吧——
```
var font = gdi.Font("Tahoma", 20, 1);

function on_paint(gr) {

    // 画实体矩形
    gr.FillSolidRect(10, 10, 100, 200, 0xff000000);
    // 画实体圆
    gr.FillEllipse(200, 200, 100, 100, 0xff00aa00);
    // 画框
    gr.DrawRect(300, 10, 100, 50, 1, 0xffaa0000);
    // 画圆框
    gr.DrawEllipse(400, 200, 100, 100, 1, 0xff000000);
    
    
    gr.GdiDrawText("Hello world", font, 0xff000000, 10, 300, window.Width, 50, format = 0);
}
```

在编辑器里编辑以上代码后，WSHMP 的面板应该显示的是这些东西：

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_201602040010.png)

看起来不过是一些简单的图形，或者文字。不过要意识到：再复杂的界面，都是这些简单的图形文字组成的，只不过他们图形画的很精致，他们的文字排版很好。而形状更复杂的图标等等则做成图片或字体。

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_201602040011.png)

比如看上面这个例子，看起来很复杂，实际不过是用 `gr.DrawImage()` 画图标，用 `gr.FillSolidRect()` 画背景色以及进度条，`gr.FillEllipse()` 画的进度条上的圆，`gr.GdiDrawText()` 画的进度条两边的时间而已。只不过在哪里画，画多大这些，需要事先设计好——哪怕偏离一个像素，就可能会导致看起来不自然。

再回到第一个例子，我们发现圆很粗糙，不光滑，需要加点东西：

```
var font = gdi.Font("Tahoma", 20, 1);

function on_paint(gr) {
    // 设置光滑度，0-4, 4最高
    gr.SetSmoothingMode(4);
    
        // 画实体圆
    gr.FillEllipse(200, 200, 100, 100, 0xff00aa00);
    // 画圆框
    gr.DrawEllipse(400, 200, 100, 100, 1, 0xff000000);
    
    // 归位，像矩形那样直上直下直来直去的不需要设置光滑度，否则反而发虚
    gr.SetSmoothingMode(0);
    
    // 画实体矩形
    gr.FillSolidRect(10, 10, 100, 200, 0xff000000);
        // 画框
    gr.DrawRect(300, 10, 100, 50, 1, 0xffaa0000);

    gr.GdiDrawText("Hello world", font, 0xff000000, 10, 300, window.Width, 50, format = 0);
    
}
```

上面那些画圆、矩形等等的方法，在 Interface.txt 文件里都能找到，里面还有更多的方法。一定要了解的是，软件界面一般都是简单的矩形圆形文字，辅佐以图片组成的，所谓自定义界面，不过就是在面板上画各种各样的东西。

`on_paint` 是一个事件函数：面板重绘一次，这个函数就执行一次（比如移动窗口，调整窗口大小，或者用 `window.Repaint()` 执行重绘命令。事件函数在 `callbacks.txt` 文件里有。

事件函数，意思就是函数跟某个事件绑定在一起，如果事件发生了，就要执行绑定的函数。

## 第二个例子

这次我们画个看起来更简单点的东西：黑灰色的背景，中间是绿色的文字背景，然后是 "hello world" 这样的文字。

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_201602040013.png)

可以分析下，这个简单的图包括三个要素：两个矩形+一句文字，仅此而已。

```
// 颜色函数，hex 格式的颜色很难用，所以我们定义一个 RGB 颜色格式函数
function RGB(r, g, b) {
	return (0xff000000 | (r << 16) | (g << 8) | (b));
}

var DT_CENTER = 0x00000001;
var DT_VCENTER = 0x00000004;
var DT_CALCRECT = 0x00000400;

var ww = 0, wh = 0;
var font = gdi.Font("Tahoma", 20, 1);

function on_size() {
    ww = window.Width;
    wh = window.Height;
}
    
function on_paint(gr) {
    // 画背景
    gr.FillSolidRect(0, 0, ww, wh, RGB(30, 30, 30));
    // 画中间矩形背景
    gr.FillSolidRect(ww/2 - 150, wh/2 - 25, 300, 50, RGB(0, 255, 0));
    
    // 画文字
    gr.GdiDrawText("Hello world", font, RGB(0, 0, 0), ww/2-150, wh/2-25, 300, 50, DT_CENTER | DT_VCENTER | DT_CALCRECT);
    
}
```

然而程序却并不比上一个短，虽然图形更简单了——因为多了个『中间』条件。要知道哪里是中间位置，需要先知道面板的大小；因为面板大小是可以用鼠标调整的，所以我们要在面板大小变化之后重新计算位置。

这里多了个事件函数 `on_size`，顾名思义，当面板大小变化后，立即执行这个函数。我们用 `ww` 和 `wh` 这两个全局变量存储面板的宽和高，并且一旦面板大小改变了，就需要重新读取一次 `window.Width` 和 `window.Height` 的值。这样，在 `on_paint` 里用到的 `ww` 和 `wh` 永远都是最新的窗口大小。

作为比较，你可以试试这个脚本，看看改变窗口大小的时候，有什么不同：
```
// 颜色函数
function RGBA(r, g, b, a) {
	return ((a << 24) | (r << 16) | (g << 8) | (b));
}

function RGB(r, g, b) {
	return (0xff000000 | (r << 16) | (g << 8) | (b));
}

var DT_CENTER = 0x00000001;
var DT_VCENTER = 0x00000004;
var DT_CALCRECT = 0x00000400;

var ww = 0, wh = 0;
var font = gdi.Font("Tahoma", 20, 1);

//function on_size() {
    ww = window.Width;
    wh = window.Height;
//}
    
function on_paint(gr) {
    // 画背景
    gr.FillSolidRect(0, 0, ww, wh, RGB(30, 30, 30));
    // 画中间矩形背景
    gr.FillSolidRect(ww/2 - 150, wh/2 - 25, 300, 50, RGB(0, 255, 0));
    
    // 画文字
    gr.GdiDrawText("Hello world", font, RGB(0, 0, 0), ww/2-150, wh/2-25, 300, 50, DT_CENTER | DT_VCENTER | DT_CALCRECT);
    
}
```

下一篇，说说看怎么做一个封面显示器。

    
    

    
    





