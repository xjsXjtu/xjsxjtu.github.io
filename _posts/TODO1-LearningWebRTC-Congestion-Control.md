---
layout: post
title: "LearningWebRTC: RTP/RTCP协议"
description: "总结RTP协议关键概念，Header关键字段，RTCP常见作用，以及WebRTC相关实现。"
date: 2017-06-25
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

总结RTP协议关键概念，Header关键字段，RTCP常见作用，以及WebRTC相关实现。

### RTP关键概念

 * RTP session

### RTP Header关键字段

![rtp_header](/images/LearningWebRTC/rtp_header.png)

 * SSRC(Synchronization SouRCe identifier)
   * 不同session里SSRC是否冲突？
 * PT(PayloadType)
   * 不同session里PT是否可能相同？
 * SN(Sequence Number)
   * 哪些条件构成独立的SN空间？
 * TS(TimeStamp)


### RTCP常用类型

![rtcp_packet_directions](/images/LearningWebRTC/rtcp_packet_directions.png)

#### QoS(Quality of Service)

 * SR(Sender Report)
 * RR(Receiver Report)
 * XR...

### 参考

 * [RFC3550：RTP: A Transport Protocol for Real-Time Applications](https://tools.ietf.org/html/rfc3550)
 * book2004 RTP-Audio and Video for the Internet














