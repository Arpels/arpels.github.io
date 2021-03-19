---
title: '浅谈 Dual_EC_DRBG'
date: 2021-03-19
permalink: /posts/2021/03/dual_ec_backdoor/
tags:
  - ecc
  - backdoor
---

上周在 `UTCTF 2021` 里看到了一道相关题目 `Sleeves`。虽然难度并不大，不过背景故事还挺有意思的。

*  目录
{:toc}

上周在 `UTCTF 2021` 里看到了一道相关题目 `Sleeves`。虽然难度并不大，不过背景故事还挺有意思的。

前两个参考链接对这个问题讲述的非常详细，因此这里只是简单说一说，用作自己备忘。

## 0. 参考链接
* [wikipedia - Dual_EC_DRBG](https://en.wikipedia.org/wiki/Dual_EC_DRBG)

* [Dual EC: A Standardized Back Door](https://eprint.iacr.org/2015/767)

* [hukc's writeup](https://github.com/cscosu/ctf-writeups/tree/master/2021/utctf/Sleeves)

## 1. 背景
`Dual_EC_DRBG` 曾经是 `NIST` 给出的四个 `CSPRNG` 标准算法之一，它的设计者是 `NSA`，2006 年它被 `NIST` 作为标准，到 2014 年被移除。

在它刚被推出后不久，就受到许多研究者的质疑存在后门。后来后门被研究者们发现，而舆论认为这个后门有可能是 `NSA` 某个计划的一部分。而后续的斯诺登泄露的备忘录大大增加了这种情况的可能性。

这个后门如今已经非常知名，在许多涉及密码学 backdoor 的问题下经常可以看见它被用作例子。

## 2. 原理
第二个参考链接的 5.3 部分讲了这个后门最基本的原理，原文如下。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_23326befbd5e56e254e91fbac9f160ee.png)

原理就是 $P = dQ$ 关系中的 $d$ 如果被攻击者知道了，那么攻击者就可以利用 $d$ 构造出之后每一步的 $state$，从而可以成功预测每一步的 $r_i$，这个随机数生成器也就被攻破了。

简单的分析与推导如下：

我们可以先列出这个 RNG 生成下一个随机数的相关式子。

设每一步的 $state$ 为 $s_i$，随机数为 $r_i$，随机数对应的椭圆曲线上的点为 $R_i$。

那么对于攻击者来说，已知 $P$、$Q$、$d$、$R_i$，而 $s_i$ 未知。于是有：

$$
\begin{cases}
x(x(s_i \cdot P) \cdot P) & \rightarrow s_{i+1} \\
x(x(s_i \cdot P) \cdot Q) & \rightarrow r_i
\end{cases}
$$

显然，我们需要求得某一步的 $s_i$，那么之后每一步的 $s_i$ 就都知道了，从而之后的 $r_i$ 我们也都可以预测了。

而论文里的这个后门，就是构造 $d \cdot r_{i-1}$，其恰好是 $s_i$，于是看似安全的体制就被攻破了。

那么两者为何相等呢？

记 $k_i = x(s_i \cdot P)$，推导如下：

$$
\begin{align}
d \cdot r_{i-1} & = {x(d \cdot R_{i-1}) = x(d \cdot k_{i-1} \cdot Q)} \\
& = {x(k_{i-1} \cdot d \cdot Q) = x(k_{i-1} \cdot P)} \\
& = {x(x(s_{i-1} \cdot P) \cdot P) = s_i}
\end{align}
$$

于是得证。

这就是这个后门的原理。

## 3. 实例：UTCTF 2021 - `Sleeves`
### （1）题目
```python
from random import randint

class RNG:
    def __init__(self):
        # This elliptical curve is NIST P-256, a well-known curve that should
        # be unexploitable.
        p = 115792089210356248762697446949407573530086143415290314195533631308867097853951
        b = 0x5ac635d8aa3a93e7b3ebbd55769886bc651d06b0cc53b0f63bce3c3e27d2604b
        self.curve = EllipticCurve(GF(p), [-3,b])

        # These are the generator points for the PRNG.
        self.P = self.curve.lift_x(110498562485703529190272711232043142138878635567672718436939544261168672750412)
        self.Q = self.curve.lift_x(67399507399944999831532913043433949950487143475898523797536613673733894036166)

        self.state = randint(1, 2**256)

    def next(self):
        # In elliptical curve cryptography, scalar multiplication is a trapdoor
        # function.
        # This means that recovering `self.state` from `self.P` and `sP` is infeasible.
        sP = self.state * self.P
        r = Integer(sP[0])
        self.state = Integer((r*self.P)[0])
        rQ = r*self.Q
        # Though we lose 8 bits of rQ, 2**8 is easily bruteforceable
        return Integer(rQ[0])>>8
```
```python
from challenge.eccrng import RNG
from Crypto.Cipher import AES
from Crypto.Hash import SHA256

rng = RNG()
# I'll just test it out a few times first
print("r1 = %d" % rng.next())
print("r2 = %d" % rng.next())

r = str(rng.next())
aes_key = SHA256.new(r.encode('ascii')).digest()
cipher = AES.new(aes_key, AES.MODE_ECB)
print("ct = %s" % cipher.encrypt(b"????????????????????????????????????????").hex())
```

### （2）题解
思路就和上面原理部分分析的一样。由于这个 `ECDLP` 很容易求解，因此 $d$ 就被我们知道了。

```python
p = 115792089210356248762697446949407573530086143415290314195533631308867097853951
b = 0x5AC635D8AA3A93E7B3EBBD55769886BC651D06B0CC53B0F63BCE3C3E27D2604B
E = EllipticCurve(GF(p), [-3, b])
P = E.lift_x(110498562485703529190272711232043142138878635567672718436939544261168672750412)
Q = E.lift_x(67399507399944999831532913043433949950487143475898523797536613673733894036166)
print(Q.discrete_log(P))

173
```

于是直接利用这个后门即可。

代码如下：

```python
from Crypto.Cipher import AES
from hashlib import sha256
from tqdm import tqdm
from Crypto.Util.number import long_to_bytes, bytes_to_long


p = 115792089210356248762697446949407573530086143415290314195533631308867097853951
b = 0x5AC635D8AA3A93E7B3EBBD55769886BC651D06B0CC53B0F63BCE3C3E27D2604B
E = EllipticCurve(GF(p), [-3, b])
P = E.lift_x(110498562485703529190272711232043142138878635567672718436939544261168672750412)
Q = E.lift_x(67399507399944999831532913043433949950487143475898523797536613673733894036166)

d = Q.discrete_log(P)

def do_next(s):
    sP = s * P
    r = Integer(sP[0])
    s_new = Integer((r * P)[0])
    rQ = r * Q
    return Integer(rQ[0]), s_new


def do_guess(r1):
    try:
        rQ1 = E.lift_x(r1)
    except ValueError:
        return None
    sP2 = d * rQ1
    s2 = Integer(sP2[0])
    r2, s3 = do_next(s2)
    return r2, s3


def decrypt(r):
    ct = long_to_bytes(0xc2c59febe8339aa2eee1c6eddb73ba0824bfe16d410ba6a2428f2f6a38123701)
    aes_key = sha256(str(r).encode()).digest()
    cipher = AES.new(aes_key, AES.MODE_ECB)
    pt = cipher.decrypt(ct)
    print(pt)


r1 = 135654478787724889653092564298229854384328195777613605080225945400441433200
r2 = 16908147529568697799168358355733986815530684189117573092268395732595358248

# bruteforce 8 bits unknown
for i in tqdm(range(256)):
    r1_guess = (r1 << 8) + i
    res = do_guess(r1_guess)

    if res:
        r2_guess, s3 = res
        if r2_guess >> 8 == r2:
            r3, s4 = do_next(s3)
            decrypt(r3 >> 8)
            break
```

结果如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_239c386ee7664396380681cde98a39ee.png)
