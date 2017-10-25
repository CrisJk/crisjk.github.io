---
layout: post
title: LDA主题模型的简要概括
autour: Kuang
tags: LDA
categories: ML 
---



本文为学习LDA主题模型的笔记，主要是对LDA主题模型进行一个简单的概括，具体的细节及推导可以参见:





[非常详细的参考资料][1]

## 一、问题提出

什么是主题模型？什么是LDA?

> 将文档集中，每篇文档的主题按照概率分布的形式给出，属于无监督的学习算法。需要的输入仅仅是文档集和指定的文档主题数量K

> 隐含狄利克雷分布(Latent Dirichlet allocation)简称LDA。LDA是一种典型的词袋模型，词与词之间没有顺序及先后关系。一篇文档可以包含多个主题，文档中的每个词都由其中一个主题生成。

## 二、必备知识

* 蒙特卡洛方法、马尔科夫链、MCMC采样和M-H采样，**Gibbs采样** 。[参考资料][2]
* EM算法。参考资料:统计学习方法EM算法
* 概率论相关知识以及一些优化算法(如牛顿法等)

## 三、概述

[LDA维基百科][3]

LDA模型中，一篇文档的生成方式如下

* 从狄利克雷分布中取样生成文档i的主题分布$$\theta_i$$
* 从主题的多项式分布$$\theta_i$$ 中取样生成文档i第j个词的主题$$z_{i,j}$$
* 从狄利克雷分布$$\beta$$ 中取样生成主题$$Z_{i,j}$$的词语分布$$\Phi_{z_{i,j}}$$
* 从词语的多项式分布$$\Phi_{z_{i,j}}$$中采样最终生成词语$$\omega_{i,j}$$

即：主题分布--> 主题-->词语分布-->词语

### 共轭先验分布

> 在贝叶斯统计中，如果后验分布与先验分布属于同类，则先验分布与后验分布被称为共轭分布，而先验分布被称为似然函数的共轭先验

Beta分布是二项分布的共轭先验分布，Dirichlet分布是多项分布的共轭先验分布

### LDA主题模型

目标:找到每一篇文档中的主题分布和每一个主题中词的分布

LDA的模型图如下(图片来自[pinard博客][1])

![LDA模型][4]

LDA假设主题中词的先验分布是Dirichlet分布，即对任一文档d，其主题分布$$\theta_d$$为

$$\theta_d=Dirichlet(\vec{\alpha})$$ ,$$\vec{\alpha}$$是K维(主题数)向量

LDA假设主题中词的先验分布是Dirichlet分布，即对任一主题k，其词分布为

$$\beta_k=Dirichlet(\vec{\eta})$$ , $$\vec\eta$$ 是V维(词汇表中词数)向量

对任一文档d中第n个词，可以从主题分布$$\theta_d$$ 中获得它的主题编号$$z_{dn}$$ 的分布为

$$z_{dn}=multi(\theta_d)$$

而对于该主题，得到我们看到的词$$W_{dn}$$的概率为

$$W_{dn} = multi(\beta_{z_{dn}})$$

设在文档d中，第k个主题的次数为$$n_d^{(k)}$$ ,则对应多项分布的计数为$$\vec{n_d}=(n_d^{(1)},n_d^{(2)},...,n_d^{(k)})$$

根据Dirichlet-Multi共轭，有$$\theta_d$$的后验分布为

$$Dirichlet(\theta_d \vert \vec{n_d}+\vec\alpha)$$  (新的文档-主体分布)

同理，设第k个主题中，第v个词的个数为$$n_k^{(v)}$$ ,则对应的多项分布的计数为$$\vec{n_k}=(n_k^{(1)},n_k^{(2)},...,n_k^{(V)})$$ 

根据Dirichlet-Multi共轭，有$$\beta_k$$ 的后验分布为

$$Dirichlet(\beta_k\vert\vec{n_k}+\vec\eta)$$

主题产生词不依赖具体文档，这说明文档-主题分布和主题-词分布是独立的。

以上是LDA基本原理，剩下需要解决的问题是

**基于一个LDA模型，如何求解每一篇文档的主题分布和每个主题的词分布，即如何求解 $\theta_d$ 和$\beta_k$ **

求解LDA的方法有Gibbs采样和变分推断EM两种

## 四、Gibbs采样求解LDA

Gibbs采样解决的是使采样数据符合指定分布的问题。在了解Gibbs采样之前，先了解一下蒙特卡洛方法。

### 蒙特卡洛方法

蒙特卡洛方法是一种随机的方法。我们在中小学时学习过使用投针实验估计圆周率的问题就是属于蒙特卡洛方法。一般地，蒙特卡洛方法可用来解决定积分(投针实验可以看作是求解0~r上圆形曲线的定积分问题)。

对于$$\int_a^b f(x)dx$$ ,若f(x)的原函数较难求解，则可以使用蒙特卡洛方法近似

> 在[a,b]上随机采样n个点，$$x_1,x_2,...,x_n$$
>
> $$\int_a^bf(x)dx\thickapprox\frac{b-a}{n}\sum_{i=1}^{n}f(x_i)$$

如果数学过关，那么很容易可以发现上述方法必须基于x在[a,b]上是均匀分布。然而多数情况下x并不均匀。若x在[a,b]上的概率分布函数为p(x),那么

$$\int_a^bf(x)dx=\int_a^b\frac{f(x)}{p(x)}\cdot p(x) \thickapprox\frac{1}{n}\sum_{i=1}^n\frac{f(x_i)}{p(x_i)}$$

以上我们知道了如何求任意概率分布下的f(x)的定积分，但是还有一个非常重要的问题没有解决，即如何使采样数据符合我们指定的概率分布，这也是我们需要研究的重点。

对于均匀分布，采样十分简单，一般通过线性同余发生器就可以产生0~1之间的伪随机数，并且很多常见的分布都可以根据均匀分布转化。

对于无法转化的分布，可采用**接受-拒绝采样**的方法。

### 马尔科夫链

#### 马尔科夫链介绍

使用**接受-拒绝** 采样的时间复杂度通常很高，如果需要对大量的数据进行采样，不太适用。因此引入马尔科夫链。

马尔科夫链的每一个时刻状态转移只依赖于上一个状态。

![][5]

上图构成的马尔科夫链，定义$$p(i,j)$$为$$ p(j\vert i)$$, 定义牛市状态为0，熊市状态为1,很盘状态为2,这样得到的马尔科夫链模型的状态转移矩阵为

$$p=\begin{pmatrix} 0.9 & 0.075 & 0.025 \\ 0.15 & 0. 8 & 0.05 \\ 0.25 & 0.25 & 0.5   \end{pmatrix}$$

#### 马尔科夫链性质

马尔科夫链具有一个非常好的性质，就是对应不同的初始状态，按照马尔科夫链进行转移，最终会收敛到同一状态，也就是说收敛到的稳定概率分布与初始状态概率分布无关。

这个性质为什么好呢？因为一旦我们知道了某个概率分布对应的状态转移矩阵，则可以用任意的概率分布样本开始，代入马尔科夫链的状态转移矩阵，最终得到符合对应概率分布的样本。

该性质对于大部分的马尔科夫链都成立，并且连续状态时也成立。

#### 基于马尔科夫链的采样

如果可以得到某个分布对应的马尔科夫链状态转移矩阵，就可以采样出符合该分布的样本集。

首先根据高斯分布或其他分布$$\pi_0(x)$$ 采样得到初始状态$$x_0$$ 基于条件概率分布$$p(x_0\vert x)$$ 采样得到x1，如此进行下去，直到马尔科夫状态转移矩阵收敛。n次后，马尔科夫链收敛到平稳分布$$\pi(x)$$ 再开始采样

接下来就是如何根据指定分布得到状态转移矩阵了。

### MCMC采样和M-H采样

具体的推导过程可以参考[pinard的博客][6] ， 这里不再推导证明，只简述采样过程。

马尔科夫链的细致平稳条件为

$$\pi P=\pi$$

也就满足马尔科夫链的收敛性，也就是说，只要找使概率分布$$\pi (x)$$满足细致平稳分布的矩阵。

#### MCMC采样

一般情况下

$$\pi (i)Q(i,j)\ne \pi (j)Q(j,i) $$

引入一个$$\alpha(i,j)$$ ，使

$$\pi(i) Q(i,j) \alpha(i,j) = \pi (j) Q(j,i)\alpha(j,i)   $$

只要令

$$\alpha(i,j) = \pi(j)Q(j,i)$$

$$\alpha(j,i)=\pi(i)Q(i,j)$$

一般称$$\alpha(i,j) $$ 为接受率。取值在[0,1]之间。这也就是说，**目标矩阵P可以通过任意一个马尔科夫链转移矩阵以一定的接受率获得**。

#### MCMC采样过程

[摘自Pinard博客][6]

* 输入任意选定的马尔科夫链状态转移矩阵Q,平稳分布$$\pi (x)$$ ,设定状态转移次数阈值n1,需要的样本次数n2

* 从任意简单的概率分布采样得到初始状态X0

* 从 t=0~n1+n2-1

  ​      a) 从条件概率分布$$Q(x\vert x_t)$$ 中采样得到样本$$X_*$$

  ​      b) 从均匀分布[0,1]中采样得到u

  ​      c) 如果$$u<\alpha(x_t,x_*) = \pi(x_*)Q(x_*,x_t) $$ 则接受转移$$x_t\rightarrow x_*$$ ，即$$x_{t+1}  = x_*$$

  ​      d) 否则不接受,t=max(t-1,0) 

得到样本集$$(x_{n1},x_{n1+1},...,x_{n1+n2-1})$$

#### MCMC存在的问题

当接受率较小时，采样所需时间开销很大。

### M-H采样

由于MCMC采样存在的问题，因此提出M-H采样

马尔科夫链的细致平稳条件

$$\pi (i) Q(i,j)\alpha(i,j) = \pi(j) Q(j,i) \alpha(j,i)$$

此时如果把两边同时扩大，细致平稳条件依然满足，因此，假设原来的$$\alpha(i,j)=0.1$$ $$\alpha(j,i)=0.2$$ 则两边都乘5，以增大接受率

M-H采样过程和MCMC采样类似，唯一的区别是第三步的c

c) 如果$$\alpha(x_t,x_{*}) = min(\frac{\pi (j)Q(j,i)}{\pi(i)Q(i,j)}),1) > u $$ ,则接受

### Gibbs 采样

M-H存在的缺陷主要有两点，一是由于接受率的问题导致算法收敛时间变长；二是对于有些高维数据，联合分布概率不好求，而条件概率好求时，M-H不适用。

Gibbs采样克服了以上两点问题。

同样的，这里不详细推导，只说明Gibbs采样的流程。

观察马尔科夫平稳细致条件可以发现，对于二维数据，在$$x_1= x_1^{(1)}$$ 这条直线上，如果使用条件概率分布$$\pi(x_2\vert x_1^{(1)})$$ 作为马尔科夫链的状态转移概率，则任意两个点满足细致平稳条件。

结论：平面上任意两点E,F，满足细致平稳条件

$$\pi(E)P(E\rightarrow F) = \pi(F)P(F\rightarrow E)$$

### 二维Gibbs采样

过程

* 输入平稳分布$$\pi(x1,x2)$$ , 设定状态转移次数阈值n1,需要的样本个数n2

* 随机初始化初始状态值$$x_1^{(1)}$$ 和 $$x_2^{(1)}$$

* 从 t=0 ~ n1+n2-1

  ​    a) 从条件概率分布$$P(x_2\vert x_1^{(t)})$$ 中采样得到样本$$x_2^{(t+1)}$$

  ​    b) 从条件概率分布$$P(x_1\vert x_2^{(t+1)})$$ 中采样得到样本$$x_1^{(t+1)}$$

多维Gibbs采样类比

### Gibbs 采样方法解LDA

求解LDA即根据已知所有文档形成的词向量$$\vec \omega$$ , 先求出词$$\omega$$ 和主题$$z$$ 的联合分布$$P(\vec \omega,\vec z)$$，进而求出某一个词对应主题的条件概率分布$$P(z_i=k\vert\vec \omega,\vec {z_{\Gamma i}})$$ ,$$\vec {z_{\Gamma i}}$$ ,表示去掉下标为i的词后的主题分布。有了这个条件概率分布，就可以进行Gibbs采样，在采样收敛后得到第i个词的主题。

得到每个词的主题后，通过统计每个主题的词数，得到各个主题的词分布。通过统计各个文档对应词的主题分布，得到文档主题分布。

### 得到联合分布

$$p(\vec w, \vec z)  \propto p(\vec w, \vec z\vert \vec \alpha,  \vec \eta) = p(\vec z\vert\vec \alpha) p(\vec w\vert\vec z, \vec \eta) =  \prod_{d=1}^M \frac{\triangle(\vec n_d +  \vec \alpha)}{\triangle( \vec \alpha)}\prod_{k=1}^K \frac{\triangle(\vec n_k +  \vec \eta)}{\triangle( \vec \eta)} $$

### 得到条件概率

$$\begin{align} p(z_i=k\vert \vec w,\vec z_{\neg i})  &  \propto p(z_i=k, w_i =t\vert \vec w_{\neg i},\vec z_{\neg i}) \\ & = \int p(z_i=k, w_i =t, \vec \theta_d , \vec \beta_k\vert \vec w_{\neg i},\vec z_{\neg i}) d\vec \theta_d d\vec \beta_k  \\ & =  \int p(z_i=k,  \vec \theta_d \vert  \vec w_{\neg i},\vec z_{\neg i})p(w_i=t,  \vec \beta_k \vert  \vec w_{\neg i},\vec z_{\neg i}) d\vec \theta_d d\vec \beta_k  \\ & =  \int p(z_i=k\vert\vec \theta_d )p( \vec \theta_d \vert  \vec w_{\neg i},\vec z_{\neg i})p(w_i=t\vert\vec \beta_k)p(\vec \beta_k \vert  \vec w_{\neg i},\vec z_{\neg i}) d\vec \theta_d d\vec \beta_k  \\ & = \int p(z_i=k\vert\vec \theta_d ) Dirichlet(\vec \theta_d \vert \vec n_{d, \neg i} + \vec \alpha) d\vec \theta_d \\ & * \int p(w_i=t\vert\vec \beta_k) Dirichlet(\vec \beta_k \vert \vec n_{k, \neg i} + \vec \eta) d\vec \beta_k \\ & = \int  \theta_{dk} Dirichlet(\vec \theta_d \vert \vec n_{d, \neg i} + \vec \alpha) d\vec \theta_d  \int \beta_{kt} Dirichlet(\vec \beta_k \vert \vec n_{k, \neg i} + \vec \eta) d\vec \beta_k \\ & = E_{Dirichlet(\theta_d)}(\theta_{dk})E_{Dirichlet(\beta_k)}(\beta_{kt})\end{align} $$ 

$$E_{Dirichlet(\theta_d)}(\theta_{dk}) = \frac{n_{d, \neg i}^{k} + \alpha_k}{\sum\limits_{s=1}^Kn_{d, \neg i}^{s} + \alpha_s} $$

$$E_{Dirichlet(\beta_k)}(\beta_{kt})= \frac{n_{k, \neg i}^{t} + \eta_t}{\sum\limits_{f=1}^Vn_{k, \neg i}^{f} + \eta_f} $$

因此

$$p(z_i=k\vert \vec w,\vec z_{\neg i})  = \frac{n_{d, \neg i}^{k} + \alpha_k}{\sum\limits_{s=1}^Kn_{d, \neg i}^{s} + \alpha_s}   \frac{n_{k, \neg i}^{t} + \eta_t}{\sum\limits_{f=1}^Vn_{k, \neg i}^{f} + \eta_f} $$

有了条件概率公式，就能进行Gibbs采样，当Gibbs采样收敛时，就能得到所有词的采样主题。

### LDA Gibbs采样算法流程

#### 训练流程

1) 选择合适的主题数K ,选择合适的超参数向量$$\vec \alpha , \vec \eta$$ 

2) 对应语料库中每个篇文档的每一个词，随机赋予一个主题编号z

3) 重新扫描语料库，对于每一个词，利用Gibbs采样公式，更新它的topic编号，并更新语料库中该词的编号

4) 重复3直至Gibbs采样收敛

5) 统计语料库中各个文档各个词的主题，得到文档主题分布$$\theta_d$$ ,统计语料库中各个主题的词的分布，得到LDA的主   题与词的分布$$\beta_k$$



#### 预测流程

当新文档出现时，如何统计得到该文档的主题？此时LDA的主题-词分布的参数$$\beta_k$$ 已经确定，需要得到的是文档的主题分布。

**换句话说，就是在Gibbs采样时，$E_{Dirichlet(\beta_k)}(\beta_{kt})$ 已知，只需对前半部分 $E_{Dirichlet(\theta_d)}(\theta_{dk})$  进行采样计算即可**

1) 对当前文档主题的每个词，随机赋予一个主题编号z

2) 重新扫描当前文档，对每一个词，利用Gibbs采样公式，更新它的topic编号

3) 重复2直至Gibbs采样收敛

4) 统计文档中各个词的主题，得到该文档的主题分布 

## 五、变分推断EM求解LDA

### EM算法

EM算法解决的问题是**求解含有隐变量的概率模型的参数极大似然估计或极大后验概率估计**

* 输入:观测变量数据Y，隐变量数据Z，完全数据的联合分布$$P(Y,Z\vert\theta)$$，隐变量的条件分布$$P(Z\vertY,\theta)$$
* 输出:模型参数

EM算法一般分为E步和M步两步，E步是求完全数据对于隐变量的期望，M步为最大化这个期望，至于为什么是求期望，是根据极大似然估计推导得出。

### 变分推断EM算法求解LDA

[参考资料][7]

变分推断EM通过变分推断和EM来得到LDA的文档主题分布和主题词分布

LDA模型中的隐藏变量为$$\theta , \beta , z$$ ，模型的参数为$$\alpha,\eta$$ 。求解它的主要思想是，使用EM算法在E步求出隐藏变量$$\theta,\beta,z$$ 的基于条件概率分布的期望，然后再M步极大化这个期望，然后得到更新后的参数$$\alpha,\eta$$ 。

然而现在的问题是，我们难以求出$$\theta,\beta,z$$ 的条件概率分布。因此需要“变分推断”方法，通过变分假设，假设所有的隐藏变量都是通过各自的独立分布形成。即

![][8]

接下来就是如何求$$\gamma, \Phi,\lambda$$  以及解LDA了，具体推导参见[该链接][7]



[1]: http://www.cnblogs.com/pinard/p/6831308.html
[2]: http://www.cnblogs.com/pinard/p/6625739.html
[3]: https://zh.wikipedia.org/wiki/%E9%9A%90%E5%90%AB%E7%8B%84%E5%88%A9%E5%85%8B%E9%9B%B7%E5%88%86%E5%B8%83
[4]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/LDA_model.png
[5]: https://raw.githubusercontent.com/CrisJk/SomePicture/master/blog_picture/600px-Finance_Markov_chain_example_state_space.svg.png
[6]: http://www.cnblogs.com/pinard/p/6638955.html
[7]: http://www.cnblogs.com/pinard/p/6873703.html
[8]: http://images2015.cnblogs.com/blog/1042406/201705/1042406-20170518154844557-29824317.png