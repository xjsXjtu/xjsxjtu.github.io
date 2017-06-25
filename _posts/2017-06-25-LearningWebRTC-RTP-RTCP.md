---
layout: post
title: "LearningWebRTC: RTP/RTCP协议"
description: "总结RTP协议关键概念，Header关键字段，RTCP常用类型，以及WebRTC相关实现。"
date: 2017-06-25
tags: [webrtc]
comments: true
share: false
---

### RTP关键概念

 * RTP session

### RTP Header关键字段

![rtp_header](/images/LearningWebRTC/rtp_header.png)

 * SSRC
   * 不同session里SSRC是否冲突？
 * PT(PayloadType)
   * 不同session里PT是否可能相同？
 * TS(TimeStamp)


### RTCP常用类型

![rtcp_packet_directions](/images/LearningWebRTC/rtcp_packet_directions.png)

#### QoS(Quality of Service)

 * SR(Sender Report)
 * RR(Receiver Report)
 * XR...

### 参考

 * [RFC3550](https://tools.ietf.org/html/rfc3550)
 * book2004 RTP ....














