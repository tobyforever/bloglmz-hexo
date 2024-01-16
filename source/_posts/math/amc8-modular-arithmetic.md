---
title: 模运算
date: 2023-12-18 23:47:51
tags: [数学]
toc: false
published: true
---

## 定义

$$
a \equiv b \mod n
$$

这里 $\equiv$ 叫做同余（a is congruence to b mod n)

(注：markdown里数学公式写法是LaTex语法 [参考](https://shareg.pt/PRY92qc))

## 求法

## 性质
模运算满足加法和乘法

模(Mod)在一些场合(如计算机编程)，可以使用符号%表示，它是一个二元运算。x mod y的值都介于0和模之间：


其中y=0,为了避免用零做除数，为了完整起见，我们定义 x mod 0 = x.



模（Mod）运算的规则、公式与基本四则运算有些类似，但是除法例外。其规则如下：

(a+b)%p=(a%p+b%p)%p

(a-b)%p=(a%p-b%p)%p

(a*b)%p=(a%p * b%p)%p

由第1个公式可以推导出 
模运算满足结合律、交换律、分配率，具体如下：

A. 结合律

((a+b)%p+c)%p=(a+(b+c)%p)%p

((a*b)%p * c)%p= (a * (b*c)%p)%p

B. 交换律

(a+b)%p=(b+a)%p

(a*b)%p=(b*a)%p

C. 分配率

(a+b)%p=(a%p+b%p)%p

((a+b)%p*c)%p = ( (a*c)%p + (b*c)%p )%p

## 应用


![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20231219174141.png)
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20231219174044.png)

```
提示：求个位可以看做是mod10，然后利用mod运算的加法性质
```

解法2：
观察原式=2+2x4+2x4x6...
个位就是2+4+6+...+2022的个位。
等差数列求出和为1012X1011,所以个位为2


