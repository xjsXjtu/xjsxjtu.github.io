---
layout: post
title: "LearningWebRTC: Opus"
description: "Opus简介，包括基本模块及FEC抗丢包策略。"
date: 2017-07-08
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

WebRTC M59使用Opus作为默认的音频编解码器，这里只简答介绍，太深的暂时也不懂。

### Opus Overview

Opus的目标应用场景为“**Interactive** audio over the Internet”，第一版在2012年发布。

从官网的介绍来看，压缩效率和延迟比AAC在内的绝大多数音频编解码器都要优秀。

![opus_compress_rate](/images/LearningWebRTC/opus_compress_rate.png)

![opus_delay](/images/LearningWebRTC/opus_delay.png)

### Opus internal modules

Opus是一个混合编码器，由SILK和CELT两种编码器混合而成，SILK主要负责wideband(8khz)以下的语音编码，CELT主要负责高频编码，如音乐等。

![opus_modules1](/images/LearningWebRTC/opus_modules1.png)
![opus_modules2](/images/LearningWebRTC/opus_modules2.png)

SILK由Skype Speex改良而来，像通常的语音编解码器一样，使用LPC(Linear Predictive Coding)在时域完成压缩。

CELT则像通常的音频编解码器一样，使用MDCT在频域完成压缩。

为了尽量减小延迟，SILK帧长最小为10ms，最大60ms；而CELT帧长则在2.5ms~20ms。

### Opus anti packet loss

Opus FEC只用在低频部分，用更低的码率编码出码流(redundant bitstream)，然后与原始码流(primary bitstream)在同一个RTP session里发送。

![opus_fec](/images/LearningWebRTC/opus_fec.png)

### 参考
 * WebRTC M59源码
 * [Presentation @2012](http://opus-codec.org/presentations/ )
 * [Opus wiki]( https://zh.wikipedia.org/wiki/Opus_codec )
 * [RFC6716 @2012 Definition of the Opus Audio Codec]( https://tools.ietf.org/html/rfc6716 )
 * [RFC7587 @2015 RTP Payload Format for the Opus Speech and Audio Codec]( https://tools.ietf.org/html/rfc7587 )
 * Book @2003 RTP Audio and Video for the Internet
 * [Range coding wiki]( https://en.wikipedia.org/w/index.php?title=Range_encoding&oldid=509582757 )
 * [Introduction to CELP Coding](https://www.speex.org/docs/manual/speex-manual/node9.html) of [The Speex Codec Manual](https://www.speex.org/docs/manual/speex-manual/manual.html)

----
xjs.xjtu@gmail.com

2017-07-08












