---
title: '浅谈高次剩余的求解'
date: 2021-01-19
permalink: /posts/2021/01/rth_root/
tags:
  - Number Theory
---

本文主要通过 `*CTF 2021` 的一道题目，浅谈高次剩余求解。

*  目录
{:toc}

虽然已经很久不打 CTF 了，但有比赛了还是习惯性的瞅一眼 crypto 类题目。

最近在 `*CTF 2021` 碰到了一题 `little case`，看了一眼就确定是熟悉的套路。首先 `wiener's attack`，然后 `AMM算法` 解有限域上的高次剩余。

说起来这已经是第四次碰到这类题了，不妨做个总结。

## 1. 背景
### （1） 如何求解二/高次剩余？

在学密码学时，大部分教科书都有专门的一个章节“二次剩余”，一般紧跟在原根章节之后。

它们往往会从“整数a何时是模素数 p 完全平方数”这个古老的问题说起，给出二次剩余的定义与基本性质，再谈到欧拉判别法、勒让德符号、雅可比符号、二次互反律，最后引申到欧拉伪素数、Solovay-Strassen 检验法、Rabin 密码体制和零知识证明。

然而，国内不少教科书并不会告诉学生，对于一个很一般的、模数为素数或者不是素数的二次剩余，如何求解？以及如果不是二次，而是三次或者更高次，如何求解？

这个问题相关论文有很多，发展到最后，产生了几种比较主流的，适用于大多数情况的方法。

关于有限域上的二次剩余，比较主流的两种方法是 `Cipolla-Lehmer Algorithm` 和 `Tonelli-Shanks Algorithm`。

关于有限域上的高次剩余，比较主流的两种方法是在二次剩余的基础上形成的，由 `H.C. Williams`改进后的`Cipolla-Lehmer` 算法和基于 `Tonelli-Shanks` 的 `AMM` 算法。

而当模数为合数时，一般简单的处理方法是根据算术基本定理，将模数分解为素数乘积，分别求解后再使用中国剩余定理。此外，对于一些模数和指数较为特殊的情况，有其相对于一般算法更快的特殊算法，比如 `Pocklington Algorithm`。

二次剩余两个算法的原理详见[二次剩余：OI Wiki](https://oi-wiki.org/math/quad-residue/)。

高次剩余两个算法的原理详见第二部分提到的论文。

[更多请见：二次剩余 wiki](https://en.wikipedia.org/wiki/Quadratic_residue)

### （2） CTF 中相关题目

之所以说 `*CTF 2021` 这次是第四次碰到，因为我自从 2017 年开始打 CTF 以来，已经碰到过三次了。

第一次是 `Hackergame 2019` 的 `十次方根`，出题人在 wp 中提到，肯定有比他的方法更优雅的开根方法。

此外，他还提到 2018 高校运维赛的 `AzureRSA` 以及 `De1taCTF 2019` 的 `babyrsa`，让他怀疑两次比赛出题人是否认为模 n 意义下开高次方是不可能的。

题目详见[Hackergame 2019 - 十次方根](https://github.com/ustclug/hackergame2019-writeups/blob/master/official/%E5%8D%81%E6%AC%A1%E6%96%B9%E6%A0%B9/README.md)。

确实，在这次 `Hackergame` 之前，我对这个问题也不太清楚。在几年前准备某年的 `CNSS` 的招新题目的时候，有一题用了 `Rabin` 体制，涉及到二次剩余，我当时用的是 `Tonelli-Shanks`。当时曾经问过 `Nu1L` 的 `gmcn` 师傅高次剩余该用什么算法，不过我们当时都不知道有啥算法。

第二次是紧随第一次的 `NCTF 2019` 的 `easyrsa`。南邮的 `soreat_u` 师傅找到了 `AMM算法` 的论文，并在实现之后出成了题目。具体细节详见[NCTF 2019 - easyrsa](https://blog.soreatu.com/posts/intended-solution-to-crypto-problems-in-nctf-2019/#easyrsa909pt-2solvers)。

第三次是 `CTFshow` 的练习题 `unusualrsa5`，和第二次题目几乎相同，只是改了一下参数，而且由于指数设置的太小，不需要 `AMM算法`，用 `SageMath` 或者 `Mathematica` 可以直接开出来。
算上这次 `*CTF 2021` 就是第四次了。


## 2. 理论与论文
### （1） Cipolla 与 AMM 区别

两者各有其优势劣势。

大多数情况下，我们倾向于用 `AMM算法`，因为 `Cipolla算法` 中很关键的几步需要域扩张，这是很麻烦的。

而 `AMM` 在 $r^v \mid (q-1)$ 中的 $v$ 很大时，运算速度会非常非常慢，这时我们就又倾向于 `Cipolla` 了。

此外，在 2019 年 `Gook Hwa Cho` 的一篇论文中，他对 `Cipolla` 做了改进，降低了算法复杂度，并且 $r$ 不一定必须要是素数。而 `AMM` 算法中 $r$ 必须是素数，如果不是素数的情况，那我们必须自己先对 $r$ 进行一些处理，比如分解 $r$，之后解两次。

### （2） Cipolla-Lehmer

原理就不赘述了，论文讲的很清楚，跟着推导一遍就完事了。
<iframe src="https://blog.arpe1s.xyz/files/Cho2019_cipolla.pdf" style="width:700px; height:800px;" frameborder="0"></iframe>

### （3） Adleman-Manders-Miller

原理就不赘述了，论文讲的很清楚，跟着推导一遍就完事了。
<iframe src="https://blog.arpe1s.xyz/files/AMM algorithm.pdf" style="width:700px; height:800px;" frameborder="0"></iframe>


## 3. *CTF 2021： little case
### （1） 题目

题目如下：
```python
from Crypto.Util.number import *
from libnum import *
from secret import flag,special,p,q,n


def little_trick(msg):
    p1 = getPrime(1024)
    q1 = getPrime(1024)
    n1 = p1 * q1
    d1=random.randint(1,2**256)
    e1=inverse(d1,(p1-1)*(q1-1))
    print(n1)
    print(e1)
    print(pow(msg,e1,n1))


def real_trick():
    assert (special > (ord("*")*100) and gcd(special,(p-1)*(q-1))!=1 )
    print(n)
    print(pow(libnum.s2n(flag),special,n))


if __name__ == '__main__':
    little_trick(p-1)
    real_trick()
```
```python
21669699875387343975765484834175962461348837371447024695458479154615348697330944566714587217852888702291368306637977095490953192701450127798670425959768118384915082017373951315699899009631834471691811815393784748930880954114446745814058132752897827717077886547911476575751254872623927783670252969995075629255541621917767501261249192653546875104532649043219697616464205772025267019328364349763854659490144531087349974469079255236823096415094552037488277752927579909539401311624671444833332618177513356173537573280352724384376372955100031534236816681805396608147647003653628203258681097552049114308367967967184116839561

20717541468269984768938524534679430706714860712589983300712432366828367981392533792814384884126053081363266457682162675931547901815985830455612301105504518353600255693451085179954519939635263372257973143178677586338992274607959326361412487748088349413448526455377296931144384663805056580662706419414607407821761761574754611275621927387380065975844282519447660467416826579669726178901884060454994606177784839804528666823956703141147239309978420776148158425922031573513062568162012505209805669623841355103885621402814626329355281853436655713194649170570579414480803671531927080535374958180810697826214794117466378050607

17653913822265292046140436077352027388518012934178497059850703004839268622175666123728756590505344279395546682262531546841391088108347695091027910544112830270722179480786859703225421972669021406495452107007154426730798752912163553332446929049057464612267870012438268458914652129391150217932076946886301294155031704279222594842585123671871118879574946424138391703308869753154497665630799300138651304835205755177940116680821142858923842124294529640719629497853598914963074656319325664210104788201957945801990296604585721820046391439235286951088086966253038989586737352467905401107613763487302070546247282406664431777475

22346087036331379968192118389403047568445805414881948978518580277027027486284293415097623011228506968071753709256352246733181304513713003096615266613365080909760605498017330085960699607777361429562376124376340215426398797920168016137830563564636922257215066266075494625782943973857490781916694118187094786034792437781964601089843549995939887939410763350338658901108020658475956489391300528691289604149598720803012371765770928211044755626045817053870803040863722458554924076011151695567147976903053993914859714631837755435592006986598006207692599019026644753575853382810261910332197447386727419606073948645238377595719

12732299056226934743176360461051108799706450051853623472248552066649321279227693844417404789169416642586313895494292082308084823101092675162498154181999270703392144766031531668783213589136974486867571090321426005719333327425286160436925591205840653712046866950957876967715226097699016798471712274797888761218915345301238306497841970203137048433491914195023230951832644259526895087301990301002618450573323078919808182376666320244077837033894089805640452791930176084416087344594957596135877833163152566525019063919662459299054294655118065279192807949989681674190983739625056255497842063989284921411358232926435537518406L

```

观察到 `d` 很小，适用 `wiener's attack`，直接用现成的轮子即可解出 `msg`。

[Wiener's Attack, Python Implementation](https://github.com/pablocelayes/rsa-wiener-attack)

然后 `AMM` 算法解出一个解，之后求出所有的根即可。

如何求所有的根？参考 `soreat_u` 师傅博客给出的 `stackexchange` 上的两个链接即可。

[How to get the other roots?](https://stackoverflow.com/questions/6752374/cube-root-modulo-p-how-do-i-do-this)

[Finding the n-th root of unity in a finite field](https://crypto.stackexchange.com/questions/63614/finding-the-n-th-root-of-unity-in-a-finite-field)

### （2） 解题代码
```python
from Crypto.Util.number import long_to_bytes
import random
import time
import gmpy2

# About 3 seconds to run
def AMM(o, r, q):
    start = time.time()
    print('\n----------------------------------------------------------------------------------')
    print('Start to run Adleman-Manders-Miller Root Extraction Method')
    print('Try to find one {:#x}th root of {} modulo {}'.format(r, o, q))
    g = GF(q)
    o = g(o)
    p = g(random.randint(1, q))
    while p ^ ((q-1) // r) == 1:
        p = g(random.randint(1, q))
    print('[+] Find p:{}'.format(p))
    t = 0
    s = q - 1
    while s % r == 0:
        t += 1
        s = s // r
    print('[+] Find s:{}, t:{}'.format(s, t))
    k = 1
    while (k * s + 1) % r != 0:
        k += 1
    alp = (k * s + 1) // r
    print('[+] Find alp:{}'.format(alp))
    a = p ^ (r^(t-1) * s)
    b = o ^ (r*alp - 1)
    c = p ^ s
    h = 1
    for i in range(1, t):
        d = b ^ (r^(t-1-i))
        if d == 1:
            j = 0
        else:
            print('[+] Calculating DLP...')
            j = - discrete_log(a, d)
            print('[+] Finish DLP...')
        b = b * (c^r)^j
        h = h * c^j
        c = c ^ r
    result = o^alp * h
    end = time.time()
    print("Finished in {} seconds.".format(end - start))
    print('Find one solution: {}'.format(result))
    return result

def findAllPRoot(p, e):
    print("Start to find all the Primitive {:#x}th root of 1 modulo {}.".format(e, p))
    start = time.time()
    proot = set()
    while len(proot) < e:
        proot.add(pow(random.randint(2, p-1), (p-1)//e, p))
    end = time.time()
    print("Finished in {} seconds.".format(end - start))
    return proot

def findAllSolutions(mp, proot, cp, p):
    print("Start to find all the {:#x}th root of {} modulo {}.".format(e, cp, p))
    start = time.time()
    all_mp = set()
    for root in proot:
        mp2 = mp * root % p
        assert(pow(mp2, e, p) == cp)
        all_mp.add(mp2)
    end = time.time()
    print("Finished in {} seconds.".format(end - start))
    return all_mp


c = 12732299056226934743176360461051108799706450051853623472248552066649321279227693844417404789169416642586313895494292082308084823101092675162498154181999270703392144766031531668783213589136974486867571090321426005719333327425286160436925591205840653712046866950957876967715226097699016798471712274797888761218915345301238306497841970203137048433491914195023230951832644259526895087301990301002618450573323078919808182376666320244077837033894089805640452791930176084416087344594957596135877833163152566525019063919662459299054294655118065279192807949989681674190983739625056255497842063989284921411358232926435537518406
p = 199138677823743837339927520157607820029746574557746549094921488292877226509198315016018919385259781238148402833316033634968163276198999279327827901879426429664674358844084491830543271625147280950273934405879341438429171453002453838897458102128836690385604150324972907981960626767679153125735677417397078196059
q = 112213695905472142415221444515326532320352429478341683352811183503269676555434601229013679319423878238944956830244386653674413411658696751173844443394608246716053086226910581400528167848306119179879115809778793093611381764939789057524575349501163689452810148280625226541609383166347879832134495444706697124741
e = 4919

cp = c % p
cq = c % q
mp = AMM(cp, e, p)
mq = AMM(cq, e, q)
p_proot = findAllPRoot(p, e)
q_proot = findAllPRoot(q, e)
mps = findAllSolutions(mp, p_proot, cp, p)
mqs = findAllSolutions(mq, q_proot, cq, q)


def check(m):
    h = m.hex()
    if len(h) & 1:
        return False
    tmp = long_to_bytes(m)
    if tmp.startswith(b'*CTF'):
        print(tmp)
        return True
    else:
        return False


# About 16 mins to run 0x1337^2 == 24196561 times CRT
start = time.time()
print('Start CRT...')
for mpp in mps:
    for mqq in mqs:
        solution = CRT_list([int(mpp), int(mqq)], [p, q])
        if check(solution):
            print(solution)
            exit()
    print(time.time() - start)
            
end = time.time()
print("Finished in {} seconds.".format(end - start))
```
### （3） 结果
![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_9ab78fb548ed45f8dbb0f2ae7087be00.png)

***

> **声明：本文采用 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) 授权。**
> 
> **This work is licensed under a [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).**


