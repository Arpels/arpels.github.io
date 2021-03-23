---
title: '浅谈 Oblivious transfer'
date: 2021-03-18
permalink: /posts/2021/03/oblivious_transfer/
tags:
  - OT
---

最近不经意传输相关的题目在 CTF 里出现了不少次，学习之后记录一下。

*  目录
{:toc}

最近不经意传输相关的题目在 CTF 里出现了不少次，学习之后记录一下。

## 1. Multi-party Computation
* 参考链接：[wikipedia - Federated_Learning](https://en.wikipedia.org/wiki/Federated_learning)

近一年来，联邦学习 `Federated Learning` 风靡一时。它的一项核心技术就是 `Multi-party Computation`，也就是几个互相不信任的人如何合作计算得到一个结果。

实现联邦学习的协议有很多，比较常见的两种是混淆电路 `Garbled Circuit` 和秘密共享 `Secret Sharing`。

`Secret Sharing`，简单来说就是必须要凑齐 $k$ 个人才能得到秘密信息 $D$。一种简单的实现是利用多项式 $D_i = D + a_1i + ... + a_{k-1}i^{k-1}$。每个人拥有一个 $D_i$，当我们凑齐 $k$ 个人时，显然解线性方程组就可以解出 $D$。这是 1979 年由 Shamir 提出的。此外的方案还有很多，如果你记性还可以，你会记得在本科的密码学课程中讲中国剩余定理时，提到过 `Asmuth-Bloom` 方案。

`Garbled Circuit`，混淆电路，作用是让两个人能在互相不知晓对方数据的情况下计算某一能被逻辑电路表示的函数。比如通过一个比大小的逻辑电路，Alice 和 Bob 可以在不让对方知道自己具体成绩的情况下，知道谁考的比较好。而它背后的密码学思想，则是不经意传输。


## 2. Oblivious transfer
* 参考链接：[wikipedia - Oblivious_transfer](https://en.wikipedia.org/wiki/Oblivious_transfer)

简单来说，不经意传输就是发送者把多条信息中的一条发送给了接收者，但他并不知道是哪一条信息。不经意传输的概念一开始是在 1981 年由 Rabin 提出的。

Rabin 的不经意传输协议和 Rabin 密码体制原理是一样的，都是利用了模 $n$ 上的二次剩余。接收者选择 $x$，发送 $x^2 \pmod n$ 给发送者，发送者解二次剩余的结果发送回去。于是这样接收者就有 $1/2$ 的几率分解模数 $n$。而由于有四个根，显然发送者无法知道接收者选择的是不是 $x$。

关于 Rabin 密码体制的题目可以参考 [CNSS 2019 招新赛 - 坤坤的 Oracle](https://zhangche0526.github.io/CNSS%20Recruit%202019%20Crypto%20Writeup/#more) 或 [Hackergame 2020 - 永不溢出的计算器](https://github.com/USTC-Hackergame/hackergame2020-writeups/blob/master/official/%E6%B0%B8%E4%B8%8D%E6%BA%A2%E5%87%BA%E7%9A%84%E8%AE%A1%E7%AE%97%E5%99%A8/README.md)。

不经意传输有很多实现方式，并不局限于 Rabin 这一种。

比如 wikipedia 给出了基于 RSA 的一种实现，如下图所示。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_6743416aaa4140af9fb72a7a10df98f3.png)

比如知乎上 [某篇文章](https://zhuanlan.zhihu.com/p/126396795) 提到了另一种实现。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_ed68994d4514065eb982a02c9a1db497.png)


## 3. 相关题目：Hackergame 2020 不经意传输
* 参考链接：[官方 writeup](https://github.com/USTC-Hackergame/hackergame2020-writeups/blob/master/official/%E4%B8%8D%E7%BB%8F%E6%84%8F%E4%BC%A0%E8%BE%93/README.md)

### （1）题目
```python
from Crypto.PublicKey import RSA
from random import SystemRandom
import os


if __name__ == "__main__":
    random = SystemRandom()

    key = RSA.generate(1024)
    print("n =", key.n)
    print("e =", key.e)

    m0 = int.from_bytes(os.urandom(64).hex().encode(), "big")
    m1 = int.from_bytes(os.urandom(64).hex().encode(), "big")

    x0 = random.randrange(key.n)
    x1 = random.randrange(key.n)
    print("x0 =", x0)
    print("x1 =", x1)

    v = int(input("v = "))
    m0_ = (m0 + pow(v - x0, key.d, key.n)) % key.n
    m1_ = (m1 + pow(v - x1, key.d, key.n)) % key.n
    print("m0_ =", m0_)
    print("m1_ =", m1_)

    guess0 = int(input("m0 = "))
    guess1 = int(input("m1 = "))
    if guess0 == m0:
        print(open("flag1").read())
        if guess1 == m1:
            print(open("flag2").read())
    else:
        print("Nope")
```

### （2）思路
看完代码，这题显然是用了 wikipedia 上的那个 RSA 的实现。

我们能控制的部分只有 $v$，而我们需要求得的是两个 $m$。

求得其中一个 $m$ 是很容易的，只需要 $v = x_0$。但如果要同时求出两个 $m$，就需要巧妙地构造 $v$了。

如何构造？官方 writeup 里给出了很巧妙的构造：

$$
v = \frac{scale^e*x_0 - x_1}{scale^e - 1}
$$

这样我们就能让 $k_1 = scale*k_0$。

因此，$scale \cdot m_0^{'} - m_1^{'} = m_0 \cdot scale - m_1$。

首先，如果恰巧随机数 $m_0$ 和 $m_1$ 都小于 $scale$，那么直接可以从 $m_0 \cdot scale - m _1$ 将两者求出。

其次，如果 $scale$ 取值不能取得太大，而导致两个 $m$ 总是比它大，那我们只能老老实实穷举可能的低字节，再向高字节搜索。

### （3）代码
直接贴官方的了：

```python
from pwn import *
import time
from tqdm import tqdm

threshold = 24

while True:
    re = remote('127.0.0.1', 10031)

    re.recvuntil("token: ")
    re.sendline('这里填写你的 token')

    re.recvuntil("n = ")
    n = int(re.recvline())
    re.recvuntil("e = ")
    e = int(re.recvline())
    re.recvuntil("x0 = ")
    x0 = int(re.recvline())
    re.recvuntil("x1 = ")
    x1 = int(re.recvline())

    cs = "0123456789abcdef"
    bits = 1024
    scale = 19

    table = [[] for _ in range(256)]
    for i in cs:
        for j in cs:
            v = ord(i) + scale * ord(j)
            table[v % 256].append((i, j, v // 256))

    v = (x0 + pow(scale, e, n) * x1) * pow(1 + pow(scale, e, n), -1, n) % n

    re.recvuntil("v = ")
    re.sendline(str(v))

    re.recvuntil("m0_ = ")
    m0_ = int(re.recvline())
    re.recvuntil("m1_ = ")
    m1_ = int(re.recvline())

    start_r = (m0_ + m1_ * scale) % n
    while True:
        cands = {start_r: ([], 1)}
        for b in range(bits // 8):
            ncands = {}
            for r, (m0, cnt) in cands.items():
                for i, j, carry in table[r % 256]:
                    new_r = r // 256 - carry
                    if new_r < 0:
                        continue
                    if new_r not in ncands:
                        ncands[new_r] = [], 0
                    l, old_cnt = ncands[new_r]
                    l.append((i, m0))
                    ncands[new_r] = l, old_cnt + cnt
            cands = ncands
            if not cands:
                break
        if cands:
            total = sum(cnt for _, (_, cnt) in cands.items())
            print(total, total.bit_length())
            break
        start_r += n
    if total < 2 ** threshold:
        break
    else:
        print("Too long")
        re.close()
        time.sleep(5)

def generate(l):
    for c, suffix in l:
        if suffix:
            for s in generate(suffix):
                yield c + s
        else:
            yield c

pbar = tqdm(total=total, position=0, leave=True)

target = (v - x0) % n

for _, (root, _) in cands.items():
    for m0 in generate(root):
        m0 = int.from_bytes(m0.encode(), 'big')
        if pow(m0_ - m0, e, n) == target:
            m1 = (start_r - m0) // scale
            m1t = m1.to_bytes(bits // 8, 'big')
            re.recvuntil("m0 = ")
            re.sendline(str(m0))
            re.recvuntil("m1 = ")
            re.sendline(str(m1))
            re.interactive()
            exit()
        pbar.update(1)
pbar.close()
```


## 4. 相关题目：PWNHUB 2020 BabyOT
* 参考链接：[官方 writeup](https://mp.weixin.qq.com/s/eAJWraah9OOgJfZOhm4Sqg)

这题和上一题的出题人都是中科大的师傅。

### （1）题目
```python
#!/usr/bin/env -S python3 -u

import os
import string
from Crypto.PublicKey import RSA
from Crypto.Util.number import bytes_to_long
from random import SystemRandom


def getkey():
    if os.path.isfile("key.pem"):
        with open("key.pem", "rb") as f:
            key = RSA.importKey(f.read())
    else:
        key = RSA.generate(2048)
        with open("key.pem", "wb") as f:
            f.write(key.exportKey("PEM"))
    return key


def random_str(n):
    return "".join([random.choice(string.ascii_letters) for _ in range(n)])


if __name__ == "__main__":
    random = SystemRandom()

    key = getkey()

    print(key.n)
    print(key.e)

    while True:
        msg0 = bytes_to_long(random_str(255).encode())# 2048 // 8 - 1
        msg1 = bytes_to_long(random_str(255).encode())

        x0 = random.randrange(key.n) # 0 - n-1
        x1 = random.randrange(key.n)

        print(x0)
        print(x1)

        v = int(input())
        print((msg0 + pow(v - x0, key.d, key.n)) % key.n)
        print((msg1 + pow(v - x1, key.d, key.n)) % key.n)

        guess0 = int(input())
        guess1 = int(input())
        if guess0 == msg0:
            print("You are on the half way of success, work harder!")
            if guess1 == msg1:
                print(open('flag').read())
                exit()

```
### （2）思路
第一种思路，是可以用不经意传输那道题的做法来解。

第二种思路，也就是官方给出的，发现题目中重复使用了 RSA key，于是保持原来连接的同时建立一条新连接以使 key 相同。通过巧妙构造 $v = x_0 - x_1 + x_0^{'}$ 使得每次返回结果都是 $k_0+m_0^{'}$，其中 $k_0$ 保持不变，而 $m_0^{'}$ 每次随机生成。于是尝试多次我们就可以得到一组 $k_0 + m_{0,i}$。而我们每次根据 $m_{0,i}$ 中每个字节可能的范围，可以把 $k_0$ 中每个字节不可能的范围筛掉，使一定次数后 $k_0$ 每个字节情况数小于一个较小的值，然后我们可以对其进行穷举。

### （3）代码
官方 writeup 给的代码如下：

```python
from pwn import *
from Crypto.Util.number import long_to_bytes, bytes_to_long
import string
from itertools import product

# context.log_level = 'debug'

host = "39.107.33.72"
port = 10001

def dec(c):
   length = 2048 // 8
   candidates = [set(range(256)) for i in range(length)]
   charmin = ord(min(string.ascii_letters))
   charmax = ord(max(string.ascii_letters))

   r1 = remote(host, port)
   n = int(r1.recvline())
   e = int(r1.recvline())

   cnt = 0
   while True:
       x0 = int(r1.recvline())
       x1 = int(r1.recvline())
       r1.sendline(str((x0 + c) % n))
       m0 = int(r1.recvline())
       m1 = int(r1.recvline())
       r1.sendline("0")
       r1.sendline("0")

       b = long_to_bytes(m0, length)[::-1]
       for i in range(length):
           if i == 0:
               carrymin = 0
               carrymax = 0
           else:
               if min(candidates[i - 1]) + charmin >= 256:
                   carrymin = 1
               else:
                   carrymin = 0
               if max(candidates[i - 1]) + charmax < 256:
                   carrymax = 0
               else:
                   carrymax = 1

           if i == length - 1:
               start = b[i] - carrymax
               end = b[i] - carrymin
           else:
               start = b[i] - charmax - carrymax
               end = b[i] - charmin - carrymin
           possible = set(x % 256 for x in range(start, end + 1))
           candidates[i] &= possible

       cnt += 1
       possibles = 1
       for i in range(length):
           possibles *= len(candidates[i])

       print(cnt, [len(c) for c in candidates], possibles.bit_length())

       if possibles < 50000:
           print("Trying all possibilities")
           for b in product(*candidates):
               m = bytes_to_long(bytes(b[::-1])) % n
               if pow(m, e, n) == c:
                   return m
           print("Failed")
           exit()


if __name__ == "__main__":
   r0 = remote(host, port)
   n = int(r0.recvline())
   e = int(r0.recvline())
   x0 = int(r0.recvline())
   x1 = int(r0.recvline())
   r0.sendline(str(x0))
   m0 = int(r0.recvline())
   m1 = int(r0.recvline())
   r0.sendline(str(m0))
   r0.sendline(str((m1 - dec(x0 - x1)) % n))
   r0.interactive()
```

## 5. 相关题目：D3CTF 2021 EasyCurve
* 参考链接：[官方 writeup](https://github.com/shal10w/d3ctf2021_EasyCurve)

### （1）题目
```python
from sage.all import *
import random
class MyCurve:
    def __init__(self, p, D , u):
        self.p = p
        self.R = GF(self.p)
        self.u = self.R(u)
        self.D = self.R(D)
        self.zero = (u, 0)

    def check_point(self, P):
        x, y = P
        return (x**2 - self.D*y**2 - self.u**2 == 0)

    def add(self, P1, P2):
        x1, y1 = P1
        x2, y2 = P2
        if x1 - self.u == 0:
            return P2
        elif x2 - self.u == 0:
            return P1
        m1 = y1 / (x1 - self.u)
        m2 = y2 / (x2 - self.u)

        m3 = (self.D * m1 * m2 + 1) /((m1 + m2) * self.D)
        
        x3 = self.u * (self.D * m3 **2 + 1) / (self.D * m3 **2 - 1)
        y3 = 2 * self.u * m3 / (self.D * m3 **2 - 1)
        
        P3 = (int(x3), int(y3))
        assert self.check_point(P3)
        return P3

    def mul(self, n, P):
        Q = self.zero

        while n:
            if n & 1:
                Q = self.add(Q, P)
            P = self.add(P, P)
            n >>= 1
        return Q

    def lift_x(self, x):
        y2 = (x**2 - self.u**2) / self.D
        y = int(y2.sqrt())
        P = (x, y)
        assert self.check_point(P)
        return P

    def getPoint(self):
        while 1:
            x = random.randint(1 , self.p)
            try:
                return self.lift_x(x)
            except:
                pass
```

```python
import socketserver
from Crypto.PublicKey import RSA
from Crypto.Util.number import getPrime , bytes_to_long
from Curve import MyCurve
from hashlib import sha256
import os
import string
import random
import signal
from secret import flag

BIT = 2048
p = 9688074905643914060390149833064012354277254244638141162997888145741631958242340092013958501673928921327767591959476890238698855704376126231923819603296257

class Task(socketserver.BaseRequestHandler):

    def proof_of_work(self):
        random.seed(os.urandom(8))
        proof = ''.join([random.choice(string.ascii_letters+string.digits) for _ in range(20)])
        _hexdigest = sha256(proof.encode()).hexdigest()
        self.send(f"sha256(XXXX+{proof[4:]}) == {_hexdigest}".encode())
        self.send(b'Give me XXXX: ')
        x = self.recv()
        if len(x) != 4 or sha256(x+proof[4:].encode()).hexdigest() != _hexdigest:
            self.send('wrong')
            return False
        return True

    def recv(self):
        data = self.request.recv(1024)
        return data.strip()

    def send(self, msg, newline=True):
        if isinstance(msg , bytes):
            msg += b'\n'
        else:
            msg += '\n'
            msg = msg.encode()
        self.request.sendall(msg)

    def key_gen(self , bit):
        key = RSA.generate(bit)
        return key

    def ot(self , point):
        x , y = point
        random.seed(os.urandom(8))

        key = self.key_gen(BIT)
        self.send('n = ' + str(key.n))
        self.send('e = ' + str(key.e))
        x0 = random.randint(1 , key.n)
        x1 = random.randint(1 , key.n)
        self.send("x0 = " + str(x0))
        self.send("x1 = " + str(x1))

        self.send("v = ")
        v = int(self.recv())
        m0_ = (x + pow(v - x0, key.d, key.n)) % key.n
        m1_ = (y + pow(v - x1, key.d, key.n)) % key.n
        self.send("m0_ = " + str(m0_))
        self.send("m1_ = " + str(m1_))
    def handle(self):
        signal.alarm(180)
        if not self.proof_of_work():
            return 0
        e = bytes_to_long(os.urandom(32))
        u = random.randint(1 , p)
        D = random.randint(1 , p)
        curve = MyCurve(p , D , u)
        self.send('p = ' + str(p))
        self.send('D = ' + str(D))
        for i in range(3):
            G = curve.getPoint()
            self.ot(G)
            P = curve.mul(e , G)
            self.ot(P)
            self.send("do you know my e?")
            guess = int(self.recv())
            if guess == e:
                self.send("oh no!")
                self.send(flag)
                return 0
            else:
                self.send("Ha, I know you can't get it.")

class ForkedServer(socketserver.ForkingMixIn, socketserver.TCPServer):
    pass

if __name__ == "__main__":
    HOST, PORT = '0.0.0.0', 10000
    server = ForkedServer((HOST, PORT), Task)
    server.allow_reuse_address = True
    server.serve_forever()
 
```


### （2）思路
题目给了一条曲线，以及随机生成两个点满足 $P = eG$，要求我们解这个 `ECDLP` 解三次。而这两个点的信息是通过不经意传输告诉我们的。

显然这里因为只需要三次，那么有不小的概率随机生成的点的坐标是小于我们选择的很大的 $scale$ 的，因此我们可以用不经意传输的那个方法获取这两个点，然后解 `ECDLP` 即可。

由于关于这个 `ECDLP` 的群的阶 $p-1$ 光滑，因此使用 `Pohlig–Hellman algorithm` 即可求解。

* 相关链接：[wikipedia - Pohlig–Hellman algorithm](https://en.wikipedia.org/wiki/Pohlig%E2%80%93Hellman_algorithm)

上面的这个方法是 `Kap0K`的师傅发在安全客上的非预期解。官方给出的解法是利用一篇论文里给出的映射，将 `ECDLP` 转化为模 $p$ 的 `DLP` 问题。

* 相关论文：[A PUBLIC KEY CRYPTOSYSTEM BASED ON PELL EQUATION](https://eprint.iacr.org/2006/191.pdf)

### （3）代码
非预期解的代码：

```python
import string, gmpy2
from hashlib import sha256
from pwn import *
context.log_level = "debug"

dic = string.ascii_letters + string.digits

def solvePow(prefix,h):
    for a1 in dic:
        for a2 in dic:
            for a3 in dic:
                for a4 in dic:
                    x = a1 + a2 + a3 + a4
                    proof = x + prefix.decode("utf-8")
                    _hexdigest = sha256(proof.encode()).hexdigest()
                    if _hexdigest == h.decode("utf-8"):
                        return x

def getData():
    r.recvuntil("n = ")
    n = int(r.recvuntil("\n", drop = True))
    r.recvuntil("e = ")
    e = int(r.recvuntil("\n", drop = True))
    r.recvuntil("x0 = ")
    x0 = int(r.recvuntil("\n", drop = True))
    r.recvuntil("x1 = ")
    x1 = int(r.recvuntil("\n", drop = True))
    offset = 2 << 1024
    offset_e = int(pow(offset, e, n))
    v = ((offset_e * x0 - x1) * gmpy2.invert(offset_e - 1, n)) % n
    r.sendlineafter("v = ",str(v))
    r.recvuntil("m0_ = ")
    m0 = int(r.recvuntil("\n", drop = True))
    r.recvuntil("m1_ = ")
    m1 = int(r.recvuntil("\n", drop = True))
    m = (m0 * offset - m1) % n
    x = m // offset + 1
    y = x * offset - m
    return x,y

r = remote("47.100.50.252",10000)
r.recvuntil("sha256(XXXX+")
prefix = r.recvuntil(") == ", drop = True)
h = r.recvuntil("\n", drop = True)
result = solvePow(prefix,h)
r.sendlineafter("Give me XXXX: \n",result)

r.recvuntil("p = ")
r.recvuntil("\n", drop = True)
r.recvuntil("D = ")
D = int(r.recvuntil("\n", drop = True))

Gx,Gy = getData()
Px,Py = getData()

with open("data.txt","wb") as f:
    f.write(str(D).encode() + b"\n")
    f.write(str(Gx).encode() + b"\n")
    f.write(str(Gy).encode() + b"\n")
    f.write(str(Px).encode() + b"\n")
    f.write(str(Py).encode() + b"\n")

s = process(argv=["sage", "exp.sage"])
e = int(s.recv())
s.close()
r.sendline(str(e))

r.interactive()
```
```python
# exp.sage
load("Curve.sage")

p = 9688074905643914060390149833064012354277254244638141162997888145741631958242340092013958501673928921327767591959476890238698855704376126231923819603296257
F = GF(p)
fac = [2^21,3^10,7^4,11,13^2,17,19,29,31,37,43^3,47,71,83,89,97,223,293,587,631,709,761,1327,1433,1733,1889,2503,3121,6043,6301,49523,98429,140683,205589,1277369,1635649,5062909,45698189,67111151,226584089,342469397]

def bsgs(g, y, p):
    m = int(ceil(sqrt(p - 1)))
    S = {}
    point = (u,0)
    for i in range(m):
        point = curve.add(point,g)
        pointg = point[0] << 800 | point[1]
        S[pointg] = i

    gs = curve.mul(m,g)
    for i in range(m):
        pointy = y[0] << 800 | y[1]
        if pointy in S:
            return S[pointy] - i * m + 1
        y = curve.add(y,gs)
    return None

def Pohlig_Hellman(G,P):
    ea = []
    na = []
    for i in range(len(fac)):
        c = fac[i]
        n = (p - 1) // c
        gi = curve.mul(n, G)
        yi = curve.mul(n, P)
        ei = bsgs(gi,yi,c)
        ea.append(ei%c)
        na.append(c)
    ee = crt(ea,na)
    return ee

data = open("data.txt","rb").read().decode("utf-8")
data = data.split("\n")

D = int(data[0])
Gx = int(data[1])
Gy = int(data[2])
Px = int(data[3])
Py = int(data[4])

G = (F(Gx),F(Gy))
P = (F(Px),F(Py))

u2 = (Gx ^ 2 - D * Gy ^ 2)
u2 = F(u2)
u = int(u2.sqrt())
curve = MyCurve(p , D , u)
e = Pohlig_Hellman(G,P)
e %= p - 1
print(e)
```
官方 writeup 的代码：
```python
from sage.all import *
from pwn import *
from Curve import MyCurve
from pwnlib.util.iters import mbruteforce
import string
from hashlib import sha256
from Crypto.Util.number import *
#context.log_level = 'debug'
port = 10000
ip = '47.100.50.252'
io = remote(ip , port)
p = 9688074905643914060390149833064012354277254244638141162997888145741631958242340092013958501673928921327767591959476890238698855704376126231923819603296257
R = GF(p)
def proof_of_work(p):
    p.recvuntil("XXXX+")
    suffix = p.recv(16).decode("utf8")
    p.recvuntil("== ")
    cipher = p.recvline().strip().decode("utf8")
    proof = mbruteforce(lambda x: sha256((x + suffix).encode()).hexdigest() ==
                        cipher, string.ascii_letters + string.digits, length=4, method='fixed')
    p.sendlineafter("Give me XXXX: ", proof) 


def hackOT(d):
    io.recvuntil('n = ')
    n = int(io.recvline()[:-1])
    io.recvuntil('e = ')
    e = int(io.recvline()[:-1])
    io.recvuntil('x0 = ')
    x0 = int(io.recvline()[:-1])
    io.recvuntil('x1 = ')
    x1 = int(io.recvline()[:-1])
    v = (x0 + pow(-d , e, n) * x1) * inverse(1 + pow(-d , e , n) , n) % n
    io.sendline(str(v))
    io.recvuntil("m0_ = ")
    m0_ = int(io.recvline()[:-1])
    io.recvuntil("m1_ = ")
    m1_ = int(io.recvline()[:-1])
    res = (m0_ - d * m1_) % n - n
    return R(res)
def getd():
    io.recvuntil('D = ')
    D = R(int(io.recvline()[:-1]))
    if pow(D , (p-1)//2 , p) != 1:
        return -1
    else:
        d = int(D.sqrt(0))
        assert pow(d ,2 , p) == D
        return d   


while 1:
    proof_of_work(io)
    d = getd()
    if d == -1:
        io.close()
        io = remote(ip , port)
        continue
    a1 = hackOT(d)
    a2 = hackOT(d)
    io.recvuntil("do you know my e?")
    io.sendline('0')
    io.recvuntil("I know you can't get it.")
    b1 = hackOT(d)
    b2 = hackOT(d)
    e = discrete_log(b2/a2,b1 / a1)
    print(e)
    io.recvuntil("do you know my e?")
    io.sendline(str(e))
    io.interactive()
    break
```

## 6. 相关题目：Zer0pts CTF 2021 OT or not OT
* 参考链接：[hellman's writeup](https://affine.group/writeup/2021-01-Zer0pts#ot-or-not-ot)

* 参考链接：[Joseph's writeup](https://www.josephsurin.me/posts/2021-03-07-zer0pts-ctf-2021-crypto-writeups#ot-or-not-ot)

### （1）题目
```python
import os
import signal
import random
from base64 import b64encode
from Crypto.Util.number import getStrongPrime, bytes_to_long
from Crypto.Util.Padding import pad
from Crypto.Cipher import AES
from flag import flag

p = getStrongPrime(1024)

key = os.urandom(32)
iv = os.urandom(AES.block_size)
aes = AES.new(key=key, mode=AES.MODE_CBC, iv=iv)
c = aes.encrypt(pad(flag, AES.block_size))

key = bytes_to_long(key)
print("Encrypted flag: {}".format(b64encode(iv + c).decode()))
print("p = {}".format(p))
print("key.bit_length() = {}".format(key.bit_length()))

signal.alarm(600)
while key > 0:
    r = random.randint(2, p-1)
    s = random.randint(2, p-1)
    t = random.randint(2, p-1)
    print("t = {}".format(t))

    a = int(input("a = ")) % p
    b = int(input("b = ")) % p
    c = int(input("c = ")) % p
    d = int(input("d = ")) % p
    assert all([a > 1 , b > 1 , c > 1 , d > 1])
    assert len(set([a,b,c,d])) == 4

    u = pow(a, r, p) * pow(c, s, p) % p
    v = pow(b, r, p) * pow(c, s, p) % p
    x = u ^ (key & 1)
    y = v ^ ((key >> 1) & 1)
    z = pow(d, r, p) * pow(t, s, p) % p

    key = key >> 2

    print("x = {}".format(x))
    print("y = {}".format(y))
    print("z = {}".format(z))
```

### （2）思路
题目名字虽然有不经意传输，但解题思路其实和它关系不是很大。

题目给我们条件如下，要求我们解出 $k_1, k_2$：

$$
\begin{cases}
x \equiv a^rc^s \bigoplus k_1 \\
y \equiv b^rc^s \bigoplus k_2 \\
z \equiv d^rt^s 
\end{cases}
\pmod p
$$

我们的思路是想办法消去 $r,s$ 的影响，以方便爆破 $k$。

为了消去 $r$ 的影响，需要 $abd = 1$。

为了消去 $s$ 的影响，需要 $c^2t = 1$。

因此不妨让 $(a, b, c, d) = (m, m^{-1}, t, -1)$，则有：

$$
\begin{cases}
x \equiv m^rt^s \bigoplus k_1 \\
y \equiv m^{-r}t^s \bigoplus k_2 \\
z \equiv (-1)^{r}t^s 
\end{cases}
\pmod p
$$

于是可以推导出关系如下：

$$
xy \equiv
	\begin{cases}
	z^2 \pmod p , & \text{if} (k_1, k_2)=(0,0) \\
	z^2 \pm y \pmod p , & \text{if} (k_1, k_2)=(1,0) \\
	z^2 \pm x \pmod p , & \text{if} (k_1, k_2)=(0,1) \\
	\text{others} , & \text{if} (k_1, k_2)=(1,1)
	\end{cases}
$$

根据此关系我们可以得到每轮的 $k_0, k_1$，从而恢复 $key$。

### （3）代码
```python
import os
os.environ['PWNLIB_NOTERM'] = 'True'
from pwn import *
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
from Crypto.Util.number import long_to_bytes
from base64 import b64decode
from tqdm import tqdm

proof.arithmetic(False)

conn = remote('crypto.ctf.zer0pts.com', 10130)
ciphertext = b64decode(conn.recvline().decode().split('flag: ')[1])
print('[!] ciphertext:', ciphertext)
p = int(conn.recvline().decode().split('p = ')[1])
key_bitlength = int(conn.recvline().decode().split(' = ')[1])
F = GF(p)

m = 2 
a = m
b = F(m)^-1
d = F(-1)
results = []
for i in tqdm(range(ceil(key_bitlength/2))):
    t = F(int(conn.recvline().decode().split('t = ')[1]))
    c = t
    conn.sendlineafter('a = ', str(a))
    conn.sendlineafter('b = ', str(b))
    conn.sendlineafter('c = ', str(c))
    conn.sendlineafter('d = ', str(d))
    x = int(conn.recvline().decode().split('x = ')[1])
    y = int(conn.recvline().decode().split('y = ')[1])
    z = int(conn.recvline().decode().split('z = ')[1])
    results.append((x, y, z))

key = ''
for x,y,z in results:
    xy = F(x*y)
    zz = F(z^2)
    if xy == zz:
        key = '00' + key
    elif xy - zz == y or xy - zz == -y:
        key = '01' + key
    elif xy - zz == x or xy - zz == -x:
        key = '10' + key
    else:
        key = '11' + key

print('[+] key:', key)
key = long_to_bytes(int(key, 2))
aes = AES.new(key=key, mode=AES.MODE_CBC, iv=ciphertext[:16])
flag = aes.decrypt(ciphertext[16:])
print('[*] flag:', flag)
```

## 7. 相关题目：0CTF/TCTF 2020 Finals Oblivious
* 参考链接：[官方 writeup](https://github.com/Septyem/My-Public-CTF-Challenges/tree/master/0ctf-tctf-2020-final/Oblivious)

* 参考链接：[cr0wn's writeup](https://cr0wn.uk/2020/0ctf-oblivious/)

* 参考链接：[Ashen's writeup](https://www.cnblogs.com/ashen0/p/TCTF-20-Finals-writeup.html)

### （1）题目
```python
import os
from hashlib import sha256
import SocketServer
from random import seed,randint,choice
from Crypto.Util.number import getStrongPrime, inverse
from flag import flag
import hashlib
import string

BITS = 2048
assert flag.startswith('flag{') and flag.endswith('}')
assert len(flag) < BITS/8
padding = os.urandom(BITS/8-len(flag))
flagnum = int((flag+padding).encode('hex'), 16)

class Task(SocketServer.BaseRequestHandler):
    def pow(self):
        res = "".join([choice(string.ascii_letters) for i in range(20)])
        self.request.sendall("md5(??????+%s).startswith('000000')" % (res))
        pre = self.recvn(6)
        return hashlib.md5(pre+res).hexdigest().startswith("000000")

    def genkey(self):
        '''
        NOTICE: In remote server this key is generated like below but hardcoded, since genkey is time/resource consuming
        and I don't want to add annoying PoW, especially for a final event.
        This function is kept for your local testing.
        '''
        p = getStrongPrime(BITS/2)
        q = getStrongPrime(BITS/2)
        self.p = p
        self.q = q
        self.n = p*q
        self.e = 0x10001
        self.d = inverse(self.e, (p-1)*(q-1))

    def genmsg(self):
        '''
        simply xor looks not safe enough. what if we mix adjacent columns?
        '''
        m0 = randint(1, self.n-1)
        m0r = (((m0&1)<<(BITS-1)) | (m0>>1))
        m1 = m0^m0r^flagnum
        return m0, m1

    def recvn(self, sz):
        '''
        add a loop in recv to avoid truncation by network issues
        '''
        r = sz
        res = ''
        while r>0:
            res += self.request.recv(r)
            if res.endswith('\n'):
                r = 0
            else:
                r = sz - len(res)
        res = res.strip()
        return res

    def handle(self):
        seed(os.urandom(0x20))
        if not self.pow():
            self.request.close()
            return
        self.genkey()
        self.request.sendall("n = %d\ne = %d\n" % (self.n, self.e))
        try:
            while True:
                self.request.sendall("--------\n")
                m0, m1 = self.genmsg()
                x0 = randint(1, self.n-1)
                x1 = randint(1, self.n-1)
                self.request.sendall("x0 = %d\nx1 = %d\n" % (x0, x1))
                v = int(self.recvn(BITS/3))
                k0 = pow(v^x0, self.d, self.n)
                k1 = pow(v^x1, self.d, self.n)
                self.request.sendall("m0p = %d\nm1p = %d\n" % (m0^k0, m1^k1))
        finally:
            self.request.close()

class ForkedServer(SocketServer.ForkingTCPServer, SocketServer.TCPServer):
    pass

if __name__ == "__main__":
    HOST, PORT = '0.0.0.0', 10002
    server = ForkedServer((HOST, PORT), Task)
    server.allow_reuse_address = True
    server.serve_forever()
```

### （2）思路
思路一：

这个思路比较奇特，来自于 Ashen（貌似是六室的大哥）。

题目考察的是不经意传输，但作者发现由于题目提供了大量的数据，因此代码里用的 `random.randint()` 这种基于梅森旋转来生成随机数的函数，就变得可以预测了。

与之解法类似的一题：[BalsnCTF 2019 - unpredictable](https://github.com/sasdf/ctf/tree/master/tasks/2019/BalsnCTF/crypto/unpredictable)

作者原文如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_2f9731141a55e169a4acd974b5421e13.png)

思路二：

`the cr0wn` 的思路。

由题目可知，$m_1' = m_1 \oplus D(v \oplus x_1) = m_0 \oplus ROR(m_0) \oplus flag \oplus D(v \oplus x_1)$。

因此可以构造 `single-bit adaptive chosen-ciphertext RSA oracle`，然后使用 `the Bleichenbacher attack`。

* 相关论文：[the Bleichenbacher attack](http://archiv.infsec.ethz.ch/education/fs08/secsem/bleichenbacher98.pdf)

oracle 的构造利用了汉明重量，`Hamming Weight`，又叫 `popcount`。

可知 $pp(m_1'(i, x_1(i) \oplus y)) = pp(m_0(i) \oplus ROR(m_0(i)) \oplus flag \oplus D(y)) = pp(flag \oplus D(y))$。

作者核心思想如下：

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_16ebdd17337d8e78def769e750e7550e.png)

体现在代码里，就是我们可以借助汉明重量 $pp(D(y))$ 和 $pp(D(y \cdot 2^e))$ 的这个关系，来缩小搜索明文的范围。

```python
upper = n
lower = 0
ct = x0^x1
for i in range(2048):
    print i,'/ tot'
    power = pow(2,(i+1),n)
    _ct = (pow(power, e, n)*ct)%n
    lsb = oracle(_ct)
    if lsb == 0:
        upper = (upper + lower)/2
    else:
        lower = (upper + lower)/2
    if upper < lower:
        break

found = False
for i in range(-1000, 1000):
    if pow(upper+i, e, n) == ct%n:
        found = True
        pt = (upper+i)%n
        break
```


思路三：

官方思路，只有几行，我没太看明白是什么意思。

但看解题代码，貌似和思路二差不多？

```
The original idea is the MSB of randint(0,n) can be biased for most n in RSA. And if you collect sufficient data it can be used as an oracle for this bit under certain condition.

At this challenge, the LSB is also mixed with MSB, which reduces to usual parity oracle of RSA.
```


### （3）代码
官方：

```python
from pwn import *
from math import log,exp

#c = remote('127.0.0.1', 10001)
c = remote('chall.0ops.sjtu.edu.cn', 10002)
c.recvuntil('n = ')
n = int(c.recvline())
ratio = exp(log(2**2047)-log(n))
if ratio<0.7:
    assert False, "bad n"
c.recvuntil('e = ')
e = int(c.recvline())

c.recvuntil('x0 = ')
x0 = int(c.recvline())
c.recvuntil('x1 = ')
x1 = int(c.recvline())

c.sendline(str(x0))

c.recvuntil('m0p = ')
m0p = int(c.recvline())
c.recvuntil('m1p = ')
m1p = int(c.recvline())

def oracle(ct):
    msb = 2**2047
    stats = [0,0]
    cnt = 0
    while cnt < 40 or abs(stats[0]-stats[1])<cnt*0.4*0.6:
        cnt += 1
        c.recvuntil('x0 = ')
        x0 = int(c.recvline())
        c.recvuntil('x1 = ')
        x1 = int(c.recvline())
        c.sendline(str(ct^x0))

        c.recvuntil('m0p = ')
        m0p = int(c.recvline())
        c.recvuntil('m1p = ')
        m1p = int(c.recvline())
        if m1p&msb != 0:
            val = 1
        else:
            val = 0
        val ^= (m0p&1)
        stats[val] += 1
    print stats
    if stats[0] >= stats[1]:
        return 0
    else:
        return 1

upper = n
lower = 0
ct = x0^x1
for i in range(2048):
    print i,'/ tot'
    power = pow(2,(i+1),n)
    _ct = (pow(power, e, n)*ct)%n
    lsb = oracle(_ct)
    if lsb == 0:
        upper = (upper + lower)/2
    else:
        lower = (upper + lower)/2
    if upper < lower:
        break

found = False
for i in range(-1000, 1000):
    if pow(upper+i, e, n) == ct%n:
        found = True
        pt = (upper+i)%n
        break

assert found

m1=m1p^pt
m0=m0p
m0r = (((m0&1)<<2047) | (m0>>1))
flagnum = m0^m0r^m1
print hex(flagnum)
```

Ashen 预测随机数：

```python
from pwn import *
import time
import hashlib
import string
import os
from Crypto.Util.number import long_to_bytes

BITS = 2048
k = BITS
n = 19537672993921510910953800210784804463906011801348944134382259677098515591468020354186917058659291508782207012207322759124661039955163907358060182234684997838303129553612091765074441858018987479764884871179221087985572587060253497705505070405152688906445392906317500619032806029443372743631700328868047923922273766615053104519261361069287938437682793053653603535934093590530631032737414606160770584158459833468735707963661279153502660376802573242852076645275762942169376811866451825822378845156284080472507828885812988167574335939801962133577967403542809570426652088681810875263525518970234197229449528868110799345007
e = 65537

from mt19937predictor import MT19937Predictor

def md5(candidate):
    return hashlib.md5(str(candidate).encode('ascii')).hexdigest()

def md5pow(suffix):
    for i in string.ascii_letters:
        for j in string.ascii_letters:
            for k in string.ascii_letters:
                for l in string.ascii_letters:
                    for m in string.ascii_letters:
                        for n in string.ascii_letters:
                            if (md5(i+j+k+l+m+n+suffix)[:6] == '000000'):
                                return i+j+k+l+m+n

for _ in range(512):
    predictor = MT19937Predictor()
    sh = remote("chall.0ops.sjtu.edu.cn",10002)

    # md5(??????+TNIdqVwjSqAmJdanUPIm).startswith('000000')
    has = str(sh.recvuntil("startswith('000000')"),encoding='ascii').strip()
    
    print(has)

    suffix = has[11:31]

    md5p = md5pow(suffix)
    sh.sendline(md5p)
    n = str(sh.recvline(),encoding='ascii').strip()
    e = str(sh.recvline(),encoding='ascii').strip()
    
    for i in range(int(624/3)):
        sh.recvline()
        x0 = str(sh.recvline(),encoding='ascii').strip()
        x1 = str(sh.recvline(),encoding='ascii').strip()

        x0 = int(x0[5:])
        x1 = int(x1[5:])
        v = x0
        sh.sendline(str(v))

        m0p = str(sh.recvline(),encoding='ascii').strip()
        m1p = str(sh.recvline(),encoding='ascii').strip()

        m0 = int(m0p[6:])

        predictor.setrandbits(x0-1, k)
        predictor.setrandbits(x1-1, k)
        predictor.setrandbits(m0-1, k)

    pre = predictor.getrandbits(k)+1
    m0 = pre
    m0r = (((m0&1)<<(BITS-1)) | (m0>>1))

    sh.recvline()
    x0 = str(sh.recvline(),encoding='ascii').strip()
    x1 = str(sh.recvline(),encoding='ascii').strip()

    x0 = int(x0[5:])
    x1 = int(x1[5:])
    v = x1
    sh.sendline(str(v))

    pre = predictor.getrandbits(k)+1
    if pre == x0:
        print('x0 predict')
    pre = predictor.getrandbits(k)+1
    if pre == x1:
        print('x1 predict')

    m0p = str(sh.recvline(),encoding='ascii').strip()
    m1p = str(sh.recvline(),encoding='ascii').strip()

    m1p = int(m1p[6:])

    flagnum = m1p^m0^m0r
    flag = long_to_bytes(flagnum)
    print(flag)

    sh.close()
    time.sleep(1)
    print(_)
```

## 8. 总结
不经意传输虽然在 2020 年以前的 CTF 里没怎么出现过，但近几年算是一个小热点。还是值得学习和积累相关知识的。

***

> **声明：本文采用 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) 授权。**
> 
> **This work is licensed under a [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).**
