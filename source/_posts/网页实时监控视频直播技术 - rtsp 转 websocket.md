---
updated: '2020-01-21 08:00:00'
categories: code
excerpt: 按照上面的分析，要实现摄像头实时视频的网页直播，可行的技术方案之一是，接入摄像头 RTSP 协议视频流，然后通过 Websocket 推送到页面播放。
date: '2020-01-21 08:00:00'
tags:
  - Media
urlname: stream-live-video-websocket
title: 网页实时监控视频直播技术 - rtsp 转 websocket
---

# 分析


本文讨论的是监控行业的视频直播，这类应用的特点是实时性要求特别高，一般需要在1秒内，这样当用户控制摄像头云台时才能有较好效果。


已经有很多文章和讨论对相关技术以及特点进行了总结，在此不再赘述：

- [HLS](https://zh.wikipedia.org/zh-hans/HTTP_Live_Streaming) 10秒级延时
- [RTMP](https://zh.wikipedia.org/wiki/%E5%AE%9E%E6%97%B6%E6%B6%88%E6%81%AF%E5%8D%8F%E8%AE%AE) 秒级延时
- [RTSP](https://zh.wikipedia.org/wiki/%E5%8D%B3%E6%99%82%E4%B8%B2%E6%B5%81%E5%8D%94%E5%AE%9A) 毫秒级延时

其中 RTSP 以及 RTP 广泛运用于监控行业，主流的摄像头厂商都会支持该协议，然而在 Web 的世界里，HTTP 才是标准，其他的直播技术都是基于 HTTP 实现的，比如 HLS、HTTP-FLV，由于是基于文件分发、所以这些方案的缺点是延迟较大。


有没有既延迟小，又能在网页中播放的视频技术呢，随着新技术的出现这个问题也有了解决办法，首先视频传输：

- WebSocket 可从服务器端主动推送数据到客户端，支持二进制数据、支持数据分发（广播模式）；
- Web RTC 适用于点对点的通信，可将内部网络的视频源通过 P2P 的方式发到客户页面，延迟更小；

然后视频解码： * H264 现代浏览器以及摄像头都支持 H264 视频编解码，可以采用 HTML 的 MediaSourceExtension 进行硬解码；

- H265 较新的摄像头一般推荐采用 H264（HEVC）对视频编码，遗憾的是目前（2020-01-21）浏览器都不支持，网上有通过 WASM FFmpeg 解码的案例，但是性能不佳；

# 实现


按照上面的分析，要实现摄像头实时视频的网页直播，可行的技术方案之一是，接入摄像头 RTSP 协议视频流，然后通过 Websocket 推送到页面播放。


## RTSP 接入


使用 FFMpeg 的 libav 进行 RTSP 视频数据的读取，其中关键点是低延迟，可参考以下 FFmpeg 参数


```text
ffplay -i "rtsp://admin:123456@192.168.1.99/h264/ch3/sub/av_stream" -fflags nobuffer -flags low_delay -framedrop -strict experimental -probesize 32

```


## fMp4 封包


由于 HTML5 MediaSourceExtension 支持的 H264 数据格式为 fragmented MP4，所以需要对 RTSP 中读取到的 H264 NAL 进行打包处理，打包完成后通过 Websocket 发送到页面。


## MediaSourceExtension 解码


页面收到 fMP4 数据后将其发送到 SourceBuffer 即可实现解码和播放，需要注意的是有时由于网络抖动导致数据发送不及时，会导致播放进度落后，这是可以通过调整 player 的 currentTime 追赶播放进度。


基于以上流程的 Demo 示例如下


![20200122002551_%281%29.gif](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbb39313-8950-40fc-9abf-5c7412d9778c/cc09ac82-3175-4676-9455-1691f01bc6cd/20200122002551_%281%29.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240926%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240926T042913Z&X-Amz-Expires=3600&X-Amz-Signature=d777c6294b3e2d4ef6fe2388fb2852eabaf28382b5185b8ce8b80693f36e9236&X-Amz-SignedHeaders=host&x-id=GetObject)

