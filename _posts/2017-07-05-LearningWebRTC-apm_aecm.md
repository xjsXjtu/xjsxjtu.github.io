---
layout: post
title: "LearningWebRTC: AECM"
description: "总结WebRTC M59 AECM(Acoustic Echo Control for Mobile)算法流程"
date: 2017-07-05
tags: [webrtc]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

AECM(Acoustic Echo Control for Mobile)是WebRTC专为Android系统设计的低运算量的回声消除模块。
iOS系统自带的回声消除模块一般性能优良，不需要使用应用层的；Windows由于运算量充足，一般使用AEC/AEC3模块。

### 算法原理

AECM算法的核心是Adaptive Filter，运用的是NLMS(Normalized Least Mean Square) + Wiener Filter结合的方法。 
NLMS主要用于估算回声能量谱，Wiener主要根据回声能量谱计算最终结果。

#### NLMS原理

![aecm_nlms_theory](/images/LearningWebRTC/aecm_nlms_theory.png)
    
 * Known:  farend x(n), nearend s(n) + d(n), where d(n) is the real echo
 * Target: estimate adaptive filter h^(n), so that to estimate echo d^(n)
 * Cost function: L is the filter length
        ![aecm_nlms](/images/LearningWebRTC/aecm_nlms.png)
 * Iteration result after the n’th iteration
        ![aecm_nlms_formula](/images/LearningWebRTC/aecm_nlms_formula.png)
   * should only updates when nearend speech s(n) is silent
   * Freq domain(WebRTC used): the same (from 《现代信号处理》P195 张贤达)

#### Wiener Filter原理

![aecm_wiener_theory](/images/LearningWebRTC/aecm_wiener_theory.png)

Wiener滤波器同样使用均方误差最小作为优化目标，不过Wiener滤波不使用迭代解，而是显示解：
 * ![aecm_wiener_result](/images/LearningWebRTC/aecm_wiener_result.png)

结合AECM场景，可以简单推到得到，基于NLMS噪声估计的传递函数为：
 * H = Syx/Sxx = Ss(s+d)/S(s+d)(s+d) = Sss/(Sss+Sdd) = 1 – Sdd/(Sss+Sdd) = 1 – echoEnergy/nearendEnergy
 * AECM会在echoEnergy前乘以一个"置信度” suppressGain，表示的是回声估计越准确，“置信度”越高。
  
### 关键步骤

 * 使用Hanning窗的DFT变换
   * 防止频谱泄露
 * delay estimate
   * 目标： AECM里，delay一共有三段，farend Playout delay + echo path delay + capture delay，
     其中playout和capture由外面提供，这里主要估计echo path delay。
     * AECM在频域做，因此最小延迟单位为一个block（64samples = 4ms），默认支持最大延迟为100block，即400ms。
   * 方法： 分别求远端信号和近端采集信号的二值谱，并根据二值谱的相似程度（XOR后1的个数）进行匹配。
 * Farend VAD检测 & DT检测：
   ![aecm_delay_est](/images/LearningWebRTC/aecm_delay_est.png)
   * 作用：
     * 只有区域③允许更新滤波器系数，其他区域必须停止更新(即NLMS stepsize=0）； 
     * 根据farendVAD和DT结果，决定Winer滤波”置信度“(suppressGain).
   * AECM里没有显式的检测DT，而是根据large variations in estimated echo来判断DT的存在 
 * Adaptive Filtering
   * 迭代更新NLMS滤波器系数，估算回声能量谱
   * 计算Winer滤波器系数，并使用NLP(Non-Linear Process)限幅。
 * CNG(Comfort Noise Generator)
   * 原因： Winer filter抑制回声时，也抑制了噪声，需要补回来
   * 幅度谱根据nearend能量估计，相位谱为均匀分布； 然后算出实部虚部值，加到Winer滤波的结果里
 * DFT加窗反变换

### 可能的优化方法
 * 延迟估计是否准确，最大延迟是否超过代码的限度
 * echo路径变化时，如插拔耳机，是否对滤波器做了重新收敛
 * 提高near VAD + double talk检测准确性。
 * 强制用AEC/AEC3替换AECM

### 评价指标
 * ERLE(Echo Return Loss Enhancement)
 
### 遗留问题
 * 为什么不能只用NLMS？

### 参考
 * WebRTC M59源码

----
xjs.xjtu@gmail.com

2017-07-05












