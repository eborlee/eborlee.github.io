---
layout:     post
title:      "Notes - Quant Job Interview Q & A | 量化红宝书"
subtitle:   " \"Every little bit helps\""
date:       2023-04-27 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
mathjax: true
catalog: true
tags:
    - Notes
    - Math
    - Quant
---

# Notes on Quant Job Interview Questions and Answers

2023-06-01

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

* TOC
{:toc}

---

## 2. Option Pricing

**<mark><u>2.1 Derive BS equation for a stock S. What boundary conditions are satisfied at S=0 and S=∞？</u></mark>**

***See the Long Way derivation of PDE on the other note: QMF.*** 

BS equation:
$$
\begin{aligned}
\frac{\partial C}{\partial t}+\frac{\partial C}{\partial S}rS_t+\frac{1}{2}\frac{\partial ^2C}{\partial S^2}\sigma^2S_t^2-rC=0
\end{aligned}
$$
About the boundary conditions:

(1) S=0

In intuition, for:
$$
\begin{aligned}
S_T=S_t~exp\{(r-\frac{1}{2}\sigma^2)(T-t)+\sigma(W_T-W_t)\}
\end{aligned}
$$
If `St=0`, then so is `ST`. Thus the call option would be worthless and we could have the boundary condition `C(t,0) = 0, t∈[0,T]`.

As a more concrete approach, substitute St=0 into BS equation above:
$$
\begin{aligned}
\frac{\partial C}{\partial t}(t,0)=rC(t,0)\\
C(t,0)=e^{rt}C(0,0)
\end{aligned}
$$
Since `C(T,0) = max{0-K,0} = 0` ---> `C(0,0)=0` ---> `implies C(t,0) = 0 for all t`.

(2) S=∞

For a very large St, the option is almost certain to finish in-the-money. Thus for every dollar the stock price rises, we are almost certain to receive a dollar at payoff written as:
$$
\begin{aligned}
\lim _{S\to\infty}\frac{\partial C}{\partial S_t}(t,S_t)=1
\end{aligned}
$$
And as the option gets deeper and deeper ITM, the optionality gets worth less and less so the boundary condition is that `C=St-K` instead of `MAX(St-K)` anymore.


**Summary: Lower bound -> 0, Upper bound -> St-K**


<hr>

**<mark><u>2.2 Derive the BS equation so that an undergrad can understand it.</u></mark>**

Essential point is that we evolve such a stochastic differential equation of **`arbitrage-free`** stock price in BS world:
$$
\begin{aligned}
dS_t=rS_tdt+\sigma S_tdW_t
\end{aligned}
$$
At the same time, we need  one other asset, the risk-free bank account that grows at the continuously compounding rate `r`. Hence its value `Bt` is given by:
$$
\begin{aligned}
B_t=e^{rt}\to dB_t=rB_tdt
\end{aligned}
$$
Then apply Ito's formula to get `SDE` of `f(t, St)`:
$$
\begin{aligned}
df(t,S_t)=\frac{\partial f}{\partial t}(t,S_t)dt+\frac{\partial f}{\partial S}(t,S_t)dS_t+\frac{1}{2}\frac{\partial ^2f}{\partial S^2}(t,S_t)(dS_t)^2
\end{aligned}
$$
Evaluating this requires the relations:
$$
\begin{aligned}
(dt)^2=(dW_t)(dt)=0,~~~ (dW_t)^2=dt
\end{aligned}
$$
And then we could denote the price of a derivative to be a function of the current time t and the current stock price St,  i.e. `C(t, St)`.

Finally we need that:
$$
\begin{aligned}
C(t,S_t)B_t^{-1} ~~is~~a~~martingale
\end{aligned}
$$
Simple explanation for the statement above: We expect it to have zero growth. Our option price is expected to grow a the same rate as the bank account and hence the growth of each cancels out in the given process. There will be changes in the discounted price and we expect it to be zero on average. So the discounted price should have a zero drift term.

Then we apply Ito formula to calculate the drift, equate to zero and get:
$$
\begin{aligned}
\frac{\partial C}{\partial t}+\frac{\partial C}{\partial S}rS_t+\frac{1}{2}\frac{\partial ^2C}{\partial S^2}\sigma^2S_t^2-rC=0
\end{aligned}
$$

<hr>

**<mark><u>2.3 Explain the BS equation.</u></mark>**

This is a PDE describing the evolution of the  option price as a function of the current stock price and the current time. **The equation does not change if we vary the payoff function of the derivative.** **However the associated boundary conditions, which are required to solve the equation either closed form or by simulation, do vary.**

And the underlying assumptions are important: 

(1) The `SDE` of stock price is under the `risk-neutral measure`;

(2) The existence of a risk-free asset which grows at the continuously compounding rate `r`.


<hr>


**<mark><u>2.4 Suppose two assets in a BS world have the same volatility but different drifts. How will the price of call options on them compare? Now suppose one of the assets undergoes downward jumps at random times. How will this affect option prices?</u></mark>**




<hr>


**<mark><u>2.5 Suppose an asset has a deterministic time dependent volatility. How would I price an option on it  using the BS theory? How would I hedge it?</u></mark>**
