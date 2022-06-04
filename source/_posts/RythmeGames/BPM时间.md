---
title: BPM时间、游戏Tick、刷新帧以及正常时间。
date: 2022-04-12 20:00:00
categories:
- 音游开发日志
tags: [音游]
mathjax: true
---

## BPM时间值

BPM时间是一个针对于 BPM 而言的时间刻度。

### BPM 
BPM是Beat Per Minute的简称，中文名为拍子数，释义为每分钟节拍数的单位。最浅显的概念就是在一分钟的时间段落之间，所发出的声音节拍的数量，这个数量的单位便是BPM。
——(百度百科)

60BPM就是一分钟60拍。

### 游戏Tick
对于本人而言，游戏Tick是对于帧率的和数值。

在60FPS的情况下，一秒对应的Tick为60。

### 刷新帧
游戏引擎会需要无时无刻刷新，一帧一帧绘图。
例如ExcaliburJs使用了Canvas的刷新（？
Unity有FixUpdate不是靠帧刷新的。

## 转换
BPM时间值通常有两种形式
- [a,b,c] 代表 第 a:b/c 拍
- a.b 例如 120.50 代表 120:1/2拍

转换就是把BPM时间转为Tick

$ \frac{BPM时间}{BPM}\times 60 \times 设定帧率 $

