---
title: 基于Hypercolumns神经网络的的图像分割（译）
date: 2017-12-09 21:44:42
tags:
---

# 前言

通过VGG-16神经网络得到pixel hypercolumns信息给图片上色 [Link](http://tinyclouds.org/colorize/)

本篇文章将使用Keras与Scikit-Learn去提取pixel hypercolumns信息。

# Hypercolumns

很多算法都利用卷积神经网络的思想，是用全连接层来提取特定特征。但是对于一些空间特征却提取得很粗糙，同时，第一层卷积层可能能够较好的空间特征，但往往缺少语义信息。为了解决这个问题，这篇[Hypercolumns Paper](http://arxiv.org/pdf/1411.5752v2.pdf)的作者提出了
**TODO**

![Hypercolumns](http://otbwgn2nv.bkt.clouddn.com/de198f05ff938be7ca6f867938bf14cc.png)

提取Hypercolumns信息的第一步便是将图片送入CNN获取图像每个区域的feature map activations。但是当feature maps比输入图片小的时候，例如，在池化操作完成后，作者对feature map进行了bilinear upsample去将feature map扩展为跟输入图片一样的大小。并且在FC（全连接层）进行相同步骤，因为

# VGG-16

在我们开始提取Hypercolumns信息前我们需要构建一个VGG-16神经网络，
