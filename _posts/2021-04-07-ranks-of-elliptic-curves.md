---
title: '浅谈 Ranks of Elliptic Curves'
date: 2021-04-07
permalink: /posts/2021/04/ranks_of_elliptic_curves/
tags:
  - ECC
---

清明假期虎符赛道的一道题 `cubic`，之前见过，不难但很有意思，记录一下。


*  目录
{:toc}


清明假期虎符赛道的一道题 `cubic`，之前见过，不难但很有意思，记录一下。

## 0. 参考资料
* [wikipedia - BSD 猜想](https://en.wikipedia.org/wiki/Birch_and_Swinnerton-Dyer_conjecture)
* [Congruent number](https://en.wikipedia.org/wiki/Congruent_number)
* [360doc - BSD 猜想：不定方程的有理解问题](http://www.360doc.com/content/20/1207/18/32196507_949999641.shtml)
* [中国科学院大学实景课堂 - 有限生成阿贝尔群](https://v.ucas.ac.cn/course/getplaytitle.do?menuCode=2&code=ce56ac697921449f9bfa669b2cde01c6&classcode=1&classid=52bb07b81d6d4e2ab65a66101a83b6ab&sectionNumber=4&sectionDisplay=0)
* [stanford notes - elliptic curve](https://crypto.stanford.edu/pbc/notes/elliptic/)
* [知乎问题 - 数论中的 BSD 猜想是指什么？](https://www.zhihu.com/question/20473905)
* [知乎文章 - 椭圆曲线的一些基础性质](https://zhuanlan.zhihu.com/p/27159180)
* [知乎文章 - 你能找到苹果、香蕉、菠萝的正整数解吗](https://zhuanlan.zhihu.com/p/33853851)
* [看雪论坛 - Mordell 定理](https://bbs.pediy.com/thread-128083.htm)
* [Quora - How do you find the positive integer solutions to $\frac{x}{y+z} + \frac{y}{x+z} + \frac{z}{x+y} = 4$](https://www.quora.com/How-do-you-find-the-positive-integer-solutions-to-frac-x-y+z-+-frac-y-z+x-+-frac-z-x+y-4)
* [An unusual cubic representation problem](https://ami.uni-eszterhazy.hu/uploads/papers/finalpdf/AMI_43_from29to41.pdf)
* [Ranks of Elliptic Curves - Math Berkeley](https://math.berkeley.edu/~molsson/KConrad-ranks.pdf)
* [On Computing the Rank of Elliptic Curves](https://www.math.colostate.edu/~achter/math/brown.pdf)
* [SageMath - BSD lecture](https://sage.math.leidenuniv.nl/home/pub/46/)
* [Integer solutions to the equation $a^3+b^3+c^3 = 30$](https://math.stackexchange.com/questions/1386034/integer-solutions-to-the-equation-a3b3c3-30)
* [Sums of three cubes](https://math.mit.edu/~drew/NTW2020.pdf)


## 1. 概述
清明假期的虎符赛道，有一道 crypto 题目 `cubic`。

题目和曾经一些营销号经常发的一个问题很像，如下图：


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_a41d23269ccf00973b0f9cc26a2b1332.png" width = "300" height = "200"/>

只不过题目里把 $4$ 换成了 $6$。

要解这题并不难，Google 一搜就可找到相关的问题与回答，代码思路都非常详细。不过在阅读后，我发现国内知乎上有一篇文章翻译时出现了一点小错误，将 `rank` 翻译成了 `阶`，而正确的翻译应该是 `秩`，`order` 才是 `阶`。

这两者是完全不同的两个概念。`秩` 是指椭圆曲线上有理数点中，阶数为无穷的那一部分点，所能用的最少数量 `r` 个点来生成，这个 `r` 就是 `rank`，`秩`。`阶` 则一般指点的个数：有限域上椭圆曲线的 `阶` 一般指椭圆曲线上点的个数；椭圆曲线上某个点的 `阶` 则一般指该点作为生成元，生成的循环群中点的个数；`秩` 所涉及的那些点，由于永远不会循环，所以它们的 `阶` 是无穷的。

这个问题的解决恰好要用到秩，于是不妨稍微深入的了解一下秩，学完开贴记录一下。

至于 `秩` 及其相关问题的发展历史，可以看 [参考资料](#0-参考资料)，其中关于 `BSD 猜想` 的那个知乎问题，下面有一个回答很详细的介绍了相关历史。

## 2. 知识铺垫
### （1） 椭圆曲线基本知识
椭圆曲线的基本知识这里就不说了，不太了解的可以看 [参考资料](#0-参考资料)，其中的斯坦福笔记和那篇知乎文章比较适合入门。

### （2） 有限生成阿贝尔群
相关细节可以看 [参考资料](#0-参考资料)。


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_0bdd2c63b3143f69a6d0af45db898478.png" width = "400" height = "400"/>

<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_625695861188a530f4d14f377d09b630.png" width = "400" height = "180"/>

### （3） Nagell–Lutz theorem
这个定理告诉我们 `torsion points` 的性质，`torsion points` 就是那些有限阶的点，它们是一个个由生成元（循环群某个点）生成的循环群。这个定理常常被用来计算 `torsion points`。


![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_bf10ae4508687b17870c9e9583d0e5e8.png)


### （4） Mordell(-Weil) theorem
这个定理是从有限生成阿贝尔群的角度来看待椭圆曲线。这个定理告诉我们，椭圆曲线上的有理数点分两部分，挠部分和秩部分，挠部分是所有 `torsion points`，秩部分是那些无穷阶的点。挠部分根据前一个定理是好求的，但秩部分并不好求，目前还没人知道有什么简单的公式来表示 `秩`。


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_2ca9bd4e968c0b1b5156e756a52af4c8.png" width = "400" height = "300"/>

### （5） BSD conjecture
`BSD 猜想` 起源于上世纪五十年代，数学家们对椭圆曲线上有理点的个数的探索，它给计算 `秩` 提供了一种可能的途径。关于 `BSD 猜想` 的相关历史和细节，可以看 [参考资料](#0-参考资料) 部分关于 `BSD` 的链接。


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_1c11d3d23e27532068571a101eee7d69.png" width = "400" height = "280"/>

<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_faa517ec675ef99b3919e580788fe0ee.png" width = "420" height = "300"/>


## 3. 什么是秩
在知识铺垫的 `Mordell Theorem` 部分，我们就提到过 `秩` 的概念。

椭圆曲线上的有理数点，分为挠部分和秩部分。所有的 `torsion points` 属于挠部分，它们的 `阶` 是正整数。而其他那些 `阶` 为无穷的点，构成了秩部分。秩部分所有的点数量是无限的，但它们可以用有限个数的点来生成，而这个最小的有限个数 `r`，就是 `秩`。

举例如下图：


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_d77799d304de2c6a5c417500df191786.png" width = "380" height = "320"/>

## 4. 秩的求解
`秩` 的求解目前并没有什么好的方法，没有直接的公式。

`BSD 猜想` 是求解 `秩` 的一种可能的途径，但目前依然还是猜想。

一般只能通过分析来求所谓的 `analytic rank`。

比如 [参考资料](#0-参考资料) 中有的链接就是通过分析，证明 `rank` 的上下界恰好都为某个数，通过夹逼，求得 `rank`。

或者我们可以用 `SageMath`，文档里说貌似它用的是 `BSD 猜想` 来给出一个 `analytic rank`。

尝试如下：

```python
from tqdm import tqdm

R, dic = [2010 .. 2013], {}
for a, b in tqdm(cartesian_product([R, R])):
    try:
        E = EllipticCurve(QQ, [a, b])
        E.two_descent(second_limit=13, verbose=False)
        r = E.rank(only_use_mwrank=False)
        dic[(a, b)] = r
        # print(f'({a}, {b}) -> {r}')
    except Exception:
        dic[(a, b)] = None

print(dic)
```

结果如下：


![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_a154ddda59bfafae12232598a80c4363.png)


## 5. 相关应用
### （1） Congruent Number
中文里一般叫 `同余数问题`，当然，此同余数非彼同余。

问题是：对任意整数，是否存在直角三角形，面积为该整数，三边均为有理数？


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_48ed05bc5b115b21b07007bb0ba2e78b.png" width = "400" height = "310"/>

显然，转化为椭圆曲线，那么如果要有有理数点，`秩` 应该至少是 $1$。


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_01ebcaf805c471b2d9d8dd20405b1512.png" width = "400" height = "310"/>

此外，目前对这个问题一些研究进展，可以参考 wikipedia。

### （2） Sylvester’s conjecture
在 $x^3+y^3 = n$ 这个问题中的应用：


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_1a5bae4b198d2dbd34848c361fdeee44.png" width = "400" height = "310"/>

在另一个常见的营销号问题 $x^3+y^3+z^3 = n$ 中也经常会提到这个应用：


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_96c8b704b038b3a05abde31212e827f5.png" width = "450" height = "300"/>

### （3） Hilbert’s 10th problem
在希尔伯特第十问题中的应用：


<img src="https://codimd.s3.shivering-isles.com/demo/uploads/upload_a5f4ab0216e79d5c8c3474bdeac4990d.png" width = "400" height = "300"/>


## 6. 2021 虎符网络安全赛道 - cubic
### （1） 题目
```python
from math import gcd
from functools import reduce
from fractions import Fraction as Frac


N = 6

def read_num(prompt):
    try:
        num = int(input(prompt))
    except:
        return 0
    return num if num > 0 else 0


print(f"Please give me {N} pairs of positive integers (x,y,z) "
      f"satisfying the equation `x/(y+z) + y/(z+x) + z/(x+y) = {N}`\n")
anss = []
mark = 0
for i in range(N):
    x = read_num("[>] x: ")
    y = read_num("[>] y: ")
    z = read_num("[>] z: ")
    if x * y * z == 0: # positive integer
        mark = 1
        print("This is not what i want!\n")
        break
    if reduce(gcd, [x, y, z]) != 1: # (kx, ky, kz)
        mark = 1
        print("This is not what i want!\n")
        break
    if Frac(x, y+z) + Frac(y, z+x) + Frac(z, x+y) != N:
        mark = 1
        print("This is not what i want!\n")
        break
    ans = tuple(sorted([x, y, z])) # (y, x, z)
    if ans in anss:
        mark = 1
        print("This is not what i want!\n")
        break
    else:
        print("You are right!\n")
        anss.append(ans)
if mark == 0:
    flag = open('/flag', 'r').read()
    print("flag is: " + flag + "\n")
else:
    print("Something wrong!\n")
```

### （2） 思路
显然是要找六组正整数解，满足：${a \over b+c}+{b \over a+c}+{c \over a+b} = 6$。

Google 一搜就能搜到这篇论文：[An unusual cubic representation problem](https://ami.uni-eszterhazy.hu/uploads/papers/finalpdf/AMI_43_from29to41.pdf)。

以及 Quora 上的 [这个问题](https://www.quora.com/How-do-you-find-the-positive-integer-solutions-to-frac-x-y+z-+-frac-y-z+x-+-frac-z-x+y-4)。

论文里指出，这个问题实质上是椭圆曲线的问题，并且把右式拓展到了任意整数 $N$ 上。

对应关系如下：

$$
E_n:y^2 = x^3+(4N^2+12N-3)x^2+32(N+3)x \\
a={8(N+3)-x+y \over 2(N+3)(4-x)} \\ 
b={8(N+3)-x-y \over 2(N+3)(4-x)} \\ 
c={-4(N+3)-x(N+2) \over (N+3)(4-x)}
$$

所以我们只要在曲线上找六个有理数点。

而且由于原本的式子是齐次的，所以我们并不用担心 $a,b,c$ 解出来不是整数，同乘上分母的最小公倍数即可。

那怎么保证能在这条曲线上找六个点呢？这就用到本篇博客的主题了。Quora 里的回答指出，这条曲线的 `秩` 为 $1$。那问题就简单了，这条曲线分挠部分和秩部分，秩部分有无穷个有理数点，它们都可以由某一个有理数点生成。我们只要随机在曲线上取一个点，这个点不是 `torsion points` 的概率很大，然后我们对它不断自加，很容易得到要求的六个点。

最开始怎么随机找一个点？直接用 `SageMath` 自带的 `ec.gens()` 即可。


### （3） 代码
```python
ec = EllipticCurve([0, 213, 0, 288, 0])
P = ec.gens()[0]

def check(num):
    return num if num > 0 else 0

def orig(P, N):
    x = P[0]
    y = P[1]
    a = (8*(N+3)-x+y) / (2*(N+3)*(4-x))
    b = (8*(N+3)-x-y) / (2*(N+3)*(4-x))
    c = (-4*(N+3)-(N+2)*x) / ((N+3)*(4-x))
    da = denominator(a)
    db = denominator(b)
    dc = denominator(c)
    l = lcm(da, lcm(db, dc))
    return [a*l, b*l, c*l]

i = 1
result = []
while True:
    f = True
    ans = orig(i*P, 6)
    for j in ans:
        if check(j) == 0:
            f = False
    if f:
        result.append(ans)
    i += 1
    if len(result) == 6:
        break
print(ans)
```


## 7. 类似问题
一个看起来非常相似的问题：求 $a^3+b^3+c^3 = 33$ 的整数解。

2019 年 Andrew Booker 在论文 `Cracking the problem with 33` 中给出了整数解以及研究方法。

与这个问题相关的更多的细节可以参考 [参考资料](#0-参考资料) 中 `sums of three cubes` 那个链接。

这个问题我打算之后的博客再讲。


***

> **声明：本文采用 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) 授权。**
> 
> **This work is licensed under a [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).**
