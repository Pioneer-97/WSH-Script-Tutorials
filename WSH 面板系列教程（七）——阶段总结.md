# WSH 面板系列教程（七）——阶段总结

标签（空格分隔）： JScript WSH foobar2000 

---

前面的内容，如果懂个差不多的话，或者就算没懂个差不多，但直到怎么应用 `TextButton` 和 `Slider` 这两个类的用法，写出一个类似

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Image%201.png)

这样的界面，是问题不大的。这个是 windows10 的默认播放器 Groove 的播放控制面板（当然，那些动态效果，以及一些细节，暂时还是不去管它们为好）。我自己用的 foobar2000 的播放控制面板前面贴了好多次了，之所以不拿那个做例子，是因为这些图标，windows10 系统里自带的，就算不是 windows10 系统，新装个字体就可以了。

> ###字体下载地址：
vdisk.weibo.com/s/qd8l7QWqaHeZy，里面那个 **segmdl2.ttf** 就是的。

嗯，我们这一篇的内容，就是模仿上面的图片，做出一个类似的播放控制面板出来。

## 先冷静一下

在我开始接触 WSH 脚本的时候，还很少看到一块 WSH 脚本上有这么多东西的，那时候的脚本一般很单纯，要么全是按钮，要么只有一个进度条（复杂的脚本也有，不过不多），然后杂七杂八的面板，用 CUI+PSS 组装起来。单纯又单纯的好处，复杂也有复杂的优点，只要别怕就行了，因为难度差不多。

就像画画一样，在什么地方画什么东西，心里事先要有个大概的估计，然后做的过程中一步一步的调整细节。而我们现在是照着例子画，已经省去了设计这个过程，只要考虑编程了。

这个面板，分上下两部分，上面是进度条，进度条两边是时间；下面是六个按钮，六个按钮均布且居中。

- 时间是文字，用 `GdiDrawText()` 方法就可以画出
- 进度条参考之前写过的进度条，但上面多了个圈圈，这个圈圈可以用图片做，画图片用 `DrawImage()` 这个接口
- 下面的按钮，图标形状在前面下载的字体里都有，全都有！直接画也好，做成图片再画也好，都可以。

好了，到这里应该知道，复杂的东西之前已经全部准备好了，按钮和进度条控件都是现成的，虽然要做一些修改，但那已经不麻烦了。余下的，就是要准备图片文件，之前想说来着，这一篇正好说明一下。

## 编程准备

有些东西，是几乎每次写脚本都要写的，这次我们也事先准备好：

```
// 常用函数

// 用在 gr.DrawString() 接口中
function StringFormat() {
	var h_align = 0,
		v_align = 0,
		trimming = 0,
		flags = 0;
	switch (arguments.length) {
		case 4:
			flags = arguments[3];
		case 3:
			trimming = arguments[2];
		case 2:
			v_align = arguments[1];
		case 1:
			h_align = arguments[0];
			break;
		default:
			return 0;
	};
	return ((h_align << 28) | (v_align << 24) | (trimming << 20) | flags);
}

function RGBA(r, g, b, a) {
	return ((a << 24) | (r << 16) | (g << 8) | (b));
}

function RGB(r, g, b) {
	return (0xff000000 | (r << 16) | (g << 8) | (b));
}

// 常用常数，参考 Flag.txt
var DT_LEFT = 0x00000000;
var DT_CENTER = 0x00000001;
var DT_RIGHT = 0x00000002;
var DT_VCENTER = 0x00000004;
var DT_WORDBREAK = 0x00000010;
var DT_CALCRECT = 0x00000400;
var DT_NOPREFIX = 0x00000800;
var DT_END_ELLIPSIS = 0x00008000;
var DT_SINGLELINE = 0x00000020;

```

> 每个经常给 foobar2000 写脚本的人，都会准备一个公共的文件，里面就是这些常用的常数，对象，类，函数之类的东西，本人也有，如果你们下载过 Mnlt2（改）这个界面配置，就能在里面找到个 `common3.js` 文件。不然每次都要复制一遍到脚本中，略烦。

然后我们先画好面板背景，包括固定面板的高度：

```
var ww, wh;
var back_color = RGB(50, 89, 3);
var text_color = RGB(250, 250, 250);
// 限制面板高度的最大和最小值，两值相等
// 则面板的高度固定不变，同样的还有 window.MaxWidth 
// 和 window.MinWidth.
window.MaxHeight = window.MinHeight = 130;

function on_size() {
    ww = window.Width;
    wh = window.Height;
}

function on_paint(gr) {
    // Bg, 如果看到颜色却不知道 RGB 值，可以用取色软件，
    // 比如本人用的是 PicPick，可以截图，取色
    gr.FillSolidRect(0, 0, ww, wh, back_color);
}
```

### 准备进度条上的 nob 图片

进度条上有个小圈，你可以在 PS 中先画好，然后用 `var image = gdi.Image(path_to_image)` 来载入，不过这样简单的图形，我们也可以用脚本直接画：

``` 
var nob_image = null;

function prepare_images() {
    var g;
    
    nob_image = gdi.CreateImage(14, 14);
    // 这里的 g 和 on_paint 的参数 gr 是同个品种
    g = nob_image.GetGraphics();
    g.SetSmoothingMode(4);
    g.FillEllipse(1, 1, 12, 12, text_color);
    g.FillEllipse(3, 3, 8, 8, back_color);
    g.SetSmoothingMode(0);
    nob_image.ReleaseGraphics(g); 
}

prepare_images();
```

### 画出进度条和时间

因为之前写的 `Slider` 是没有那个圈圈的，加上这个进度条很细，鼠标操作的空间太小，所以我们要做点修改：

```
var Slider = function(nob_img, func_get, func_set) {
	this.is_drag = false;
	this.get = (function () {
		return typeof func_get == "function" ? func_get : function() {};
	})();
	this.set = (function () {
		return typeof func_set == "function" ? func_set : function () {};
	})();
	this.pos = this.get();
    this.nob_img = nob_img ? nob_img : null;
}
Slider.prototype.draw = function(gr, x, y, w, h, y_offset, active_color, inactive_color) {
    if (h <= y_offset * 2) {
        y_offset = 0;
    }
    // 进度条背景
	gr.FillSolidRect(x, y+y_offset, w, h - y_offset * 2, inactive_color);
	if (this.pos > 0 && this.pos <= 1) {
		gr.FillSolidRect(x, y+y_offset, w * this.pos, h-y_offset*2, active_color);
	}
    // nob 图片
    if (this.nob_img) {
        var img_w = nob_image.Width;
        if (!(this.pos >= 0)) {
            this.pos = 0;
        }
        gr.DrawImage(this.nob_img, x+w*this.pos - img_w/2, (h - img_w)/2+y, img_w, img_w,
            0, 0, img_w, img_w, 0, 255);
    };
    
	this.x = x;
	this.y = y;
	this.w = w;
	this.h = h;
}
Slider.prototype.is_mouse_over = function(x, y) {
	return (x > this.x && x < this.x + this.w && y > this.y && y < this.y + this.h);
}

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
		x -= this.x ;
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

然后，我们把进度条画下来，然后顺便也把两边的时间画下来。以后写脚本也是这样，不管这个功能多复杂，先在面板上画下来，然后再想其它的。

```
// playback seeker
var sk; 
var time_font = gdi.Font("Segoe UI Semibold", 12);
var length_time = "0:00",
     playback_time = "0:00";

sk = new Slider(nob_image, 
    function() {
        return fb.PlaybackTime / fb.PlaybackLength;
    },
    function (pos) {
        fb.PlaybackTime = fb.PlaybackLength * pos;
    });
     
// 改写 on_paint

function on_paint(gr) {
    // Bg, 如果看到颜色却不知道 RGB 值，可以用取色软件，
    // 比如本人用的是 PicPick，可以截图，取色
    gr.FillSolidRect(0, 0, ww, wh, back_color);
    
    // 进度条
    sk.draw(gr, 70, 25, ww-140, 16, 7, text_color & 0xeeffffff, text_color & 0x20ffffff);
    // 进度条两边的时间
    gr.GdiDrawText(playback_time, time_font, text_color, 10, 20, 50, 26, DT_VCENTER | DT_CENTER | DT_CALCRECT);
    gr.GdiDrawText(length_time, time_font, text_color, ww-60, 20, 50, 26, DT_VCENTER | DT_CENTER | DT_CALCRECT);
    
}

// 相关的回调函数，等按钮全画好后再添加
// ...
```

进度条跟时间应该出来了，差不多是这个样子：

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160210003.png)

### 准备按钮图片

嗯，这次按钮跟上一次不太一样了，这次是图片按钮，所以先要准备图片。一般图片都是矩形（别被那些畸形怪状的图标给迷惑了，它们都是方的）。

修改下上面的 `prepare_images()`，就像创建 `nob_image` 一样，这次我们要再创建几个图标图片。正如四大天王有五个一样，六个图标，肯定不止六张图片了，本人掐指一算，觉得至少得 7 张才能糊弄过去。

还记得之前下载的那个字体吗？用**字符映射表**软件打开，可以查看某个字符对应的 unicode 编码：

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160210004.png)

那个 `U+E100` 就是所谓的 unicode 编码。

```
var nob_image = null;
var bt_images = {};

function prepare_images() {
    var g;
    var ico_font = gdi.Font("Segoe MDL2 Assets", 15, 0);
    
    nob_image = gdi.CreateImage(14, 14);
    // 这里的 g 和 on_paint 的参数 gr 是同个品种
    g = nob_image.GetGraphics();
    g.SetSmoothingMode(4);
    g.FillEllipse(1, 1, 12, 12, text_color);
    g.FillEllipse(3, 3, 8, 8, back_color);
    g.SetSmoothingMode(0);
    nob_image.ReleaseGraphics(g); 
    
    // 开始画图标
    // 图标图片都是相同的，只有中间的字符不同
    
    // 之前估计有误，得 8 个图标图片
    var ico_name = ["prev", "pause", "play", "next", "volume", "shuffle", "repeat", "repeat1"];
    // 对应的 unicode 编码
    var ico_code = ["\uE100", "\uE103", "\uE102", "\uE101", "\uE15D" , "\uE14B", "\uE149", "E1CC"];
    var len = ico_code.length;
    var w = 30;
    
    for (var i = 0; i < len; i++) {
		bt_images[ico_name[i]] = gdi.CreateImage(w, w);
		g = bt_images[ico_name[i]].GetGraphics();

		g.SetTextRenderingHint(3);
		g.DrawString(ico_code[i], ico_font, text_color, 0, 0, w, w, StringFormat(1, 1));
		g.DrawString(ico_code[i], ico_font, text_color, 0, 0, w, w, StringFormat(1, 1));
		g.SetTextRenderingHint(0);

		bt_images[ico_name[i]].ReleaseGraphics(g);
	}
	
	// repeat 和 shuffle 图标的状态背景，一个半透明圆
	bt_images["round"] = gdi.CreateImage(w, w);
	g = bt_images["round"].GetGraphics();
	g.SetSmoothingMode(4);
	g.FillEllipse(0, 0, w, w, 0x20000000);
	g.SetSmoothingMode(0);
	bt_images["round"].ReleaseGraphics(g);
    
}

prepare_images();

```

### 新的按钮 Button

这次是图片按钮，对前面的 `TextButton` 做一些修改：

```
var Button = function(func) {
	this.func = func;
	this.state = 0;
}
Button.prototype.is_mouse_over = function(x, y) {
	return (x > this.x && x < this.x + this.w &&
			y > this.y && y < this.y + this.h);
};
Button.prototype.draw = function(gr, img, x, y) {
	this.x = x;
	this.y = y;
	this.w = img.Width;
	this.h = img.Height;
	var alpha = 255;
	if (this.state == 2) {
		alpha = 100;
	}
	gr.DrawImage(img, x, y, this.w, this.h, 0, 0, this.w, this.h, 0, alpha);
}

Button.prototype.is_mouse_over = function(x, y) {
	return (x > this.x && x < this.x + this.w &&
			y > this.y && y < this.y + this.h);
}

Button.prototype.change_state = function(s) {
	if (s == this.state) {
		return;
	}
	this.state = s;
	window.Repaint();
}
Button.prototype.down = function (x, y) {
	if (this.is_mouse_over(x, y)) {
		this.change_state(2);
	}
}

Button.prototype.up = function(x, y) {
	if (this.is_mouse_over(x, y)) {
		this.change_state(1);
		return true;
	} else {
		this.change_state(0);
		return false;
	}
}

Button.prototype.move = function(x, y) {
	if (this.state == 2) {
		return;
	} else {
		if (this.is_mouse_over(x, y)) {
			this.change_state(1);
		} else {
			this.change_state(0);
		}
	}
}

Button.prototype.leave = function() {
	this.change_state(0);
}

Button.prototype.on_click = function(x, y) {
	if (!this.func || typeof this.func != "function") {
		return ;
	}
	this.func(x, y);
}
```

变动的地方很少. 

然后将按钮画上去，再写出需要的事件函数，就 OK 了，前面东一段西一段代码很乱，到[这里]()看完整代码吧。
（音量功能未完成。）

这下，基本就完工了，差不多下图那个样子。不过还有很多细节可以修改，比如我觉得这个面板太高了点，比如各个部件的大小，位置等等。

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160210007.png)

另外还可以扩展的就是，Groove 的底栏是响应式布局的，它很窄的时候才是这个样子，如果窗口被拉宽，就不是这个样子了。响应式布局这个名字很拉风——然而在 windows10 还没有诞生的时候 foobar2000 的界面就已经有应用了，不信看看 eiko 的播放列表，eiko 是什时候的事情了... 那时候还没 win7 呢。

