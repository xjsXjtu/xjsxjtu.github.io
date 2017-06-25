---
layout: post
title: "LearningFFMPEG: VS2015里单步调试FFMPEG"
description: "基于ShiftMediaProject,使用VS2015编译、运行、单步调试FFMPEG。"
date: 2017-05-20
tags: [ffmpeg]
comments: true
share: false
---

* auto-gen TOC:
{:toc}

### MOTIVATION

相比于GDB+VIM，VS2015在读代码方面还是方便太多了。

### STEPS

1.  Clone repository：https://github.com/ShiftMediaProject/FFmpeg
2.  找到FFmepg/SMP/project_get_dependencies.bat, 双击运行。下载第三方依赖库。
3.  下载YASM汇编器：https://github.com/ShiftMediaProject/VSYASM/releases/tag/0.2。解压，管理员模式运行install_script.bat
4.  打开FFmpeg/SMP/ffmpeg_deps.sln。编译ffmpeg/ffplay/ffprobe。
    *  TroubleShooting：如果编译时遇到GL/glext.h找不到，则到官网下载即可：https://www.khronos.org/registry/OpenGL/api/GL/
5.  以ffplay为例，拷贝sample.mp4文件到ffmpeg_deps.sln同级目录，设置debug comand line：sample.mp4。开始Debug~~~~~~
    *  TroubleShooting：如果遇到“Unable to start program FFmpeg/SMP/../../../../msvc/ffplay.exe",很可能需要把ffplay工程的Linker->General->OutputFile改成error msg的路径。

### REFERENCE

 * ShiftMediaProject：https://github.com/ShiftMediaProject/FFmpeg
  * 本文主要参考FFmpeg/SMP/readme.txt
 * 使用vs2013或vs2015 编译ffmpeg：使用vs2013或vs2015 编译ffmpeg
