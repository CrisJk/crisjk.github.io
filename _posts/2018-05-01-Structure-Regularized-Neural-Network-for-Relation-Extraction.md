---
layout: post
title: Structure Regularized Neural Network for Entity Relation Classification for Chinese Literature Text 笔记
author: Kuang
categories: knowledgeGraph
tags: RelationExtraction

---
### 1. Background

本文提出了一种从中文文学作品中提取实体关系的方法。相对于英文的关系提取，中文的关系提取存在以下几个难点:

* 中文的文学作品倾向于直觉和情感的表达，而科学的分析较少
* 中文的文学作品往往缺乏严谨的逻辑，并且喜欢使用各种各样不同的句式来表达不同的情感
* 中文的句子之间没有明显的连词关联
* 中文是一门topic-prominent的语言，主语通常比较隐蔽，词的用法相对复杂

总而言之，中文的文学作品往往强调”意合“，并不依赖句法结构来表达逻辑，一些文学作品在语法表达上更是天马行空；而英文强调”形合“，即通过一些特定的句法结构来显性地表达逻辑. 因此，中文文本中往往含有大量没有用的词，并且往往有复杂而又灵活的结构。而现有的很多方法主要是利用句法信息(例如词性标注、依存关系)来进行关系提取。对于中文文本来说，直接产生的句法信息往往可信度不高，质量较低，因此很难达到理想的效果.

基于以上难点，本文提出的方法，利用结构正则化(Structure regularization)去除句法信息中的噪声. 通过正则化后的句法信息(最短依赖路径, Shortest Dependency Path, SDP) ，进行关系提取.








### 2. Related Work

#### 2.1 Relation Classification

***Bunescu and Mooney (2005)*** 首先在关系提取中应用SDP来表达两个实体间的句法信息. ***Xu et al. (2015)*** 在LSTM上应用SDP进行关系提取.

当然，还有一些没有用到SDP，但是非常有意思的工作. 例如***Wang et al.(2016)*** 提出一种使用two level attention的卷积神经网络. ***Wang et al. (2017)*** 将神经网络和句法分析树结合.***Liu et al. (2017)*** 提出一种可以容忍远程监督学习中存在的wrong label problem的方法.

#### 2.2 Structure Regularization

结构预测(Structured Prediction)广泛应用于解决各种各样的”结构依赖“问题，***Sun (2014a), Sun et al. (2017a) 以及Sun et al. (2017b)*** 指出，复杂的结构更可能导致过拟合问题. ***Sun (2014b)*** 指出，无论从经验上还是理论上,结构正则化都能够有效地缓解过拟合问题. 



关于这篇论文，可以参考作者的另外两篇论文，一篇是关于语料库的获取[A Discourse-Level Named Entity Recognitionand Relation Extraction Dataset for Chinese Literature Text](https://arxiv.org/pdf/1711.07010.pdf) ;另一篇是关于BRCNN模型[Structure Regularized Bidirectional Recurrent Convolutional Neural Network for Relation Classification](https://arxiv.org/pdf/1711.02509.pdf)

### 3. Corpus

本文的一大贡献点便是提供了一个高质量的中文语料库. 作者也专门在另一篇文章中介绍了该语料库. ***Xu et.al(2017)*** .该语料库可以在Github下载[(下载地址)](https://github.com/lancopku/Chinese-Literature-NER-RE-Datase) . 

图1是语料库的一个示例

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/chinese-literature-text-corpus.png)

​                                            Figure 1: Examples from the Chinese literature text corpus.

实际上，标注的数据有两种，一种用于关系提取，另一种用于命名实体识别.

* 用于关系提取的标注数据

  ```

  T1	Time-Implicit 2 11	上世纪50年代中期
  T2	Person-Pronoun 12 13	我
  T3	Location-Nominal 14 18	完全中学
  T4	Person-Nominal 65 83	一位矮瘦并戴着高度近视眼镜的青年老师
  T5	Person-Pronoun 203 204	他
  *	Coreference T5 T4 T8 T10 T12 T14 T15 T18 T19 T20 T22 T33 T34
  .
  .
  .
  R1	Social Arg1:T101 Arg2:T102	
  *	Coreference T101 T103 T106 T110 T112
  R2	Social Arg1:T103 Arg2:T104	
  ```

  其中，第一列表示ID,实体用T表示，关系用R表示，共指关系用`*`表示.

  * 对于实体的标注: 第2列表示实体的类型;第3列和第4列表示实体的起始位置和终止位置;第5列表示实体内容
  * 对于共指的标注: 列出所有共指的实体id
  * 对于关系的标注: 第2列表示关系类型;第3列和第4列表示实体对.

* 用于命名实体识别的标注

  ```
  记 O
  得 O
  小 B_Time
  时 I_Time
  候 I_Time
  ， O
  妈 B_Person
  妈 I_Person
  说 O
  起 O
  哪 O
  个 O
  典 O
  型 O
  败 B_Person
  家 I_Person
  子 I_Person
  形 O
  象 O 　
  ```

直接给每个词打上词性标注.

关于语料库的获取，本文爬取了1000+的中文文章，筛除掉太短或者噪声太多的文章，从剩下的837篇文章中获得语料. 语料的标注经过3个阶段.

* 首先根据定义好的实体和关系进行手工标注，由于不同人对于语句的理解不同，可能造成标注数据不一致的问题.例如,如图2所示,Hamlett本来是个人名，但在文中是指代一只兔子.而有些标注者会把Hamlett标注成"Person"，有些标注者则会将其标注为"Thing".

  ![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/chinese-text-annotation-error.png)

  ​                                                                 Figure 2: A tagging example

* 为了解决标注数据不一致的问题，作者使用启发式的方法进行数据标注，即定义一系列消歧规则.例如，去除所有的形容词，只保留"entity header"（例如，将"穿红衣服的女孩"转换成"女孩").

* 仅仅通过启发式方法并不能完全解决标注数据不一致的问题,作者最后还利用机器辅助标注的方法，即利用全部数据的一个子集学习一个标注器，再在其他数据上用这个标注器预测标签. 文章中作者使用的模型是基于简单2元语法的CRF模型.最后，在预测的标签上，再进行人工校对，得到最终的标注数据.

本文定义了7种实体类型和9种关系类型,如表1和表2所示

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/chinese-literature-entity-type.png)

Table 1: The set of entity tags. The last column indicates each tag’s relative frequency in the full annotated data.

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/chinese-literature-relation-type.png)

Table 2: The set of relation tags. The last column indicates each tag’s relative frequency in the full annotated data

###  4. Model

#### 4.1 Basic BRCNN

Basic BRCNN(Bidirectional Recurrent Convolutional Neural Network)用于学习最短依赖路径(Shortest Dependency Path, SDP)上的信息表示.其中"Recurrent"部分使用LSTM实现，利用SDP学习词法和依存关系的隐含表示. 而卷积层则是从隐含表示中，捕捉相邻两个词的"local feature".卷积层之后是一个max pooling, 用于聚集(gather)local feature. 最后,一个softmax层用于分类.整个模型的框架如图3所示.

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/BRCNN-architecture.png)

Figure 3: The overall architecture of BRCNN. Two-Channel recurrent neural networks with LSTM units pick up information along the shortest dependency path, and inversely at the same time. Convolution layers are applied to extract local features from the dependency units. In the example, we conduct the process between the entities of “土船”(boat) and “清江”(river)

从图中可以看到，两个双向的LSTMs，一个用于捕捉单词的特征，一个用于捕捉依存关系的特征. 

附常见依存关系

| 缩写   | 含义    |
| ---- | ----- |
| SBV  | 主谓关系  |
| VOB  | 动宾关系  |
| IOB  | 间宾关系  |
| FOB  | 前置宾语  |
| DBL  | 兼语    |
| ATT  | 定中关系  |
| ADV  | 状中结构  |
| CMP  | 动补结构  |
| COO  | 并列关系  |
| POB  | 介宾关系  |
| LAD  | 左附加关系 |
| RAD  | 右附加关系 |
| IS   | 独立结构  |
| HED  | 核心关系  |

将words和relations进行embedding后，输入到LSTMs中.经过LSTMs编码后的向量为$h_t$ . 将每两个相邻的词以及它们之间的依存关系编码后的向量级联得到$[h_a h_{ab} h_b]$ ,级联后的向量经过卷积层得到依存单元$L_{ab}$ 的表示.

$$L_{ab} = f(W_{con}\cdot [h_a\oplus h_{ab}^{'}\oplus h_b]+b_{con})$$

其中, $W_{con}$ 是权重矩阵,$b_{con}$ 是偏置项, 激活函数采用正切函数.在卷积层之后是一个最大池化.

softmax层有3个softmax分类器，其中一个粗粒度的分类器，将数据划分成K+1类；两个细粒度的分类器，分别根据正向和反向的SDP产生的结果$\overrightarrow G$ 和 $\overleftarrow G$ ,产生分类结果$\overrightarrow y$ 和 $\overleftarrow y$ .

$$\overrightarrow y  = softmax(W_f \cdot \overrightarrow G + b_f)$$

$\overleftarrow y = softmax(W_f \cdot \overleftarrow G + b_f )$



损失函数是基于三个分类器的交叉熵来设置的

$$J = \sum_{i=1}^{2K+1}\overrightarrow {t_i}log\overrightarrow{y_i} + \sum_{i=1}^{2K+1}\overleftarrow {t_i}log \overleftarrow{y_i} + \sum_{i=1}^{K} t_ilog y_i + \lambda\cdot {\Vert\theta\Vert}^2$$

最终的预测结果是$\overrightarrow y$ 和$\overleftarrow y$ 的结合

$$y_{text} = \alpha \cdot \overrightarrow y + (1-\alpha)\overleftarrow y$$



#### 4.2 Structure Regularized BRCNN

直接产生的依存树往往结构复杂，从复杂的结构中提取的最短依赖路径中往往有许多冗余的词，这点在长句中更加明显，***Sun(2014b)*** 中指出，无论是在理论上还是实际上，structure regularization都能够控制过拟合的风险.因此，本文首先对依存树进行structure regularization, 基于一些启发式的规则,选取树中部分节点作为根节点，得到森林，再将这些根节点连接起来，得到正则化后的树结构. 再从树中得到最短依存路径, 称之为SR-SDPs. 图4所示的是一个structure regularization的例子

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/SR-SDPs.png)

Figure 4: An example of the proposed method. The two words in circles are the entities, and the thick edges form the SDP. By flattening the dependency tree, the path becomes shorter.

本文采用了3种不同的正则化方法:

* 利用标点的自然分割:根据文本中的标点，将句子进行分割，每一段生成一棵树，最后得到一片森林
* 随机选择切割的点: 字面意思
* 基于介词分割: 根据文本中的介词进行分割，这种分割最能保持原有的信息

### 5. Experiment

实验结果如表3所示

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/SR-BRCNN-Result.png)

Table 3:  Comparison of relation classification systems on Chinese literature text 

表4所示的是不同的正则化方法的结果,实验证明，随机分割的结果于基于标点分割；基于介词的分割优于以上两种分割方法. 这是由于仅仅基于标点的方法，不能充分地正则化；而随机的方法，尽管容易丢失一些信息，但能够很好的对结构进行正则化；而基于介词的方法则是一种更加合理的办法，不仅使得正则化更加充分，并且减少了信息的丢失.

![](https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/differentRegularizationResult.png)

​                             Table 4: Different structure regularization results on Chinese literature texts

### 6. Contributions

* 得到了一个有标注的中文文本语料库
* 提出了一种对于树结构的正则化方法，此前的structure regularization都是基于序列结构
* 将结构正则化应用于关系提取,缓解了了中文文本中冗余信息较多的问题

### 7. References

Razvan Bunescu and Raymond Mooney. 2005. A shortest path dependency kernel for relation extraction. In Proceedings of Human Language Technology Conference and Conference on Empirical Methods in Natural Language Processing, pages 724–731, Vancouver, British Columbia, Canada. Association for Computational Linguistics. 

Rui Cai, Xiaodong Zhang, and Houfeng Wang. 2016. Bidirectional recurrent convolutional neural network for relation classification. In ACL (1).

Iris Hendrickx, Su Nam Kim, Zornitsa Kozareva, Preslav Nakov, Diarmuid O. S´eaghdha, Sebastian ´ Pad ´o, Marco Pennacchiotti, Lorenza Romano, and Stan Szpakowicz. 2010. Semeval-2010 task 8: Multi-way classification of semantic relations between pairs of nominals. In Proceedings of the 5th International Workshop on Semantic Evaluation, SemEval ’10, pages 33–38, Stroudsburg, PA, USA. Association for  Computational Linguistics. Sepp Hochreiter and J¨urgen Schmidhuber. 1997. Long short-term memory. Neural computation, 9(8):1735–1780.

Tianyu Liu, Kexiang Wang, Baobao Chang, and Zhifang Sui. 2017. A soft-label method for noisetolerant distantly supervised relation extraction. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing, pages 1790–1795.

Yang Liu, Furu Wei, Sujian Li, Heng Ji, Ming Zhou, and Houfeng Wang. 2015. A dependency-based neural network for relation classification. In In Proceedings of the 53rd Annual Meeting of the Association for Computational Linguistics and the 7th International Joint Conference on Natural Language Processing (Volume 2: Short Papers. Citeseer.

Tomas Mikolov, Ilya Sutskever, Kai Chen, Greg S Corrado, and Jeff Dean. 2013. Distributed representations of words and phrases and their compositionality. In Advances in neural information processing systems, pages 3111–3119.

Cicero Nogueira dos Santos, Bing Xiang, and Bowen Zhou. Classifying relations by ranking with convolutional neural networks.

Richard Socher, Jeffrey Pennington, Eric H Huang, Andrew Y Ng, and Christopher D Manning. 2011.
Semi-supervised recursive autoencoders for predicting sentiment distributions. In Proceedings of the
conference on empirical methods in natural language processing, pages 151–161. Association for Computational Linguistics.

Xu Sun. 2014a. Structure regularization for structured prediction. In Advances in Neural Information Processing Systems, pages 2402–2410.

Xu Sun. 2014b. Structure regularization for structured prediction: Theories and experiments. CoRR,
abs/1411.6243.
Xu Sun, Xuancheng Ren, Shuming Ma, Bingzhen Wei, Wei Li, and Houfeng Wang. 2017a. Training simplification and model simplification for deep learning: A minimal effort back propagation method. CoRR, abs/1711.06528.

Xu Sun, Weiwei Sun, Shuming Ma, Xuancheng Ren, Yi Zhang, Wenjie Li, and Houfeng Wang. 2017b. Complex structure leads to overfitting: A structure regularization decoding method for natural language processing. CoRR, abs/1711.10331.

Linlin Wang, Zhu Cao, Gerard de Melo, and Zhiyuan Liu. 2016. Relation classification via multi-level attention cnns. In ACL (1).
Yizhong Wang, Sujian Li, Jingfeng Yang, Xu Sun, and Houfeng Wang. 2017. Tag-enhanced tree-structured neural networks for implicit discourse relation classification. In Proceedings of the Eighth International Joint Conference on Natural Language Processing, IJCNLP 2017, Taipei, Taiwan, November 27 - December 1, 2017 - Volume 1: Long Papers, pages 496–505.

Ji Wen. 2017. Structure regularized bidirectional recurrent convolutional neural network for relation classification. CoRR, abs/1711.02509.

Jingjing Xu, Ji Wen, Xu Sun, and Qi Su. 2017. A discourse-level named entity recognition and relation extraction dataset for chinese literature text. CoRR, abs/1711.07010.

Yan Xu, Lili Mou, Ge Li, Yunchuan Chen, Hao Peng,
and Zhi Jin. 2015. Classifying relations via long short term memory networks along shortest dependency paths. In EMNLP, pages 1785–1794. Matthew D Zeiler. Adadelta: An adaptive learning rate method.

Daojian Zeng, Kang Liu, Siwei Lai, Guangyou Zhou, Jun Zhao, et al. 2014. Relation classification via
convolutional deep neural network. In COLING, pages 2335–2344.

Shu Zhang, Dequan Zheng, Xinchen Hu, and Ming Yang. 2015. Bidirectional long short-term memory networks for relation classification. In PACLIC.
