---
title: 'SEAL 库 CKKS 方案 "scale out of bounds" 报错分析'
date: 2021-03-10
permalink: /posts/2021/03/seal_scale_error/
tags:
  - Homomorphic Encryption
  - SEAL
---

"scale out of bounds" 报错分析。

*  目录
{:toc}

最近在尝试写一些同态加密的小玩意，其中在用SEAL库的时候，一开始没怎么注意乘法次数，以为 `16384` 作为 `poly_modulus` 已经足够了，结果跑代码时报错 `"scale out of bounds"`。

于是查了一些资料，尝试了一下解决这个报错。

## 0. 参考的Issues
[Issues with defining CoeffModulus for CKKS](https://github.com/microsoft/SEAL/issues/128)

[Polynomial moduli beyond 32768](https://github.com/microsoft/SEAL/issues/261)

[Multiply plain repeatedly and scale out of bound error](https://github.com/microsoft/SEAL/issues/182)

[SEAL-CKKS max multiplication depth](https://stackoverflow.com/questions/55268399/seal-ckks-max-multiplication-depth)

[Implementing and Accelerating Bootstrap](https://github.com/microsoft/SEAL/issues/125)

## 1. 初始参数如何设置
首先给出初始参数设置的说明：

```
There are two encryption parameters that are necessary to set:
        - poly_modulus_degree (degree of polynomial modulus);
        - coeff_modulus ([ciphertext] coefficient modulus);
```

第一个参数 `poly_modulus_degree` 推荐的是 `4096, 8192, 16384, 32768`。如果需要更大的，则要在初始化的时候用 `sec_level::none` 忽视安全级别来开启。

第二个参数 `coeff_modulus`的`bit-length` 之和受到 `poly_modulus_degree` 的限制，如下表。

```
        +----------------------------------------------------+
        | poly_modulus_degree | max coeff_modulus bit-length |
        +---------------------+------------------------------+
        | 1024                | 27                           |
        | 2048                | 54                           |
        | 4096                | 109                          |
        | 8192                | 218                          |
        | 16384               | 438                          |
        | 32768               | 881                          |
        +---------------------+------------------------------+
```

如果不满足限制，会报错如下：

```shell
terminate called after throwing an instance of 'std::invalid_argument'
  what():  encryption parameters are not set correctly
```

其次，除去首尾，中间的要设置的相近，并和 `scale` 大小相近，这样在 `rescale` 时会很方便。

此外，指定 `bit-length` 的作为 `coeff_modulus` 的数是有限的，并不是想找多少就有多少。



## 2. 为什么这么设置
### （1） `coeff_modulus` 受 `poly_modulus_degree` 限制的规则
可以参考这个问题：[Issues with defining CoeffModulus for CKKS](https://github.com/microsoft/SEAL/issues/128)

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_bd1bd3efd2ff3d9582a5ee509dc65e72.png)



### （2） 模交换 modulus switching
官方文档：

```
    Modulus switching is a technique of changing the ciphertext parameters down
    in the chain. The function Evaluator::mod_switch_to_next always switches to
    the next level down the chain, whereas Evaluator::mod_switch_to switches to
    a parameter set down the chain corresponding to a given parms_id. However, it
    is impossible to switch up in the chain.
    Due to the modulus switching chain, the order of the 5 primes is significant.
    The last prime has a special meaning and we call it the `special prime'. Thus,
    the first parameter set in the modulus switching chain is the only one that
    involves the special prime. All key objects, such as SecretKey, are created
    at this highest level. All data objects, such as Ciphertext, can be only at
    lower levels. The special prime should be as large as the largest of the
    other primes in the coeff_modulus, although this is not a strict requirement.

              special prime +---------+
                                      |
                                      v
    coeff_modulus: { 50, 30, 30, 50, 50 }  +---+  Level 4 (all keys; `key level')
                                               |
                                               |
        coeff_modulus: { 50, 30, 30, 50 }  +---+  Level 3 (highest `data level')
                                               |
                                               |
            coeff_modulus: { 50, 30, 30 }  +---+  Level 2
                                               |
                                               |
                coeff_modulus: { 50, 30 }  +---+  Level 1
                                               |
                                               |
                    coeff_modulus: { 50 }  +---+  Level 0 (lowest level)
```

### （3） 为什么数量有限
因为这些数需要满足一定条件，如下图。

下图论文出处：[A Full RNS Variant of Approximate Homomorphic Encryption](https://eprint.iacr.org/2018/931.pdf)

所以对于确定的 `bit-length`，它们是有限的，写代码时并不是想找多少就有多少。

相对的，如果需要较多的数，自然是要选大一点的 `poly_modulus_degree`，这样 `coeff_modulus` 可以取得更多，`bit-length` 也可以设置的相对较大。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_9fc22e5875938ca73ff779a974a8d62e.png)

### （4） rescaling
`rescaling` 是 CKKS 方案独有的操作，正是这个操作让它不同于其他方案，可以很方便地处理浮点数，从而被 AI 等领域青睐。

官方文档：
```
    We saw in `2_Encoders.cs' that multiplication in CKKS causes scales in
    ciphertexts to grow. The scale of any ciphertext must not get too close to
    the total size of CoeffModulus, or else the ciphertext simply runs out of
    room to store the scaled-up plaintext. The CKKS scheme provides a `rescale'
    functionality that can reduce the scale, and stabilize the scale expansion.

    Rescaling is a kind of modulus switch operation (recall `3_Levels.cs').
    As modulus switching, it removes the last of the primes from CoeffModulus,
    but as a side-effect it scales down the ciphertext by the removed prime.
    Usually we want to have perfect control over how the scales are changed,
    which is why for the CKKS scheme it is more common to use carefully selected
    primes for the CoeffModulus.

    More precisely, suppose that the scale in a CKKS ciphertext is S, and the
    last prime in the current CoeffModulus (for the ciphertext) is P. Rescaling
    to the next level changes the scale to S/P, and removes the prime P from the
    CoeffModulus, as usual in modulus switching. The number of primes limits
    how many rescalings can be done, and thus limits the multiplicative depth of
    the computation.

    It is possible to choose the initial scale freely. One good strategy can be
    to is to set the initial scale S and primes P_i in the CoeffModulus to be
    very close to each other. If ciphertexts have scale S before multiplication,
    they have scale S^2 after multiplication, and S^2/P_i after rescaling. If all
    P_i are close to S, then S^2/P_i is close to S again. This way we stabilize the
    scales to be close to S throughout the computation. Generally, for a circuit
    of depth D, we need to rescale D times, i.e., we need to be able to remove D
    primes from the coefficient modulus. Once we have only one prime left in the
    coeff_modulus, the remaining prime must be larger than S by a few bits to
    preserve the pre-decimal-point value of the plaintext.

    Therefore, a generally good strategy is to choose parameters for the CKKS
    scheme as follows:

        (1) Choose a 60-bit prime as the first prime in CoeffModulus. This will
            give the highest precision when decrypting;
        (2) Choose another 60-bit prime as the last element of CoeffModulus, as
            this will be used as the special prime and should be as large as the
            largest of the other primes;
        (3) Choose the intermediate primes to be close to each other.

    We use CoeffModulus.Create to generate primes of the appropriate size. Note
    that our CoeffModulus is 200 bits total, which is below the bound for our
    PolyModulusDegree: CoeffModulus.MaxBitCount(8192) returns 218.
    
    We choose the initial scale to be 2^40. At the last level, this leaves us
    60-40=20 bits of precision before the decimal point, and enough (roughly
    10-20 bits) of precision after the decimal point. Since our intermediate
    primes are 40 bits (in fact, they are very close to 2^40), we can achieve
    scale stabilization as described above.
```

## 3. 为什么会报错
![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_72c8b85413b5e39563301a2136bc268f.png)

```
Generally, for a circuit of depth D, we need to rescale D times, i.e., we need to be 
able to remove D primes from the coefficient modulus. Once we have only one prime 
left in the coeff_modulus, the remaining prime must be larger than S by a few bits 
to preserve the pre-decimal-point value of the plaintext.
```

正如图中所说，每一步 `rescale`，也就是 `rescale_to_next_inplace()`，都会用掉一个素数。而每次乘法之后必然是要 `rescale` 的。因此电路的深度，也就是乘法的深度，不能超过初始参数中 `coeff_modulus` 的个数。否则就会报错 `"scale out of bounds"`。

当然，目前的大部分需要用 CKKS 的运算还是可以满足的，它们的 `level` 基本都在20以下。对于一些 `level` 非常高的，或许可以考虑用其他方法。

改进的方法的话，或许未来 SEAL 库的 CKKS 方案会有 `bootstrapping`，但目前来看微软的开发者还没有这个打算。SEAL 库目前是不支持 `bootstrapping` 的。因为据开发者说，CKKS 的 `bootstrapping` 会积累错误，它和 Gentry 提出的 `bootstrapping` 概念是不一样的。

![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_bec421203c3eb16445eeb6c59831d874.png)


## 4. 应对方法
目前我还没有找到什么特别好的方法。

### （1） 方法一
我所知的第一种方法是增大所用的参数，这样就能满足较多次数的 `rescale`。

当然相对的，这样会需要更多的内存和运行时间。

一般可以如下设置：
```c
poly_modulus_degree = 32768;
precision        = 40;
modulus_size     = 20;
default_value    = 40;
sentenial_value  = 50;

vector<int> mod(modulus_size, default_value);
mod[0] = sentenial_value;
mod[modulus_size-1] = sentenial_value;

EncryptionParameters parms(scheme_type::ckks);
parms.set_poly_modulus_degree(poly_modulus_degree);
parms.set_coeff_modulus(CoeffModulus::Create(poly_modulus_degree, mod));
```

如果有更大的需求，可以用 `sec_level::none` 来开启。

参考这个 issue：[Polynomial moduli beyond 32768](https://github.com/microsoft/SEAL/issues/261)

### （2） 方法二
第二种方法则是不用 SEAL。在 CKKS 方案的提出者的论文链接里可以找到他们写过一个叫 `HEAAN` 的库。这个库是支持 `bootstrapping` 的。虽然是 2017 年的库，但拿来改一改应该还是可以用的。

[github-HEAAN](https://github.com/snucrypto/HEAAN)

```
HEAAN is software library that implements homomorphic encryption (HE) that supports 
fixed point arithmetics. This library supports approximate operations between 
rational numbers. The approximate error depends on some parameters and almost 
same with floating point operation errors. The scheme in this library is on the paper 
"Homomorphic Encryption for Arithmetic of Approximate Numbers" 
(https://eprint.iacr.org/2016/421.pdf).
```
