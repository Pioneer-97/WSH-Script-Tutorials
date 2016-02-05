# WSH 面板系列教程（四）——完善封面显示

标签（空格分隔）： foobar2000 WSH JScript

---


上一篇中，我们做了个很不错的封面显示面板，效果很好，不过太简陋了点，如果是作为如下图 fusion 的左下角，那确实够了，不过作为单独的一块能显示大封面的面板，还不够：起码，我们需要它能显示封底、碟片和艺术家，可以右键切换图片，并且提供是否纠正宽高比的选项。

![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160205002.png)

好了，分析下需求：

- 可以切换封面
- 提供是否纠正宽高比选项

那在脚本中就需要多加两个变量来存储我们的要求（的状态）：
```
// 默认显示封面(front)
var cover_id = 0; 
var keep_aspect_ratio = window.GetProperty("纠正宽高比", true);
```
上面，因为我希望每次重启 foobar2000 它都能知道上次关闭时`keep_aspect_ratio`的状态，所以这个变量值要存在一个保险的地方，等到重启 foobar2000 的时候可以读取到上一次的状态，而`window.GetProperty(prop_name, value)`这里就很有用。

通过`右键菜单 > 属性`打开属性窗口，可以查看、编辑保存的属性。
![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160205009.png)

再对昨天的程序做一点修改：
```
var g_metadb;
var AlbumArtId = {
	front: 0,
	back: 1,
	disc: 2,
	icon: 3
};

function on_metadb_changed(metadb, fromhook) {
	// 
	g_metadb = fb.IsPlaying ? fb.GetNowPlaying() : null;
	if (g_metadb) {
	// 这里 AlbumArtId.front 改为 cover_id.
		utils.GetAlbumArtAsync(window.ID, g_metadb, cover_id);
	} else {
		g_img = null;
		window.Repaint();
	}
}

on_metadb_changed();

function on_playback_new_track(metadb) {
	on_metadb_changed();
}

function on_playback_stop(reason) {
	if (reason != 2) {
		on_metadb_changed();
	}
}


var g_img = null;

function on_get_album_art_done(metadb, art_id, image, image_path) {
	// 找到了封面，就存到 g_img 这个全局变量里，
	// 没找到，image == null, g_img 也会被清空为 null
	g_img = image;
	// 
	window.Repaint();
}

var ww = 0, wh = 0;

function on_size() {
	ww = window.Width;
	wh = window.Height;
}

function on_paint(gr) {

	//bg
	gr.FillSolidRect(0, 0, ww, wh, 0xffeeeeee);
	if (g_img) {
	// 这里加个判断
		if (keep_aspect_ratio) {
			var scale_w = ww / g_img.Width;
			var scale_h = wh / g_img.Height;
			var scale = Math.min(scale_w, scale_h);
			var pos_x = 0;
			var pos_y = 0;
			// 计算图片位置 (x, y)
			if (scale_w < scale_h) {
				pos_y = (wh - g_img.Height * scale) / 2;
			} else {
				pos_x = (ww - g_img.Width * scale) / 2;
			}
			// 画图
			gr.DrawImage(g_img, pos_x, pos_y, g_img.Width * scale, g_img.Height * scale,
					0, 0, g_img.Width, g_img.Height, 0, 255);
		} else {
			gr.DrawImage(g_img, 0, 0, ww, wh, 0, 0, g_img.Width, g_img.Height, 0, 255);
		}
	}

}

```

然后是添加右键菜单：

回想一下鼠标右键点击然后弹出菜单的过程：鼠标移动到面板上，右键按下，松开，然后菜单弹出——OK~ 查查`callbacks.txt`文档，很容易就能发现跟鼠标右键有关的事件函数有这两个：

- `on_mouse_rbtn_down`
- `on_mouse_rbtn_up`

由于菜单只在右键松开的时候弹出，那么很显然我们只需要用到`on_mouse_rbtn_up`。

> 创建菜单是个很繁但很容易理解的事情，本人当初开始写脚本的时候，菜单写过一次忘一次，然后把别人的模板再拿过来抄...

先设计一下菜单该是什么样子，比如可以这样：

- 保持宽高比
`-------------'
- 封面
- 封底
- 碟片
- 艺术家

全是一级菜单。然后脚本这么写：

```
var MF_STRING = 0x00000000;

function on_mouse_rbtn_up(x, y, mask) {
    // 按照传统，当 SHIFT 键按下时，右键菜单是 WSH 面板的默认菜单，
    // 而正常情况下则是自定义菜单
    // 是否弹出自定义菜单根据 on_ouse_rbtn_up 的 return 值来判断
    if (mask == 4) {
        return false;
    }
    // 创建菜单 _menu
    var _menu = window.CreatePopupMenu();
    var ret;
    // 给 _menu 添加菜单项， 
    _menu.AppendMenuItem(MF_STRING, 1, "保持宽高比");
    // 复选框选项检查
    _menu.CheckMenuItem(1, keep_aspect_ratio);
    _menu.AppendMenuSeparator(); // 分割线
    _menu.AppendMenuItem(MF_STRING, 2, "封面");
    _menu.AppendMenuItem(MF_STRING, 3, "封底");
    _menu.AppendMenuItem(MF_STRING, 4, "碟片");
    _menu.AppendMenuItem(MF_STRING, 6, "艺术家");
    // 单选按钮状态检查
    _menu.CheckMenuRadioItem(2, 6, cover_id + 2)

    // 获取点击的菜单 ID
    ret = _menu.TrackPopupMenu(x, y);
    // 根据获得的菜单 ID 决定执行哪个命令
    if (ret == 1) {
        keep_aspect_ratio = !keep_aspect_ratio;
        window.SetProperty("保持宽高比", keep_aspect_ratio);
        window.Repaint();
    } else if (ret >= 2 && ret <= 6) {
        cover_id = ret - 2;
        on_metadb_changed();
    }
    // 用过之后，就要抛弃它，释放内存
    _menu.Dispose();
    return true;
}
```
![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160205010.png)

这样就得到想要的封面显示面板了，达到了跟 DUI 自带的封面显示器一样的效果。但因为它是 WSH 面板，所以它的用处还是比默认的 DUI 组件要广点——可以自定义背景色，可以自定义没有封面时的显示内容，可以在上面**加料**。

当然，各路大神可不会满足于实现一个仅仅相当于普通 DUI 组件的 WSH 封面显示器，太掉价了——WSH 封面显示的比较复杂的脚本有好几个，其中最强大的是 Jensen （此人是 WSH 脚本的拓荒者，WSH Panel Mod 一诞生，随之而来的就是他的一系列高质量、高效率的脚本）的 WSH-Cover，这个脚本你们肯定不陌生，早在 **Eiko**，**Shutter** 上就已经在使用了，dreamawake 大神的 dreamix，foobox 这两个界面配置也用的它，松尾庆军的各个皮肤基本也是用的它……嗯从 WSH Panel Mod 一诞生开始你们就已经在使用着最好的 WSH 封面显示脚本了。

封面脚本就到这里，后面还会用到，到时候再说。