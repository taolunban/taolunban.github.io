---
layout: post
permalink: 2017-06-21-Skyfire: Data-Driven Seed Generation for Fuzzing
title: Skyfire: Data-Driven Seed Generation for Fuzzing
---

## 发表会议
IEEE Symposium on Security and Privacy 2017

## 作者信息
Junjie Wang, Bihuan Chen†, Lei Wei, and Yang Liu 
>* Nanyang Technological University, Singapore
>* {wang1043, bhchen, l.wei, yangliu}@ntu.edu.sg 
>* †Corresponding Author 

## Abstract
对于输入格式是高度结构化文件的程序来说，其处理流程一般是：语法解析--语义检查--程序执行。程序深层次的漏洞一般隐藏在程序执行阶段，
而对于自动化的模糊测试(Fuzzing)来说很难触发该类漏洞。<br>
该论文提出了一种数据驱动的种子生成方法，叫做Skyfire。Skyfire通过从大量的已知样本中学习而生成覆盖良好的种子作为Fuzzing的输入对处
理高度结构化输入的程序进行测试。Skyfire接收输入样本集合和文法，通过自动化学习PCSG（一种改了的上下文敏感的文法，包含语义规则和语
法特征），并利用其生成种子文件。<br>
本文利用收集的样本和Skyfire生成的种子作为AFL的seed对开源的XSLT、XML等引擎进行测试，证明skyfire生成的种子文件分布（提高了20%行覆
盖率和15的函数覆盖率）和发现漏洞能力。同时也对闭源的IE11的JavaScript引擎测试。其发现了19个内存破坏型缺陷和32个DOS缺陷。

## Introduction
Fuzzing是一种自动化的随机测试技术，其通过变异或者生成的方法生成大量的测试样本，并利用生成的测试样本对目标程序进行测试和监控，以发现
程序异常和缺陷。模糊测试的输入种子文件的质量是对测试效果的重要影响因素。<br />

	一个高效的Fuzzer需要实现大部分的生成样本可以到达处理执行阶段（execution stage)。

基于变异的方法是通过随机或者启发式的方法对合法的输入种子文件进行变异生成测试用例，大部分的生成用例在早起的语法检查阶段就被拒绝而导致
程序退出。然而，基于生成的方法是利用格式描述或文法描述来生成测试用例，可以快速的通过语法检查阶段，但是大部分程序在语义检查阶段也难以
通过，这都限制了这些方法难以挖掘程序的深层次漏洞。<br />
!["结构化输入的程序处理流程"](../assets/Skyfire-image1.png)
> 基于生成的方法能够实现对语法规则的描述和生成，但是想要通过语义规则确实非常困难的。一方面，对于不同的程序有不同的语义规则，编写的生
成规则难以复用，另一方面，这样的手动描述方法是非常耗时费力的，而且有时候甚至是难以实现的。<br />

