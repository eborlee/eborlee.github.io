---
layout:     post
title:      "Notes - QMF and MathModelling"
subtitle:   " \"Let's Change of Measure!\""
date:       2023-04-24 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
mathjax: true
catalog: true
tags:
    - Math
    - Notes
    - Quant
---

# Notes on Courses QMF & MathModelling of CUHK

2023-04-24

Thanks for my professors, Dr. Willem J. Van Vliet, Mr. Keith Law and Mr. Kelvin Lam

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

[TOC]



## 1 Probability

##### 1.1 Introductory Background: Flipping Coins

Flipping Two fair coins. Use H and T to denote heads and tail.

P[HH] = P[HT] = P[TH] = P[TT] = 1/4

P[H on first coin] = P[HH] + P[TT] = 1/2

**[Outcomes]** we do not directly assign probabilities to outcomes but to events. An **event** is a collection of outcomes. It's a way of grouping together outcomes.

For the flipping coins, there are 4 outcomes: HH, HT, TT, TH.

**A prob measure is a function that maps events to [0,1]**

16 possible events and their corresponding probs:

<img src="/Users/admin/Library/Application Support/typora-user-images/image-20230424134815699.png" alt="image-20230424134815699" style="zoom: 33%;" />

P(HH) makes no sense. Right expression is P({HH}).

##### 1.2 How to define probability

Requires 3 parts:

- *Sample Space Ω as the set of all possible outcomes*
- *Event Space as the set of events that we will assign probs to*
- *Prob Measure, which assigns probs to events*

An event is a subset A ⊆ Ω including ∅(nothing happens) and Ω(something happens)

So the event space would be a set of subsets of Ω.

**Sigma-Algebra**

Def: Ω be a set. The set F of subsets of Ω is called a sigma-algebra if it satisfies:

- *Ω  ∈ F*
- *If A ∈ F, then Ac ∈ F*
- *F is closed under countable unions. If A1, A2 ... are in F, then ∪Ai ∈ F.*

Then we have the definition of Event Space:

<img src="/Users/admin/Library/Application Support/typora-user-images/image-20230424140435230.png" alt="image-20230424140435230" style="zoom:33%;" />

Some examples:

<img src="/Users/admin/Library/Application Support/typora-user-images/image-20230424140510038.png" alt="image-20230424140510038" style="zoom:33%;" />

The discrete sigma-algebra is the largest sigma-algebra.

**Measureable Space**
<img src="https://github.com/eborlee/imgs/blob/main/qmf/image-20230424140621977.jpg?raw=true" alt="image-20230424140621977" style="zoom:33%;" />

**Prob Measure:**

<img src="/Users/admin/Library/Application Support/typora-user-images/image-20230424140649431.png" alt="image-20230424140649431" style="zoom:33%;" />

*P*(Ω) is not Ω, but much larger.

<img src="/Users/admin/Library/Application Support/typora-user-images/image-20230424141126188.png" alt="image-20230424141126188" style="zoom: 33%;" />

Now we only modeled "group together". Actually we should also consider conditional prob:

<img src="/Users/admin/Library/Application Support/typora-user-images/image-20230424141630154.png" alt="image-20230424141630154" style="zoom: 33%;" />



## 2 Random Variables


## 10 Derivation of Black-Scholes-Merton Fomula

### Method 1: Long Way


Firstly, we could derive the SDE of the European Call:

$$
\begin{align*}
&\text{Let}~r_{t} = r, \text{ constant}\\
&\Rightarrow dB_{t}=rB_{t}\mathrm{d}_{t}\to \frac{dB_{t}}{B_{t}}=r\mathrm{d}_{t}\\
&\text{Let}~u=\ln B_{t} \to \frac{\mathrm{d}u}{\mathrm{d}B_{t}}=\frac{1}{B_{t}}\to \mathrm{d}u=\frac{\mathrm{d}B_{t}}{B_{t}}\\
&\int_{0}^{t}\frac{\mathrm{d}B_{s}}{B_s} = \int_{0}^{t}r\mathrm{d}_{s}=rt\\
&\text{The left side also equals to: }\int_{\ln B_{0}}^{\ln B_{t}}\mathrm{d}u=\ln B_{t}-\ln B_{0}=\ln\frac{B_{t}}{B_{0}}\\
&\ln\frac{B_{t}}{B_{0}}=rt\\
&B_t=e^{rt}B_{0}\\
&\text{Assume the underlying stock return follows: }\\
&\frac{\mathrm{d}S_t}{S_t}=\mu \mathrm{d}_t+\sigma \mathrm{d}W_t\\
&\text{And the payoff of a European Call is: }C_T=\max(S_T-K,0)=(S_T-K)^+\\
&\text{From Markov,}C_T\text{ only depends on }S_T\to C_t=C(t,S_t)\\
&\text{From Ito Lemma: }\mathrm{d}C=\frac{\partial C}{\partial t}\mathrm{d}_t + \frac{\partial C}{\partial S}\mathrm{d}_S+\frac{1}{2}\frac{\partial^2 C}{\partial S^2}(\mathrm{d}_S)^2\\
&\Rightarrow \mathrm{d}C_t=\left(\frac{\partial C}{\partial t}+\frac{1}{2}\frac{\partial^2 C}{\partial S^2}\sigma^2S_t^2 + \frac{\partial C}{\partial S}\mu S_t\right)\mathrm{d}_t+\frac{\partial C}{\partial S}\sigma S_t\mathrm{d}W_t\\
\end{align*}
$$




Then we discount the Ct to Time 0:
$$
\begin{align*}
&e^{-rt}C_t=e^{-rt}C(t,S_t)\\
&\Rightarrow\\
&\text{Let} ~f(t,C)=e^{-rt}C_t\\
&\mathrm{d}(e^{-rt}C_t)= \mathrm{d}f=-re^{-rt}C_t\mathrm{d}t+e^{-rt}\mathrm{d}C_t\\
&=-re^{-rt}C_t\mathrm{d}t+e^{-rt}\left(\left(\frac{\partial C}{\partial t}+\frac{1}{2}\frac{\partial^2 C}{\partial S^2}\sigma^2S_t^2 + \frac{\partial C}{\partial S}\mu S_t\right)\mathrm{d}t+\frac{\partial C}{\partial S}\sigma S_t\mathrm{d}W_t\right)\\
&=e^{-rt}\left(\frac{\partial C}{\partial t}+\frac{1}{2}\frac{\partial^2 C}{\partial S^2}\sigma^2S_t^2 + \frac{\partial C}{\partial S}\mu S_t-rC\right)\mathrm{d}t+e^{-rt}\frac{\partial C}{\partial S}\sigma S_t\mathrm{d}W_t
\end{align*}
$$

According to self-financing strategy, we want to construct a portfolio to replicate the call option exposing to money market fund and the stock.

$$
\begin{aligned}
&\text{Let}~X_t=\theta_{B,t}B_t+\theta_{S,t}St~\text{denote the of the portfolio, }\theta_{B,t}~\text{and}~\theta_{S,t}~\text{are the exposures} \\
&\mathrm{d}X_t=\theta_{B,t}\mathrm{d}B_t+\theta_{S,t}\mathrm{d}S_t \\
&=\theta_{B,t}rB_t\mathrm{d}t+\theta_{S,t}(\mu S_t\mathrm{d}t+\sigma S_t\mathrm{d}W_t)
\end{aligned}
$$

Then we consider:

$$
\begin{aligned}
f(t,X_t)&=e^{-rt}X_t\\
\Rightarrow\\
\mathrm{d}(e^{-rt}X_t)&=\frac{\partial f}{\partial t}\mathrm{d}t+\frac{\partial f}{\partial X_t}\mathrm{d}X_t\\
&=-re^{-rt}X_t\mathrm{d}t+e^{-rt}\mathrm{d}X_t\\
&=-re^{-rt}(\theta_{B,t}B_t+\theta_{S,t}S_t)\mathrm{d}t+e^{-rt}\mathrm{d}X_t\\
&=-re^{-rt}(\theta_{B,t}B_t+\theta_{S,t}S_t)\mathrm{d}t+e^{-rt}(\theta_{B,t}\mathrm{d}B_t+\theta_{S,t}\mathrm{d}S_t)\\
&=-re^{-rt}(\theta_{B,t}B_t+\theta_{S,t}S_t)\mathrm{d}t+e^{-rt}(\theta_{B,t}\mathrm{d}B_t+\theta_{S,t}(\mu S_t\mathrm{d}t+\sigma S_t\mathrm{d}W_t))\\
\text{Simplify:}\\
&=-re^{-rt}\theta_{S,t}S_t\mathrm{d}t+e^{-rt}\theta_{S,t}(\mu S_t\mathrm{d}t+\sigma S_t\mathrm{d}W_t)\\
&=(\mu -r)e^{-rt}\theta_{S,t}S_t\mathrm{d}t+e^{-rt}\theta_{S,t}\sigma S_t\mathrm{d}W_t
\end{aligned}
$$

Now we combine (2) and (4), we could get the 
<span style="background-color: yellow;">**<u>BSM PDE</u>**</span>:

$$
\begin{align*}
&\text{Since}~C_t=X_t,~\text{we have: }d(e^{-rt}C_t)=d(e^{-rt}X_t)\\
&\begin{cases}
e^{-rt}\left(\frac{\partial C}{\partial t}+\frac{1}{2}\frac{\partial^2 C}{\partial S^2}\sigma^2S_t^2 + \frac{\partial C}{\partial S}\mu S_t-rC\right)\mathrm{d}_t = (\mu -r)e^{-rt}\theta_{S,t}S_t\mathrm{d}_t \\
e^{-rt}\frac{\partial C}{\partial S}\sigma S_t\mathrm{d}W_t = e^{-rt}\theta_{S,t}\sigma S_t\mathrm{d}W_t~\to~\frac{\partial C}{\partial S}=\theta_{S,t}
\end{cases}\\
&\text{Simplify:}\\
&\frac{\partial C}{\partial t}+\frac{1}{2}\sigma^2\frac{\partial ^2C}{\partial S^2}S_t^2+rS_t\frac{\partial C}{\partial S}-rC=0
\end{align*}
$$





### Method 2: Short Way




