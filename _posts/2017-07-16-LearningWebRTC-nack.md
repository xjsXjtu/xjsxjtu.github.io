---
layout: post
title: "LearningWebRTC: NACK(Negative ACKnowledgement)"
description: "主要介绍使用NACK做视频抗丢包时的细节问题。"
date: 2017-07-16
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

### Overview

NACK(Negative ACKnowledgement)在RTT较小时，非常有效的抗丢包策略。

由于本文涉及很多包的往复，因此用编码端代表RTP发送方，解码端代表RTP接收方。

![nack_overview](/images/LearningWebRTC/nack_overview.png)

NACK Request是Payload Type为PT_RTPFB=205的RTCP包，一般与其他RTCP包打在一起，称为一个RTCP compound包。

```cpp
    enum RTCPPT
    {
        PT_IJ    = 195,
        PT_SR    = 200,
        PT_RR    = 201,
        PT_SDES  = 202,
        PT_BYE   = 203,
        PT_APP   = 204,
        PT_RTPFB = 205,
        PT_PSFB  = 206,
        PT_XR    = 207
    };
```
一个只包含NACK Request的RTCP包的格式如下，PID表示丢包的SN，BLP为SN之后16个RTP包丢包的bit mask。
其他字段的含义参考[RFC4588 6.2.1.  Generic NACK](https://tools.ietf.org/html/rfc4585#section-6.2.1)

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |V=2|P|  FMT=1  | PT=205(RTPFB) |          length in 32bit - 1  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  SSRC of this packet sender                   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                  SSRC of media source                         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |            PID1               |            BLP1               |
   |            PID2               |            BLP2               |
   : ....                                                          ：
```

NACK Response为重发的RTP包，可以与原来的RTP一模一样，但这样会影响接收端丢包率的计算、以及拥塞控制的带宽估计，因此，一般会按照RFC4588，增加一个RTX Header。

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         RTP Header                            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | OSN(original sequence number) |                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
   |                  Original RTP Packet Payload                  |
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### 编码端实现细节

细节：RTP History保留多久的历史包？
 * 一般是一个有限循环队列，队列的大小取决于RTP数据内容。例如，如果RTP为视频数据，GOP为3sec，那么RTP History能够保存3sec的视频RTP包就足够了。

细节：FEC包是否会缓存重发？
 * 应该不需要。但是WebRTC M59专门为Flex FEC分配一个RtpPacketHistory对象，具体原因待查。

### 解码端实现细节

细节：哪种包算丢了，应该insert到NACK list里？
 * 维护两个SN，一个SN_LATEST，表示收到的所有包里SN的最大值(需要考虑Wrap的情况)，一个为SN_OLDEST，表示有用的最老的包，对于视频而言，可以认为已解码的帧下一个SN为SN_OLDEST。
 * 所有在SN_OLDEST，SN_LATEST之间，却又没有收到的包，都认为是丢了。

细节：多久发一次NACK Request？
 * WebRTC周期性(10ms)检查NACK list是否有包，有就发NACK Request。

细节：某个包在Encoder可能就没有(Encoder可能是中转服务器)，怎么防止NACK Request一直包含这个包？
 * WebRTC M59 ModuleRtpRtcpImpl::SendNACK()中处理了这种情况
   * 每(5 + RTT * 1.5)发送NACK list里所有的包，而在其他时候，保证一个包只被发送一次。这样，每个包在(5 + RTT * 1.5)时间间隔内至多发送两次。
 * Decoder端可以通过[RTCP RRTime与DLRR计算](https://xjsxjtu.github.io/2017-06-25/LearningWebRTC-RTP-RTCP/#qosrtt).

细节：如何防止NACK Response导致Encoder上行拥塞，从而导致更加恶劣的丢包？
 * Encoder端会周期性估算带宽，并根据历史数据，把估算的带宽分别分配给video encoder, FEC, NACK 每次发送NACK Response前，都会检查是否还有充足的NACK带宽用来发送NACK包。
 
细节：NACK对音视频同步、视频render time的影响？
 * TODO

### 参考
 * WebRTC M59源码
 * [RFC4588 @2006:RTP Retransmission Payload Format](https://tools.ietf.org/html/rfc4588)

----
xjs.xjtu@gmail.com

2017-07-16












