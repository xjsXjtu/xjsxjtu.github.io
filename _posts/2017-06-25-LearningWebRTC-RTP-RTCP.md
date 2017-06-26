---
layout: post
title: "LearningWebRTC: RTP/RTCP协议"
description: "TODO"
date: 2017-06-25
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

总结RTP协议关键概念，Header关键字段，RTCP常见作用。
对于协议中理解比较模糊的地方，查看WebRTC M59相关实现。

### RTP关键概念

#### RTP session

 * Address+Port确定了一个RTP session,一般RTCP端口号比RTP大1。
 * 一个session只传输一种媒体数据，音频和视频应该(SHOULD)在不同的session里。
   * 目的（其实不理解...)： 
     * 相比于把音视频用不同的SSRC在同一个session里发送的方案，音视频在不同的session里，可以"to allow some participants in the conference to receive only one medium if they choose."
     * 其他原因可参考[RFC3550 section 5.2](https://tools.ietf.org/html/rfc3550#section-5.2)
   
#### stream(media stream)

 * RTP并没有定义什么是stream，但是这个概念却到处都在用，实在烦人。
 * 我的理解：
   * 发送摄像头是一路stream，发送本地视频文件里的视频是一路stream，发送麦克风是一路stream，发送本地mp3文件是一路stream。
   * 同一种media类型的stream（如：摄像头和本地视频文件），在同一个RTP session里，即共享同一个网络端口，但两路stream的SSRC不同。

##### 遗留问题
   * 同一种media类型的stream共享网络端口，需要看WebRTC代码确认。
   * 为什么WebRTC M59里，cricket::StreamParams里有多个SSRC，如primary ssrc，fec ssrc， fec-fr ssrc， sim ssrc？


### RTP Header关键字段

![rtp_header](/images/LearningWebRTC/rtp_header.png)

#### SSRC(Synchronization SouRCe identifier)
   * 作用：在一个session内代表一路media stream，也可以代表视频stream内的FEC/RTX包。因此，一个participant在一个RTP session内可能有多个SSRC。
   * 谁生成，怎么生成：当participantA进入房间时，会给自己的每路media stream分配一个SSRC。为了尽量减少冲突的概率，不应该仅仅使用time作为随机数的种子。
   * 冲突了怎么办：当已在房间内的participantB发现participantA在使用冲突的SSRC时，ParticipantB会发送RTCP BYE退出房间，然后进入房间并重新分配SSRC。
   
#### PT(PayloadType)
   * 作用：告诉接收端如何处理RTP包，即找到正确的解码器。

#### SN(Sequence Number)
   * 作用：表示media stream的编码顺序，同时用于接收端QoS的计算。
   * 哪些条件构成独立的SN空间：每个SSRC下可以有一个独立的SN空间。

#### TS(TimeStamp)
   * 作用：表示媒体数据的采集、渲染顺序。
   * 单位：音频TS的单位为采样率，视频TS的单位一般为90khz。注意这个不是ms或us，。
   * 如何映射成local playout time，将在RTCP里介绍
   
#### 遗留问题
 * 一个participant的音频、视频在不同session里，那么音频ssrc和视频ssrc需要保证不冲突吗？WebRTC如何实现？
 * 音频和视频的PT允许用同一个PT吗？


### RTCP

首先，提到何时该发送RTCP包。

其次，RTCP最重要的作用就是接收端把QoS反馈给发送端，这里主要介绍的就是这个（丢包率，延迟、抖动）。

最后，还需要介绍，如何利用RTCP SR把TS映射成本地时间，这对音视频同步是至关重要的。

其他作用，包括用SDES发送CNAME，RTCP BYE等，比较简单，这里不介绍。

![rtcp_packet_directions](/images/LearningWebRTC/rtcp_packet_directions.png)

由于RTCP在接收端和发送端之间往返，清晰起见，RTP 发送端用Encoder表示，反之为Decoder。

#### 何时发送RTCP包
 * 目标：RTCP周期性发送。为了防止RTCP占用太多带宽（一般<5%），一种方法是把各种类型的RTCP尽量打成一个UDP包，即compound RTCP；另一种做法是控制RTCP发送频率，participant越多，RTCP发包间隔越大。
 * RFC3550: 动态变化，考虑因素有：RTCP带宽，RTCP包平均大小，session内总人数和总sender人数。
 * WebRTC M59实现(RTCPSender)
   * 音频以[2.5 sec, 7.5 sec]均匀分布的间隔发送
   * 视频以[0.5 sec, 1.5 sec]均匀分布的间隔发送，总体带宽较大时，会适当减小间隔。  

#### QoS：丢包率
 * WebRTC M59(StreamStatisticianImpl)遵守RFC3550算法: fractionLost = max(0, (expected - received) / expected)
   * expected: 间隔内最大的SN - 最小SN
   * received： 间隔内收到的所有包数，包括前一个间隔晚到的包，重传的包都算。因此，expected 有可能小于received。

#### QoS：RTT
 * Encoder如何计算RTT——根据RTCP RR：
   * WebRTC M59(RTCPReceiver::HandleReportBlock)遵守RFC3550算法： RTT = T - LSR - DLSR
     * T： 表示Encoder收到RR的NTP时间。
     * LSR： 为Encoder上一次发SR的NTP时间，记录在SR里，并被Decoder在RR里发送回来。
     * DLSR： Decoder收到SR后，发送RR前的延迟。
 * Decoder如何计算RTT(NACK时需要)
   * 类似，只是LSR记录在RRTime包里，DLSR记录在DLRR包里。

#### QoS：Jitter
 * Jitter表示每个RTP包传输时长的抖动，最理想的是用方差计算，但由于是实时计算的，因此RFC3550的计算方法是一个moving average。
 * WebRTC M59（StreamStatisticianImpl::UpdateJitter)遵守RFC3550算法: 
   * ![jitter](/images/LearningWebRTC/jitter.png)
   * D(i-1, i)，表示第i、i-1个RTP包传输时间的差。由于Encoder/Decoder时钟不一致，无法准确计算，因此用RTP TS和包到达时间近似表示。

#### 如何把RTP TS映射成本地时间
 * 根据RTCP SR，把RTP TS映射为本地时间，是音视频同步的基础。
 * ![ts_2_ntp](/images/LearningWebRTC/ts_2_ntp.png)
   * Tsr, Msr分别表示上一个RTCP SR里的NTP时间和RTP TS. 一个TS为M的RTP对应的NTP时间为T。


### 参考

 * [RFC3550：RTP: A Transport Protocol for Real-Time Applications](https://tools.ietf.org/html/rfc3550)
 * book2004 RTP-Audio and Video for the Internet














