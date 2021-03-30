---
title: '浅谈 Weak Prime Factors'
date: 2020-12-11
permalink: /posts/2020/12/Weak Prime/
tags:
  - Lattice
  - RSA
---

因为某个比赛的一道题去搜索论文，没有搜到但碰巧看了另一篇论文。想不到很快就在另一个比赛用上了。实在是“有心栽花花不开，无心插柳柳成荫”。

*  目录
{:toc}


## 1. 从CryptoCTF 2020 说起
今年暑假的 `CryptoCTF2020`，有一道 `Decent RSA` 让我印象挺深刻。常规的大数分解方法并不能分解给出的 `n`。

然而巧妙的是，在十一进制下，`n` 表现出的形式非常漂亮。
首先，其中出现了大量的 `0`，并且没有紧挨着的非零位，也就是所有非零的数之间至少有一个零隔开。
其次，非零的数除了 `1`，其余均为 `2`、`4`、`8`，都是 `2` 的倍数。

那很显然，我们可以将其 `11` 进制的形式表示为多项式，利用多项式分解，完成对 `n` 的分解。

这个方法很巧妙，但我自己尝试了一下，和题目中的 `n` 性质类似的，出现了大量的 `0` 和并且在某种进制下很好分解的 `n`，并不是很容易找到。

于是我试图搜索有关的论文，深入探究一下这个问题。然而，由于自己搜索水平有限，最终没有搜到和这个方法直接相关的论文。
但也并不是一无所获。搜索过程中我复习了一遍 `Coppersmith` 相关攻击，并读了一些论文。
而这篇论文：[`Factoring RSA moduli with weak prime factors`](https://eprint.iacr.org/2015/398.pdf)， 让我意外的是居然在之后不久的 `N1CTF2020` 里用到了，几乎是完全一样的方法。

## 2. 关于论文
论文大致的意思是作者对 `Coppersmith` 的 `Stereotyped messages Attack` 进行了改进。

`Coppersmith` 的情况适用于 $p = M_1u_1+M_0$ $(u_1, u_0 < N ^{0.25})$。
而作者改进后的版本适用于 $ap = u_0 + M_1u_1 + M_2u_2 + ... + M_ku_k$。

这种形式生成的素数被作者称为 `weak prime factors`。
由于论文中已经给出了详细的证明和各种例子，所以此处就不再赘述了。

论文如下：
<iframe src="https://blog.arpe1s.xyz/files/Factoring RSA moduli with weak prime factors.pdf" style="width:700px; height:800px;" frameborder="0"></iframe>




## 3. N1CTF2020 easyRSA
很巧的是之后在 `N1CTF` 中就用上了这篇论文。

题目如下：
```python
from Crypto.Util.number import *
import numpy as np

mark = 3**66

def get_random_prime():
    total = 0
    for i in range(5):
        total += mark**i * getRandomNBitInteger(32)
    fac = str(factor(total)).split(" * ")
    return int(fac[-1])

def get_B(size):
    x = np.random.normal(0, 16, size)
    return np.rint(x)

p = get_random_prime()
q = get_random_prime()
N = p * q
e = 127

flag = b"N1CTF{************************************}"
secret = np.array(list(flag))

upper = 152989197224467
A = np.random.randint(281474976710655, size=(e, 43))
B = get_B(size=e).astype(np.int64)
linear = (A.dot(secret) + B) % upper

result = []
for l in linear:
    result.append(pow(l, e, N))

print(result)
print(N)
np.save("A.npy", A)

```

显然，题目分为两部分，第一部分中的 `n` 就是由这篇论文所讲的 `weak prime` 所相乘得到的。
因此我们使用论文中描述的方法，首先根据 `n` 确定 `M` 的大小，再根据 `M` 选取符合要求的 `k` 和 `c`，然后构造一个格如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_af03ea77e978f4bf19d4880d2fd41659.png)

最后用 `LLL` 算法进行格基规约，将规约后的某个向量作为多项式系数，再对多项式进行分解，即可完成对 `n` 的分解。


## 4. 代码实现
我在 `SageMath9.1` 中给出了代码实现，并用论文中的两个例子，以及 `N1CTF2020 easyRSA` 的数据进行了测试。

代码如下：
```python
from tqdm import tqdm
import gmpy2


class success(Exception):
    pass


def attack_weak_prime(basenum, exp, n):
    m = basenum^exp
    k = len(n.str(base=basenum))//(2*exp) + 1
    c = gmpy2.iroot(2*k^3, int(2))
    # assert c[1] == True
    tmp = int(c[0])

    try:
        for c in tqdm(range(1, tmp)):
            amount = 2*k+1

            M = Matrix(RationalField(), amount, amount)
            for i in range(amount):
                M[i, i] = 1
                M[i, amount-1] = c*m^(2*k-i)
            M[amount-1, amount-1] = -c*n

            new_basis = M.LLL(delta=0.75)
            for j in range(amount):
                last_row = list(new_basis[j])
                last_row[-1] = last_row[-1]//(-c)

                poly = sum(e * x^(k*2-i) for i,e in enumerate(last_row))
                fac = poly.factor_list()
                if len(fac) == 2:
                    p_poly, q_poly = fac
                    p_coefficient = p_poly[0].list()
                    q_coefficient = q_poly[0].list()
                    ap = sum(m^i * j for i,j in enumerate(p_coefficient))
                    bq = sum(m^i * j for i,j in enumerate(q_coefficient))
                    p = gcd(ap, n)
                    q = gcd(bq, n)

                    if (p*q == n) and (p != 1) and (q != 1):
                        raise success

    except:
        print ('n =', n)
        print ('p =', p)
        print ('q =', q)
        print ('p*q == n ?', bool(p*q == n))


if __name__ == '__main__':
    print ('[+] Weak Prime Factorization Start!')
    print ('-------------------------------------------------------------------------------------------------------------------------------')
    basenum, exp = (3, 66)
    n = 32846178930381020200488205307866106934814063650420574397058108582359767867168248452804404660617617281772163916944703994111784849810233870504925762086155249810089376194662501332106637997915467797720063431587510189901
    attack_weak_prime(basenum, exp, n)
    print ('-------------------------------------------------------------------------------------------------------------------------------')
    basenum, exp = (2, 300)
    n = 2225247841130921784107286036701370588942955257324445844921877791958982583022374460167661056809469768595526508244174896515945933598642030611401384177196278679610792649992710545439547737674344167389994475132339971873048849649589042797025394919188452806473589845335273211757008624589940594696926424952951236975084566789
    attack_weak_prime(basenum, exp, n)
    print ('-------------------------------------------------------------------------------------------------------------------------------')
    basenum, exp = (2, 100)
    n = 10009752886312109988022778227550577837081215192005129864784685185744046801879577421186031638557426812962407688357511963709141
    attack_weak_prime(basenum, exp, n)
```
得到的结果如下：
![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_1a67ce9f828b7c03d7329f7c6ca33b95.png)

***

> **声明：本文采用 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) 授权。**
> 
> **This work is licensed under a [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).**
