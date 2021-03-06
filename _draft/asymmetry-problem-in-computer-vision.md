---
layout: post
title: 机器视觉：Asymmetry Problem in Computer Vision
categories: [计算机视觉]
tags: 计算机视觉
---

> 自然法则无时不刻不给予着人类以对称性的恩惠，从一片树叶到人类自身，其形态都是对称的。对称性的特性，大大减轻了人类的记忆和认知负担。然而，弱相互作用中互为镜像的物质的运动不对称却暗藏着自然法则对非对称性的偏爱。

在计算机视觉中，对称性是一个很好的先验，如果某一个特定的物体具备对称性的话，通过引入对称性可以提升系统的精度。常见的对称性包括：

- 物体本身具备对称性，且这种对称性不容易受大视角变化的影响。主要应用场景为在训练诸如检测模型的时候，可以将这一信息加入到训练样本的扩增上；
- 相似性度量具备对称性。这种对称性常体现在设计的相似性度量准则上，A与B计算出的相似性和B与A得到的相似性是一样。

我们的大脑深受对称性法则的影响，所以对于第一种情况，我们可以非常直观的将这一法则应用到我们的视觉模型训练中，但是对于第二种情况，特别是当相似性度量由非对称性在对称性上转化而来的时候，我们的第一意识却频频出错。计算机视觉中非对称现象和非对称相似性度量如此干扰我们的第一意识，以至于小白菜对这个问题曾做出过若干的思考，下面是小白菜结合自己的一些思考和经验做的总结和整理。

### 局部特征匹配中的非对称问题

在用局部特征进行匹配的时候，比如SIFT，用A图匹配B图和用B图匹配A图，得到的匹配结果是不一样的，如下图所示：

![](http://ose5hybez.bkt.clouddn.com/2017/1029/sift_matching_diff.jpg)

在这匹配的过程中，所有的度量方式比如最近邻、次近邻、几何校验等都是具备对称性的，所以很容易给我们造成一种错觉就是他们的匹配结果应该是一样的。这里面造成匹配结果不对称的根本因素在于**A图和B图它们的局部特征数目是不相等的**，比如A有500个局部特征，B有800个局部特征，用500个局部特征去匹配800局部个特征和用800个局部特征去匹配500个局部特征，势必造成匹配的结果不一样。一个鲁棒的匹配算法，应该在用A匹配B和用B匹配A时都能获得比较好的匹配结果，以避免单一结果匹配较好的情形，像下面的匹配算法就不是一种好的匹配算法：

![](http://ose5hybez.bkt.clouddn.com/2017/1029/false_sift_matching.jpg)

在设计匹配算法的时候，我们应避免我们的算法出现单一匹配好的情形，以提高匹配的鲁棒性。

### Logo识别中的非对称问题

Logo的识别问题，可以分为两类:  

- 基于检测的方式
- 基于检索的方式

基于检测方式的缺点在于新的Logo样式来临时，会面临训练样本如何获取以及人工标注的问题，虽然可以自动通过往图片上贴Logo的方式来构造训练样本，但这种方式容易造成过拟合；基于检索的方式的优点在于，它不会面临训练样本获取和标注的问题，能够以较快的速度响应新Logo检测的请求，但是基于Logo检索的方式，面临一个很大的问题就是尺寸不对称性。

这种尺寸不对称性体现在：对于待检测的图片，Logo所占的区域是很小的，通常区域面积占总体面积比的5%都不到，这样就导致提取的Logo区域的特征（比如局部特征）被非Logo区域的特征给“淹没”掉，如下图所示：

![](http://ose5hybez.bkt.clouddn.com/2017/1029/logo-example.jpg)

上图这个图选取得不是很合适，因为背景比较干净，Logo区域的特征还是其了比较大的作用。对于一般的情况，由于Logo区域的特征被非Logo区域的特征给淹没掉，从而使得在Logo库里检索的时候，在top@K（K取得比较小）里面比较难以检索到相关的Logo，而且即便是做重排，也比较难以将最相关的Logo排到最前面。

针对这种由于尺寸不对称性造成的有用特征（信号）被无用信号淹没的问题，很难在不对检索精度造成较大影响的前提下找到比较有效的解决办法。如果要见过无用信号造成的干扰，一种比较好的方法是先进行粗略的检测，这一步不要求检测的准确率很高，只要保证召回率很高即可，然后可以根据定位到的框提取特征，再进行检索。这种方式由于剔除了非相关区域特征的干扰，所以准确率和召回率通常能够得到较好的保证，唯一不足的是，引入了检测，而检测势必要求对数据进行标注（半自动）。不过总体来说，这仍然是一种非常不错的方法。

[Google Object Detection API to detect Brand Logos Part 1](https://medium.com/towards-data-science/google-object-detection-api-to-detect-brand-logos-fd9e113725d8)

### PQ中的非对称距离

在此前的文章[图像检索：再叙ANN Search](http://yongyuan.name/blog/ann-search.html)中，小白菜曾对PQ做过比较详细的介绍，这里对PQ中非对称距离的计算做一详述。

![](http://ose5hybez.bkt.clouddn.com/2017/0408/PQ_search_zpskgugtocx.PNG)

如上图所示，非对称距离计算方式（红框标示）在计算查询向量$x$到库中某一样本$y$之间的距离时，并不需要对查询向量$x$自身进行量化，而是直接计算查询向量$x$到量化了的$y$之间的距离，这种距离计算方式可以确保非距离计算方式以更大的概率保证计算的距离更接近于真实的距离（与对称距离计算方式相比较），而且这种方式比对称距离计算方式在实施的过程中，来得更直接。

非对称性的应用，在这里展示了它有利的一面，通过将非对称性延展到相似性度量上，可以进一步缩小量化造成的损失。

### 累计最小距离非对称问题
