---
title: '浅谈 NTRU Lattice Attack'
date: 2021-03-29
permalink: /posts/2021/03/ntru_lattice_attack/
tags:
  - NTRU
  - Lattice
---

CTF 里 `NTRU` 见过很多次了，总结一下。

*  目录
{:toc}

上周的比赛有道 `NTRU`，用的还是非常经典的那个格。碰巧有认识的朋友不太会来问我，正好写篇博客总结一下。记得自己第一次碰到 `NTRU` 还是在 19 年的巅峰极客，不知不觉已过两年，时光匆匆，白驹过隙。

## 0. 参考资料
* [wikipedia - NTRU](https://en.wikipedia.org/wiki/NTRUEncrypt)

* [Lattice Attacks on NTRU](https://link.springer.com/chapter/10.1007/3-540-69053-0_5)

* [从一道CTF题初探NTRU格密码](https://xz.aliyun.com/t/7163)

* [StackExchange - Significance of parameter q in NTRU lattice attack](https://crypto.stackexchange.com/questions/87747/significance-of-parameter-q-in-ntru-lattice-attack)

* [StackExchange - What is the largest parameter broken for NTRU?](https://crypto.stackexchange.com/questions/88567/what-is-the-largest-parameter-broken-for-ntru)

* [StackOverflow - How to make a message into a polynomial?](https://stackoverflow.com/questions/1562548/how-to-make-a-message-into-a-polynomial/2404582)

* [Lattice Basics](https://public.csusm.edu/ssharif/crypto/LatticeBasics.pdf)

* [Shortest Vector from Lattice Sieving](https://eprint.iacr.org/2017/999.pdf)

* [NTRU 算法](https://www.cnblogs.com/xdyixia/p/12597290.html)

* [Lattice in Cryptography](https://www.cs.tau.ac.il/~tromer/PKC2003/pkclattice2.pdf)

* [LatticeHacks - NTRU](https://latticehacks.cr.yp.to/ntru.html)

## 1. 知识铺垫
### （1） 格相关
以下概念都是常见概念，详细可以自行 Google，或查看列出的 [参考资料](#0-参考资料)。

* SVP

格上的最短向量问题。

* CVP

格上的最近向量问题。

* The Hermite bound

`Hermite's Theorem` 给出了最短向量长度的上界。它的证明基于 `Minkowski's Theorem`。


* Guassian Heuristic

`Gaussian Heuristic` 给出了上界的进一步缩小。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_84e8545fa8c70aa78520e73482d5a4b1.png)

* Lagrange-Gauss reduction

19 世纪由 Guass 提出的格基规约算法。

* LLL algorithm

1982 年由 Lenstra 等人提出的格基规约算法。

### （2） NTRU
* 概述

`NTRU` 是 1996 年由 Hoffstein 等数学家提出的加密方案。

该方案建立在多项式环的基础之上，安全性基于 `SVP` 问题。它的提出主要是作为 `RSA` 和 `ECC` 的替代者。

2011年，Damien Stehlé 在理想格上基于 `R-LWE` 问题构造了选择明文攻击安全的 `NTRU` 加密体制。2012 年，Ron Steinfeld 等人在理想格上提出了选择密文攻击安全的 `NTRU` 加密体制。

* 特点

参数选取上，多项式的 `degree`，$n$ 一般在 250 到 2500 之间。

$q$ 一般取 $2$ 的幂次，$p$ 一般取 $3$。

加法和多项式加法一样，乘法则是 `cyclic convolution`。

对于明文的编码，一般采用三进制。

相比于 `RSA` 和 `ECC`，`NTRU` 速度更快，但密钥密文占用更多空间。

过去的二十年该方案被很多研究人员尝试着攻击。目前，`NTRU signature` 处境艰难，但 `NTRU encryption` 在更改过参数后依然坚挺。

* 参数设定

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_344232eb11feecfa3d46216e9e56865e.png)


* 密钥生成：

私钥：$f$，$g$，多项式系数为三进制 $\{-1, 0, 1\}$，其中用 $-1$ 代表 $2$，因为 `NTRU` 对参数进行中心化。

公钥：$h = p \cdot g \cdot f^{-1} \pmod q$

* 加密：

注意：因为 `NTRU` 对参数进行中心化，因此这里的模 $q$ 实际上是 $[-q/2, q/2]$，而不是 $[0, q)$。

明文：$m$ 按字母拆分，每个字母先转为 `ascii`，再由十进制转为三进制，然后作为多项式系数编码到多项式上。由此可见，多项式的 `degree` 会限制需要加密的明文长度。

随机选取的多项式：$r$

密文：$c = r \cdot h + m \pmod q$

* 解密：

密文：$c$

私钥：$f$，$g$

中间步骤：$a = f \cdot c = f \cdot (r \cdot h +m) = f \cdot m + 3 \cdot r \cdot g \pmod q$

中间步骤：$f_3 = f^{-1} \pmod 3$

明文：$m = a \cdot f_3 = f \cdot f^{-1} \cdot m = m \pmod 3$

## 2. Lattice Attack on NTRU
### （1） 攻击思想
最早提出这种攻击方法的是 1997 年 Coppersmith 和 Shamir 的一篇论文，论文链接在 [参考资料](#0-参考资料) 部分。

* 格子的构造

格子的构造如下图所示:

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_f9129d12dc379579b08129a8f8a1d37d.png)

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_730b2599ef7cf54fba6048b296865027.png)

* 格子的形式

构造的格子形式是：

$$
\begin{pmatrix} I & H \\ 0 & q \\ \end{pmatrix}
$$

其中 $H$ 是根据公钥多项式的系数生成的循环矩阵。

* 为什么 LLL 可以求解

观察上面论文里格子的构造，显然，因为 

$$
\begin{pmatrix} f & k \\ \end{pmatrix} \cdot \begin{pmatrix} I & H \\ 0 & q \\ \end{pmatrix} = \begin{pmatrix} f & g \\ \end{pmatrix}
$$ 

，所以私钥构成的向量确实在这个格子上。

那为什么私钥的求解等价于求解近似 `SVP` 问题呢？

假设参数 $n = N$，则多项式的度为 $N-1$。设构造的这个格为 $L$，则 $\det(L) = q^N$，$\dim(L) = 2N$。

根据 `Guassian Heuristic`，我们可以知道这个格子的 `SVP` 问题的上界 $B$ 为 $B = \sqrt{N}\det(L)^{1/\dim(L)} = \sqrt{qN}$，而最短向量的长度必然是小于这个上界。

同时，我们发现由于采用三进制编码，多项式系数的平方不是 $1$ 就是 $0$，因此有 $||(f, g)|| < 2N < B$。

于是，我们可以认为，如果我们使用 `LLL` 算法，那解出来的最短向量必然是小于这个上界的。因为 $(f, g)$ 也在格上，并且也小于这个上界，因此我们可以期望最好的情况它就是这个最短向量。即使不是最短的这个，求出来的其他小于上界的向量，我们也可以考虑来验证是否是我们要求解的私钥。

于是我们用 `LLL` 解近似 `SVP` 问题即可。

* 改进的构造

除了最初始的 Coppersmith 构造的这个格子外，后来又有研究人员在此基础上进行了改进。细节可以看相关论文。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_c21f1e4bebcc45e0b9d65b5317d9c710.png)

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_52aacc3aad6113a21c9edffa5960eb32.png)

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_5fb62778869ace099fe7755f13006e27.png)



### （2） 代码实现
```python
Zx.<x> = ZZ[]

# the multiplication operation used in NTRU
def convolution(f, g):
    return (f * g) % (x^n - 1)

def balancedmod(f, q):
    g = list(((f[i] + q//2) % q) - q//2 for i in range(n))
    return Zx(g)

def invertmodprime(f, p):
    T = Zx.change_ring(Integers(p)).quotient(x^n-1)
    return Zx(lift(1 / T(f)))

def invertmodpowerof2(f, q):
    assert q.is_power_of(2)
    g = invertmodprime(f, 2)
    while True:
        r = balancedmod(convolution(g, f), q)
        if r == 1: return g
        g = balancedmod(convolution(g, 2 - r), q)

def randomdpoly():
    assert d <= n
    result = n*[0]
    for j in range(d):
        while True:
            r = randrange(n)
            if not result[r]: break
        result[r] = 1-2*randrange(2)
    return Zx(result)

def keypair():
    print ("----------------------------------")
    print ("[+] Keypair Generation Start...")
    while True:
        try:
            f = randomdpoly()
            f3 = invertmodprime(f, 3)
            fq = invertmodpowerof2(f, q)
            break
        except:
            pass
    print ("[-] f Generation Finished.")
    g = randomdpoly()
    print ("[-] g Generation Finished.")
    publickey = balancedmod(3 * convolution(fq,g), q)
    secretkey = f, f3
    return publickey, secretkey


def encrypt(message, publickey):
    r = randomdpoly()
    return balancedmod(convolution(publickey, r) + message, q)

def randommessage():
    result = list(randrange(3) - 1 for j in range(n))
    return Zx(result)

def decrypt(ciphertext,secretkey):
    f, f3 = secretkey
    a = balancedmod(convolution(ciphertext,f), q)
    return balancedmod(convolution(a, f3), 3)

def attack(publickey):
    recip3 = lift(1/Integers(q)(3))
    publickeyover3 = balancedmod(recip3 * publickey, q)
    M = matrix(2 * n)
    for i in range(n):
        M[i, i] = q
    for i in range(n):
        M[i+n, i+n] = 1
        c = convolution(x^i, publickeyover3)
        for j in range(n):
            M[i+n, j] = c[j]
    M = M.LLL()
    for j in range(2 * n):
        try:
            f = Zx(list(M[j][n:]))
            f3 = invertmodprime(f, 3)
            return (f, f3)
        except:
            pass
    return (f, f)

def toStr(n, base):
    convString = "0123456789ABCDEF"
    if n < base:
        return convString[n]
    else:
        return toStr(n//base,base) + convString[n%base]

n = 7
q = 256
d = 5

publickey, secretkey = keypair()

donald = attack(publickey)

try:
    # m = randommessage()
    flag = "a"
    tmp = []
    for i in flag:
        tmp_m = list(toStr(ord(i), 3))[::-1]
        length = len(tmp_m)
        if length < 6:
            tmp_m.extend((6-length)*['0'])
        tmp.extend(tmp_m)
    initial_m = Zx(tmp)
    for i in range(len(tmp)):
        if tmp[i] == '2':
            tmp[i] = '-1'
    m = Zx(tmp)
    c = encrypt(m, publickey)
    res = decrypt(c, donald)
    assert res == m
    print ("[-] Attack successfully finished.")
except:
    print ("[x] Attack was unsuccessful.")



m_coeff = res.list()
coeff = m_coeff[::-1]

if len(coeff)%6 != 0:
    coeff = (6 - len(coeff)%6)*[0] + coeff

for i in range(len(coeff)):
    if coeff[i] == -1:
        coeff[i] = 2

message = ""
for i in range(1):
    tmp_m = chr(int(''.join(str(i) for i in coeff[i*6:i*6+6]), 3))
    message += tmp_m

print ("[*] Plaintext is {}.".format(message[::-1]))
```

结果如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_49a4fd629ad37cbcc649afa9d78b94d4.png)

### （3） 例题
* 题目

```
n =160
p =3
q =65536
h =10171*x^159 - 24360*x^158 - 8906*x^157 - 20686*x^156 + 13437*x^155 + 9798*x^154 - 3161*x^153 - 19074*x^152 + 24152*x^151 + 1155*x^150 + 14699*x^149 - 4117*x^148 - 16008*x^147 - 12276*x^146 + 381*x^145 - 29393*x^144 - 2191*x^143 + 21846*x^142 + 1248*x^141 + 3284*x^140 - 12680*x^139 - 21480*x^138 + 8649*x^137 + 1339*x^136 + 6235*x^135 - 30436*x^134 - 9229*x^133 - 13940*x^132 - 9793*x^131 + 966*x^130 - 23699*x^129 + 31156*x^128 + 2496*x^127 + 1781*x^126 + 22781*x^125 + 10547*x^124 + 2018*x^123 + 25948*x^122 + 10119*x^121 - 7689*x^120 - 11904*x^119 - 11057*x^118 + 16297*x^117 - 19990*x^116 - 3235*x^115 + 18569*x^114 + 7173*x^113 - 15884*x^112 - 24731*x^111 + 23173*x^110 + 28633*x^109 - 32383*x^108 - 31333*x^107 + 1073*x^106 - 9999*x^105 + 5401*x^104 + 22508*x^103 - 8972*x^102 - 20172*x^101 - 12615*x^100 - 29992*x^99 + 4147*x^98 + 6013*x^97 + 28305*x^96 - 14608*x^95 + 7037*x^94 - 26058*x^93 - 19660*x^92 + 10635*x^91 + 30368*x^90 - 1755*x^89 + 5875*x^88 + 32206*x^87 + 9025*x^86 + 13719*x^85 - 20530*x^84 + 12461*x^83 + 7779*x^82 + 7393*x^81 - 21342*x^80 - 418*x^79 - 24026*x^78 + 19278*x^77 - 26341*x^76 - 14251*x^75 - 18687*x^74 + 7043*x^73 + 11782*x^72 - 5955*x^71 - 21259*x^70 + 8716*x^69 + 3943*x^68 + 12499*x^67 + 16485*x^66 + 31438*x^65 + 13380*x^64 - 28445*x^63 - 31888*x^62 + 3727*x^61 - 21604*x^60 + 11027*x^59 - 26655*x^58 + 31669*x^57 + 4897*x^56 + 8751*x^55 + 13549*x^54 - 17732*x^53 - 21368*x^52 + 17746*x^51 - 18922*x^50 + 15075*x^49 + 29277*x^48 + 4550*x^47 + 5172*x^46 + 3939*x^45 - 2885*x^44 - 12414*x^43 + 28449*x^42 + 718*x^41 - 26248*x^40 + 22333*x^39 - 2486*x^38 - 5105*x^37 + 15900*x^36 - 2999*x^35 + 21728*x^34 - 4434*x^33 + 27659*x^32 + 19110*x^31 + 20626*x^30 + 28754*x^29 + 3456*x^28 - 13710*x^27 - 26417*x^26 - 31900*x^25 - 15286*x^24 + 29499*x^23 - 115*x^22 + 2829*x^21 - 1530*x^20 - 1077*x^19 - 28359*x^18 - 23568*x^17 - 4675*x^16 - 25671*x^15 + 32184*x^14 - 24582*x^13 - 26015*x^12 - 26730*x^11 + 1983*x^10 + 13322*x^9 - 12713*x^8 + 30320*x^7 + 8408*x^6 - 12385*x^5 + 14225*x^4 - 19590*x^3 + 1648*x^2 - 5765*x - 10605
e =-4126*x^159 + 32110*x^158 + 22695*x^157 - 30757*x^156 + 20367*x^155 - 453*x^154 - 13871*x^153 - 23537*x^152 - 21693*x^151 - 17201*x^150 - 20437*x^149 - 12190*x^148 - 4511*x^147 + 13602*x^146 - 9057*x^145 + 15099*x^144 + 18782*x^143 + 24823*x^142 + 1158*x^141 - 12398*x^140 - 4766*x^139 + 19492*x^138 + 18654*x^137 - 3026*x^136 - 25320*x^135 - 20085*x^134 - 32598*x^133 + 12968*x^132 + 18498*x^131 - 8235*x^130 + 22028*x^129 + 11043*x^128 + 13378*x^127 - 17880*x^126 - 17706*x^125 + 25932*x^124 + 21293*x^123 + 13396*x^122 - 6787*x^121 - 8983*x^120 - 30106*x^119 + 22344*x^118 - 3820*x^117 + 28336*x^116 - 7669*x^115 - 21546*x^114 + 12760*x^113 - 30361*x^112 - 21005*x^111 - 20834*x^110 - 29353*x^109 + 4430*x^108 - 19316*x^107 - 31661*x^106 + 7340*x^105 + 203*x^104 + 3712*x^103 + 4758*x^102 - 3055*x^101 - 23828*x^100 - 27376*x^99 - 10030*x^98 + 28830*x^97 + 8641*x^96 + 7581*x^95 - 26755*x^94 + 29794*x^93 + 25240*x^92 - 6560*x^91 - 19259*x^90 + 10331*x^89 + 1896*x^88 + 25504*x^87 + 727*x^86 - 11016*x^85 - 27090*x^84 + 9005*x^83 - 22960*x^82 - 30452*x^81 + 17833*x^80 + 31869*x^79 + 30436*x^78 - 16647*x^77 - 22362*x^76 - 31801*x^75 + 897*x^74 + 17819*x^73 + 29993*x^72 - 29783*x^71 - 14472*x^70 + 27174*x^69 + 19605*x^68 + 13850*x^67 - 24561*x^66 - 20419*x^65 - 29391*x^64 + 8570*x^63 - 814*x^62 + 24626*x^61 - 18486*x^60 - 19433*x^59 - 23857*x^58 - 18977*x^57 + 27027*x^56 + 9325*x^55 + 14269*x^54 + 30166*x^53 + 6364*x^52 + 27842*x^51 - 22210*x^50 + 4196*x^49 + 14784*x^48 - 31125*x^47 - 32399*x^46 + 11458*x^45 - 26458*x^44 + 26129*x^43 + 10576*x^42 - 30095*x^41 - 18843*x^40 + 4172*x^39 - 29128*x^38 - 14177*x^37 - 16219*x^36 + 23148*x^35 - 24828*x^34 - 8287*x^33 - 19887*x^32 - 31485*x^31 - 32499*x^30 - 29689*x^29 - 5414*x^28 - 13435*x^27 - 15201*x^26 + 24378*x^25 - 22085*x^24 + 18116*x^23 - 3533*x^22 + 26465*x^21 + 6590*x^20 - 18678*x^19 - 25146*x^18 - 23266*x^17 + 17573*x^16 + 17149*x^15 - 25867*x^14 + 24994*x^13 - 589*x^12 + 22898*x^11 + 7174*x^10 - 15836*x^9 - 28613*x^8 + 30859*x^7 + 19602*x^6 + 4445*x^5 + 259*x^4 - 5980*x^3 - 2778*x^2 + 24123*x + 23313
```

* 解题代码

和标准的解法略有不同的是这里用的是二进制而不是三进制，稍微改一下代码就行。

```python
Zx.<x> = ZZ[]

# the multiplication operation used in NTRU
def convolution(f, g):
    return (f * g) % (x^n - 1)

def balancedmod(f, q):
    g = list(((f[i] + q//2) % q) - q//2 for i in range(n))
    return Zx(g)

def invertmodprime(f, p):
    T = Zx.change_ring(Integers(p)).quotient(x^n - 1)
    return Zx(lift(1 / T(f)))

def decrypt(ciphertext,secretkey):
    f,f3 = secretkey
    a = balancedmod(convolution(ciphertext, f), q)
    return balancedmod(convolution(a, f3), 3)

def attack(publickey):
    recip3 = lift(1 / Integers(q)(3))
    publickeyover3 = balancedmod(recip3 * publickey, q)
    M = matrix(2 * n)
    for i in range(n):
        M[i,i] = q
    for i in range(n):
        M[i+n, i+n] = 1
        c = convolution(x^i, publickeyover3)
        for j in range(n):
            M[i+n, j] = c[j]
    M = M.LLL()
    for j in range(2 * n):
        try:
            f = Zx(list(M[j][n:]))
            f3 = invertmodprime(f, 3)
            return (f, f3)
        except:
            pass
    return (f, f)

n = 160
q = 65536

publickey = 10171*x^159 - 24360*x^158 - 8906*x^157 - 20686*x^156 + 13437*x^155 + 9798*x^154 - 3161*x^153 - 19074*x^152 + 24152*x^151 + 1155*x^150 + 14699*x^149 - 4117*x^148 - 16008*x^147 - 12276*x^146 + 381*x^145 - 29393*x^144 - 2191*x^143 + 21846*x^142 + 1248*x^141 + 3284*x^140 - 12680*x^139 - 21480*x^138 + 8649*x^137 + 1339*x^136 + 6235*x^135 - 30436*x^134 - 9229*x^133 - 13940*x^132 - 9793*x^131 + 966*x^130 - 23699*x^129 + 31156*x^128 + 2496*x^127 + 1781*x^126 + 22781*x^125 + 10547*x^124 + 2018*x^123 + 25948*x^122 + 10119*x^121 - 7689*x^120 - 11904*x^119 - 11057*x^118 + 16297*x^117 - 19990*x^116 - 3235*x^115 + 18569*x^114 + 7173*x^113 - 15884*x^112 - 24731*x^111 + 23173*x^110 + 28633*x^109 - 32383*x^108 - 31333*x^107 + 1073*x^106 - 9999*x^105 + 5401*x^104 + 22508*x^103 - 8972*x^102 - 20172*x^101 - 12615*x^100 - 29992*x^99 + 4147*x^98 + 6013*x^97 + 28305*x^96 - 14608*x^95 + 7037*x^94 - 26058*x^93 - 19660*x^92 + 10635*x^91 + 30368*x^90 - 1755*x^89 + 5875*x^88 + 32206*x^87 + 9025*x^86 + 13719*x^85 - 20530*x^84 + 12461*x^83 + 7779*x^82 + 7393*x^81 - 21342*x^80 - 418*x^79 - 24026*x^78 + 19278*x^77 - 26341*x^76 - 14251*x^75 - 18687*x^74 + 7043*x^73 + 11782*x^72 - 5955*x^71 - 21259*x^70 + 8716*x^69 + 3943*x^68 + 12499*x^67 + 16485*x^66 + 31438*x^65 + 13380*x^64 - 28445*x^63 - 31888*x^62 + 3727*x^61 - 21604*x^60 + 11027*x^59 - 26655*x^58 + 31669*x^57 + 4897*x^56 + 8751*x^55 + 13549*x^54 - 17732*x^53 - 21368*x^52 + 17746*x^51 - 18922*x^50 + 15075*x^49 + 29277*x^48 + 4550*x^47 + 5172*x^46 + 3939*x^45 - 2885*x^44 - 12414*x^43 + 28449*x^42 + 718*x^41 - 26248*x^40 + 22333*x^39 - 2486*x^38 - 5105*x^37 + 15900*x^36 - 2999*x^35 + 21728*x^34 - 4434*x^33 + 27659*x^32 + 19110*x^31 + 20626*x^30 + 28754*x^29 + 3456*x^28 - 13710*x^27 - 26417*x^26 - 31900*x^25 - 15286*x^24 + 29499*x^23 - 115*x^22 + 2829*x^21 - 1530*x^20 - 1077*x^19 - 28359*x^18 - 23568*x^17 - 4675*x^16 - 25671*x^15 + 32184*x^14 - 24582*x^13 - 26015*x^12 - 26730*x^11 + 1983*x^10 + 13322*x^9 - 12713*x^8 + 30320*x^7 + 8408*x^6 - 12385*x^5 + 14225*x^4 - 19590*x^3 + 1648*x^2 - 5765*x - 10605

secretkey = attack(publickey)

c = -4126*x^159 + 32110*x^158 + 22695*x^157 - 30757*x^156 + 20367*x^155 - 453*x^154 - 13871*x^153 - 23537*x^152 - 21693*x^151 - 17201*x^150 - 20437*x^149 - 12190*x^148 - 4511*x^147 + 13602*x^146 - 9057*x^145 + 15099*x^144 + 18782*x^143 + 24823*x^142 + 1158*x^141 - 12398*x^140 - 4766*x^139 + 19492*x^138 + 18654*x^137 - 3026*x^136 - 25320*x^135 - 20085*x^134 - 32598*x^133 + 12968*x^132 + 18498*x^131 - 8235*x^130 + 22028*x^129 + 11043*x^128 + 13378*x^127 - 17880*x^126 - 17706*x^125 + 25932*x^124 + 21293*x^123 + 13396*x^122 - 6787*x^121 - 8983*x^120 - 30106*x^119 + 22344*x^118 - 3820*x^117 + 28336*x^116 - 7669*x^115 - 21546*x^114 + 12760*x^113 - 30361*x^112 - 21005*x^111 - 20834*x^110 - 29353*x^109 + 4430*x^108 - 19316*x^107 - 31661*x^106 + 7340*x^105 + 203*x^104 + 3712*x^103 + 4758*x^102 - 3055*x^101 - 23828*x^100 - 27376*x^99 - 10030*x^98 + 28830*x^97 + 8641*x^96 + 7581*x^95 - 26755*x^94 + 29794*x^93 + 25240*x^92 - 6560*x^91 - 19259*x^90 + 10331*x^89 + 1896*x^88 + 25504*x^87 + 727*x^86 - 11016*x^85 - 27090*x^84 + 9005*x^83 - 22960*x^82 - 30452*x^81 + 17833*x^80 + 31869*x^79 + 30436*x^78 - 16647*x^77 - 22362*x^76 - 31801*x^75 + 897*x^74 + 17819*x^73 + 29993*x^72 - 29783*x^71 - 14472*x^70 + 27174*x^69 + 19605*x^68 + 13850*x^67 - 24561*x^66 - 20419*x^65 - 29391*x^64 + 8570*x^63 - 814*x^62 + 24626*x^61 - 18486*x^60 - 19433*x^59 - 23857*x^58 - 18977*x^57 + 27027*x^56 + 9325*x^55 + 14269*x^54 + 30166*x^53 + 6364*x^52 + 27842*x^51 - 22210*x^50 + 4196*x^49 + 14784*x^48 - 31125*x^47 - 32399*x^46 + 11458*x^45 - 26458*x^44 + 26129*x^43 + 10576*x^42 - 30095*x^41 - 18843*x^40 + 4172*x^39 - 29128*x^38 - 14177*x^37 - 16219*x^36 + 23148*x^35 - 24828*x^34 - 8287*x^33 - 19887*x^32 - 31485*x^31 - 32499*x^30 - 29689*x^29 - 5414*x^28 - 13435*x^27 - 15201*x^26 + 24378*x^25 - 22085*x^24 + 18116*x^23 - 3533*x^22 + 26465*x^21 + 6590*x^20 - 18678*x^19 - 25146*x^18 - 23266*x^17 + 17573*x^16 + 17149*x^15 - 25867*x^14 + 24994*x^13 - 589*x^12 + 22898*x^11 + 7174*x^10 - 15836*x^9 - 28613*x^8 + 30859*x^7 + 19602*x^6 + 4445*x^5 + 259*x^4 - 5980*x^3 - 2778*x^2 + 24123*x + 23313
m = decrypt(c, secretkey)

m_coeff = m.list()
coeff = m_coeff[::-1]

if len(coeff)%8 != 0:
    coeff = (8 - len(coeff)%8)*[0] + coeff

message = ""
for i in range(20):
    tmp_m = chr(int(''.join(str(i) for i in coeff[i*8:i*8+8]), 2))
    message += tmp_m

print (message[::-1])
```

* 结果

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_3efb455f566c632bad5567c0ec0ba9a8.png)


### （4） 如何防范
从原理上来看，要想有更好的安全性，增大 $n$，减小 $q$，但 $q$ 也不能取得太小。减小 $q$ 会增加解密出错的可能性。

具体如何选择安全的参数，只需要参照官方给出的标准即可。

[NTRU 官网](https://ntru.org/)


## 3. 其他攻击
列出如下，有兴趣可以阅读相关论文。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_32c5ed200698abc7ecf905415a511f87.png)

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_add87a1e95f855874dcda63d8a7cbad7.png)


## 4. 相关题目
### （1） VolgaCTF Quals 2015 - `CPKC`
最简单的类型，甚至不是在多项式上。直接格基规约得到私钥。

建议参考：[hellman's writeup](http://mslc.ctf.su/wp/volgactf-quals-2015-cpkc-crypto-400-writeup/)

### （2） ASIS CTF Quals 2015 - `FalseCrypt`
参数非常小，爆破即可。

建议参考：[writeup](https://github.com/ctfs/write-ups-2015/tree/master/asis-quals-ctf-2015/crypto/falsecrypt)

### （3） PlaidCTF 2016 - `sexec`
构造的格和 `NTRU` 的有点像，是一个 `Ring-LWE` 问题，需要用 `Babai + LLL` 来解 `CVP`。

建议参考：[hellman's writeup](http://mslc.ctf.su/wp/plaidctf-2016-sexec-crypto-300/)

### （4） CTFZone 2019 Quals - `NTRU`
利用 `oracle` 和 `NTRU` 的乘法的性质来解题，和格没什么关系。

建议参考：[hellman's writeup](https://gist.github.com/hellman/07510ee55ca1822de683ad8c9b6f425e)

### （5） 巅峰极客 2019 - `NTRUE`
最简单的类型，甚至不是在多项式上。直接格基规约得到私钥。

不过印象中这是我 18 年加入 CNSS 之后，第一次在国内赛碰到 `NTRU`。

建议参考：[Soreatu's writeup](https://xz.aliyun.com/t/7163#toc-0)

### （6） Codegate CTF 2020 Preliminary - `Polynomial`
没什么好说的，老生常谈，还是用那个经典的格来格基规约。

建议参考：[hellman's writeup](https://gist.github.com/hellman/019de1367f39ba73583d55aaddcbc1f8) 或 [0ops' writeup](https://github.com/0ops/ctfs-2020/blob/master/codegate-quals/Crypto/Polynomials/solve.sage)


### （7） DASCTF 2021 三月赛 - `threshold`
这个比赛两道 `NTRU`，一道就是实数，一道在多项式上。都是用那个经典的格来格基规约。出过无数次的题再拿来出，挺没意思的。


***

> **声明：本文采用 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) 授权。**
> 
> **This work is licensed under a [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).**
