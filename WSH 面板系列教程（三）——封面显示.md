# WSH 面板系列教程（三）——封面显示
标签（空格分隔）： foobar2000 WSH JScript

---

这次我们要画一个封面面板，顺便学点其它的东西。

封面面板，就是一块板上贴一张正在播放的歌曲的封面图片。审下题，先看看条件——正在播放的歌曲、封面、贴（画）。也就是我们：

- 要得到正在播放的歌曲是哪个；
- 还要获取它的封面图片； 
- 最后才是画上去。

获取正在播放的歌曲的句柄，在 `on_metadb_changed` 函数里：如果正在播放，那就发一个消息（`utils.GetAlbumArtAsync`，如果没有播放的歌曲，那就拉倒pass。

**正在播放的歌曲**并不是不变的，哪些情况下会变？无非三种：从停止状态开始播放、播放下一曲、停止播放。三种情况，用两个事件函数就可以监视了，`on_playback_new_track`: 播放新歌曲了，正在播放歌曲肯定变了，那我们重新获取一次句柄；`on_playback_stop`: 正在播放的歌曲没了，null 了，那就再去获取一次句柄。`on_metadb_changed` 也是事件函数，如果句柄变化了就执行它。

`utils.GetAlbumArtAsync()`：我上面说它是『发消息』，因为它不会立即返回图片，它只是告诉 foobar2000: “你给我找封面图片来！”然后 foobar2000 就屁颠屁颠去找了，因为找封面的时间一般会比较长，所以我们不能在这里等它找到封面再去做下一步，我们继续做我们该干的事情，等它找到了，载入了，我们再去处理。否则在干等的这段时间，foobar2000 是卡死的。
```
var AlbumArtId = {
    front: 0,
    back: 1,
    disc: 2,
    icon: 3
};
var g_metadb;

function on_metadb_changed(metadb, fromhook) {
    // fb.GetNowPlaying() 获取当前正在播放歌曲的句柄
    // fb.IsPlaying 判断是否正在播放
    g_metadb = fb.IsPlaying ? fb.GetNowPlaying() : null;
    if (g_metadb) {
        // 如果获取到了句柄，那就发个获取封面的消息
        utils.GetAlbumArtAsync(window.ID, g_metadb, AlbumArtId.front);
    } else {
        // 没获取到，则刷新图片
        g_img = null;
        window.Repaint();
    }
}
// 脚本载入的时候立即就执行一次找封面的任务
// 不然你要等播放下一首歌曲它才开始找封面
on_metadb_changed();

function on_playback_new_track(metadb) {
    on_metadb_changed();
}

function on_playback_stop(reason) {
    // reason: (integer, begin with 0): user, eof, starting_another
    if (reason != 2) {
        on_metadb_changed();
    }
}
```

『找到了封面图片』是一个事件，我们自然希望这个事件发生后会有个函数立即执行，我们可以在那个函数里刷新图片，事件函数自然是有的: `on_get_album_art_done`, 它有四个参数，foobar2000 找到图片后，会把图片载入内存，然后告诉我们：当初的句柄是哪个，当初找的是哪种图片，内存里的图片是哪个，图片路径是什么。这里我们只需要存一下图片本身就行。
```
var g_img = null;

function on_get_album_art_done(metadb, art_id, image, image_path) {
    // 找到了封面，就存到 g_img 这个全局变量里，
    // 没找到，image == null, g_img 也会被清空为 null
    g_img = image;
    // 
    window.Repaint();
}
```

到这里，我们已经可以获取我们想要的封面图片`g_img`，我们要在面板上把它画下来：
```
var ww = 0, wh = 0;

function on_size() {
    ww = window.Width;
    wh = window.Height;
}

function on_paint(gr) {
    //画 bg
    gr.FillSolidRect(0, 0, ww, wh, 0xffeeeeee);
    // 如果获取到了封面，就画封面
    if (g_img) {
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
    }
    
}
```
![](https://raw.githubusercontent.com/elia-is-me/WSH-Script-Tutorials/master/images/doc1/Pic_20160204016.png)

好了，三段程序一起，就能在 WSHMP 上显示封面了（如图示）。尽管例子很简单，却是完全可以去实战的代码，抄去使用完全没有问题。

这次的封面显示，用到的事件函数，主要是监视播放状态变化的两个事件函数。WSHMP 监视播放的事件函数好几个，参考 `callbacks.txt` 文件，搜索 `on_playback_xxx` 就行。

现成的例子不用可惜，下次就加点东西，把它变成一个比较完善的封面显示面板吧，加个右键菜单，可以切换显示不同种的图片：封面、封底、碟片、艺术家。下次的主要内容是怎么弄菜单。




