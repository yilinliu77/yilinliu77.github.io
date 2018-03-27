---
title: Coherent Occlusion Culling(CHC) 算法的原理与实现
date: 2018-03-22 21:04:06
tags:
---

> CHC算法是Jiˇrí Bittner在2004年提出的一种遮挡剔除的算法，其中主要运用到了渲染的（Temporal coherence）时间相干性以及（Spatial coherence）空间相干性，同时运用BVH（Bounding Volume Hirachical）管理场景并且通过基于硬件的遮挡查询对遮挡物进行剔除，提高渲染速度。

# 算法背景

## Hardware Occlusion Culling

CHC算法主要建立在基于硬件的遮挡查询（Hardware Occlusion Culling）的基础上。Hardware Occlusion Culling基于一个十分简单的思想，在判断物体是否被遮挡时，将物体的深度值传递给GPU，GPU将自己现存的最低的深度值与物体的深度值进行对比，并返回查询结果（可见像素点个数）。

## 优点
 
- API简单易用，不需要额外管理信息，能迅速集成进现有系统
- 基于GPU进行判断，效率高，充分挖掘GPU性能

# 存在问题

- 基于GPU的判断效率高，但涉及数据传输，在大规模场景下会造成CPU一直等待GPU查询结果返回造成GPU性能饥饿的情况
- 虽然基于GPU判断效率高，不过在复杂场景下仍有大量渲染目标需要被测试，仍有提升空间

# Coherent Occlusion Culling（CHC）

## 目标

- 减少进行遮挡查询的次数
- 避免CPU出现瓶颈而GPU饥饿的情况

## 算法核心

- 利用BVH（Bounding Volume Hirachical）管理场景，并利用**Spatial Coherence**（空间相干性）减少每一帧需要进行遮挡查询的次数。
- 重用上一帧的查询结果，利用**Time Coherence**（时间相干性）进一步减少查询次数
- 维护一个查询队列，延迟进行查询的时间，并利用渲染部分场景的时间来填充CPU等待GPU查询结果返回的时间，减少GPU的饥饿。

## 算法介绍

### BVH

在实际的渲染过程中，一个渲染场景通常会有大量物体，如果将他们都渲染出来会对GPU造成极大的性能负担，所以需要对看不见的物体从渲染队列中剔除。例如View Frustum Culling算法：只渲染在视角范围内的物体。而对于一个复杂场景来说，经过View Frustum裁剪过后的物体对于渲染管线仍有很大压力。

而对于CHC算法的基础Hardware Occlusion Culling算法来说，对每一个物体都进行一次查询仍然是难以负担的任务。为了减少进行查询的次数，CHC算法的作者采用了一种层次化管理场景的方法。将场景中的物体基于空间信息利用树状结构进行管理，相邻的物体存在于叶节点中。

![Bounding Volume Hirachical](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2a/Example_of_bounding_volume_hierarchy.svg/750px-Example_of_bounding_volume_hierarchy.svg.png)(图片来源[Wikipedia](https://en.wikipedia.org/wiki/Bounding_volume_hierarchy))

如图所示，场景中一共有6个物体，而树状结构一共有4个内部节点，每个物体都被它的父节点包围，每个内部节点也被它的父节点包围。

算法流程如下：在渲染开始时会从根节点依次往下遍历，并利用Hardware Occlusion Culling的方法检测节点包围盒对于当前场景是否可见，如果可见则遍历子节点直至叶节点。如不可见则证明该包围盒下所有节点都不可见，则跳过该节点继续遍历。

![CHC-1](http://otbwgn2nv.bkt.clouddn.com/CHC-1.png)

如上图，从根节点A开始遍历，发现右下角部分可视，则遍历子节点B与C。对于B节点的可见性测试失败，则跳过B节点的所有子节点，开始遍历C节点，直至遍历到叶节点开始渲染。

从上面例子中可以见到，



**的**



