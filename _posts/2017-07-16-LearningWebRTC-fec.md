---
layout: post
title: "LearningWebRTC: FEC(Forward Error Correction)"
description: "主要介绍RTP级别的FEC在视频抗丢包上的应用，包括，码流格式，传输层方案，相关联的模块以及调参时考虑因素。"
date: 2017-07-16
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

WebRTC里使用的FEC有两种：编码器相关的FEC，如[Opus内置FEC](https://xjsxjtu.github.io/2017-07-08/LearningWebRTC-opus/#opus-anti-packet-loss); RTP级别的FEC。

这篇文章主要介绍RTP级别的FEC如何应用在视频抗丢包上。

### WebRTC FEC模块图

FEC原理比较简单，M个Media包生成F个FEC包，每个FEC包都是M个Media包中若干个XOR(异或)得来，包括Media包的RTP Header和Payload部分。

![fec_module](/images/LearningWebRTC/fec_module.png)

### WebRTC FEC码流结构

WebRTC M59中有两种可选的FEC， ULP FEC和Flex FEC。 两者共同需要解决的问题有：
 * 哪一路FEC保护哪一路或哪几路Media
 * 某个FEC包保护了哪些Media包。

#### ULP(Unequal Level Protection) FEC

遵循RFC2198 section 1/2/3和RFC5109 section 6/7/8，使用xor生成冗余数据。
  * Media包： 在RFC5109中，Media包并不会做任何修改，目的是兼容不支持FEC的解码端。但是，WebRTC参考RFC2198，给Media包增加了一个RED头，感觉没啥必要，后面会有解释。

```
    0                   1                    2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3  4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | V |P|X|  CC   |M| RED PT(96)  |         SN                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         timestamp                             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         SSRC                                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |0| VP9 PT(101) |                                               |
   +-+-+-+-+-+-+-+-+                                               +
   +                           VP9 Payload                         +
   +                                               +---------------+
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

  * FEC包： 参考RFC5109，不过只使用一个level。 RFC5109多level的目的，是区分不同level的重要性，从而尽量多的保护更重要的level。

```
    0                   1                    2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3  4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | V |P|X|  CC   |M| RED PT(96)  |         SN                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         timestamp                             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         SSRC                                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |0| FEC PT(97)  |E|L|P|X|  CC   |M| PT recovery |   SN base     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |   SN base     |               TS recovery                     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  TS recovery  |        length recovery        |   Protection  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |   Length      |             mask              |  mask cont.   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                mask cont(only when L=1)       |               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
   +                        FEC Payload                            +
   +                                               +---------------+
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

进一步思考：
  * Media包应该不需要RED头，正如RFC5109规范中定义的。
    * FEC保护哪一路media包：
      * 假如同时有两路视频在发送，这两路视频的ssrc应该是不一样的，Rx端可以根据FEC包里RTP header里的ssrc确定保护的是哪一路视频。
      * 假如音频和视频都开了RTP层次的FEC，则必须使用两种RED PT，否则即使media包使用了RED头传输，也不能区分FEC包保护的是音频还是视频。
    * FEC包保护哪几个media包？
      * FEC Header里的SN base和mask可以得到，不需要从Media RED header里获取。
  * FEC Header里的Protection Length不需要
    * 这个字段来源于RTP5109：RTP Payload Format for Generic Forward Error Correction，这个协议里同一个FEC包可以包含多个Level，Protection Length字段的目的，就是为了标记当前level的结束，从而得知下一个level header的起始。而WebRTC只实现了一个level，因此这个字段失去了意义。

#### Flexible FEC

遵循并大幅修改RFC:RTP Payload Format for Flexible Forward Error Correction. 该RFC设计的目的之一是解决RFC5109扩展性上的不足：“address the scalability issues experienced with the earlier specifications including RFC 2733, RFC5109 and SMPTE 2022-1, and offer several improvements.”

  * Media包：不变
  * FEC包：不同的ssrc(因此SN空间不同),甚至可以支持一路FEC可以保护多路Media包(SSRC_CNT）

![flex_fec_bs.png](/images/LearningWebRTC/flex_fec_bs.png)

### FEC包RTP Header重要字段、以及包传输方法

相比于Media包，FEC包重要字段为：

|         | FEC SSRC    | FEC SN             | FEC port |
| - | - | - | - |
| ULPFEC  | same        | 与Media共享SN空间[1] | same |
| FlexFEC | different[2]| 与Media使用独立SN空间 | same |

[1] ULPFEC包与Media包共享SN空间，会导致JB内的连续性判断有误：即使所有的Media包都到达了，而FEC包丢失了，仍然无法从JB里pop解码。

[2] FlexFec接收端怎么知道哪个FEC SSRC保护哪个Media SSRC。 答案在remote SDP里，如a=ssrc-group:FEC-FR 1234 2345，就是把1234和2345两个SSRC绑在一起。

### WebRTC FEC调参

FEC调参过程中，既要知道各个参数的含义，又需要了解相关模块的相互影响，对于FEC来说，相关联的模块主要是： JB(JitterBuffer), MO(MediaOptimzation)。

#### 固定FEC rate or 自适应rate

WebRTC中，FEC rate必须是网络自适应的。对于WebRTC二次开发，可以根据业务需求，改成固定FEC rate。

固定FEC rate调整时，主要权衡因素是：
 * 视频卡顿次数与卡顿时长
 * FEC带宽
 
网络自适应FEC rate参考[这里](https://xjsxjtu.github.io/2017-07-16/LearningWebRTC-adafec/)

#### Bursty/Random mask type

根据m个media包生成f个FEC包时，选择哪几个Media包来做异或运算得到FEC包，就是所谓的Mask。注意，2个Media生成一个FEC，4个Media生成2个FEC，两者的mask是不一样的。

Random Mask对抗随机丢包。 

Bursty Mask更有利于抗连续丢包，更具体的说：

```cpp
// They have the following property: for any packet mask defined by the
// parameters (k,m), where k = number of media packets, m = number of FEC
// packets, all "consecutive" losses of size <= m are completely recoverable.
```

例如，7个media生成3个FEC包，使用的Mask为：

![fec_mask7_3](/images/LearningWebRTC/fec_mask7_3.png)

如果前三个Media包都丢失，使用Random Mask无法恢复，而使用Bursty mask，可以按照下面的顺序依次恢复：
![fec_mask7_3_recovery](/images/LearningWebRTC/fec_mask7_3_recovery.png)

另外，为了使抗Bursty丢包效果最大化，还应使一个group内的Media包尽量多。
例如，如果一个group内只有2个Meida包生成一个FEC包，那么只要有连续丢包，就没办法恢复了。而如果一个group内有4个Media包，生成2个FEC包，就可以在连续丢两个包时仍然可以恢复出来。

#### MaxFrames

MaxFrames的调参，必须考虑接收端JB的实现。

如果JB允许一帧在不收全的情况下就Pop(kWithError),则MaxFrames必须为1，否则绝大部分Media包即使靠FEC恢复了，也会被JB认为是OldPacket扔掉。
如果JB必须等待一帧收全才Pop(kNoError),则MaxFrames可以按照下面的因素权衡。

MaxFrames权衡的因素：
 * 延迟，原因是一个group内所有Media包都准备好之后才生成FEC包。
   * 如果没有丢包，则FEC不会引入额外的延迟。
   * 如果丢包，且发生在一个group内的第一帧的Media包丢失，不考虑网络抖动的情况下，则至少需要等待MaxFrames-1帧的时间，才可能收到这个group的FEC包。
 * 抗Bursty丢包
   * 原理同Bursty mask type中解释的，一个group内Media包越多，抗Bursty丢包性能就越好。
   * 在一定码率、帧率情况下，每帧所平均包含的Media往往是固定的，因此，MaxFrames越大，就意味着一个group内Media包越多。

### 参考
 * WebRTC M59源码
 * [RFC2198 @1997：RTP Payload for Redundant Audio Data](https://tools.ietf.org/html/rfc2198)
 * [RFC5109 @2007：RTP Payload Format for Generic Forward Error Correction](https://tools.ietf.org/html/rfc5109)
 * [RFC???? @2017:RTP Payload Format for Flexible Forward Error Correction](https://tools.ietf.org/html/draft-ietf-payload-flexible-fec-scheme-05)
 * [WebRTC SDP anatomy](https://webrtchacks.com/sdp-anatomy/)

----
xjs.xjtu@gmail.com

2017-07-16












