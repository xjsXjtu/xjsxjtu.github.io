---
layout: post
title: "LearningWebRTC: 拥塞控制"
description: "总结GCC(Google Congestion Control)算法原理，以及WebRTC实现。"
date: 2017-07-18
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

我不是拥塞控制专家，只懂皮毛，以后随着对WebRTC拥塞控制理解的加深，会进一步更新本文。

### 原理

GCC(Google Congestion Control)算法架构如下图:

![gcc_arch](/images/LearningWebRTC/gcc_arch.png)

主要分为DBCC(Delay Based Congestion Control)和LBCC(Loss Based Congestion Control)。

#### DBCC(Delay Based Congestion Control)

![gcc_delay_gradient](/images/LearningWebRTC/gcc_delay_gradient.png)

RTP包网络延迟定义为：t - T

DBCC原理，直观上可以理解为：根据网络延迟变化，判断网络状态：
 * 延迟变大 ==> Overuse；
 * 延迟变小 ==> Underuse；
 * 延迟比较平稳 ==> Normal。

分成下面几个部分,大致的功能如下：

 * Arrival filter
   * 估算网络延迟梯度。使用Kalman滤波器，融合上次估算值m(ti-1)和本次观测值dm(ti)，得出本次梯度估计值m(ti)。
![gcc_dbcc_dm](/images/LearningWebRTC/gcc_dbcc_dm.png)
![gcc_dbcc_m](/images/LearningWebRTC/gcc_dbcc_m.png)

 * Adaptive threshold
   * 网络延迟梯度m(ti)通过与阈值gamma比较，判断网络状态。但是gamma必须是自适应的，否则与TCP流竞争时，RTP可用带宽会越来越小。
   * 阈值gamma的更新，是基于|m(ti)|平滑滤波得到.
![gcc_dbcc_gamma](/images/LearningWebRTC/gcc_dbcc_gamma.png)

 * Overuse detector
   * 比较m(ti)和gamma(ti)，得出网络状态。
![gcc_dbcc_detector](/images/LearningWebRTC/gcc_dbcc_dector.png)
 
 * Remote rate control
   * 根据网络状态，使用有线状态机更新带宽估计。
![gcc_dbcc_state](/images/LearningWebRTC/gcc_dbcc_state.png)
   
#### LBCC(Loss Based Congestion Control)

LBCC就简单很多，在发送端收到RTCP Receive Report后，根据丢包率，更新带宽估计：
![gcc_lbcc_formula](/images/LearningWebRTC/gcc_lbcc_formula.png)

然后，取DBCC和LBCC带宽估计中较小的值，作为最终结果。

### WebRTC实现

WebRTC实现主要牵涉到三个模块：
 * RBE(Remote Bitrate Estimator)，负责算法核心部分，即带宽估计。
 * CC(Congestion Control)，封装RBE，负责实现DBCC。
 * BC(BitrateController)：实现LBCC，并取min(DBCC, LBCC)作为最终结果。
 
各自的功能在下面会看到详细的介绍。

#### DBCC

DBCC的实现在CC(Congestion Controller)模块里，接口类为ReceiveSideCongestionController和SendSideCongestionController。

##### 接收端实现

ReceiveSideCongestionController把包的大小和到达时间转发给RemoteBitrateEstimatorAbsSendTime或者RemoteBitrateEstimatorSingleStream。
![gcc_dbcc_rx](/images/LearningWebRTC/gcc_dbcc_rx.png)

接收端实现时，需要知道发送端的时间，有两种方法：
 * 一种为RTP HeaderEx里的absoluteSendTime，对应上面的RemoteBitrateEstimatorAbsSendTime类。
   * absoluteSendTime为包的绝对发送时间，因此如果有服务器转发RTP，则需要更新该字段，从而得到分段的带宽估计。
   * GCC Spec里建议用这种方法。
 * 一种为RTP HeaderEx里的transmissionTimeOffset，对应上面的RemoteBitrateEstimatorSingleStream。
   * transmissionTimeOffset表示相对于RTP Timestamp时间，因此，只能用在p2p的场景；如果有服务器转发RTP，则只能估算端到端的带宽。

##### 发送端实现

在接收端：ReceiveSideCongestionController则把包的大小和到达时间转发给RemoteBitrateEstimatorProxy，然后以RTCP RTPFB包发给发送端。

在发送端：收到RTCP RTPFB包后，转给SendSideCongestionController并在DelayBasedBwe里完成带宽估计。

![gcc_dbcc_tx](/images/LearningWebRTC/gcc_dbcc_tx.png)

##### 如何选择：@Rx or @Tx

注意：这部分工程经验还不够足，只是根据仅有的皮毛知识猜测的。

接收端实现，需要把结果以RTCP REMB的形式发给发送端。
发送端实现，需要把包的到达时间以RTCP RTPFB的形式发给发送端。
因此，两种方式的延迟可以认为是一样的。

但是，RTPFB包的数量比REMB多很多，本身对网络就是一种负担。

因此：
 * 如果接收端是转发服务器，没有实现RBE，那么可以暂时用DBCC@Tx，接收端只需要实现RTPFB。
 * 如果接收端实现了RBE，就选择DBCC@Rx。
 

#### LBCC

BC模块得到丢包率后，根据前面的公式更新带宽，并得到DBCC结果取较小值。很简单，不画图了。


### 遗留问题

 * 如果同时有两路视频发送，意味着发送端和接收端分别有两个CC，那最终估算的带宽如何融合？估算的带宽如何分配？

### 参考
 
 * [Paper @2016: Analysis and Design of the Google Congestion Control for Web Real-time Communication (WebRTC)](http://c3lab.poliba.it/images/6/65/Gcc-analysis.pdf)
 * [IETF draft @2017: A Google Congestion Control Algorithm for Real-Time Communication](https://tools.ietf.org/html/draft-ietf-rmcat-gcc-02)
 * [Paper @2012: Understanding the Basis of the Kalman Filter Via a Simple and Intuitive Derivation ](https://www.cl.cam.ac.uk/~rmf25/papers/Understanding%20the%20Basis%20of%20the%20Kalman%20Filter.pdf)
 * [RFC4585：Extended RTP Profile for RTCP-Based Feedback](https://tools.ietf.org/html/rfc4585)














