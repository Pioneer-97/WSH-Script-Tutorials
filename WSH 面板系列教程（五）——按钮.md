# WSH 面板系列教程（五）——按钮

标签（空格分隔）： foobar2000 WSH JScript

---

这次我们要做点稍微复杂点的东西——按钮（Button）。

> 说起来 WSH Panel Mod 最初设计出来，就是为了做点按钮啦、进度条啦、封面显示之类的东西来的吧，后面有人用它做出整个的 foobar2000 UI 应该算是失控了……

所谓按钮，都不陌生，生活中也好，电脑上也好，到处可见，原理也很像——一个圆的或方的东西，手（鼠标）按上去，松开，会触动一些动作。现实的按钮论外，电脑上的按钮都是鼠标操作的——这就意味着，我们要监视鼠标的一系列动作，并根据不同的动作触发不同的操作。

WSHMP 提供了一系列监视鼠标动作的事件函数，我们可以利用它们做出按钮出来，先列举下这几个函数：

```
// left button > 左键
function on_mouse_lbtn_dblclk(x, y, mask) {}
function on_mouse_lbtn_down(x, y, mask) {}
function on_mouse_lbtn_up(x, y, mask) {}
// 鼠标移出
function on_mouse_leave() {}
// 中键
function on_mouse_mbtn_dblclk(x, y, mask) {}
function on_mouse_mbtn_down(x, y, mask) {}
function on_mouse_mbtn_up(x, y, mask) {}
// 移动
function on_mouse_move(x, y, mask) {}
// 右键
function on_mouse_rbtn_dblclk(x, y, mask) {}
function on_mouse_rbtn_down(x, y, mask) {}
```

可以看到，鼠标的整个关键动作过程，都在掌握之中~ 我们先做个简单到不能再简单的按钮：

## 第一个按钮

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160206002.png)

我们先做个这样的按钮：鼠标按上去，颜色由灰`RGB(100, 100, 100)`变为黑`RGB(0, 0, 0)`. 

简单归简单，在此之前先看看我们要注意哪些东西：

- 按钮有大小，所以要定义它的坐标与长宽：`x, y, w, h`
- 按钮按下去变黑，说明按钮有它的状态：正常、按下去，可以用一个变量 `state` 来表示；为了长远着想，我们还要考虑鼠标移动到按钮上时（浮动）的这个状态，那么 `state`就有三个状态：`0, 1, 2` 分别表示正常、鼠标浮动、鼠标按下。
- 按钮有外观，比如颜色，文字（，可能还有图标之类的）等，这些也可以事先定义好。

接下来就可以在编辑器中定义按钮了：

```
// 
var button = {
    x: 200,
    y: 100,
    w: 100,
    h: 28,
    text: "button",
    font: gdi.Font("Segoe UI", 12),
    state: 0,
}
```

然后先把按钮画出来：

```
var DT_LEFT = 0x00000000;
var DT_CENTER = 0x00000001;
var DT_RIGHT = 0x00000002;
var DT_VCENTER = 0x00000004;
var DT_CALCRECT = 0x00000400;
var DT_NOPREFIX = 0x00000800;
var DT_END_ELLIPSIS = 0x00008000;

function RGB(r, g, b) {
	return (0xff000000 | (r << 16) | (g << 8) | (b));
}

var ww = 0;
var wh = 0;

function on_size() {
    ww = window.Width;
    wh = window.Height;
}

function on_paint(gr) {
    // bg
    gr.FillSolidRect(0, 0, ww, wh, RGB(245, 245, 245));
    
    var bt_color;
    
    if (button.state == 2) {
        bt_color = RGB(0, 0, 0);
    } else {
        bt_color = RGB(100, 100, 100);
    }
        
    
    gr.FillSolidRect(button.x, button.y, button.w, button.h, bt_color);
    gr.GdiDrawText(button.text, button.font, RGB(255, 255, 255),
    button.x, button.y, button.w, button.h, DT_CENTER | DT_VCENTER | DT_CALCRECT);
  
}
```

这样，就可以看到上面的图片上的情形了。不过这时，还不能用鼠标操作它。再加上鼠标操作的情形：

- 按下去 - 按钮颜色变深
- 松开 - 按钮颜色归位

```
function on_mouse_lbtn_down(x, y, mask) {
    // 判断鼠标指针是否在按钮上
    if (x > button.x && x < button.x + button.w && y > button.y && y < button.y + button.h) {
        button.state = 2;
        window.Repaint();
    }
}

function on_mouse_lbtn_up(x, y, mask) {
    if (button.state == 2) {
        button.state = 0;
        window.Repaint();
    }
}
```

上面的三段编码都 copy 到编辑器上去确定，就能得到一个简单的 button 了。

不过这些还不够：按钮这东西是很常见的，两三个是常事，四五个不少见，不能每次都定义那么长的 `button = {}` 吧？而且如果好几个脚本中都有按钮，每个脚本中都要重写一遍……论谁都会烦的。

所以第二个例子，我们来利用利用 Javascript 的构造函数，做出可以复用的按钮。

## 第二个例子

Javascript 是面向对象的语言，声称`一切皆是对象`，不过它没有`类`这个概念，一般情况下用构造函数来模拟。具体不说了，一般的 Javascript 教材都会在前两三章说到。

先来写一个 button 的构造函数，因为我们这里是带文字的按钮，就叫 `TextButton` 好了：

```
// 定义函数，添加 7 个属性
function TextButton(text, font, color1, color2) {
    this.x = 0;
    this.y = 0;
    this.w = 0;
    this.h = 0;
    this.state = 0; // 0: normal, 1: hover, 2: down
    this.text = text;
    this.colors = [color1, color2];
}
// 在 prototype 上添加方法
// 我们要画按钮，还要监控它在鼠标按下、松开时的动作， 这些由 draw, lbtn_up,
// lbtn_down；判断指针是否在按钮上是个比较独立自主的过程，单独用个 is_mouse_over
// 方法
TextButton.prototype.draw = function(graphic, x, y, w, h) {
    // 设定属性值
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
    // 在 [x, y, w, h] 这个区域内画 button
    // button background
    graphic.FillSolidRect(x, y, w, h, this.state == 2 ? this.colors[1] : this.colors[0]);
    // 画文字
    graphic.GdiDrawText(this.text, this.font, RGB(255, 255, 255),
        x, y, w, h, DT_CENTER | DT_VCENTER | DT_CALCRECT);
    }
    
TextButton.prototype.is_mouse_over = function(x, y) {
    return (x > this.x && x < this.x + this.w &&
        y > this.y && y < this.y + this.h);
    }
    
TextButton.prototype.lbtn_down = function(x, y, mask) {
    if (this.is_mouse_over(x, y)) {
        this.state = 2;
        window.Repaint();
    }
}

TextButton.prototype.lbtn_up = function(x, y, mask) {
    if (this.state == 2) {
        this.state = 0;
        window.Repaint();
        return true;
    } else {
        return false;
    }
}

```

这样，一个按钮模板就完成了。我们以后只要指定文字、字体、颜色，然后就能得到一个类似的按钮。如果我们需要的按钮变化比较大，这个模板已经套不上去了，那就新定义其它的构造函数吧。

完成程序：

```
// 定义需要的变量、常量
var DT_LEFT = 0x00000000;
var DT_CENTER = 0x00000001;
var DT_RIGHT = 0x00000002;
var DT_VCENTER = 0x00000004;
var DT_CALCRECT = 0x00000400;
var DT_NOPREFIX = 0x00000800;
var DT_END_ELLIPSIS = 0x00008000;

function RGB(r, g, b) {
	return (0xff000000 | (r << 16) | (g << 8) | (b));
}

var ww = 0;
var wh = 0;
var bt_font = gdi.Font("Segoe UI", 12);
var bt_color1 = RGB(100, 100, 100);
var bt_color2 = RGB(0, 100, 100);
var bt_color3 = RGB(100, 0, 100);

// 新建实例化的 button
var button1, button2;

button1 = new TextButton("hello", bt_font, bt_color1, bt_color2);
button2 = new TextButton("world", bt_font, bt_color1, bt_color3);

// 回调(事件)函数
// 可以看到，每个事件函数都比第一个例子短很多，虽然我们在前面多定义了个构造函数
// ，但如果我们要画的按钮越多的话，那么节省的代码量也会越多（人总是很懒的）

function on_size() {
    ww = window.Width;
    wh = window.Height;
}

function on_paint(gr) {
    // bg
    gr.FillSolidRect(0, 0, ww, wh, RGB(245, 245, 245));
    
    // buttons
    button1.draw(gr, 50, 50, 100, 28);
    button2.draw(gr, 50, 100, 100, 28);
    
}

function on_mouse_lbtn_down(x, y, mask) {
    button1.lbtn_down(x, y);
    button2.lbtn_down(x, y);
}

function on_mouse_lbtn_up(x, y, mask) {
    if (button1.lbtn_up(x, y)) {
        // do something
    }
    if (button2.lbtn_up(x, y)) {
        // do something
    }
}
```

### 效果图
![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160206003.png)


## 第三个例子

这次我们做几个带有**具体功能**的按钮，而且也是第三个例子了，所以这次做个比较完善点的, 以后可以直接用，或者简单修改后用的按钮。

### 示意图

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160207003.png)

先想象下其它软件上的按钮，它们的触发过程是怎么样的？

- 鼠标移动到按钮上，按钮会变化（一般背景多出个浮动效果）
- 鼠标点下去，按钮又变成按下去的效果
- 鼠标松开，会触发按钮对应的功能
- 如果鼠标按下去后，你有点后悔按下了，那么把指针移到按钮外再松开，不会触发对应的功能

前面的 `TextButton` 是做不到上面的要求的，所以先对 `TextButton` 做点完善工作：

```
// 三个参数
// font: 按钮文字的字体
// colors: [color1, color2, color3] 这样的数组，分别表示三种状态的背景色
// func: 按钮所触发的功能函数
function TextButton(font, colors, func) {
    this.font = font;
    // colors: Array
    // [color_normal, hover, down]
    this.colors = colors;
    this.x = 0;
    this.y = 0;
    this.w = 0;
    this.h = 0;
    this.state = 0; // 0: normal, 1: hover, 2: down
    this.func = func;
}

// 添加方法
// 在 prototype 上添加的方法一般叫公共方法
TextButton.prototype = {
    is_mouse_over: function(x, y) {
        return (x > this.x && x < this.x + this.w &&
            y > this.y && y < this.y + this.h);
    },

    draw: function(gr, text, x, y, w, h) {
        this.x = x;
        this.y = y;
        this.w = w;
        this.h = h;
        if (this.colors[this.state]) {
            gr.FillSolidRect(x, y, w, h, this.colors[this.state]);
        }
        gr.GdiDrawText(text, this.font, RGB(255, 255, 255),
            x, y, w, h, DT_VCENTER | DT_CENTER | DT_CALCRECT);
    },

    change_state: function(s) {
        if (s == this.state) {
            return;
        }
        this.state = s;
        window.Repaint();
    },

    down: function(x, y, mask) {
        if (this.is_mouse_over(x, y)) {
            this.change_state(2);
        }
    },

    up: function(x, y, mask) {
        if (this.is_mouse_over(x, y)) {
            this.change_state(1);
            return true;
        } else {
            this.change_state(0);
            return false;
        }
    },

    move: function(x, y) {
        if (this.state == 2) {
            return;
        } else {
            if (this.is_mouse_over(x, y)) {
                this.change_state(1);
            } else {
                this.change_state(0);
            }
        }
    },

    leave: function() {
        this.change_state(0);
    },

    on_click: function(x, y) {
        if (!this.func || typeof this.func != "function") {
            return;
        }
        this.func(x, y);
    }
}
```

上面添加了一系列方法，其中 `draw` 方法在 `on_paint` 函数中执行，`move, down, up, leave` 则对应鼠标的移动、按下、松开、离开（面板）这几个关键动作。各个方法的逻辑并不复杂，看不懂还请有点耐心。

```
// 定义必须的常量、变量、函数
var DT_LEFT = 0x00000000;
var DT_CENTER = 0x00000001;
var DT_RIGHT = 0x00000002;
var DT_VCENTER = 0x00000004;
var DT_CALCRECT = 0x00000400;
var DT_NOPREFIX = 0x00000800;
var DT_END_ELLIPSIS = 0x00008000;

function RGB(r, g, b) {
    return (0xff000000 | (r << 16) | (g << 8) | (b));
}

var ww = 0;
var wh = 0;

function on_size() {
    ww = window.Width;
    wh = window.Height;
}

// 声明按钮的字体，颜色
var bt_font = gdi.Font("Segoe UI", 12);
var bt_colors = [RGB(100, 100, 100), RGB(0, 100, 100), RGB(0, 50, 50)];
// buttons 都存储在一个数组里，方便集中处理；
var bt = [],
    bt_len = 0;
    
// 添加实例化的按钮
// 添加三个
bt.push(new TextButton(bt_font, bt_colors, function() {
    fb.Prev();
}));
bt.push(new TextButton(bt_font, bt_colors, function() {
    fb.Next();
}));
bt.push(new TextButton(bt_font, bt_colors, function() {
    fb.PlayOrPause();
}));

bt_len = bt.length;

// 事件函数
function on_paint(gr) {
    // bg
    gr.FillSolidRect(0, 0, ww, wh, RGB(245, 245, 245));

    // buttons
    bt[0].draw(gr, "前一首", 20, 20, 60, 28);
    bt[1].draw(gr, "后一首", 100, 20, 60, 28);
    bt[2].draw(gr, "播放/暂停", 180, 20, 80, 28);
}

// 由于每一个按钮的事件都是一样的，用个循环遍历一下
function on_mouse_move(x, y) {
    for (var i = 0; i < bt_len; i++) {
        bt[i].move(x, y);
    }
}

function on_mouse_lbtn_down(x, y, mask) {
    for (var i = 0; i < bt_len; i++) {
        bt[i].down(x, y, mask);
    }
}

function on_mouse_lbtn_up(x, y, mask) {
    for (var i = 0; i < bt_len; i++) {
        if (bt[i].up(x, y, mask)) {
            bt[i].on_click();
            break;
        }
    }
}

function on_mouse_lbtn_dblclk(x, y, mask) {
    for (var i = 0; i < bt_len; i++) {
        bt[i].down(x, y);
    }
}

function on_mouse_leave() {
    for (var i = 0; i < bt_len; i++) {
        bt[i].leave();
    }
}
```

外观来看，似乎并不怎么出彩的样子，不过这只是个示范。按钮的背景，文字都是在脚本里绘制的，而且还很简单，如果想做比较复杂的或者比较漂亮的按钮，那就要看你自己的美工水平了。本人并不擅长这个，所以没法随手就做个漂亮的按钮给你们看。

代码优化的余地还有不少，比如那个 `for` 循环，怎么弄看你们自己了。按钮控件好多人做，所以例子很多，**Jensen** 和 **extremeHunter1972** 的按钮带动画效果，是目前看到的比较好的了；br3tt 的按钮则灵活性很高，用的时候很舒服；动画效果最好的目前是看到的一个日本人做的，然而他并没有分享，so...

下次说什么呢？进度条和音量条吧。话说在写教程期间所有的示例都是现写的，写完估计可以积累不少吧... 都是准备弃坑的人了，哎...
