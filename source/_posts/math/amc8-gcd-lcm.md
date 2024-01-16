---
title: GCD和LCM的积
date: 2023-12-18 23:46:51
tags: [数学]
toc: false
published: true
---

## 定义
- GCD 最大公约数Createst Common Divisor
- LCM 最小公倍数Least Common Multiple

## 求法
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20231219100422.png)


例如，
$$
24=2*2*2*3=2^3*3
$$
$$
36=2*2*3*3=2^2*3^2
$$
则
$$
GCD(24,36)=2^2*3=12
$$
(质因数取较小次幂得到GCD)
$$
LCM(24,36)=2^3*3^2=72
$$
(质因数取较高次幂得到LCM)

注：[用Markdown输入数学公式的方法](https://zhuanlan.zhihu.com/p/138532124)

上面的规律可知，如果有GCD和LCM的条件，要把GCD和LCM表示为质因数的幂积形式，才好看出隐藏的逻辑关系

例如MasteringAMC8Book里
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20231219151131.png)
要把条件看做

$（1） GCD(a,b)=4=2^2$

$（2）GCD(b,c)=18=2*3^2$

$（3）LCM(a,c)=144=2^4*3^2$

然后就根据1 2 3条件来推测a  b c各自的2和3的幂次了（还需要注意LCM的质因数肯定在a或c的质因数里，所以a和c都不会有2和3以外的质因数）

## 性质
定理 GCD和LCM的积等于两数的积

![定理1](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20231219094830.png)

例题
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20231219095052.png)

gpt解释原理（直观一些）
https://sharegpt.com/c/WOFoy32

原教材解释原理（原数用质因数的幂表示，和GCD和LCM的关系）
![](https://cdn.jsdelivr.net/gh/tobyforever/uploadpic/upload/20231219095352.png)

## 应用

Problem 19.5.1 (AMC 10)
Each piece of candy in a store costs a whole number of cents. Casper has exactly
enough money to buy either 12 pieces of red candy, 14 pieces of green candy, 15 pieces
of blue candy, or n pieces of purple candy. A piece of purple candy costs 20 cents. What
is the smallest possible value of n?

钱数金额等于12r=14g=15b=20n,所以最少的钱数是LCM(12,14,15,20)
求LCM的方法是把abcd表示为质因数幂积形式
$2^2*3$ , $2*7$, $3*5$, $2^2*5$
所以LCM=$2^2*3*5*7$=420元，所以最小的n是420/20=21
