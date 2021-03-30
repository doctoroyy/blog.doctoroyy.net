---
title: 关于 web 视频播放
date: 2021-03-30 22:23:50
tags: 前端
---

# 关于 web 视频播放

### 1. 视频的格式、编码

格式：视频格式也叫视频的封装格式，通俗地讲，视频格式就是一种规范，定义了在什么地方（位置）放什么内容。常见的格式有 mkv，mp4，mov，flv。

编码：视频是 音频 + 图像 的集合，所以视频编码实际上要分成音频编码和图像编码。编码方式实际上描述的就是数据的压缩方式。常见的编码有 h264 、vp8、vp9，以及还没普及的 h265

<!-- more -->
### 2. 浏览器如何解析一个视频？

[花椒前端基于WebAssembly 的H.265播放器研发](https://zhuanlan.zhihu.com/p/73772711)

1. video 标签创建一个 DOM 对象，实例化一个 WebMediaPlayer
2. player 驱使 Buffer 请求多媒体数据
3. FFmpeg 进行解封装和音视频解码
4. 把解码后的数据传给相应的渲染器对象进行渲染绘制
5. video 标签显示或声卡播放

![%E5%85%B3%E4%BA%8E%20web%20%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%20d9b5d2946e3f444b82a369e48f57a400/Untitled.png](https://s1.vika.cn/space/2021/03/30/589444a9095540d2942791f679ef13fd?attname=image.png)

### 3. 浏览器对各种格式、编码的支持度

[HTML5 Video Converter | Encoding.com](https://www.encoding.com/html5-video-codec/)

![%E5%85%B3%E4%BA%8E%20web%20%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%20d9b5d2946e3f444b82a369e48f57a400/Untitled%201.png](https://s1.vika.cn/space/2021/03/30/1bc1a37b368149d2990989a96a913dea)

![%E5%85%B3%E4%BA%8E%20web%20%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%20d9b5d2946e3f444b82a369e48f57a400/Untitled%202.png](https://s1.vika.cn/space/2021/03/30/d364060af66144e297b8ad7541a68ed4)

在实际测试过程中，mkv 格式、h264 编码的视频在 Mac  Chrome 器中也可以播放，但在 Mac PC Safari 中不能播放

### 4. 我们如何准确知道一个视频能不能正常播放？

先说结论：我们并不能精确知道视频能否播放。

现代浏览器提供了两个 API 用来检测浏览器对视频的支持度。

- [window.MediaSource.isTypeSupported](https://developer.mozilla.org/en-US/docs/Web/API/MediaSource/isTypeSupported), 这个方法支持传入一个包含视频 mimeType 和 codecs 的字符串，返回一个布尔值。

```jsx
var mimeCodec = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"';

if ('MediaSource' in window && MediaSource.isTypeSupported(mimeCodec)) {
  var mediaSource = new MediaSource;
  //console.log(mediaSource.readyState); // closed
  video.src = URL.createObjectURL(mediaSource);
  mediaSource.addEventListener('sourceopen', sourceOpen);
} else {
  console.error('Unsupported MIME type or codec: ', mimeCodec);
}
```

- [video.canPlayType](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/canPlayType), 这个方法接受参数与上一个函数一致, 返回一个不精确的字符串描述

```jsx
type CanPlayTypeResult = "" | "maybe" | "probably";

/**
 * 检测您的浏览器是否能播放不同类型的视频
 *
 * @param type 可播放类型，'video/mp4; codecs="avc1.64001E, mp4a.40.5"'
 */
public canPlayType(type: string): CanPlayTypeResult;
```

接下来做一个简单的实验：

样品: 

1. 一个 mp4 格式、mpeg4 编码的视频 [1602656111421.mp4](https://s1.vika.cn/space/2020/10/21/95ac31d26f4048a9b7d087106670a0f9?attname=1602656111421.mp4) (video/mp4;codecs="mpeg4,aac")
2. macOS 10.14.6
3. Chrome 89.0.4389.90

在播放前用这 2 个函数检测, 打印结果

```jsx
console.log('avinfo: ', avinfo);
console.log('isTypeSupported: ', window.MediaSource.isTypeSupported(avinfo));
console.log('canplayType: ', document.createElement('video').canPlayType(avinfo));
```

结果如下：

![%E5%85%B3%E4%BA%8E%20web%20%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%20d9b5d2946e3f444b82a369e48f57a400/Untitled%203.png](https://s1.vika.cn/space/2021/03/30/d046027ce0b7441a9956e822540dc08d)

结果表明，当前视频不支持，也确实不能正常播放。

于是我做了第二次实验：

一个 mp4 格式、h264编码的视频 [bandicam 2021-01-28 11-12-47-568.mp4](https://s1.vika.cn/space/2021/03/20/068375b3c98b455586258898e7be0aa8?attname=bandicam%202021-01-28%2011-12-47-568.mp4) (video/mp4;codecs="h264,aac")

![%E5%85%B3%E4%BA%8E%20web%20%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%20d9b5d2946e3f444b82a369e48f57a400/Untitled%204.png](https://s1.vika.cn/space/2021/03/30/5a56cd6d28684b83ba4b977675dfc213)

这次打印的结果表明依然是不支持，然而事实确是：**可以正常播放**？？？！！！

对于完全不能播放（视频和音频）的情况，算是一种异常，开发者可以明确知道；

但是对**音频**或者**视频**二者之一不能播放的情况，只有依靠这两个函数来处理了，但根据上面的测试结果，这 2 个函数的检测实际上是不准确的，所以如果想要在视频播放异常加提示这件事情就变得困难

基于上述这个浏览器自身检测 API 不可信任的结论，我们似乎只能在用户侧维护一套浏览器普遍支持的视频 格式/编码 集合，如果不在这个集合里面就弹出类似这种提示，想要准确在合适的时机弹出，暂时没有找到比较完美的解决方案

![%E5%85%B3%E4%BA%8E%20web%20%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%20d9b5d2946e3f444b82a369e48f57a400/Untitled%205.png](https://s1.vika.cn/space/2021/03/30/f0caf28d84c9430bb7db4714a771185b)

### 5. 扩展浏览器播放能力：如何在 web 端播放浏览器不支持的视频？

目前主要有 2 个方案：

1. web 端转码（本地调用类似 FFmpeg、libde265 的 .wasm 版本解码）

    淘宝前端做过类似的事情，但在实践中存在视频帧数不稳定、视频播放进度调整出现 bug 的问题，总的来说，截止到这篇文章发表的时候，该方案基本还处于小规模试验阶段（这篇文章提到一点是现代浏览器不能支持众多视频编码可能不是技术问题，而是有些编码解码器使用是要收费的）

    [Taobao FED | 淘系前端团队](https://fed.taobao.org/blog/taofed/do71ct/web-player-h265/)

    花椒直播开发过一个 h265 播放器，但尚未开源，不过在文章评论页下找到一个开源项目：

    [goldvideo/h265player](https://github.com/goldvideo/h265player)

    项目主页给了一个 demo：[https://github.com/goldvideo/h265player](https://github.com/goldvideo/h265player)， 体验下来感觉也不是十分流畅，另外就是拖进度条时会出现页面 video 加载 loading 框不断跳动的问题，不是一个十分完美的版本，但是看起来可用

    另外又有国人维护的另一个 EasyPlayer 的项目，看介绍已经投入生产环境使用，最近一次更新在 4 天前，看起来也有一点希望的样子

    [tsingsee/EasyPlayer.js](https://github.com/tsingsee/EasyPlayer.js)

2. 服务端转码

    接云服务商转码服务，返回通用格式/编码的视频流