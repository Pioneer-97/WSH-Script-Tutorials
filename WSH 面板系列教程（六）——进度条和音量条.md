# WSH 面板教程（六）——进度条和音量条

标签（空格分隔）： WSH JScript foobar2000

---

前一篇说过了按钮，这一篇说说进度条。虽然 foobar2000 的工具栏上也能显示进度条，不过样式固定，位置也不能自由设置。而 WSH 面板做进度条则可以很自由地自定义外观。

还是先看看这个图（这个是照着虾米在线播放器的样式画的）：

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160204011.png)

我们的目标就是做出能跟上面差不多的进度条，当然一开始我们先从简单的入手，我们先做播放进度条吧，先分析分析：

- 进度条是长条矩形（或其它类似，但矩形最简），那就有 x, y, w, h 这几个属性
- 需要画两个矩形，一个是全长，一个是进度的长度
- 鼠标在进度条上点下能改变进度
- 点下不放，移动也能改变进度

我们先从画开始：
```
// 声明一个对象，存储进度条的各个值
var seeker = {
    x: 20,
    y: 20,
    w: 0,
    h: 10,
    pos: 0,
}

var ww, wh;

function on_size() {
    ww = window.Width;
    wh = window.Height;
    
    // 进度条宽度随面板大小变化而变化
    seeker.w = ww - seeker.x * 2;
}

function on_paint(gr) {
        // 进度条背景
    gr.FillSolidRect(seeker.x, seeker.y, seeker.w, seeker.h,
        0xffaaaaaa);
    // 进度条进度
    if (fb.IsPlaying && seeker.pos > 0) {
        gr.FillSolidRect(seeker.x, seeker.y, seeker.w * seeker.pos,
            seeker.h, 0xff000000);
    }
}

// 进度条进度在播放时刷新
function on_playback_time(time) {
    seeker.pos = fb.PlaybackTime / fb.PlaybackLength;
    window.Repaint();
}

// 在播放停止或者播放开始时刷新（归零）
function on_playback_starting(cmd, is_paused) {
    seeker.pos = 0;
    window.Repaint();
}

function on_playback_stop(reason) {
    seeker.pos = 0;
    window.Repaint();
}
// 脚本初始化时刷新进度条的进度
if (fb.IsPlaying) {
    on_playback_time();
}
```



第一个例子还未完成，鼠标操作还没写，不过不继续了——我们直接第二个例子.

### 第二个例子

进度条这东西，上面说到了要用两种——播放进度条和音量。然而从图片上我们也能知道，这两种无论外观还是操作方式都很相似。所以我们能不能像**按钮**一样也做个通用的呢？

试试看吧，虽然写教程之前我没写过。




```
// 总之先定义一个 `Slider` 构造函数：
var Slider = function() {
    this.x = this.y = this.w = this.h = 0;
    this.is_drag = false;
    this.pos = 0; // 百分数表示
}

// 然后添加几个方法：
Slider.prototype.draw = function(gr, x, y, w, h, active_color, inactive_color) {
    // 画图
    gr.FillSolidRect(x, y, w, h, inactive_color);
    if (this.pos > 0 && this.pos <= 1) {
        gr.FillSolidRect(x, y, w * this.pos, h, active_color);
    };
    // 更新它的 x, y, w, h 值
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
}

Slider.prototype.is_mouse_over = function (x, y) {
    return (x > this.x && x < this.x + this.w && y > this.y && y < this.y + this.h);
}
```

这里 `this.pos` 是我们要注意的，进度条前进到哪了，就指着这个值呢。但现在要写个通用的，那就不能在 `Slider` 里面定义获取 `this.pos` 的方法了，比如播放进度条，我们经常添加一个方法：
```
Slider.prototype.get_pb_pos = function() {
    return fb.PlaybackTime / fb.PlaybackLength;
};
```

音量进度的话，则换一个：
```
Slider.prototype.get_vl_pos = function()｛
    // 一般不用这个
    return (100+fb.Volume)/100;
}
```

而现在我们要写的是通用的，`get_pos()` 也是要能适应各种各样不同的情况的。好在 Javascript 可以用**函数**作为参数，我们改写上面的程序：

```
// func_get: 返回一个百分比值，比如 return fb.PlaybackTime / fb.PlaybackLength;
// func_set: 可以不返回任何值，比如 
// function set(pos) {
//      fb.PlaybackTime = fb.PlaybackLength * pos;
// }
// 有一个参数 pos, 也是百分比值，设置进度
var Slider = function(func_get, func_set) {
    this.x = this.y = this.w = this.h = 0;
    this.is_drag = false;
    this.pos = 0; // 百分数表示
    //
    this.get = (function() {
        return typeof func_get == "function" ? func_get : function() {};
    })();
    this.set = (function() {
        return typeof func_set == "function" ? func_set : function() {};
    })();
}

// 然后添加几个方法：
Slider.prototype.draw = function(gr, x, y, w, h, active_color, inactive_color) {
    // 画图
    gr.FillSolidRect(x, y, w, h, inactive_color);
    if (this.pos > 0 && this.pos <= 1) {
        gr.FillSolidRect(x, y, w * this.pos, h, active_color);
    };
    // 更新它的 x, y, w, h 值
}

Slider.prototype.is_mouse_over = function (x, y) {
    return (x > this.x && x < this.x + this.w && y > this.y && y < this.y + this.h);
}

// 鼠标事件：

Slider.prototype.down = function(x, y) {
    if (this.is_mouse_over(x, y)) {
        this.is_drag = true;
        this.move(x, y);
    }
}

Slider.prototype.up = function(x, y) {
    this.is_drag = false;
}

Slider.prototype.move = function(x, y) {
        if (this.is_drag) {
            x -= this.x;
            this.pos = x < 0 ? 0 : x > this.w ? 1 : x / this.w;
            this.set(this.pos);
            window.Repaint();
        }
}

Slider.prototype.update = function() {
    this.pos = this.get();
    window.Repaint();
}
```

上面，我们把不通用的部分`get()`和`set()`作为参数，其余的部分则定义在构造函数里面。这样我们每次新建一个实例的时候，都只要实现定义好 `get` 和 `set`，然后作为参数就可以了。

例子文件在[这里](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/%E6%95%99%E7%A8%8B%E5%AE%9E%E4%BE%8B/6%20-%20slider.txt)，不在这里贴了。

这个例子参考了 **thanhdat1710** 的代码，此人也是个大神，WSH 脚本的许多新点子新特性都是他想起的。不过此人有点挖坑不埋的意思，经常写一个很牛b的脚本，然后就放着不管了。他新做（或 Mod）的界面也是，很久以前就有预览图片了，至今也没能正式放出来。（可能年龄到了——那些年活跃的大神们后来一个个都消失了，Jensen, theophile, neoashi 等等，都是过了折腾的年纪了，最重要的是毕业了（或出国留学），thanhdat 孩子都有了，花在这上面的时间肯定也是有限的。）

## 说说构造函数

按钮和进度条里，都用到了构造函数。（然而 Javascript 本人理解的并不深，所以下面的话千万别当成教条，本人也不打算说太复杂）。

构造函数仍然还是个函数，实际上任何函数你都可以 `new func()` 一个实例出来，只不过毫无意义罢了，既没法操作属性也没有方法调用。只有构造函数，你给这个函数赋予了属性，赋予了方法，新建出来的实例才有存在的意义。

构造函数，本人不知道它有什么高级的低级的用途。但看了这么多脚本，发现实际上用它仅仅两个目的：

- 做模板：就像前面的按钮、进度条，只要定义一次，后面要用的时候 new 一个实例就行。
- 封装，尽量减少全局变量：把某个模块都装在一起，减少全局变量的使用。

比如我要写个界面，界面上有一个播放列表，还有几个按钮。我可以这么来：
```
var Playlist = function() {
    this.xx = ..;
    this.yy = ..;
    this.func  = function() {};
    this.func2 = function() {};
    ...
}

var Buttons = function() {
    this.b = [];
    this.draw = function() {};
    this....
}

var pl = new Playlist();
var bts = new Buttons();
```

这样，我们看到 pl.xxx 就知道跟列表有关，看到 bts 就知道跟按钮有关，程序是模块化的，而且只占用 pl, bts 这两个全局变量，否则要用一大列的全局变量。

写简单程序，并不在乎上面这些，比如我们前面写的那些，但要是复杂界面，比如贴吧 **Keperlia** 发布的那个界面，所有功能都集中在一块面板上，将各个功能模块化是很重要的。

构造函数的属性，方法：
```
function func() {
    // 私有属性，不能直接改
    var x1 = 0;
    // 公有属性
    this.x2 = 3;
    
    // 私有方法
    function meth() {
        x1 = x1*2;
    }
    // 特权方法
    this.prevli = function() {
        x1 = 4;
        this.x2 = 2;
    }
}
// 公共方法
func.prototype.publ = function() {
    this.x2 = 100;
}
```

私有，特权，公共... 看起来很烦...确实很烦...还是建议各位看一本叫 《Javascript设计模式》的书吧（这本书内容很多，但你不打算入那一行的话不用学那么多，封装，单体模式看一下就够了，继承都不用看，遑论其它。）里面讲的比较详细。

还有个要注意的是 `this`... 注意 `this` 指的是谁。就像在贴吧，看到不少人会说“这里”...看的我一头雾水莫名其妙这里指的什么东西啊?!

我就说一点啊，`this` 是一个指针一类的东西，只在函数（或私有方法）里会改变指向。

```
var a = 0;
fb.trace(this.a); // 输出 0;

function b() {
    this.a = 1;
    fb.trace(this.a); // 输出 1
}

function c() {
    this.a = 4;
    fb.trace(this.a); // 输出4
    function d() {
        this.a = 10;
        fb.trace(this.a); 
    }
    d(); // 输出 10;
    this.m = function() {
        fb.trace(this.a);
    }
    this.m(); // 输出 4
}
```

可以看到，如果 `this` 不在函数里面的话，它指的是当前的环境对象（浏览器是 window 对象，foobar2000 和 NodeJS 都不是），在函数里面，指的则是函数对象本身，构造函数里的方法不会改变 `this`的指向，但里面定义的私有方法（也就是函数）则会。（不要担心看不懂，这个坑以后肯定会遇到的，多摔几次就懂了。）
        
