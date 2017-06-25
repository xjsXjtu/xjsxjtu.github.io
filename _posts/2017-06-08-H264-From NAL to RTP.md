---
layout: post
title: "H264: From NAL to RTP"
description: "梳理、总结H264 NAL层级结构，以及NALU如何打成RTP包。"
date: 2017-06-08
tags: [h264]
comments: true
share: false
---

* auto-gen TOC:
{:toc}


最近调试一个WebRTC H264解码的问题，熟悉了H264码率结构及RTP打包协议。这里梳理、总结一下，供以后参考。

### H264 NAL(Network Abstraction Layer)

一个H264视频序列就是NALU(NAL Unit)序列组成，如下图：

![NAL](/images/LearningVideoCodec/H264-NAL.png)

每个NALU包含NALU header和RBSP（Raw Byte Sequence Payload），如下图：

![NAL-Header](/images/LearningVideoCodec/NAL-Header.png)

 * F必须为0，NRI表征NALU优先级，NALU Type则如下所示，其中24~31将在下面的RTP打包方法中定义：
   * ![NAL-Type](/images/LearningVideoCodec/NAL-Type.png)


### 从NALU到RTP

  * 解决的问题：H264码流如何用RTP打包发送接收，即NALU如何与RTP对接。
  * 考虑的因素：
    * 在MTU的限制内，尽量提高传输效率
    * 方便generic FEC做丢包补偿
  * RFC6184的方案
    * 打包方法
      * Single NAL Unit Packet
        * 一个NALU严格对应一个RTP包
        * 如果一个NALU大于MTU，那么OS会把RTP拆成多个UDP发送。
      * Aggregation Packets
        * 为了提高传输效率，多个NALU组成一个RTP包，如size较小的SPS、PPS。
        * STAP-A：只能把相同时间的NALU聚合在一个RTP包里
          * ![STAP-A](/images/LearningVideoCodec/STAP-A-NAL.png)
        * STAP-B，MTAP16，MTAP24暂没研究
      * Fragmentation Units
        * 为了在RTP层做generic FEC抗丢包，把一个NALU拆成多个RTP包。
        * FU-A：
          * ![FU-A](/images/LearningVideoCodec/FU-A.png)
          * FU indicator就是NALU header
          * FU header如下图，表征的是一个NALU被切分的起始RTP包（S）和结束RTP包（E）
            * ![FU-Header](/images/LearningVideoCodec/FU-Header.png)
        * FU-B暂无研究。
      * 各种打包方法对应的NALU type为：
        * ![NALU-Type2](/images/LearningVideoCodec/NALU-Type2.png)
    * 打包模式——不同的mode可用的打包方法不同：
      * Single NALU Mode    ：low-delay场景，只能用Single NALU Packet打包方法
      * Non-interleaved Mode  ：low-delay场景，NALU的发送顺序必须与解码顺序一致。
      * Interleaved Mode    ：暂无研究。
      * ![Packetization-Mode](/images/LearningVideoCodec/Packetization-Mode.png)  
    * 其他：
      * RTP timestamp  ：表征采集、渲染顺序。每一帧对应的RTP timestamp必须相同
      * RTP seq number：表征编码、解码顺序。
      * ![RTP-ts-seq](/images/LearningVideoCodec/RTP-ts-seq.png)

### Wireshark实践

抓了一个GOP的前三帧，可以看出来用的是Non-interleaved Packetization Mode。I帧有7个IDR-Slice，两个P帧各有7个non-IDR-Slice。
  * ![Wireshark-NALU](/images/LearningVideoCodec/Wireshark-NALU.png)

### 遗留问题：
  * RTP传输以NALU为单位，接收端怎么知道一帧是否收全？？？？？
  * RTP解包之后保存的码流，StreamEye怎么区分哪些NALU组成一帧？？？？？


### Reference：
  * book2010: "The_H.264_advanced_video_compression_standard"
  * RFC6184：https://tools.ietf.org/html/rfc6184

  
