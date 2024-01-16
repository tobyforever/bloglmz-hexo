---
title: 从头学编程003 python
date: 2023-12-14 21:46:51
tags: [coding]
toc: false
published: true
---

Today we're gonna learn basic concepts with python.

demo1  [简单的变量simple variable](https://pythontutor.com/render.html#code=%0Ax%3D100%0Ay%3D200%0Az%3D%22hello%22%0Ay%3Dx%2B200%0Ax%3Dx%2B1%0Ay%3Dy-1%0Ay%3Dx*2%0Ay%3Dx*2%0Ay%3Dy&cumulative=false&curInstr=9&heapPrimitives=nevernest&mode=display&origin=opt-frontend.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false)

```python
x=100
y=200
z="hello"
y=x+200
x=x+1
y=y-1
y=x*2
y=x*2
y=y
```


demo2 [函数function、变量作用域variable scope](https://pythontutor.com/render.html#code=a%3D100%0Ab%3D200%0A%0Adef%20funny%28x,y%29%3A%0A%20%20%20%20magic_num%3D2%0A%20%20%20%20z%3D%28x%2By%29*magic_num%0A%20%20%20%20return%20z%0A%0Ac%3Dfunny%28a,b%29%0A%0Ac%3Dmagic_num&cumulative=false&curInstr=0&heapPrimitives=nevernest&mode=display&origin=opt-frontend.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false)

概念concepts:
- 1、函数function。函数参数，函数返回值，形参和实参
- 2、变量作用域scope、全局变量、内部变量

```python
a=100
b=200

def funny(x,y):
    magic_num=2
    z=(x+y)*magic_num
    return z

def funny2(x,y):
    x=x*2
    y=y*3
    #外部变量会被修改吗？
    a=999
    b=888

c=funny(a,b)

funny2(a,b)

a=a+1
b=b+2

# 这里c会变成2吗？
c=magic_num
```


demo3 [递归recursion](https://pythontutor.com/render.html#code=def%20fibonacci%28n%29%3A%0A%20%20%20%20if%20n%3E%3D3%3A%0A%20%20%20%20%20%20%20%20return%20fibonacci%28n-1%29%2Bfibonacci%28n-2%29%0A%20%20%20%20else%3A%0A%20%20%20%20%20%20%20%20return%201%0A%0Aa%3Dfibonacci%284%29&cumulative=false&curInstr=0&heapPrimitives=nevernest&mode=display&origin=opt-frontend.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false)

概念concepts:
- 1、条件判断if-else
- 2、递归
- 3、理解求值过程，中间过程暂存在stack里（观察global frame）。

```python

def fibonacci(n):
    if n>=3:
        return fibonacci(n-1)+fibonacci(n-2)
    else:
        return 1

# 如果改为fibonacci(10)甚至更大会怎么样？
a=fibonacci(4)
```

