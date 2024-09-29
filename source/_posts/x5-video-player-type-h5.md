---
title: x5-video-player-type='h5'
date: 2024-09-29 17:30:36
tags: 前端
---
```
<video
id="videoID"
src="video.mp4"
poster="loadbg.jpg"　视频封面
preload="auto"
x-webkit-airplay="allow"
x5-video-player-type="h5"　        启用H5播放器,是wechat安卓版特性
x5-video-player-fullscreen="true"　全屏设置，设置为 true 是防止横屏
x5-video-orientation="portraint"　 播放器支付的方向，landscape横屏，portraint竖屏，默认值为竖屏
webkit-playsinline="true"　        这个属性是iOS 10中设置可以让视频在小窗内播放，也就是不是全屏播放
playsinline="true"　               ios微信浏览器支持小窗内播放
style="object-fit:fill">
</video>
```
