---
title: 'DSA Biased K Attack'
date: 2019-10-18
permalink: /posts/2019/10/DSA Biased K Attack/
tags:
  - RSA
  - DSA
  - Lattice
---

本文主要介绍`RSA`签名与`DSA`签名在CTF中的常见题目。

*  目录
{:toc}


## 交流会上的PPT
<iframe src="https://blog.arpe1s.xyz/files/DSA.pdf" style="width:700px; height:800px;" frameborder="0"></iframe>

## 代码实现
```python
import gmpy2


q = 1078232060875050147874718424284712678513754974181
l = 8
amount = 30
two_l = gmpy2.mpz(pow(2, l, q))
two_l_inv = gmpy2.invert(two_l, q)


#u = MSB_l,p(alpha*t), solve alpha
def solve_hnp(t, u, amount):
	# http://www.isg.rhul.ac.uk/~sdg/igor-slides.pdf
	M = Matrix(RationalField(), amount+1, amount+1)
	for i in xrange(amount):
		M[i, i] = q
		M[amount, i] = t[i]

	M[amount, amount] = 1 / (2 ** (l + 1))
    
	def babai(A, w):   
		A = A.LLL(delta=0.75)   
		G = A.gram_schmidt()[0]
		t = w
		for i in reversed(range(A.nrows())):
			c = ((t * G[i]) / (G[i] * G[i])).round()
			t -= A[i] * c
		return w - t

	closest = babai(M, vector(u + [0]))
	return (closest[-1] * (2 ** (l + 1))) % q


print ('--------Start--------')

f = open('output.txt')
r = []; s = []; hash_m = []; a = []
for i in range(amount):
	r.append(int(f.readline().strip('\n')[4:]))
	s.append(int(f.readline().strip('\n')[4:]))
	hash_m.append(int(f.readline().strip('\n')[10:], 16))
	a.append(int(f.readline().strip('\n'), 2))
	f.readline()
f.close()


print('--------Reading data finished.--------')

t = []; u = []
for i in range(amount):
	inv_s = gmpy2.invert(s[i], q)
	tmp_1 = int((r[i] * inv_s * two_l_inv) % q)
	tmp_2 = int(((a[i] - inv_s * hash_m[i]) * two_l_inv) % q)
	t.append(tmp_1)
	u.append(tmp_2)

alpha = solve_hnp(t, u, amount)
print ('Alpha is ' + str(alpha) + '.')

print ('--------Solving HNP finished.--------')
```
