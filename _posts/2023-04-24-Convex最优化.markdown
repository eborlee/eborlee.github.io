---
layout:     post
title:      "Notes - Convex Optimization | 凸优化"
subtitle:   " \"Every little bit helps\""
date:       2023-09-01 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
mathjax: true
catalog: true
tags:
    - Notes
    - Math
    - Quant
---

# Convex Optimization

2023-06-01

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

* TOC
{:toc}

---

导论

Boyd书：Chapter 2 3 4 5 9 10

## 仿射/凸/凸锥/集/组合/包


$$
\begin{align*}
&minimize~f_o(x)\\
&s.t. subject~to~f_i(x)\le b_i, i=1,2,...,m\\
&x=[x_1~...~x_n]^T
\end{align*}
$$

优化问题的难易不在于其线性还是非线性，而在于其凸还是非凸。简单来说，凸规划问题较为容易解决。

凸规划/凸优化：目标函数是凸的，约束是凸集。

凸集 Convex Set：

凸函数：



**仿射集 Affine sets：**

对于直线来说，如果有点x1，x2，x1≠x2，均属于n维空间。定义θ∈R，则有方程

y=θx1+（1-θ）x2=x2+θ（x1-x2）

而对于线段：

θ∈[0,1]的R

y=θx1+（1-θ）x2=x2+θ（x1-x2）

<u>仿射集：一个集合c是仿射集，若任意x1，x2∈c，则连接x1和x2的直线也在集合内</u>

<u>直线是仿射集，而线段不是仿射集（因为线段上选的点连成的直线超出线段了）</u>

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008205402269.png?raw=true" alt="image-20231008205402269" style="zoom: 33%;" />

正方形区域就不是一个仿射集。

更一般来说：设，x1...Xk∈c，θ1...θk∈R，θ1+....+θk=1

仿射组合： θ1x1+...+θkXk，如果c是一个仿射集，则该仿射组合也在该集合内

如何证明：
$$
\begin{align*}
&有仿射集c~x_1,x_2,x_3\in C\\
&\theta1,\theta2,\theta3 \in R, \theta_1+\theta_2+\theta_3=1\\
&\frac{\theta_1}{\theta_1+\theta_2}X_1+\frac{\theta_2}{\theta_1+\theta_2}X_2必然\in C\\
&那么，(\theta_1+\theta_2)(\frac{\theta_1}{\theta_1+\theta_2}X_1+\frac{\theta_2}{\theta_1+\theta_2}X_2)+(1-\theta_1-\theta_2)X_3也必然\in C\\
&简化上式有 \theta_1X_1+\theta_2X_2+\theta_3X_3 \in C,得证
\end{align*}
$$
仿射集的性质：

若C是仿射集，对于$\alpha X_1+\beta X2 \stackrel{?}{\in} C, \alpha,\beta \in R$, 当然就不一定了，因为没有了和=1的约束。但是是否有特殊的一些情况使得对这种一般的情形也满足？

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008211018942.png?raw=true" alt="image-20231008211018942" style="zoom:33%;" />

平移直线， 当该直线经过原点时，不论何种α与β，仿射组合都会在该直线上
若集合C是仿射集，则构造 $V=C-X_o=\{X-X_0~|~X\in C\}_{\forall X_0\in C}$ ,称 V是C相关的子空间。

V也是仿射集，并有很好的性质：（此处证明感觉有点问题）
$$
\begin{align*}
&\forall V_1,V_2 \in V,~\forall \alpha,\beta\in R,~\stackrel{\to}{?}\alpha V1+\beta V2\in V?\\
&从\alpha V1+ \beta V_2 +X_0 ~是否\in C入手\\
&构造\alpha(V_1+X_0) + \beta(V_2+X_0)+(1-\alpha-\beta)X_0\\
&可知V_1+X_0,V_2+X_0,X_0都\in C,并且系数之和为1，所以\in C\\
&所以\alpha V1+\beta V2\in V
\end{align*}
$$
该子空间一定经过原点。（这也是成为空间的必要条件）

例：任意一个线性方程组的解集都是仿射集
$$
\begin{align*}
&C=\{x~|~Ax=b\},~A\in R^{m\times n},~ b\in R^m,~x\in R^n\\
&\forall x1,x2\in C, Ax_1=b, Ax_2=b\\
&那么，是否对于\theta\in R,~~有\theta x_1+(1-\theta)x_2\in C?\\
&等价于，是否 A(\theta x_1+(1-\theta)x_2)=b?\\
&上式=\theta Ax_1+(1-\theta)Ax_2=b
\end{align*}
$$
任何一个仿射集，都可以构造出来一个与它相关的子空间，那么上述这个例子的子空间是什么？
$$
\begin{align*}
&V=\{x-x_0~|~x\in C\},~\forall x_0\in C\\
&=\{x-x_0~|~Ax=b\},~Ax_0=b~(x_0不是变量)\\
&\{x-x_0~|~A(x-x_0)=0\}\\
&\{y~|~Ay=0\},该集合是一个A的化零空间\\
&对比C=\{x~|~Ax=b\}\\
\end{align*}
$$
也就是说，高维空间的平移，至少有一个点经过原点，那么新的集合也是一个仿射集，并且是与C相关的子空间。

> 化零空间（Null Space）是线性代数中的一个概念，也被称为零空间或核（Kernel）。对于一个给定的矩阵A*，其化零空间是由所有使得Ax=0成立的向量x组成的集合。在这个表达式中，0是一个零向量，其维度与矩阵A的列数相同。化零空间提供了一个矩阵的线性性质的重要信息，它是线性映射核心概念的一部分，并在解线性方程组、线性代数和许多应用领域中都有重要作用。

反过来说，任何一个仿射集也可以写成一个线性方程组的解集。
任意集合C，如何构造尽可能小的仿射集？在某集合之上构建的最小的仿射集
引入仿射包，仿射包依然是一个集合，记作 $ aff~~C=\{\theta_1 x_1+...+\theta_kx_k~|~\forall x_1...x_k\in C,~\forall \theta_1+...\theta_k=1\}$

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008225614529.png?raw=true" alt="image-20231008225614529" style="zoom:33%;" />

如图，有二维平面，有集合{x1,x2}，显然不是仿射集。但是这条直线是该集合的仿射包。而仿射包一定是个仿射集。

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008225817264.png?raw=true" alt="image-20231008225817264" style="zoom:33%;" />

当该集合有三个点的时候，则该集合的仿射包拓展到了整个平面。



**凸集：**

一个集合C是凸集，当任意两点之间的线段仍然在C内。

$C为凸集 \equiv \forall x_1,x_2\in C,\forall\theta,\theta \in [0,1]$

$\theta x_1+(1-\theta)x_2\in C$

可见，仿射集一定是个凸集。仿射集是凸集的特例。

有 $x_1,...,x_k$的凸组合：$\theta_1 x_1+\theta_2 x_2+...+\theta_k x_k$ 当：

1）$\theta_1,...,\theta_k \in R$

2）$\theta_1+...+\theta_k=1$

3）新条件： $\theta_1,...,\theta_k \in [0,1]$

更一般化，c为凸集 等价于，任意元素的凸组合也在C内。



凸包：考虑任何n维空间的集合C，满足上述三个条件的任意的组合构成的集合。是在该集合基础上最小的凸集。
<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008231526955.png?raw=trueng" alt="image-20231008231550566" style="zoom: 50%;" />


这些图形是否为凸集：是，不是，不是

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008231550566.png?raw=true" alt="image-20231008231958189" style="zoom: 50%;" />
是

它们的凸包：
<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008231903506.png?raw=true" style="zoom: 50%;" />

对于离散的集合，尽管很难是凸集，但是可以构造出来最小的凸集：凸包
<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008231958189.png?raw=true" style="zoom: 50%;" />




**锥：Cone  凸锥 Convex Cone**

C是锥，等价于对任意x属于C，θ≥0，有θx∈C

C是凸锥，等价于对任意x1，x2属于C，θ1，θ2≥0，有x1θ1+x2θ2∈C

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008232322650.png?raw=true" alt="image-20231008232322650" style="zoom:50%;" />

从原点出发三条射线。三条射线所构成的集合是一个锥。

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008232510659.png?raw=true" alt="image-20231008232510659" style="zoom: 33%;" />

Y字形不是一个锥，右图是一个锥，也是一个凸锥，是一个凸集。



凸锥组合：
$\theta_1 x_1+...+\theta_k x_k,~~\theta_1...\theta_k\ge0$

凸锥包： 
$ x1,...,x_k \in C,~~\{\theta_1 x_1+...+\theta_k~|~x_1...x_k\in C, ~\theta_1...\theta_k\ge0\} $

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008233109026.png?raw=true" alt="image-20231008231958189" style="zoom: 50%;" />
两种情况的凸锥包

<img src="https://github.com/eborlee/eborlee.github.io/blob/main/img/convex/image-20231008233242114.png?raw=true" alt="image-20231008233242114" style="zoom:50%;" />

过原点的直线是凸锥，但不是最小的凸锥包



**Summary and Comparison:**
$$
\begin{align*}
&仿射组合~~~~~\forall \theta_1...\theta_k,~~~~~\theta1+...+\theta_k=1\\
&凸组合~~~~~\forall \theta_1...\theta_k,~~~~~\theta1+...+\theta_k=1\\
&~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\theta_1...\theta_k\in[0,1]\\
&凸锥组合~~~~~\forall \theta_1...\theta_k,~~~~~\theta1,...,\theta_k\ge0\\
\end{align*}
$$
凸组合是仿射组合的特例。



如果C={x}，仿射组合是θ1x+θ2x=x，仍然在集合内，所以是一个仿射集。也一定是一个凸集。如果这个点是原点，是凸锥，如不在原点，则不是凸锥。



空集是仿射集，是凸集，也是凸锥



## 几种重要的凸集

## 凸集的交集 保凸运算

