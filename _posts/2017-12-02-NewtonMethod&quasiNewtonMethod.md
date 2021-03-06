---
layout: post
title: 优化算法——牛顿法和拟牛顿法
categories: ML
tags: optimization
author: Kuang
---



牛顿法和拟牛顿法常用于**无约束最优化问题** ，优点是收敛速度快。



## 牛顿法

对于有约束最优化问题，可以使用拉格朗日乘子法构建极大极小或者极小极大问题，再求最优解。而对于无约束最优化问题，常用牛顿法或者拟牛顿法解决。

对于无约束最优化问题

​                                                                                        $$\min _{x\in R^n} f(x) \tag{1}$$                                                                                   

其中$x^*$ 为目标函数的的极小值点。

对$$f(k)$$进行二阶泰勒展开(假设能展开) ，结果为(多元函数的泰勒展开)

​                           $$f(x)=f(x^{(k)}) + g^T_k(x-x^{(k)})+\frac{1}{2}(x-x^{(k)})^TH(x^{(k)})(x-x^{(k)})) \tag{2}$$                                           

其中，$$g_k = g(x^{(k)})=\nabla f(x^{(k)})$$ 是$$f(x)$$ 在$$x^{(k)}$$ 处的梯度向量。$$H(x^{(k)})$$ 是$$f(x)$$ 在$$x^{(k)}$$ 处对应的Hessian矩阵。

函数$$f(x)$$ 有极值点的必要条件是在极值点处一阶导数为0, 即梯度向量为0. 当$$H(x^{(k)})$$ 为正定矩阵时，函数$$f(x)$$ 的极值为极小值。

目标就是找到使梯度向量为0的点。每次迭代从$$x^{(k)}$$ 开始，求目标函数的极小点，第k+1次迭代值$$x^{(k+1)}$$ ，假设$$x^{(k+1)}$$ 满足

​                                                                                         $$\nabla f(x^{(k+1)}) = 0 \tag{3}$$

$$f(x)$$ 对$$x$$ 进行求导(对二阶泰勒展开式求导)可得

​                                                                          $$\nabla f(x) = g_k + H_k(x-x^{(k)})  \tag{4}$$                                                             

$$H_k$$ 即 $$H(x^{(k)})$$ ,综合(3).(4)得

​                                                                                 $$g_k + H_k(x^{(k+1)-x^{(k)}}) = 0 \tag{5}$$                                 

得

​                                                                                 $$x^{(k+1)}  = x^{(k)} - H_k^{-1}g_k  \tag{6}$$        

令$$H_kp_k=-g_k$$ , 有

​                                                                                   $$x^{(k+1)} = x^{(k)}+p_k  \tag{7} $$                      

用式(6) 作为迭代公式的算法就是牛顿法。



### 牛顿法算法流程

 输入：目标函数$$f(x)$$ ，梯度$$g(x)=\nabla f(x)$$ ，Hessian矩阵$$H(x)$$ ，精度要求$$\epsilon$$ 

输出：$$f(x)$$ 的极小值点$$x^*$$

* 取初始点$$x^{(0)}$$ , $$k=0$$

* 计算$$g_k=g(x^{(k)})$$ 

* 若$$\|g_k\|<\epsilon$$ ，则得到近似解$$x^*=x^{(k)}$$ ， 结束

* 否则计算$$H_k=H(x^{(k)})$$ , 并求$$p_k$$

  $$H_kp_k=-g_k$$

* 置$$x^{(k+1)} = x^{(k)}+p_k$$ , k=k+1 ,转第二步

## 拟牛顿法

事实上，求hessian矩阵的逆矩阵比较复杂，因此相办法找一个矩阵$$G_K = G(x^{(k)})$$ 来代替hessian矩阵$$H_k^{-1} = H^{-1}(x^{(k)})$$ 。

### Hessian矩阵需要满足的条件

根据公式(4)，取$$x=x^{(k+1)}$$ 得

​                                                                        $$g_{k+1}-g_k=H_k(x^{(k+1)}-x^{(k)})  \tag{8} $$                                                

记$$y_k=g_{k+1}-g_k$$ ,$$\delta_k=x^{(k+1)}-x^{(k)}$$ ，有 

​                                                                                         $$yk=H_k\delta_k \tag{9} $$                                                                                                                                                                           

或者

​                                                                                         $$H^{-1}_ky_k=\delta_k \tag{10}$$                                                                          

称(9)或(10)为拟牛顿条件。

当$$H_k$$ 正定时，可以保证牛顿法的搜索方向$$p_k$$ 是下降方向而不是上升方向。有

$$x=x^{(k)}+\lambda p_k=x^{(k)}-\lambda H^{-1}_kg_k$$ 

$$f(x)$$ 在$$x^{(k)}$$ 处的泰勒展开式可以写作

​                                                                                $$f(x)=f(x^{(k)})-\lambda g_k^TH_k^{-1}g_k \tag{11}$$                                                      

由于$$H^{-1}_k$$ 正定，故有$$g_K^TH^{-1}_kg_k>0$$ ，当$$\lambda$$ 为一个充分小的正数时，总有$$f(x) < f(x^{(k)})$$ , 也就是说，$$p_k$$ 是下降方向。

综上，$$H_k$$ 应当满足两个条件

* 为正定矩阵
* $$y_k=H_k\delta_k$$ 或 $$y_kH^{-1}_k=\delta_k$$

因此要找到一个$$G_k$$ 近似$$H_k^{-1}$$ ，应当满足同样的条件。

按照拟牛顿条件选择$$G_k$$ 作为$$H_k^{-1}$$ 的近似或选择$$B_k$$ 作为$$H_k$$ 的近似算法称为拟牛顿法，按照拟牛顿条件，每次迭代中可以选择更新矩阵$$G_{k+1}$$ :

​                                                                                $$G_{k+1} = G_k + \Delta G_k \tag{12}$$                                                 

### DFP算法

迭代 $$G_{k+1}$$ 的方法是，假设每一步迭代中矩阵$$G_{k+1}$$ 是由$$G_k$$ 加上两个附加项构成的，即

​                                                                            $$G_{k+1} = G_k+P_k+Q_k \tag{13}$$                                               

其中$$P_k$$ 和 $$Q_k$$ 是待定矩阵，这时

​                                                                      $$G_{k+1} y_k = G_ky_k+P_ky_k +Q_ky_k \tag{14}$$                                        

为使$$G_{k+1}$$ 满足拟牛顿条件，可使$$P_k$$ 和 $$Q_k$$ 满足

​                                                                              $$P_ky_k=\delta_k \tag{15}$$                                                                     

​                                                                          $$Q_ky_k=-G_ky_k \tag{16}$$                                                               

$$P_k$$ 和 $$Q_k$$ 可以取

​                                                                             $$P_k=\frac{\delta_k\delta_k^T}{\delta_k^Ty_k} \tag{17}$$                                                                     

​                                                                         $$Q_k=-\frac{G_ky_ky_k^TG_k}{y_k^TG_ky_k} \tag{18}$$                                                         



该算法称为DFP算法，可以证明，如果初始矩阵$$G_0$$ 为正定矩阵，则迭代过程中每个矩阵$$G_k$$ 都是正定。

#### DFP算法流程

输入：目标函数$$f(x)$$ ,梯度$$g(x) = \Delta f(x)$$ ,精度要求$$\epsilon$$ 

输出：$$f(x)$$ 的极小点$$x^*$$

* 选定初始点$$x^{(0)}$$ ，取$$G_0$$ 为正定对称矩阵，置$$k=0$$

* 计算$$g_k=g(x^{(k)})$$ 

* 若$$\|g_k\|<\epsilon$$ ，则得到近似解$$x^*=x^{(k)}$$ ， 结束

* 否则，置$$p_k = -G_kg_k$$

* 一维搜索：求得$$\lambda_k​$$ 使

  $$f(x^{(k)}+\lambda_kp_k)=\min_{\lambda_k\ge0} f(x^{(k)}+\lambda p_k)$$

* 置$$x^{(k+1)}=x^{(k)}+\lambda_kp_k$$

* 计算$$g_{k+1}=g(x^{(k+1)})$$ ,若$$\|g_{k+1}\|<\epsilon$$ ,则停止计算，得到近似解$$x^*=x^{(k+1)}$$ , 否则按照公式(13)计算$$G_{k+1}$$ 

* k=k+1 ,转第二步



### BFGS 算法

BFGS算法类似于DFP算法，只不过它是用$$B_k$$ 逼近Hessian矩阵$$H$$ 