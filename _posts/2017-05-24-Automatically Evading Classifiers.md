---
layout: post
permalink: 2017-05-24-Automatically Evading Classifiers
title: Automatically Evading Classifiers
---

## 动机
机器学习在信息安全领域的很多方面都有应用（恶意样本检测，入侵检测，垃圾邮件识别等等）。  

评价机器学习模型的优劣的一个重要方面就是模型对于新数据的识别能力，即模型是否过拟合。不全面的训练数据和不具有代表性的特征都会导致过拟合，因此，想要训练出一个好的模型，需要有尽可能接近目标真实分布的大量数据，还需要提取出有效的特征。  

在恶意样本检测这个问题中，这两个方面都存在固有的问题：  

* 1 攻击者总会制作新的样本或试图修改已有样本来逃避检测，这使得收集到的数据无法接近真实的分布，且随着时间推移会慢慢地越来越偏离真实的恶意样本分布；
* 2 对于提取什么特征来表示恶意行为，没有一个固定的结论。可能用到一个特征，对于收集到的数据集有较好效果，但是这一特征不是本质的，那么这就给恶意样本逃避检测指明了方向。  

本文作者提出了一种方法来检测PDF分类器是否足够健壮，能够抵御变种恶意文件的攻击。这个方法通过自动地对恶意PDF文件进行变异，在保持恶意行为的同时，使得分类器将恶意文件错误地判别为良性。

## 对象
PDF文件：从Contagio数据集中选取的500个恶意PDF文件和从Google上搜集来的几个良性PDF文件。  

PDF分类器：基于PDF文本结构的2个State of the Art分类器：PDFrate和Hidost。其中由于PDFrate无法接触到，作者使用了一个性能相近的开源重新实现Mimicus。  


## 方法
### 前提

* 1 已知特征工作在哪个级别（针对的2个分类器都是基于PDF文本结构的）；
* 2 了解模型输出的恶意性评分的含义。  

![](img_url)
### 做法  

变异PDF的body部分：PDF的body是树状的，便于进行变异。通过删除恶意文件body中的某个object、从良性文件中选取一个object插入恶意文件中和从良性文件中选取一个object替换恶意文件中的某object这3个操作随机地修改恶意文件。  

筛选：分为2个部分。第一部分是判断样本是否保持了恶意行为，这一部分是使用cuckoo沙箱进行检测的；第二部分是判断样本是否成功误导了分类器，这一部分直接把样本输入分类器然后获取输出，并根据其意义进行判断即可。  


## 结果
* 1 对于每个分类器，每个恶意样本都成功变异出了至少一个能逃避检测的样本。
* 2 对于逃避检测的样本的例子分析发现非本质的特征很容易被攻击者利用。一般的恶意样本除了payload和形成文件必要的内容以外，缺少很多正常文件具有的内容。然而这种内容是可以添加上去的，缺少之并不是恶意样本的本质特征。在例子分析中就发现，样本中font的计数这一特征占了很大的权重，通过在恶意文件中添加这类内容就能轻易地误导分类器。
* 3 有388个文件，它们对于Hidost攻击成功的样本也能成功逃避Mimicus的检测；而对于Mimicus攻击成功的，这个数字是2。这是由于Hidost的特征“蕴含”了Mimicus的特征。这个例子说明可以构造一个具有和目标分类器相近、“更本质”特征的分类器，通过对其进行攻击，获取逃逸样本，来对目标分类器进行攻击。

## 反思
* 1 使用攻击的手段判别特征的有效性，通过攻击对抗来提升分类器的能力。
* 2 变异来进行攻击是逆向的思路，正向思考，既然pdf文件使用delete变异很容易保持恶意行为，也可以将变异作为类似数据清洗的步骤，将不必要的PDF object清除后，再进行特征提取，这样也能提升模型的能力。
* 3 模型的推广。从文中假设来看，对于同样使用PDF结构作为特征依据的分类器应该可以容易地推广。针对别的样本格式、特征选取，针对性地设计变异、筛选的部分，这个框架还是可以应用的。
* 4 <欢迎补充>。