---
layout:     post
title:      "Notes - PRML&ESL | 机器学习白板推导笔记"
subtitle:   " \"Mathematical Derivation of Machine Learning\""
date:       2023-10-01 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Machine Learning
    - Math
    - Notes
---



2023-05-10

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents


* TOC
{:toc}






## Frequency and Baysian

## Math Basics

For Gaussian Distribution under 1 dimension and high dimensions,

$P(X)=\frac{1}{\sqrt(2\pi)\sigma}\exp(-\frac{(X-\mu)^2}{2\sigma^2})$

$P(X)=\frac{1}{\sqrt{2\pi}^{\frac{p}{2}}\\|\Sigma\\|^{0.5}}\exp(-\frac{1}{2}(X-\mu)^T\Sigma^{-1}(X-\mu))$

Let Data: $X=(X_1,X_2,X_3...,X_N)_{N*P}^T, X_i\in \mathbb{R}^P,\;X_i\stackrel{\text{i.i.d.}}{\sim}N(\mu, \Sigma)$

Under MLE we have, 

$\theta_{MLE}=argmax_{\theta}P(X\\|\theta),~\theta=(\mu, \sigma^2),  ~ \log P(X\\|\theta)\stackrel{i.i.d.}{=}\log \prod_{i=1}^{N}P(X_i\\|\theta)=\sum_{i=1}^N \log P(X_i\\|\theta)$

assume P=1, 
$$
\begin{align*}
&\log P(X\|\theta)=\sum_{i=1}^{N}\log \frac{1}{\sqrt{2\pi}\sigma}\exp(-\frac{1}{2}\frac{(X_i-\mu)^2}{\sigma^2})\newline 
&=\sum(\log\frac{1}{\sqrt{2\pi}}+\log\frac{1}{\sigma}-\frac{(X_i-\mu)^2}{2\sigma^2})\newline 
\end{align*}
$$
The fisrt two items are inrelevant to $\mu$, so the estimate of $\mu$ is:
$$
\begin{align*}
&\mu_{MLE}=argmax_{\mu}\log(X\|\theta)\newline 
&=argmax_{\mu}\sum(-\frac{(X_i-\mu)^2}{2\sigma^2})\newline 
&=argmax_{\mu}\sum(X_i-\mu)^2\newline 
&\frac{\partial}{\partial\mu}\sum(X_i-\mu)^2=\sum2(X_i-\mu)(-1)=0\newline 
&\sum_{i=1}^{N}(X_i-\mu)=0\newline 
&\sum X_i-\sum\mu=0,~where\sum\mu=N*\mu\newline 
&\mu_{MLE}=\frac{1}{N}\sum_{i=1}^{N}X_i\newline 
&\text{i.e. Sample Mean}\newline 
&E[\mu_{MLE}]=E[\frac{1}{N}\sum_{i=1}^{N}X_i]=\frac{1}{N}\sum E[X_i]=\frac{1}{N}\sum\mu=\mu\newline 
\end{align*}
$$
So the estimate of $\mu$ is unbiased, which means that if you use the sample mean to estimate the mean parameter of a normal distribution, this estimate is "unbiased" in the statistical sense. This is a desirable property because an unbiased estimator, over multiple sampling and estimation processes, will converge to the true parameter on average, without systematically deviating from it.

Now we estimate $\sigma^2$:

 





## Linear Regression 线性回归

## Linear Classification 线性分类

## Dimensionality Reduction 降维

## Support Vector Machine 支持向量机

SVM’s 3 key points:

间隔 对偶 核技巧

types：

Hard-margin SVM

Soft-margin SVM

Kernel-SVM



<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/prml/image-20231009010651648.png?raw=true" alt="image-20231009010651648" style="zoom: 50%;" />

$f(w)=sign(w^Tx+b)$, 本质上是一个判别模型，跟概率没关系。

存在多条可以正确分类的直线，要挑出来最好的一条。鲁棒性更强，对噪声更不敏感。最“中间”的一个超平面。

$\{(x_i,y_i)\}^N_{i=1}\;x_i\in R^P,~y_i\in \{-1,1\}$

**<u>最大间隔分类器，如何定义？</u>**

首先定义“分类”：

max margin(w,b)，$s.t.\begin{cases}w^Tx_i+b>0,~y_i=+1 \newline  w^Tx_i+b<0,~y_i=-1\end{cases} $

等价于$y_i(w^Tx_i+b) > 0\;\;\;\forall~i=1,..,N$

再定义间隔：

distance = $ \frac{1}{∥w∥}\\|w^Tx_i+b\\| $

 样本点到超平面的distance，由于有N个样本点，则有N个distance。在其中中找到最小的作为margin，有 $margin(w,b)=\stackrel{min}{w,b,x_i}distance(w,b,x_i) =min \frac{1}{∥w∥}\\|w^Tx_i+b\\|$

=> $\begin{cases}max_{w,b}~min_{x_i}\frac{1}{∥w∥}\\|w^Tx_i+b\\|\newline\;  s.t.\;y_i(w^Txi+b)>0\end{cases}$

由于下式恒大于0，而上式的绝对值符号也大于0，因此去掉绝对值而乘以yi

=> $\begin{cases}max_{w,b}~min_{x_i}\frac{1}{∥w∥}y_i(w^Tx_i+b)\newline  s.t.\;y_i(w^Txi+b)>0\end{cases}$

与w无关，继续简化

=> $\begin{cases}max_{w,b}~\frac{1}{∥w∥}min_{x_i}y_i(w^Tx_i+b)\newline  s.t.\;y_i(w^Txi+b)>0\end{cases}$

其中，下式又可以转换为

=> $\begin{cases}\exists! \gamma>0,~s.t. min~y_i(w^Tx_i+b)=\gamma\newline max_{w,b}~\frac{1}{∥w∥}min_{x_i}y_i(w^Tx_i+b)=max_{w,b}~\frac{1}{∥w∥}\gamma\end{cases}$

Let γ=1，他说是因为 超平面可以缩放，因此要固定下来w的模，对等式没有影响。 ？？？？

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/prml/image-20231009013048869.png?raw=true" alt="image-20231009013048869" style="zoom:50%;" />

=> $\begin{cases}max_{w,b}~\frac{1}{\\|w\\|}=min∥w∥=min_{w,b}~\frac{1}{2}w^Tw\newline  s.t.\;min~y_i(w^Txi+b)=1, \forall i=1,...,N\end{cases}$

目标函数中的1/2主要是为了求导方便

目标函数是二次的，约束有N个，是一个凸优化问题





**<u>5.2 有了目标函数，如何求解？</u>**

当w特征维度和样本量不大的时候，较容易求解。

但是在一些情况下，比如做了一下映射，从原空间映射到了更高维度空间就不易求解。

此处讲解基于拉格朗日乘数法的对偶法

先写出拉格朗日函数：

$L(w,b,\lambda)=\frac{1}{2}w^Tw+\sum_{i=1}^N\lambda_i(1-y_i(w^Tx_i+b))$

有$\lambda_i\ge0$ *（此处是因为KKT条件，紧约束在最优解处成立什么的，等最优化学到了再说吧。。）*

将带约束的目标函数就转换为无约束（对w和b没有限制条件）：

 $\begin{cases}min_{w,b}max_{\lambda}L(w,b,\lambda)\newline  s.t.\;\lambda_i\ge0\end{cases}$

进一步理解上式：

对于某个样本 $(x_i,y_i)$

如$1-y_i(w^Tx_i+b)>0,\;max_{\lambda}L(w,b,\lambda)=\frac{1}{2}w^Tw+∞=∞$，

如$1-y_i(w^Tx_i+b)\le0,\;max_{\lambda}L(w,b,\lambda)=\frac{1}{2}w^Tw+0=\frac{1}{2}w^Tw$

所以第二种情况下，

$min_{w,b}max_{\lambda}L=min_{w,b}(∞,\frac{1}{2}w^Tw)=min_{w,b}\frac{1}{2}w^Tw$ 与原来是等价的。

也就是说，这种方法把使这个式子大于0的w,b的部分剔除了。所以新的约束没有wb的身影，但是本质还是等价的。

这时候新的式子也还是原问题。

写出对偶问题：

 $\begin{cases}max_{\lambda}min_{w,b}L(w,b,\lambda)\newline  s.t.\;\lambda_i\ge0\end{cases}$

对偶关系里有弱对偶关系：min max L >= max min L

但是我们想要强对偶，即相等。

由于这里的原问题是凸二次函数，且约束为线性，就是强对偶，证明略。

强对偶意味着原问题和对偶问题同解。

现在先来求$min_{w,b}L$,

$\frac{\partial L}{\partial b}=\frac{\partial }{\partial b}[\sum \lambda_i-\sum \lambda_iy_i(w^Tx_i+b)]=\frac{\partial }{\partial b}[\sum \lambda_iy_ib]=-\sum\lambda_i y_i=0$

将上面的结果代入L,将L的项全部展开

$L=\frac{1}{2}w^Tw+\sum\lambda_i-\sum \lambda_iy_iw^Tx_i$

再用上式对w求偏导。

$\frac{\partial L}{\partial w}=\frac{1}{2}2w-\sum \lambda_iy_ix_i=0$

=><mark>$w^*=\sum\lambda_iy_ix_i$</mark>

$minL=\frac{1}{2}(\sum\lambda_iy_ix_i)^T(\sum\lambda_iy_ix_i)-\sum_{i=1}^{N}\lambda_iy_i(\sum_j^N\lambda_jy_jx_j)^Tx_i+\sum\lambda_i$

$=\frac{1}{2}\sum_{i=1}^T\sum_{j=1}^T\lambda_i\lambda_jy_iy_jx_i^Tx_j+\sum \lambda_i$

此时，问题变为：去除了w和b，只剩λ

 $\begin{cases}max_{\lambda}\frac{1}{2}\sum_{i=1}^T\sum_{j=1}^T\lambda_i\lambda_jy_iy_jx_i^Tx_j+\sum \lambda_i\newline  s.t.\;\lambda_i\ge0\end{cases}$

<mark>由于原问题 对偶问题 具有强对偶关系的充要条件是满足KKT条件</mark>

KKT条件：(说此处无法对lambda求导)

 $\begin{cases}\frac{\partial L}{\partial w}=0,\frac{\partial L}{\partial b}=0~①\newline \lambda_i(1-y_i(w^Tx_i+b))=0~②，(松弛互补条件)\newline \lambda_i\ge0~③\newline1-y_i(w^Tx_i+b)\le0~④\end{cases}$

此时根据KKT条件来计算出 w\*和b\*。

w\*已经由前面的过程得到。

关于b\*。 结合下图看②式④式，当④等于0时，意味着(xi,yi)就是支持向量，此时，②式中λ可以为大于等于0的任意值。而如果④式小于0，则②式的λ一定为0。

这意味着不在这两条线上的数据点其实是没有作用。这也是SVM方法稀疏性的体现，只有支持向量对决策边界有影响。

> 拉格朗日乘子 λi 表示第 i 个约束对目标函数的影响。如果  λi 为零，则意味着对应的样本点不是支持向量，不会影响决策边界。如果 λi 大于零，则意味着对应的样本点是支持向量，并有助于确定决策边界。

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/prml/image-20231010210256680.png?raw=true" alt="image-20231010210256680" style="zoom:50%;" />

假设$\exists~(x_k,y_k),~s.t.~1-y_k(w^Tx_k+b)=0$ 意味着这个样本是支持向量

$y_k(w^Tx_k+b)=1$

$由于y_k=±1，两边同乘y_k:y^2_k(w^Tx_k+b)=y_k$

$w^Tx_k+b=y_k$

<mark>$b^{*}=y_k-\sum\lambda_iy_ix_ix_k$</mark>

超平面：$w^{\*T}x+b^{\*}$

决策函数：$f(x)=sign((w^{\*})^T+b^\*)$

由结论可见，w\*本质是data的线性组合。但是由于λ只有在支持向量上的样本不是0，所以这个线性组合中的大部分项都是0。

> 1. **拉格朗日乘子 λ 的优化**：
>    - 在SVM的对偶问题中，我们主要是通过优化拉格朗日乘子λ来求解问题。通过解决对偶问题，可以得到一组最优的拉格朗日乘子 λ
> 2. **权重向量w和偏置b的解析解**：
>    - 一旦得到了最优的拉格朗日乘子λ，就可以通过它们解析地计算权重向量w和偏置b。具体来说，权重向量w可以表示为所有支持向量的线性组合，其系数由拉格朗日乘子和对应的类标签决定
>    - 偏置b可以通过任一支持向量xk和对应的类标签yk解析地计算，或者通过支持向量的平均值来计算。
>
> 通过上述分析，可以得出“w和b是解析解，λ是优化得到的”这个表述是正确的。在求解SVM时，首先通过优化对偶问题得到拉格朗日乘子λ，然后再解析地通过λ计算w和b。这种方法使得SVM能够有效地处理分类问题，并允许通过核技巧来解决非线性问题。



<u>**5.3 Soft-Margin SVM**</u>

实际的数据中，经常数据不可分或者存在噪声。

思想：允许一点点错误

$min\frac{1}{2}w^Tw+Loss$

Loss的选择：

1）loss=$\sum^N_{i=1}I[y_i(w^Tx_i+b)<1]$

但问题指示函数不连续，存在跳跃。所以无法通过这种数错误数量的方法作为loss

2）loss：距离

若$y_i(w^Tx_i+b)\ge0,~loss=0$

$y_i(w^Tx_i+b)<0,loss=1-y_i(w^Tx_i+b)$

$loss=max[0,1-(y_i(w^Tx_i+b))]=max[0,1-z]$

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/prml/image-20231010213750138.png?raw=true" alt="image-20231010213750138" style="zoom:67%;" />

就是hinge loss

所以此时目标函数为：

$min_{w,b}\frac{1}{2}w^Tw+C\sum^N_{i=1}max\{0,1-y_i(w^Tx_i+b)\}$

令$1-y_i(w^Tx_i+b)=\xi_i,又有max的条件所以\xi_i\ge0$

由于引入了loss，那么原来的目标函数的约束$y_i(w^Tx_i+b)\ge1就需要改为y_i(w^Tx_i+b)\ge1-\xi_i$

因此有：

$\begin{cases}min_{w,b}~\frac{1}{2}w^Tw+C\sum\xi_i\newline  s.t.y_i(w^Tx_i+b)\ge1-\xi_i, \xi_i\ge0\end{cases}$

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/prml/image-20231010214822831.png?raw=true" alt="image-20231010214822831" style="zoom:50%;" />



同样地，构造拉格朗日函数$L(w,b,\xi,\lambda,\mu)$ 引入拉格朗日乘数λ和$\mu$。

有$L=\frac{1}{2}‖w‖^2+C\sum^N_{i=1}\xi_i-\sum^N_{i=1}\lambda(y_i(w^Tx_i+b)-1+\xi_i)-\sum^N_{i=1}\mu_i\xi_i$

利用KKT条件，特别是互补松弛：

$\lambda_i(y_i(w_Tx_i+b)-1+\xi_i)=0,\forall i$

$\mu_i\xi_i=0,\forall i$

为解决对偶问题，需要求解拉格朗日函数的极小值，对w,b，ξ求偏导令其=0.

$\begin{cases}\frac{\partial L}{\partial w}=w-\sum\lambda_iy_ix_i=0 \newline \frac{\partial L}{\partial b}=-\sum\lambda_iy_i=0 \newline \frac{\partial L}{\partial \xi}=C-\lambda_i-\mu_i=0\end{cases}$

而又因为KKT条件，$\lambda_i\ge0,\mu_i\ge0$

可得 $0\le\lambda_i\le C$

将上述求导结果代入得到对偶问题的最终L形式

$L(\lambda, \mu) = \frac{1}{2} \left( \sum_{i=1}^n \lambda_i y_i x_i \right)^T \left( \sum_{j=1}^n \lambda_j y_j x_j \right) + C \sum_{i=1}^n \xi_i - \sum_{i=1}^n \lambda_i(y_i(w^T x_i + b) - 1 + \xi_i) - \sum_{i=1}^n \mu_i \xi_i$

$L(\lambda, \mu) = \sum_{i=1}^n \lambda_i - \frac{1}{2} \sum_{i=1}^n \sum_{j=1}^n \lambda_i \lambda_j y_i y_j x_i^T x_j$

$\text{所以，对偶问题是：}$

$\max_\lambda \left[ \sum_{i=1}^n \lambda_i - \frac{1}{2} \sum_{i=1}^n \sum_{j=1}^n \lambda_i \lambda_j y_i y_j x_i^T x_j \right]$

$\text{subject to}$

$0 \leq \lambda_i \leq C, \, \forall i = 1, \ldots, n$
$\sum_{i=1}^n \lambda_i y_i = 0$





**<u>5.4 补充：约束优化问题</u>**

假设有一个原问题 $\begin{cases}min_{x\in R^P}f(x)\newline s.t.\;~m_i(x)\le0,i=1,...,M\newline s.t.\;~n_j(x)=0,j=1,...,N\end{cases}$ 

意为有M个不等式约束，N个等式约束

(注意此处的x实际上是w，b之类的，不是指样本。)

**首先构造其拉格朗日函数**：

$L(x,\lambda,\eta)=f(x)+\sum^M_{i=1}\lambda_im_i+\sum^N_{j=1}\eta_jn_j$

$\begin{cases}min_{x}max_{\lambda,\eta}L(x,\lambda,\eta)\newline s.t.\;~\lambda_i\ge0\end{cases}$ 

也就是说，转换为了一个没有x的新的约束问题。

**从逻辑上分析为什么两者是等价的？**

若x违反了约束$m_i(x) $意味着$m_i(x)>0,导致max_{\lambda}L\to∞$,

*我的理解是，你要最大化关于λ和η的L函数，求其最大值，然而里面的“常数项”是个正数，那么在你调节λ的过程中，会无限将其放大以求其最大值，那就是正无穷了。*

而如果x符合$m_i(x)\le0,那么max_\lambda L\ne+∞$

那么 $min_{x}max_{\lambda,\eta}L=min_x\{max_{\lambda}L,+∞\}=min_xmax_{\lambda}L$

也就是说只有这个情况下，minmax这个拉格朗日目标函数才是可求解的，而这个时候正是过滤了不符合条件的x，也就是隐式地进行了x的约束。

**接着来看对偶问题：**

构造其对偶问题：

$\begin{cases}max_{\lambda,\eta}min_{x}L(x,\lambda,\eta)\newline s.t.\;~\lambda_i\ge0\end{cases}$ 

对于弱对偶性：对偶问题的值≤原问题的值，的理解：

$max_{\lambda,\eta}min_{x}L(x,\lambda,\eta)\le min_{x}max_{\lambda,\eta}L(x,\lambda,\eta)$

显然，一定有$min_{x}L\le L \le max_{\lambda,\eta}L$ （这个式子我有点难以接受）

此时$min_xL变成A(\lambda,\eta),~max_{\lambda,\eta}L变为B(x)$

$A(\lambda,\eta)\le B(x)$

$\to~A(\lambda,\eta)\le minB(x)$

$\to~maxA(\lambda,\eta)\le minB(x)$

即$max~minL\le min ~maxL$



凸优化只有在+slater条件的时候才能推出是强对偶关系，否则只有弱对偶关系

强对偶不一定有slater条件，还可能是其他条件。

slater条件：



对于大多数凸优化问题，slater条件成立

放松的slater条件：如果M中有K个仿射函数，那么只需要校验K+1到M的约束条件是否严格＜0.





**KKT条件：**

可行条件：

$\begin{cases}m_i(x^\*)\le0 \newline n_j(x^\*)=0 \newline \lambda^\*\ge0\end{cases}$

互补松弛：$\lambda_im_i=0$

梯度为0：$\frac{\partial L(x,\lambda^\*,\eta^\*)}{\partial x}\|_{x=x^\*}=0$



推导：

令 $g(\lambda,\eta)=min_xL(x,\lambda,\eta) $
$ where\;L=f(x)+\sum_i\lambda_im_i+\sum_j\eta_jn_j$


$ d^\*=max_{\lambda,\eta}g(\lambda,\eta)=g(\lambda^\*,\eta^\*)$
$= min_xL(x,\lambda^\*,\eta^\*) \le L(x^\*,\lambda^\*,\eta^\*)$
$ =f(x^\*)+\sum_i \lambda_i^\*m_i+\sum_j\eta_jn_j $
$ \le f(x^\*)=p^\* $

其中由可行条件知nj=0，所以第三项为0

而由 convex + slater = strong duality，而strong duality <=> KKT

strong duality 意味着 d* (对偶问题)= p* (原问题)

所以上面推导过程中的≤应为 =。一旦变成等号，那么第二项$\sum_i\lambda^*_im_i$意味着也为0，而这个sum为0意味着每一项应均为0，即为互补松弛条件

而第一个≤变成＝意味着 $min_xL(x,\lambda^\*,\eta^\*)=L(x^\*,\lambda^\*,\eta^\*)$

=> 梯度为0



## Kernel Methods 核方法

核心思想：

**<mark> 从模型角度来说，非线性带来的高维转换；</mark>**

| 线性可分        | 一点点错误      | 严格非线性                    |
| --------------- | --------------- | ----------------------------- |
| PLA感知机       | Pocket Algo     | φ(x)+PLA                      |
| Hard Margin SVM | Soft Margin SVM | Φ(x)+Hard-Margin = Kernel SVM |

对于PLA来说，解决非线性问题的思路是，PLA->多层感知机(Neural Network)->DeepLearning

而另一个思路是：将非线性可分的问题转换成一个线性可分的问题。

Φ(x)：就是一种非线性转换

假设有一个样本集：

Label +1: (0,1),(1,0)

Label -1: (0,0) (1,1)

做一次转换，从Input space X -> Feature space Z

$x =(x_1,x_2) \to Z=(x_1,x_2,(x_1-x_2)^2)$

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/prml/image-20231012160051674.png?raw=true" alt="image-20231012160051674" style="zoom:50%;" />

此时变成线性可分。

Cover Theorem： 高维空间比低维空间更容易线性可分

**<mark>从优化角度来看，对偶表示带来内积;</mark>**

正如前面SVM部分通过对偶问题来求解。最后对偶问题的目标函数中含有xixj，即数据本身的内积。SVM应对的是线性可分的情况。然而对于线性不可分的情况，解决方案就是要映射到高维空间。此时xixj就会变成Φ(xi)Φ(xj)的高维内积。本身高维的Φ(xi)计算量就很大，更别提所有数据的内积。

因此，引入Kernel Function就是针对求这种高维内积的时候

Kernel Function:

$K(x,x^{'})=\phi(x)^T\phi(x^{'})=<\phi(x),\phi(x^{'})>$

$\forall x,x^{'}\in X,~\exists~\phi:X\to Z,$

$s.t.~K(x,x^{'})=\phi(x)^T\phi(x’)则称K(x,x')是一个核函数$

比如说一个核函数$K(x,x')=exp(-\frac{(x-x')^2}{2\sigma^2})$避免了先将x向高维映射后再内积。



**<u>通常说的核函数是正定核函数。</u>**

核函数：$K:X\times X\to R, ~\forall x,z\in X，~有K(x,z)$

也就是说任意从X集合中取出的一个值对的笛卡尔积都可以得到一个实数结果

正定核函数：

$K:X\times X\to R, \;~\forall x,z\in X，\;\;有K(x,z)$

若 $\exists:\phi:x\to R,\phi\in Hilbert Space $
$ s.t.\; K(x,z)=<\phi(x),\phi(z)>$, 则称K(x,z)为正定核函数

若$K(x,z)满足1)对称性~2)正定性$，也可以称K(x,z)为正定核函数

其中对称性是指K(x,z)=K(z,x);

正定性是指从X中任取N个元素，对应的Gram Matrix是半正定的。

证明：$K(x,z)=<\phi(x),\phi(z)>$ 等价于 `Gram Matrix半正定`

GramMat: $K=[K(x_i,x_j)]_{N\times N}$,也就是说从X中取出的元素两两求内积（我的理解是，在核方法的上下文中，这里是通过K核函数求出来的，而不是真的通过Φ先映射到高维空间再求点积，那等于这一切都白做了）所形成的矩阵，格拉姆矩阵也就是Kernel Matrix。

希尔伯特空间：完备的，可能是无限维的，被赋予内积运算的线性空间

完备指对极限操作是封闭的，集合or函数{Kn}的limKn=K是收敛的，并且极限值依然∈H；被赋予内积是指，正定性，对称性，线性。若f,g ∈H，正定性是指<f,f>≥0, ”=“ 等价于 f=0. 线性是指<r1f1+r2f2,g>=r1<f1,g>+r2<f2,g>

 必要性证明，已知$K(x,z)=<\phi(x),\phi(z)>$ 证明GramMatrix半正定且K(x,z)对称：

首先证明对称性：

$ K(x,z)=<\phi(x),\phi(z)>  $ <br>
$K(z,x)=<\phi(z),\phi(x)>$ <br>
而内积具有对称性，即 $<\phi(x),\phi(z)> = < \phi(z),\phi(x)>$

所以 $  K(x,z)=K(z,x)$

然后证明半正定：（证明半正定的两种方式：所有特征值大于0 or $\alpha^TK\alpha>=0$

$证明：\forall\alpha \in R^N, \alpha^TK\alpha\ge0 $

$\alpha^TK\alpha=\alpha_{1\times N}^TK_{N\times N}\alpha_{N\times 1}  $ <br>
$=\sum_{i=1}^N\sum_{j=1}^N\alpha_i\alpha_jK_{ij}  $
$ =\sum_{i=1}^N\sum_{j=1}^N\alpha_i\alpha_j<\phi(x_i),\phi(x_j)>  $ <br>
$ =\sum_{i=1}^N\sum_{j=1}^N\alpha_i\alpha_j\phi(x_i)^T\phi(x_j) $
$ =\sum_{i=1}^N\alpha_i\phi(x_i)^T\sum_{j=1}^N\alpha_j\phi(x_j) $
$ =[\sum_{i=1}^N\alpha_i\phi(x_i)]^T\sum_{j=1}^N\alpha_j\phi(x_j) $
$ =<\sum_{i=1}^N\alpha_i\phi(x_i),\sum_{j=1}^N\alpha_j\phi(x_j)>  $ <br>
$ = ‖ \sum_{i=1}^N\alpha_i\phi(x_i)‖^2\ge0$

  所以K是半正定的

在很多机器学习算法中，如支持向量机（SVM），核矩阵的半正定性保证了优化问题的凸性。这使得可以找到全局最优解，并且使得优化算法的收敛性得到保证。





**计算出核矩阵之后的下一步，以核SVM为例**

1.构建核矩阵K

首先，选择一个核函数，例如线性核、多项式核或径向基函数（RBF）核等。核函数应该根据数据的特性和问题的需求来选择。

对于给定的数据集 $ \{x_1, x_2, \ldots, x_n\}$,计算核矩阵的每个元素 $ \ K_{ij} $为 $ \ K(x_i, x_j)$, 结果是一个$ \ n \times n $的矩阵，其n是数据点的数量

2.解决SVM的优化问题

$\text{SVM的目标是找到一个决策边界，使得分类间隔最大化。在线性SVM中，优化问题可以写作：}$

$\min_{w,b} \frac{1}{2} \\|w\\|^2 + C \sum_{i=1}^n \xi_i$

$\text{subject to}$

$y_i(w^T x_i + b) \geq 1 - \xi_i, \ \xi_i \geq 0$

$text{在核SVM中，通过使用核技巧，优化问题变为对偶问题，可以写作：}$

$\max_\lambda \left[ \sum_{i=1}^n \lambda_i - \frac{1}{2} \sum_{i=1}^n \sum_{j=1}^n \lambda_i \lambda_j y_i y_j K(x_i, x_j) \right]$

$\text{subject to}$

$0 \leq \lambda_i \leq C, \ \sum_{i=1}^n \lambda_i y_i = 0$

在解决这个优化问题后，我们可以得到 $ \ \lambda_i$ 的值，它们是拉格朗日乘数。

非零的$ \lambda_i 对应于支持向量$

$\text{决策函数可以写作：}$

$f(x)=w^{\*T}x+b^\*$

$f(x)=(\sum_{i=1}^N\lambda_i^\*y_ix_i)^Tx+b^\*$

$f(x) = \sum_{i=1}^n \lambda_i^\* y_i K(x, x_i) + b$

通过解决这个优化问题，我们可以得到分类决策边界或回归函数。对于分类，符号函数 $\text{sign}(f(x))$给出了类标签。对于回归，函数$f(x)$ 本身给出了预测值.





## Exponential Family Distributions 指数族分布

概率知识补充：

Family: 

Gauss，Bern，二项分布，泊松分布，Beta，Dirichlet，Gamma



指数族分布一般形式：

$P(x\|\eta) = h(x)exp(\eta^T\phi(x)-A(\eta))$

where $\eta:参数向量，x\in R^p$

$A(\eta)$ log partition function 对数配分函数 

如$P(x\|\theta)=\frac{1}{z}\hat{p}(x\|\theta), where~z=\int\hat{p}(x\|\theta)dx$

对于上面的一般形式，配分函数$z =exp(A(\eta))$，剩下的部分则为$\hat{p}(x\|\theta)$



指数族分布的特点：

充分统计量：$\phi(x)$, 什么是统计量？对样本的加工，函数，比如期望。这个统计量就足够可以表达样本的特征。比如对于高斯分布，充分统计量 $\phi(x)=(\sum x_i, \sum x_i^2)$,通过这两个统计量就可以得到期望和方差从而得到分布。

这一点对机器学习中的online learning很重要。

共轭：

对于 $P(z\|x)=\frac{P(x\|z)P(z)}{\int_zP(x\|z)P(z)dz}$ 难点在于求积分。

因此处理方式包括近似推断（MCMC，变分推断），另一个思路就是共轭。

就是对于似然函数的分布如果存在一个它的共轭先验，那么先验将与后验具有相同的形式

比如似然函数是二项式分布，则有共轭先验beta分布，那么后验也将是beta分布。

最大熵：无信息先验。

1.广义线性模型：线性组合wTx, link function: 激活函数的反函数， y\|x~指数族分布. 比如线性回归y服从N，分类y\|x服从Bern

2.概率图模型：无向图

3.变分推断





高斯分布的指数族分布形式

以一维高斯分布为例：

$P(x \\| \theta) = \frac{1}{\sqrt{2\pi}\sigma}exp(\frac{(x-\mu)^2}{2\sigma}) $
$=\frac{1}{\sqrt{2\pi\sigma^2}}exp(-\frac{1}{2\sigma^2}(x^2-2x\mu+\mu^2))$
$ =exp(log(2\pi\sigma^2)^{-\frac{1}{2}})exp(-\frac{1}{2\sigma^2}(x^2-2x\mu+\mu^2)) $
$=exp(-\frac{1}{2}log(2\pi\sigma^2)-\frac{1}{2\sigma^2}(x^2-2x\mu+\mu^2))$

其中$x^2-2x\mu$可写为 $(-2\mu\;1)(x\;x^2)^T$

将前面的-1/2σ^2乘进去，令 $\eta=(\eta1\;\eta2)^T=(\frac{\mu}{\sigma^2} \;-\frac{1}{2\sigma^2})^T$

又可得以η表示的μ和σ：

 $\mu=-\frac{\eta_1}{2\eta_2}$
 $\sigma=-\frac{1}{2\eta_2}$

代入化简得：

$=exp((\frac{\mu}{\sigma^2}\;-\frac{1}{2\sigma^2})(x\;x^2)^T-(-\frac{\eta^2_1}{4\eta_2}+\frac{1}{2}log(-\frac{\pi}{\eta_2})))$
$ 即=exp(\eta^T\phi(x)-A(n))$



充分统计量Φ(x)与对数配分函数A（η）的关联：
对数配分函数关于η的一阶导数就是Φ(x)的期望，二阶导是Φ(x)的方差
利用概率密度函数积分为1

将指数族分布的一般形式改写为：

$P(x\|\eta)=\frac{1}{exp(A(\eta))}h(x)exp(\eta^T\phi(x)) 两边同时积分$
$ 显然exp(A(\eta))=\int h(x)exp(\eta^T\phi(x))dx$

如果对上式两边同时求导

$exp(A(\eta))A'(\eta)=\frac{\partial}{\partial \eta}(\int h(x)exp(\eta^T\phi(x))dx)$

积分符号和求导先后可互换（什么条件下？）

$=\int h(x)exp(\eta^T\phi(x))\phi(x)dx $
$ \to A'(\eta)=\frac{\int h(x)exp(\eta^T\phi(x))\phi(x)dx}{exp(A(\eta))}$
$=\int h(x)exp(\eta^T\phi(x)-A(\eta))\phi(x)dx$
$ =\int P(x\|\eta)\phi(x)dx=E_{P(x\|\eta)}[\phi(x)]$

所以比如说对高斯分布的A'(η)求关于η1的一阶导，就是μ



## Probabilistic Graphical Models 概率图模型

## Expectation-Maximization Algorithm EM算法

## Gaussian Mixture Model 高斯混合模型 

## Variational Inference 变分推断 

## Hidden Markov Model 隐式马尔科夫模型

## Markov Chain Monte Carlo 马氏链蒙特卡洛

## Linear Dynamical Systems 线性动态系统

## Particle Filtering 粒子滤波

## Conditional Random Fields 条件随机场

## Gaussian Networks 高斯网络

## Bayesian Linear Regression 贝叶斯线性回归

## Gaussian Process Regression 高斯过程回归

## Restricted Boltzmann Machine 受限玻尔兹曼机 

## Spectral Clustering 谱聚类

## Feedforward Neural Networks 前馈神经网络

## Straight-Through Estimator 直面配方函数

## Approximate Inference 近似推断

## Sigmoid Belief Network

## Deep Belief Network 深度信念网络

## Boltzmann Machine 玻尔兹曼机

## Deep Boltzmann Machine 深度玻尔兹曼机

## Generative Models 生成模型

## Generative Adversarial Networks 生成对抗网络

## Variational Autoencoder 变分自编码器

 

