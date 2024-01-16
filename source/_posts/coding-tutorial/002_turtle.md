---
title: 从头学编程002 小海龟
date: 2023-12-14 21:46:51
tags: [coding]
toc: false
published: true
---

Today we're gonna learn to draw some shapes using python.

demo1  function
```python
import turtle as t


# https://docs.python.org/zh-cn/3/library/turtle.html

# 前进100
t.forward(100)
# 左转120度
t.left(120)
# 前进100
t.forward(100)
t.left(120)
t.forward(100)

t.done()
```




demo2 variable

```python
import turtle as t

x=100
y=200
z="hello"
t.color('red')
t.forward(x)
t.color('green')
t.right(90)
t.forward(y)
t.color('blue')
t.right(90)
t.forward(x*2)
t.color('red')
t.right(90)
y=x+200
t.forward(y)
t.write(z)
t.done()
```


demo3 loop

```python
import turtle as t

for steps in range(100):
    for c in ('blue', 'red', 'green'):
        t.color(c)
        t.forward(steps)
        t.right(30)

# 相当于t.getScreen().mainloop()
t.done()
```



demo4 kochline

```python
import turtle as t

# https://blog.csdn.net/hz_zhangrl/article/details/130983644

koch_ang = (0,60,-120,60)
t.setup(670,590)
t.speed(0)
t.ht()

def kochline(_len,_n):
    if _n == 0:
        t.forward(_len)
    else:
        for angle in koch_ang:
            t.lt(angle)
            kochline(_len/3, _n-1)

if __name__ == '__main__':
    t.up()
    t.goto(-270,155)
    t.pendown()
    t.pensize(1)
    level=4
    kochline(540,level)
    t.rt(120)
    kochline(540,level)
    t.rt(120)
    kochline(540,level)
```

