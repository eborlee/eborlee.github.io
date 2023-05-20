---
layout:     post
title:      "Notes - Numerical Methods in Finance with C++"
subtitle:   " \"\""
date:       2023-05-01 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C++
    - Quant
    - Notes
---

> “C++”


# Notes on Numerical Methods in Finance with C++

2023-05-08

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

[TOC]

## 1 Binomial Pricer

#### Ways of passing parameters:

1. Passed By Value: double S(double S0, double U, ...)

   When a function is called, a copy of that variable is made in a separate location in computer memory. The func could see and change the copy, but has no access to the original varible. And the calling program could also see the copy and only has access to the original variable.

2. Passed By Reference: int GetInputData(double& S0, ...)

   C++ can return only a single value in a function and this way enables to pass all the inputs back to the program.

   In this case, a single copy of the variable in computer memory is shared by the function and the calling program.

   

---

#### Separate Compilation

After defining some function protoptypes in .h file: changes to one of the .cpp files do not require the other file to be recompiled as long as the function prototypes remain unchanged. In large projects this can mean considerable savings in compilation time.

## 2 Binomial Pricer Revisited



## 3 American Options

## 4 Non-linear Solvers
**Basic Knowledge:**

BSM Formula:
$$
\begin{align*}
&C(S_0, K, T,\sigma,r)=S_0N(d_+)-Ke^{-rT}N(d_-)\\
&d_+=\frac{ln(\frac{S_0}{K})+(r+\frac{\sigma^2}{2})T}{\sigma\sqrt{T}}\\
&d_-=d_+-\sigma\sqrt{T}\\
&N(x)=\int_{-\infty}^x \frac{1}{\sqrt{2\pi}}e^{-\frac{y^2}{2}}dy
\end{align*}
$$
If the values of S0, K, T, σ,r are all known, then we can calculate the price of option. Among these items, only σ is not known and needed to be estimated using the market option price.

There are two common methods for this purpose: Bisection method and Newton-Raphson method.



**Bisection Method:**

We want to compute a solution x to an equation: f(x) = c, where f is a given function from an interval [a,b] to R. It is assumed that f is continuous on [a,b] and f(a)-c, f(b)-c have opposite signs. Then there must be an x∈[a,b] such that f(x)=c.

The bisection method works by constructing sequences l, r of left and right approximations by induction:

(l+r)/2 -> mid, f(mid)-c

If f(l) * f(mid) > 0, root lies in [mid, r], mid -> l;

If f(l) * f(mid) <0, root lies in [l, mid], mid ->r;

If f(mid) == 0, mid is the root, Stop.

And if \|mid - l \| <=  Accuracy requirement, stop. Input the mid and the corresponding f(mid)-c at that moment.



**Newton-Raphson Method:**

f is assumed to be differentiable on [a,b] and construct a sequence xn as follows:
$$
\begin{align*}
&Take ~x0∈(a,b).\\
&For~ n=0,1,2,..., let\\
&x_{n+1}=x_n-\frac{f(x_n)-c}{f'(x_n)}
\end{align*}
$$
If the equation f(x)=c has a solution x∈(a,b) such that f'(x)≠0 and x0 is chosen close enough to x, then xn will converge to x.


### 4.1 使用Function Pointer的基本实现

```c++
#ifndef Solver01_h
#define Solver01_h

double SolveByBisect(double (*Fct)(double x), double Tgt,
                     double LEnd, double REnd, double acc)
{
    double left = LEnd, right = REnd, mid = (left + right) / 2;
    double y_left = Fct(left) - Tgt, y_mid = Fct(mid) - Tgt;
    while (mid - left > acc)
    {
        if ((y_left > 0 && y_mid > 0) || (y_left < 0 && y_mid < 0))
        {
            left = mid;
            y_left = y_mid;
        }
        else
            right = mid;
        mid = (left + right) / 2;
        y_mid = Fct(mid) - Tgt;
    }
    return mid;
}

// Newton-Raphson Method
double SolveByNR(double (*Fct)(double x), double (*DFct)(double x), double Tgt,
                 double guess, double acc)
{
    double x_prev = guess;
    double x_next = x_prev - (Fct(x_prev) - Tgt) / DFct(x_prev);
    while (x_next - x_prev > acc || x_prev - x_next > acc)
    {
        x_prev = x_next;
        x_next = x_prev - (Fct(x_prev) - Tgt) / DFct(x_prev);
    }
    return x_next;
}

#endif
```

```c++
#include "Solver01.h"
#include <iostream>
using namespace std;

double F1(double x){return x*x-2;}
double DF1(double x){return 2*x;}

int main(){
    double Acc = 0.001;
    double LEnd=0.0, REnd=2.0;
    double Tgt = 0.0;
    cout << "Root of F1 by bisect:"
         << SolveByBisect(F1, Tgt, LEnd, REnd, Acc)
         <<endl;
    double Guess = 1.0;
    cout << "Root of F1 by newton-raphson:"
         << SolveByNR(F1, DF1, Tgt, Guess, Acc)
         <<endl;

    return 0;
}
```

这种方式简单却不易于expansion。如果传入的Function形如f(x)没问题，但是f(x,a) = x^2-a不适用。

### 4.2 **使用Virtual Functions**: Enable dynamic binding

Dynamic Binding :动态绑定是指在运行时根据实际对象的类型来确定要调用的函数实现，而不仅仅根据指针或引用的静态类型。这意味着，当通过基类的指针或引用调用虚函数时，实际调用的是对象的派生类中所重写的虚函数。这种动态绑定机制使得在继承体系中可以实现多态性，允许通过基类的接口来操作具体的派生类对象，而不需要显式指定对象的具体类型。

而如果在签名中声明为类对象本身而不是其指针，就会导致： <mark>虽然可以传入其子类，但是调用方法还是调用的同一个父类方法，而不会调用子类重写的方法。</mark>

```c++
#include <iostream>

class Animal {
public:
    virtual void MakeSound() {
        std::cout << "Animal is making a sound" << std::endl;
    }
};

class Dog : public Animal {
public:
    void MakeSound() override {
        std::cout << "Dog is barking" << std::endl;
    }
};

class Cat : public Animal {
public:
    void MakeSound() override {
        std::cout << "Cat is meowing" << std::endl;
    }
};

void PerformSound(Animal* animal) {
    animal->MakeSound();
}

void PerformSound2(Animal animal) {
    animal.MakeSound();
}

int main() {
    Animal animal;
    Dog dog;
    Cat cat;
    // 当使用基类指针作为参数时，可以实现运行时的多态性，使代码能够根据实际传递的派生类对象来选择相应的函数实现。
    // dynamic binding
    PerformSound(&animal); // Output: "Animal is making a sound"
    PerformSound(&dog);    // Output: "Dog is barking"
    PerformSound(&cat);    // Output: "Cat is meowing"

    // 如果直接传递基类对象本身，编译器在编译时就确定了要调用的函数实现，无法在运行时动态选择不同的函数实现。
    PerformSound2(animal); // Output: "Animal is making a sound"
    PerformSound2(dog);    // Output: "Animal is making a sound"
    PerformSound2(cat);    // Output: "Animal is making a sound"
    return 0;
}

```

继续Solver的例子：

```c++
#ifndef Solver02_h
#define Solver02_h

class Function
{
public:
    virtual double Value(double x) = 0;
    virtual double Deriv(double x) = 0;
};

double SolveByBisect(Function *Fct, double Tgt, double LEnd, double REnd, double Acc)
{
    double left = LEnd, right = REnd, mid = (left + right) / 2;
    double y_left = Fct->Value(left) - Tgt, y_mid = Fct->Value(mid) - Tgt;
    while (mid - left > Acc)
    {
        if ((y_left > 0 && y_mid > 0) || (y_left < 0 && y_mid < 0))
        {
            left = mid;
            y_left = y_mid;
        }
        else
            right = mid;
        mid = (left + right) / 2;
        y_mid = Fct->Value(mid) - Tgt;
    }
    return mid;
}

double SolveByNR(Function *Fct, double Tgt, double guess, double acc)
{
    double x_prev = guess;
    double x_next = x_prev - (Fct->Value(x_prev) - Tgt) / Fct->Deriv(x_prev);
    while (x_next - x_prev > acc || x_prev - x_next > acc)
    {
        x_prev = x_next;
        x_next = x_prev - (Fct->Value(x_prev) - Tgt) / Fct->Deriv(x_prev);
    }
    return x_next;
}

#endif
```

```c++
#include "Solver02.h"
#include <iostream>
using namespace std;

class F1 : public Function
{
public:
     double Value(double x) { return x * x - 2; }
     double Deriv(double x) { return 2 * x; }
} MyF1;

class F2 : public Function
{
private:
     double a;

public:
     F2(double a_)
     {
          a = a_;
     }
     double Value(double x) { return x * x - a; }
     double Deriv(double x) { return 2 * x; }
} MyF2(3.0);

int main()
{
     double Acc = 0.001;
     double LEnd = 0.0, REnd = 2.0;
     double Tgt = 0.0;
     cout << "Root of F1 by bisect:"
          << SolveByBisect(&MyF1, Tgt, LEnd, REnd, Acc)
          << endl;
     double Guess = 1.0;
     cout << "Root of F2 by newton-raphson:"
          << SolveByNR(&MyF2, Tgt, Guess, Acc)
          << endl;

     return 0;
}
```

使用虚函数在运行时进行动态绑定是有一些开销的，这是因为动态绑定需要在运行时查找正确的函数实现。这些开销包括：

1. 虚函数表（vtable）：为了实现动态绑定，编译器会为每个包含虚函数的类生成一个虚函数表。虚函数表是一个指针数组，用于存储每个虚函数的地址。对于每个类对象，都会包含一个指向其虚函数表的指针。这样，当调用虚函数时，需要通过指针查找正确的函数地址。
2. 虚函数调用开销：调用虚函数时，需要通过指针间接调用函数。这涉及额外的间接跳转操作，相对于直接调用非虚函数，会增加一定的开销。

总体来说可以忽略不计，但是以下情景需要注意：

1. <u>循环中使用虚函数会造成大量开销</u>
2. 使用虚函数的地方可以进行性能优化，例如使用内联（inline）修饰符来尝试避免间接跳转的开销。
3. 考虑使用其他的技术，如模板和策略模式，来避免使用虚函数带来的开销。

### 4.3 使用Templates模板

**极大的保留了虚函数的优点，但是将type checking从runtime转移到了compile time.**

代码与虚函数大体相似，但简化了：不需要定义抽象基类和其虚函数，具体的公式类也不需要再继承共同的父类。

缺点：如果有大量的函数将会带来较长的编译时间和较大的.exe文件。

```c++
#ifndef Solver03_h
#define Solver03_h

template <typename Function>
double SolveByBisect(Function *Fct, double Tgt, double LEnd, double REnd, double Acc)
{
    double left = LEnd, right = REnd, mid = (left + right) / 2;
    double y_left = Fct->Value(left) - Tgt, y_mid = Fct->Value(mid) - Tgt;
    while (mid - left > Acc)
    {
        if ((y_left > 0 && y_mid > 0) || (y_left < 0 && y_mid < 0))
        {
            left = mid;
            y_left = y_mid;
        }
        else
            right = mid;
        mid = (left + right) / 2;
        y_mid = Fct->Value(mid) - Tgt;
    }
    return mid;
}

template <typename Function>
double SolveByNR(Function *Fct, double Tgt, double guess, double acc)
{
    double x_prev = guess;
    double x_next = x_prev - (Fct->Value(x_prev) - Tgt) / Fct->Deriv(x_prev);
    while (x_next - x_prev > acc || x_prev - x_next > acc)
    {
        x_prev = x_next;
        x_next = x_prev - (Fct->Value(x_prev) - Tgt) / Fct->Deriv(x_prev);
    }
    return x_next;
}

#endif
```

```c++
#include "Solver03.h"
#include <iostream>
using namespace std;

class F1
{
public:
     double Value(double x) { return x * x - 2; }
     double Deriv(double x) { return 2 * x; }
} MyF1;

class F2
{
private:
     double a;

public:
     F2(double a_)
     {
          a = a_;
     }
     double Value(double x) { return x * x - a; }
     double Deriv(double x) { return 2 * x; }
} MyF2(3.0);

int main()
{
     double Acc = 0.001;
     double LEnd = 0.0, REnd = 2.0;
     double Tgt = 0.0;
     cout << "Root of F1 by bisect:"
          << SolveByBisect(&MyF1, Tgt, LEnd, REnd, Acc)
          << endl;
     double Guess = 1.0;
     cout << "Root of F2 by newton-raphson:"
          << SolveByNR(&MyF2, Tgt, Guess, Acc)
          << endl;

     return 0;
}
```

同时，由于不再使用虚函数，SolveByBisect和SolveByNR的声明也不需要再使用Fct指针，使用Fct就可以了。

### 4.4 Computing Implied Volatility

Define EurCall.h for computing the call option price using BSM using solver03.h, i.e. Templates version.
Define EurCall.h for computing the call option price using BSM using solver03.h, i.e. Templates version.

To compute it using Newton-Raphson method, we also need an expression for the derivative of the European call option price with respect to volatility σ calculated from BSM formula, **Vega**, is given by:
$$
\begin{align*}
&v=\frac{1}{\sqrt{2\pi}}S_0e^{\frac{-d_+^2}{2}}
\end{align*}
$$

```c++
#ifndef EurCall_h
#define EurCall_h

class EurCall
{
public:
    double T, K;
    EurCall(double T_, double K_) { T = T_, K = K_; }

    double d_plus(double S0, double sigma, double r);
    double d_minus(double S0, double sigma, double r);
    double PriceByBSFormula(double S0, double sigma, double r);

    double VegaByBSFormula(double S0, double sigma, double r);
};

#endif
```

Implement the methods in EurCall.cpp:

```c++
#include "EurCall.h"
#include <cmath>
#include <iostream>
using namespace std;
double N(double x)
{
    double result;

    if (x > 0)
    {
        result = 0.5 * (1.0 + erf(x / std::sqrt(2.0)));
    }
    else if (x == 0)
    {
        result = 0.5;
    }
    else
    {
        result = 0.5 * (1.0 - erf(-x / std::sqrt(2.0)));
    }

    return result;
}

double EurCall::d_plus(double S0, double sigma, double r)
{
    return (log(S0 / K) + (r + 0.5 * pow(sigma, 2.0)) * T) / (sigma * sqrt(T));
}

double EurCall::d_minus(double S0, double sigma, double r)
{
    return d_plus(S0, sigma, r) - sigma * sqrt(T);
}

double EurCall::PriceByBSFormula(double S0, double sigma, double r)
{
    return S0 * N(d_plus(S0, sigma, r)) - K * exp(-r * T) * N(d_minus(S0, sigma, r));
}

double EurCall::VegaByBSFormula(double S0, double sigma, double r)
{
    double pi = 4.0 * atan(1.0);
    return S0 * exp(-d_minus(S0, sigma, r) * d_plus(S0, sigma, r) / 2) * sqrt(T) / sqrt(2.0 * pi);
}
```

Run the functions in main_for_cal_imp.cpp. Here create a class as a intermediary class to translate the functions from EurCall to Solvers.

```c++
#include "Solver03.h"
#include "EurCall.h"
#include "EurCall.cpp"
#include <iostream>
using namespace std;

// 需要一个中间类进行初始化和翻译函数名
class Intermediary : public EurCall
{
private:
    double S0, r;

public:
    Intermediary(double S0_, double r_, double T_, double K_) : EurCall(T_, K_)
    {
        S0 = S0_;
        r = r_;
    }

    double Value(double sigma)
    {
        return PriceByBSFormula(S0, sigma, r);
    }

    double Deriv(double sigma)
    {
        return VegaByBSFormula(S0, sigma, r);
    }
};

int main()
{
    double S0 = 100.0;
    double r = 0.1;
    double T = 1.0;
    double K = 100.0;
    Intermediary Call(S0, r, T, K);

    double Acc = 0.001;
    double LEnd = 0.01, REnd = 1.0;
    double Tgt = 12.56;

    cout << "Root of F1 by bisect:"
         << SolveByBisect(&Call, Tgt, LEnd, REnd, Acc)
         << endl;
    double Guess = 0.23;
    cout << "Root of F2 by newton-raphson:"
         << SolveByNR(&Call, Tgt, Guess, Acc)
         << endl;

    return 0;
}
```



## 5 Monte Carlo methods

### LU Decomposition



```c++
#include <iostream>
#include <Eigen/Dense>
#include <Eigen/LU>
using namespace std;
using namespace Eigen;

int main(){
    typedef Matrix<double, 4,4> Matrix4x4;

    Matrix4x4 p;

    p<< 7,3,-1,2,
        3,8,1,-4,
        -1,1,4,-1,
        2,-4,-1,6;

    cout<< "Mat_P:\n"<<p<<endl<<endl;


    PartialPivLU<Matrix4x4> lu(p); // 此处在栈区创建了一个PPL对象，其构造函数自动对传入矩阵进行LU分解

    cout<< "Lu_Mat:\n"<<lu.matrixLU()<< endl<<endl;

    // 由于lu分解的下三角矩阵的主对角线都是1，所以需要先初始化一个单位矩阵
    // 使用到了静态函数
    Matrix4x4 l = MatrixXd::Identity(4,4);
    // 先获取一个从0，0开始的4*4的子矩阵再限定其视图为下三角，接受赋值
    l.block<4,4>(0,0).triangularView<StrictlyLower>() = lu.matrixLU();
    cout<<"L_mat:\n"<<l<<endl<<endl;

    // 由于上三角矩阵的主对角线是0，所以直接接受赋值
    Matrix4x4 u = lu.matrixLU().triangularView<Upper>();
    cout<<"u_mat:\n"<<u<<endl<<endl;
```

