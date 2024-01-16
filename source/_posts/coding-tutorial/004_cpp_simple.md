---
title: 从头学编程004 c++
date: 2023-12-14 22:46:51
tags: [coding]
toc: false
published: true
---

Today we're gonna learn the concepts again, in c++.

demo1  [变量类型variable types](https://pythontutor.com/render.html#code=%23include%20%3Cbits/stdc%2B%2B.h%3E%0Ausing%20namespace%20std%3B%0A%0Aint%20g%3D1%3B%0Along%20lg%3D1%3B%0Afloat%20f1%3D1.0%3B%0Adouble%20d1%3D1.0%3B%0A%0Aint%20main%28%29%7B%0A%0Aint%20a%20%3D1%3B%0Afloat%20b%3D1.0%3B%0Aa%2B%2B%3B%0Ab%2B%2B%3B%0Areturn%200%3B%20%20%0A%7D&cppShowBinary=true&cppShowMemAddrs=true&cumulative=false&curInstr=0&heapPrimitives=nevernest&mode=display&origin=opt-frontend.js&py=cpp_g%2B%2B9.3.0&rawInputLstJSON=%5B%5D&textReferences=false)

知识点：
- 1、编译语言/脚本语言  汇编语言
- 2、变量作用域，全局变量，内存地址，内存分区（stack、heap）
- 3、各种数据类型的内存布局（byte-level view of data），int , long, char, float, double  [IEEE754浮点数表示法](https://zhuanlan.zhihu.com/p/343033661)

[浮点数内存可视化](https://www.h-schmidt.net/FloatConverter/IEEE754.html)
  
```c++
#include <bits/stdc++.h>
using namespace std;

int g=1;
long lg=1;
float f1=1.0;
double d1=1.0;
static int s=1;

int main(){
    int a =1;
    float b=1.0;
    a++;
    b++;
    return 0;  
}
```



demo2  [函数和指针](https://pythontutor.com/render.html#code=%23include%20%3Ciostream%3E%0Ausing%20namespace%20std%3B%0A%0Avoid%20swap1%28int%20a,%20int%20b%29%7B%0A%20%20%20%20int%20t%3Da%3B%0A%20%20%20%20a%3Db%3B%0A%20%20%20%20b%3Dt%3B%0A%7D%0A%0Avoid%20swap2%28int%20%26a,%20int%20%26b%29%7B%0A%20%20%20%20int%20t%3Da%3B%0A%20%20%20%20a%3Db%3B%0A%20%20%20%20b%3Dt%3B%0A%7D%0A%0Aint%20main%28%29%0A%7B%0A%20%20%20%20int%20x%3B%20%0A%20%20%20%20int%20y%3B%0A%20%20%20%20//%20cin%3E%3Ex%3E%3Ey%3B%0A%20%20%20%20//%20cout%3C%3C%22x,y%3D%22%3C%3Cx%3C%3C%22,%22%3C%3Cy%3C%3Cendl%3B%0A%20%20%20%20x%3D10%3B%0A%20%20%20%20y%3D20%3B%0A%20%20%20%20swap1%28x,y%29%3B//%E5%9B%A0%E4%B8%BA%E5%8F%82%E6%95%B0%E6%98%AF%E6%8C%89%E5%80%BC%E4%BC%A0%E9%80%92%EF%BC%88%E6%8B%B7%E8%B4%9D%EF%BC%89%EF%BC%8C%E6%89%80%E4%BB%A5%E5%A4%96%E9%9D%A2%E7%9A%84xy%E4%B8%8D%E4%BC%9A%E5%8F%98%EF%BC%8C%E6%B2%A1%E8%BE%BE%E5%88%B0%E4%BA%A4%E6%8D%A2%E7%9A%84%E7%9B%AE%E7%9A%84%0A%20%20%20%20cout%3C%3C%22x,y%20swapp1%3A%22%3C%3Cx%3C%3C%22,%22%3C%3Cy%3C%3Cendl%3B%0A%20%20%20%20swap2%28x,y%29%3B//%E4%BD%BF%E7%94%A8%26%E5%90%8E%EF%BC%8C%E5%8F%82%E6%95%B0%E6%98%AF%E6%8C%89%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92%EF%BC%8C%E6%89%80%E4%BB%A5%E5%A4%96%E9%9D%A2%E7%9A%84xy%E4%BC%9A%E6%94%B9%E5%8F%98%0A%20%20%20%20cout%3C%3C%22x,y%20swapp2%3A%22%3C%3Cx%3C%3C%22,%22%3C%3Cy%3C%3Cendl%3B%0A%20%20%20%20return%200%3B%0A%7D&cumulative=false&curInstr=19&heapPrimitives=nevernest&mode=display&origin=opt-frontend.js&py=cpp_g%2B%2B9.3.0&rawInputLstJSON=%5B%5D&textReferences=false)

知识点：
- 1、函数，形参和实参
- 2、指针
- 3、交换的正确写法
  
```c++
#include <iostream>
using namespace std;

void swap1(int a, int b){
    int t=a;
    a=b;
    b=t;
}

void swap2(int &a, int &b){
    int t=a;
    a=b;
    b=t;
}

int main()
{
    int x; 
    int y;
    // cin>>x>>y;
    // cout<<"x,y="<<x<<","<<y<<endl;
    x=10;
    y=20;
    swap1(x,y);//因为参数是按值传递（拷贝），所以外面的xy不会变，没达到交换的目的
    cout<<"x,y swapp1:"<<x<<","<<y<<endl;
    swap2(x,y);//使用&后，参数是按引用传递，所以外面的xy会改变
    cout<<"x,y swapp2:"<<x<<","<<y<<endl;
    return 0;
}
```




demo3  [数据结构](https://pythontutor.com/render.html#code=%23include%20%3Ciostream%3E%0Ausing%20namespace%20std%3B%0A%0Avoid%20swap1%28int%20a,%20int%20b%29%7B%0A%20%20%20%20int%20t%3Da%3B%0A%20%20%20%20a%3Db%3B%0A%20%20%20%20b%3Dt%3B%0A%7D%0A%0Avoid%20swap2%28int%20%26a,%20int%20%26b%29%7B%0A%20%20%20%20int%20t%3Da%3B%0A%20%20%20%20a%3Db%3B%0A%20%20%20%20b%3Dt%3B%0A%7D%0A%0Aint%20main%28%29%0A%7B%0A%20%20%20%20int%20x%3B%20%0A%20%20%20%20int%20y%3B%0A%20%20%20%20//%20cin%3E%3Ex%3E%3Ey%3B%0A%20%20%20%20//%20cout%3C%3C%22x,y%3D%22%3C%3Cx%3C%3C%22,%22%3C%3Cy%3C%3Cendl%3B%0A%20%20%20%20x%3D10%3B%0A%20%20%20%20y%3D20%3B%0A%20%20%20%20swap1%28x,y%29%3B//%E5%9B%A0%E4%B8%BA%E5%8F%82%E6%95%B0%E6%98%AF%E6%8C%89%E5%80%BC%E4%BC%A0%E9%80%92%EF%BC%88%E6%8B%B7%E8%B4%9D%EF%BC%89%EF%BC%8C%E6%89%80%E4%BB%A5%E5%A4%96%E9%9D%A2%E7%9A%84xy%E4%B8%8D%E4%BC%9A%E5%8F%98%EF%BC%8C%E6%B2%A1%E8%BE%BE%E5%88%B0%E4%BA%A4%E6%8D%A2%E7%9A%84%E7%9B%AE%E7%9A%84%0A%20%20%20%20cout%3C%3C%22x,y%20swapp1%3A%22%3C%3Cx%3C%3C%22,%22%3C%3Cy%3C%3Cendl%3B%0A%20%20%20%20swap2%28x,y%29%3B//%E4%BD%BF%E7%94%A8%26%E5%90%8E%EF%BC%8C%E5%8F%82%E6%95%B0%E6%98%AF%E6%8C%89%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92%EF%BC%8C%E6%89%80%E4%BB%A5%E5%A4%96%E9%9D%A2%E7%9A%84xy%E4%BC%9A%E6%94%B9%E5%8F%98%0A%20%20%20%20cout%3C%3C%22x,y%20swapp2%3A%22%3C%3Cx%3C%3C%22,%22%3C%3Cy%3C%3Cendl%3B%0A%20%20%20%20return%200%3B%0A%7D&cumulative=false&curInstr=19&heapPrimitives=nevernest&mode=display&origin=opt-frontend.js&py=cpp_g%2B%2B9.3.0&rawInputLstJSON=%5B%5D&textReferences=false)

知识点：
- 1、简单的数据结构
  
```c++

```
