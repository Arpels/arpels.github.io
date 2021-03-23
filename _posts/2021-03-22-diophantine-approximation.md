---
title: '浅谈 Diophantine approximation'
date: 2021-03-22
permalink: /posts/2021/03/diophantine_approximation/
tags:
  - diophantine approximation
  - rational reconstruction
  - continued fraction
---

去年看的东西，如何从实数恢复成分数。最近突然想起来，补一篇博客。

*  目录
{:toc}


去年暑假推免之后，导师让我看了一本 Shoup 的计算数论。其中看到了很有意思的一节，叫 `rational reconstruction`。

查了一些资料，发现它主要讲的一个有意思的应用，其实是 `Diophantine approximation`。

当时忙着做毕设，没怎么写博客。最近突然想起来这个知识点，记录一下。

## 0. 参考资料
* [A Computational Introduction to Number Theory and Algebra](https://www.shoup.net/ntb/)

* [Elementary Number Theory and Its Applications](https://www.fmf.uni-lj.si/~lavric/Rosen%20-%20Elementary%20number%20theory%20and%20its%20applications.pdf)

* [wikipedia - Diophantine_approximation](https://en.wikipedia.org/wiki/Diophantine_approximation)

* [Rational Reconstruction](https://files.oakland.edu/users/steffy/web/reconstruct/)

* [Theory of Linear and Integer Programming](https://promathmedia.files.wordpress.com/2013/10/alexander_schrijver_theory_of_linear_and_integerbookfi-org.pdf)

* [Sage - Continued fractions](https://doc.sagemath.org/html/en/reference/diophantine_approximation/sage/rings/continued_fraction.html)

* [Sage - Arbitrary Precision Real Numbers](https://doc.sagemath.org/html/en/reference/rings_numerical/sage/rings/real_mpfr.html)

* [连分数入门 - 超理论坛](https://chaoli.club/index.php/2756)

## 1. 问题背景
或许有时候会有人问你这个问题：

```
"Given a floating point number, how do I figure out what number the computer really means?"
```

你可能会说，现在很多涉及数学的编程语言，或者是某些语言的数学库，是支持分数运算的，并不会返回浮点数来代替分数结果。确实如此，现在很多涉及数学的语言做的相当不错，但我们依然可以来探讨一下这个有趣的小问题。

这个问题的前提是结果是一个无限循环小数，并且我们并不知道整个循环节。或者是让我们给出无理数的一个分数逼近。因为如果是有限小数，那我们很容易将其写成分数。如果是无理数，那我们只能给出它的一个近似的分数，比如圆周率的密率和约率，而不可能将其表示为完全相等的一个分数。如果是已经知道整个循环节的无限循环小数，那我们也很容易将其写为分数，方法这里就不细说了，可以 Google 关键字 `编程之美 循环节`。

这个问题在数学里被称为 `diophantine approximation` 或 `rational roundoff`，有的书里也会把它归为 `rational reconstruction` 的情况之一。对它的中文称呼是 `丢番图逼近`。

说起丢番图，我们第一印象是丢番图方程，也就是不定方程。丢番图逼近这个名称的由来，正是因为它主要是用于有理数逼近实数，而实数的有理逼近这个问题可以被归结为求不定方程整数解的问题，而并不是因为这个方法由丢番图研究发现。事实上，丢番图逼近与不定方程的研究颇多相关。

丢番图逼近的首要问题是寻找实数的最佳有理逼近，也就是对于给定的实数 $\alpha$，要找到距离其“最近”的 $p/q$，使 $p/q - \alpha$ 要最小。而在 18 世纪对连分数的研究中，这个问题已经得到解决。也就是说，我们用连分数相关的知识，已经足够回答开篇提出的问题。

次要问题则是对有理逼近的误差进行度量和估计，并给出尽可能精确的上下界。而根据研究，上下界取决于被逼进的实数 $\alpha$。当其为有理数、代数数、超越数时，下界的大小是不同的。而这一点使 Liouville 受到了启发，于是他提出了 Liouville 定理。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_a99f55a66162a8c800b9cf71b2765668.png)

而当时人们普遍认知是任意代数数都可以被有理数逼近，于是这就产生了矛盾。

根据这个矛盾，在 1844 年 Liouville 构造出了第一个无理数。由此可见，丢番图逼近与数论的另一个分支超越数论紧密相关。

此外，丢番图逼近在不定方程等领域也有着重要的应用。


## 2. 解法思想
### （1） 连分数概念
* 有限连分式

形如下面的式子被称为有限连分式。我们一般用 $[a_0;a_1,a_2,...,a_n]$ 来表示。

$$
x = a_0 + \cfrac{1}{a_1 +
            \cfrac{1}{a_2 +
              \cfrac{1}{a_3 +
                \cfrac{1}{a_4 + 
                  \cdots +
                  \cfrac{1}{a_{n-1} +
                    \cfrac{1}{a_n}  
                  }
                }
              }
            }
          }
$$

* 每一个有限简单连分数都可以表示一个有理数。（数学归纳法可证明）

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_3ee23b12ff2c42ddf6ca46eb090d0e0d.png)

* 每一个有理数都可以表示为有限简单连分数。（用欧几里得算法写一下即可证）

* 第 $k$ 个收敛子或渐进分数 $C_k = [a_0;a_1,a_2,...,a_k]$

* $C_k = {p_k}/{q_k}$，其中 $p_k = a_k p_{k-1} + p_{k-2}$，$q_k = a_k q_{k-1} + q_{k-2}$

* 第 $k$ 个中间收敛子或中间渐进分数 $C_{k,t} = [a_0;a_1,...,a_{k-1}, t]$

* $C_{k,t} = p_{k,t}/q_{k,t}$，其中 $p_{k,t} = tp_{k-1}+p_{k-2}$，$q_{k,t} = tq_{k-1}+q_{k-2}$

* $\text{(Dirichlet's theorem)}$ 对于实数 $\alpha$，存在整数 $p$，$q$ 满足 $\mid \alpha - p/q \mid < \epsilon / q$，其中 $0 < \epsilon \leq 1$ 且 $1 \leq q \leq \epsilon ^{-1}$

* 对实数 $\alpha$ 和其渐进分数 $p_k/q_k$，有：

$$
\frac{1}{q_k(q_k+q_{k+1})}<\left|\alpha-\frac{p_k}{q_k}\right|<\frac{1}{q_kq_{k+1}}
$$

### （2） 最优逼近
* 第一类最优逼近

对实数 $\alpha$ 和有理数 $p/q$，如果对 $\forall b$ 满足 $0<b\leq q$，$a,b\in \mathbb{Z}$，且 $a/b\neq p/q$ 都有：

$$
\left|\alpha-\frac{a}{b}\right|>\left|\alpha-\frac{p}{q}\right|
$$

则称 $p/q$ 是 $\alpha$ 一个第一类最优逼近。

* 第二类最优逼近

对实数 $\alpha$ 和有理数 $p/q$，如果对 $\forall b$ 满足 $0<b\leq q$，$a,b\in \mathbb{Z}$，且 $a/b\neq p/q$ 都有：

$$
\left|b\alpha-a\right|>\left|q\alpha-p\right|
$$

也就是

$$
\left|{\alpha}-\frac{a}{b}\right|>\frac{q}{b}\left|{\alpha}-\frac{p}{q}\right|
$$

则称 $p/q$ 是 $\alpha$ 一个第二类最优逼近。

显然由于前面多了一个系数，第二类逼近强于第一类。

* 第一类最优逼近一定是中间渐进分数。

* 第二类最优逼近一定是渐进分数，渐进分数一定是第二类最优逼近。

* 偶项渐进分数全都是下逼近，奇项渐进分数全都是上逼近。渐进分数列是下上交错的逼近。

* 注：以上三个定理的证明较长，看相关论文即可。

### （3） 主要思想
基于上面的理论，我们就知道应该如何从一个不知道循环节的无限循环小数，求得分数形式了。

首先，我们将这个已知部分的浮点数写成连分式的形式。根据 `每一个有理数都可以表示为有限简单连分数`，这显然是可以做到的。

其次，我们求其渐进分数。根据渐进分数的定义，显然这也是很好求的。

再次，根据最优逼近的性质可知，`奇项渐进分数都是上逼近`，而我们是根据小数的一部分来逼近的，因此和该小数相等的分式应该是一个上逼近。

最后，根据连分数部分的最后一条性质，我们可知，当渐进分数分母越大时，越能逼近原本的浮点数。

因此综上，当我们知道分母的上界，并且给出的浮点数位数足够时，我们就可以对该浮点数给出一个或多个分数形式的最优逼近。该逼近形式为渐进分数，可能是分母最靠近上届的那个渐进分数，也可能存在多个，如果可能的话，我们可以根据具体的语境来排除得到唯一的那一个分数。


## 3. 相关代码
### （1） 自己实现的

最开始是在参考资料部分的 Shoup 的那本数论书里看到丢番图逼近的，根据那本书上给出的算法，自己在 SageMath 里写了一个小玩意：

```python
s = 1157920892373161954235709850086879078 # 511
t = 1399840463861127631598401425355277673 # 710
assert s < t

k = 80
all_b = hdiv(s, t, k) # hdiv 是自己写的一个高精度除法

print ('[+] I will tell you some of the high order digits in the decimal expansion of b, b = s/t:')
print ('   ', all_b)
print ('\n[+] Please guess s and t:')

n = 10^k
b = int(all_b[2:])
r_star = 10^ceil(log(t, 10))
assert t < r_star

def exgcd(a, b, x=1, y=0):
    r = a
    if r < r_star:
        return r, x, y
    r, y, x = exgcd(b, a%b, y, x)
    y -= (a//b)*x
    return r, x, y

tmp, s_, t_ = exgcd(n, b)

print ('[+] I guess:')
print ('s = ', s_, '\nt = ', -t_)
if (s_ == s)&(-t_ == t):
    print ('\n[+] Congratulations! You\'re right!')
```

运行结果如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_7219add456b051c2216fa78d9c1f4269.png)


### （2） Sage 自带的
后来查了一下 SageMath 的文档，发现已经有实现好的函数可以直接用。

具体怎么用，以及实现任意精度可以参考下面两个文档链接。

* [Sage - Continued fractions](https://doc.sagemath.org/html/en/reference/diophantine_approximation/sage/rings/continued_fraction.html)

* [Sage - Arbitrary Precision Real Numbers](https://doc.sagemath.org/html/en/reference/rings_numerical/sage/rings/real_mpfr.html)

代码如下：

```python
from Crypto.Util.number import getPrime
from tqdm import tqdm
import gmpy2


for i in tqdm(range(100)):
    print ("----------------------------------------------------------------")
    print ("[+] Test {} start.".format(i+1))
    precision = 500
    RA = RealField(precision)

    bit_length = 200
    s = getPrime(bit_length)
    t = getPrime(bit_length)

    bound = bit_length

    if s > t:
        s, t = t, s

    task = RA(s)/RA(t)

    print ("[+] (Invisible) s =", s)
    print ("[+] (Invisible) t =", t)
    print ('[+] I will tell you some of the high order digits in the decimal expansion of b, b = s/t:')
    print ("[*] s/t =", task)
    print ("[*] bound = 2^{}".format(bound))
    print ('\n[+] Please guess s and t:')


    recv_str = str(task)
    recv_num = int(recv_str[2:])

    bit_length = len(bin(recv_num)) - 2
    bound = 2^200
    RA = RealField(bit_length)
    result = []

    tmp = continued_fraction_list(RA(recv_str), bits = bit_length, partial_convergents = True)
    for i in tmp[1][::-1]:
        if i[1] < bound:
            if gmpy2.is_prime(i[1]):
                if gmpy2.is_prime(i[0]):
                    result.append(i)

    print ('[+] I guess:')
    print ('[+] s = {}'.format(result[0][0]))
    print ('[+] t = {}'.format(result[0][1]))
    if (result[0] == (s, t)):
        print ('\n[-] Congratulations! You\'re right!')
    else:
        print ('\n[x] Error!')
        break
```

运行结果如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_a5814b347571a1958c7a4f320bd64614.png)

由于一开始为了方便生成数据，设定的分子分母都是大素数，所以最后满足条件的渐进分数里再把位数也满足的大素数找出来就可以了，有唯一解。

尝试了一百组，一百组都成功了。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_bdb5b1e38b608ba968d723d16b74edff.png)

### （3） C++ 实现

有个国外的大哥基于 `GMP` 和 `NTL` 库实现了一个。

* [Rational Reconstruction](https://files.oakland.edu/users/steffy/web/reconstruct/)

## 4. CNSS 2021 Recruit misc
丢番图逼近我尝试搜索了一下，`OI WIKI` 里倒是有这个条目，不过之前好像并没有被出过题目。因此我打算用这个在 CNSS 今年招新里出一个 misc 玩玩。由于招新只看单榜，misc 并不招人，所以并不会影响招新。

大概思路如下：

```python
from Crypto.Util.number import getPrime

precision = 500
RA = RealField(precision)

bit_length = 200
s = getPrime(bit_length)
t = getPrime(bit_length)

bound = bit_length

if s > t:
    s, t = t, s

print ("[+] s =", s)
print ("[+] t =", t)
print ("[+] bound = 2^{}".format(bound))
print ("[*] s/t =", RA(s)/RA(t))
```

题目背景可以设置成读心术或是类似情境。

预期解是用丢番图逼近从浮点数恢复原来的分数，与服务器交互一百次之后得到 flag。

分数的生成方面，分子分母都用 200 bits 的素数。根据素数定理，200 bits 的素数多如牛毛，如果你要在服务器允许时间内爆破出结果那根本是不可能的事。

而使用素数的另一个好处就是保证了唯一解。

## 5. 其他
和丢番图逼近相关的一个问题是 `rational reconstruction`，在 Shoup 的数论书里直接把丢番图逼近和它归为一类，但其实两者还是有些区别的。

详细内容可以参考 [SageMath - rational reconstruction](https://doc.sagemath.org/html/en/reference/rings_standard/sage/arith/misc.html) 或是 wikipedia。

***

> **声明：本文采用 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) 授权。**
> 
> **This work is licensed under a [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).**
