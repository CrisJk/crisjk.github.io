---
layout: post
author: Kuang
title: 《Joint Extraction of Entities and Relations Based on a Novel Tagging Scheme》阅读笔记
categories: knowledgeGraph
tags: RaltionExtraction
---

> 从非结构化的文本中获取三元组关系，通常有两种做法:
> 一种方法是将实体识别和关系抽取当做两个任务，串行执行。这种方法的好处在于可以简化问题，并且每个模块可以更加灵活，但这种方法忽略了这两个子任务之间的联系，并且实体识别的误差会传播到关系提取模块，造成整体效果的下降。
> 另一种方法是使用一个模型对实体识别和关系提取联合进行学习，现有工作中，很多联合抽取的模型都是给予特征工程的，这就意味着需要手工的选择特征，这对算法工程师是个很大的考验。同时，基于特征工程的方法依赖于自然语言处理工具，这意味着自然语言处理工具的错误会传播到算法模型。当然，近期也有人提出基于神经网络的方法(Makoto Miwa and Mohit Bansal. 2016. End-to-end re- lation extraction using lstms on sequences and tree structures. In Proceedings of the 54rd Annual Meet- ing of the Association for Computational Linguis- tics.)。尽管他们提出的方法能用一个共享参数的模型同时解决实体和关系提取问题，但是其实质上仍然是将这两个任务分开进行，因此会产生一些冗余的信息。



### 1 Introduction

从非结构化的文本中获取三元组关系，通常有两种做法:

一种方法是将实体识别和关系抽取当做两个任务，串行执行。这种方法的好处在于可以简化问题，并且每个模块可以更加灵活，但这种方法忽略了这两个子任务之间的联系，并且实体识别的误差会传播到关系提取模块，造成整体效果的下降。

另一种方法是使用一个模型对实体识别和关系提取联合进行学习，现有工作中，很多联合抽取的模型都是给予特征工程的，这就意味着需要手工的选择特征，这对算法工程师是个很大的考验。同时，基于特征工程的方法依赖于自然语言处理工具，这意味着自然语言处理工具的错误会传播到算法模型。当然，近期也有人提出基于神经网络的方法(Makoto Miwa and Mohit Bansal. 2016. End-to-end re- lation extraction using lstms on sequences and tree structures. In Proceedings of the 54rd Annual Meet- ing of the Association for Computational Linguis- tics.)。尽管他们提出的方法能用一个共享参数的模型同时解决实体和关系提取问题，但是其实质上仍然是将这两个任务分开进行，因此会产生一些冗余的信息。

本文提出的方法，直接从语句中提取出一对实体和这两个实体间的关系，即直接得到（实体-关系-实体）三元组。该方法将实体提取和关系提取任务看成标注问题，提出一种将标注方法和end-to-end方法相结合的模型来解决联合抽取任务。

### 2 Contributions

* 提出一种新的标注方法，将联合抽取实体和关系的任务转化为标注任务
* 使用不同的end-to-end模型来解决该问题，效果好于绝大多数现有的串行抽取和联合抽取方法
* 提出一种偏置损失函数，使用该损失函数可以使相关实体间的联系变得更强

### 3 Method

##### Tagging Scheme

图1是一个tagging的结果，从图中可以看出，每一个词被分配一个label，“O"表示"Other"标签。其它标签由３个部分组成:该单词在实体中的位置，关系类型，以及实体在三元组关系中的角色。使用"BIES"(Begin,Inside,End,Single)来表示单词在实体中的位置。关系的类型信息是从一个预先定义好的集合中获得，实体在三元组关系中的角色用"1”和"2"来表示。提取的结果用三元组来表示(Entity1,RelationType,Entity2)，"1"表示单词属于三元组中的第一个实体，"2"表示单词属于三元组中的第二个实体。

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/taggingResult.png)

图1:Gold standard annotation for an example sentence based on our tagging scheme, where “CP” is short for “Country-President” and “CF” is short for “Company-Founder”.

从图1所示的标注结果来看，"Trump"和"United States"共享同一个关系类型”Country-President“，"Apple Inc"和"Steven Paul Jobs"共享同一个关系类型"Company-Founder"。将关系类型相同的实体组成三元组得到最后的结果。当一个句子中有两个或更多的三元组有相同的关系类型，那么根据就近原则组合实体。例如，假设图１中，United States和Trump的关系类型是"Company-Founder"，那么根据就近原则，最后结果为{United States,Company-Founder,Trump}，{App Inc,Company-Founder,Steven Paul Jobs}。本文只讨论一个实体只属于一个三元组的情况。

##### The End-to-End Model

近年来基于神经网络的end-to-end模型广泛应用于序列标注任务。本文提出的end-to-end模型如图2所示:

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/end-to-endModel.png)

图2:　An illustration of our model. (a): The architecture of the end-to-end model, (b): The LSTM memory block in Bi-LSTM encoding layer, (c): The LSTM memory block in LSTMd decoding layer

该模型分为5层: 输入层、Embedding层、Encoding层、Decoding层和输出层

* Embedding层

  Embedding层将每个词的1-hot表示转换成embedding向量，则词序列可以表示为：

  $W=\{W_1,W_2,W_3,...,W_t,W_{t+1},...,W_n\}$,　$W_t$ 表示序列中第t个词汇对应的向量，维度为d,n表示给定句子的长度。

* 双向LSTM编码层

  Word embedding层之后是两个平行的的LSTM层，即前向和后向的LSTM层，由一系列循环神经子网组成。每一个时间步长是一个LSTM记忆块，每一个隐层向量$$h_t$$ 是由上一时刻的隐层向量$h_{t-1}$ 、cell向量$c_{t-1}$ 以及当前输入的词向量$W_t$计算得到，如图2(b)所示。计算公式如下:

  ![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/formula1.png)

  其中i,f和o分别是input gate，forget gate和output gate。b是偏置项，c是记忆细胞，W(.)是参数。对于每个单词$\omega_t$ ,前向的LSTM层利用$\omega_1$ 到$\omega_t$ 的上下文信息，得到向量$\overrightarrow {h_t}$ 。类似地，反向的LSTM层利用从$\omega_n$ 到$\omega_t$ 的上下文信息得到向量$\overleftarrow {h_t}$ 。最后，将$\overrightarrow {h_t}$ 和 $\overleftarrow {h_t}$连接来表示单词t编码后的结果$[\overrightarrow {h_t},\overleftarrow {h_t}]$ 。

* LSTM解码层

  本文使用LSTM来标注序列,当标注$\omega_t$的标签时，解码层的输入为: Bi-LSTM的输出$h_t$　，前一时刻预测标签对应的embedding$T_{t-1}$ ,前一时刻的记忆细胞值$c_{t-1}^{(2)}$ ,以及解码层前一时刻的输出$h_{t-1}^{(2)}$ ，计算过程如图2(c)所示，计算公式如下:

  ![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/formula2.png)

  ![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/formula3.png)

  最后，通过softmax层计算归一化的实体标签概率

  ![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/formula4.png)

### 4 The Bias Objective Function

利用极大似然估计来优化模型，损失函数如下式所示:

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/JointExtractionLossFunction.png)

$\vert \mathbb D\vert$ 是训练集的大小，$L_j$是句子$x_j$的长度，$y_t^{(j)}$是句子$x_j$中单词t的label，$p_t^{(j)}$ 是正则化的实体标注预测概率值，$I(O)$ 是一个"开关函数"，用来区分tag是否是'O'(即Other)。其表达式如下:

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/switchingFunction.png)

$\alpha$是偏置权重，$\alpha$越大，关系标注的结果对模型的影响越大(即实体关系标注对损失函数的影响更大)。

### 5 Experiments

#### Experimental setting

**Dataset**: 使用NYT(纽约时报)数据集进行验证，该数据集的训练集部分，是由远程监督的方法自动标注的，而测试集部分则是手工进行标注的，训练集包括353k对三元组，测试集包括3880对三元组。关系集合的大小为24。

**Evaluation**: 采用标准的Precision、Recall和F1-score来评估结果。与传统方法不同的是，本文没有利用实体类型信息来进行三元组提取，因此在评估时不需要考虑实体类型的准确度。当一个三元组的关系类型和两个对应的实体位置都正确时，该三元组被认为是正确的。同时，为了使实验结果更具有说服力，本文将测试数据分为两部分，一部分为全部数据的10%用来得到实验结果，剩下的90%用来验证实验结果的准确性。重复10次实验得到均值和标准差。

**Hyperparameters **: 本文通过word2vec来进行word embedding，word embeddings的维数是300维。编码层的LSTM单元是300个，解码层的LSTM单元数目是600个。Bias Objective Function中的$\alpha$设为10。实验结果如表１所示:

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/jointExtractionofTable1.png)

表 1: The predicted results of different methods on extracting both entities and their relations. The first part (from row 1 to row 3) is the pipelined methods and the second part (row 4 to 6) is the jointly extracting methods. Our tagging methods are shown in part three (row 7 to 9). In this part, we not only report the results of precision, recall and F1, we also compute their standard deviation.

**Baselines**:　Baseline分为３类，一类是将实体抽取和关系抽取当成是两个串行的任务进行，还有一类是其他的联合抽取方法，最后一类是传统的end-to-end标注模型。对比实验的选择详见论文。

#### Experimental Results

如表１所示，本文提出的方法效果是最好的。并且当编码层和解码层都使用LSTM时，相比较于解码层采用CRF，可以在precision和recall之间达到一个平衡(F1-score)。对比LSTM-LSTM和LSTM-LSTM-Bias可以发现，使用Bias Objective Function可以有效的实体标记的影响，从而达到比一般的LSTM-LSTM模型更好的效果。

### 6 Analysis and Discussion

#### Error Analysis

算法对于三元组中各个元素的预测效果如表2所示,E1表示对第一个实体的预测效果，E2表示对第二个实体的预测效果，(E1,E2)表示对一对实体的预测效果。

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/jointExtractionofTable2.png)

表 2: The predicted results of triplet’s elements based on our tagging scheme

从表２中可以看到，(E1,E2)相比于E1和E2具有更高的Precision，但是具有更低的Recall。这意味着一些实体没有形成实体对。也就是说只找到了第一个实体，而没有找到第二个实体，或者只找到了第二个实体，没找到第一个实体。通过将(E1,E2)的结果和表１中相应的结果对比，发现Precicon上升了约3%，这意味着有3%的错误是由于关系类型错误导致的。

#### Analysis of Biased Loss

图4所示的是LSTM-CRF、LSTM-LSTM、LSTM-LSTM-Bias三种算法得到单个实体的比例，即只发现一个实体而未发现与其配对实体的比例。从图４中可以看出，LSTM-LSTM-Bias发现单个实体的比例最低，这意味着LSTM-LSTM-Bias更容易发现有关系的实体对。

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/jointExtractionfigure4.png)

图 3: The ratio of predicted single entities for each method. The higher of the ratio the more entities are left.



### 7 Conclusion

本文提出了一种新的标注模型，并对端到端的模型进行研究，从而对实体和关系进行联合提取。但是对于一对实体具有多种关系的情况无法处理。
