---
title: 'Project Euler Problem 1-50'
date: 2021-04-16
permalink: /posts/2021/04/project_euler_1_to_50/
tags:
  - Project Euler
---

最近在做 `Project Euler` 上的题目，这篇博客是前 50 题的记录。

*  目录
{:toc}

```python
# 1. Multiples of 3 and 5
# 改进：筛法打表，线性筛，欧式筛，大大减少时间复杂度

ans = []
n = 1000
for i in range(1, n):
    if (i%3 == 0) or (i%5 == 0):
        ans.append(i)
sum(ans)
```




    233168




```python
# 2. Even Fibonacci numbers
# 算法方面可能有改进，但我懒得去想了

upper = 4000000

first = 1
second = 2
third = 3
cnt = 2
while third <= upper:
    tmp = second + third
    second = third
    third = tmp
    if third%2 == 0:
        cnt += third
cnt
```




    4613732




```python
# 3. Largest prime factor
# 相关算法：各种大数分解算法

def takeFirst(elem):
    return elem[0]

tmp = list(factor(600851475143))
tmp.sort(key = takeFirst, reverse = True)

for i in tmp:
    if i[0].is_prime():
        print (i[0])
        break
```

    6857



```python
# 4. Largest palindrome product
# 参考：https://leetcode-cn.com/problems/largest-palindrome-product/solution/die-dai-shu-xue-fa-by-tooooo_the_moon-5ynx/
# 利用回文数分两半，sqrt(a^2 - 4*lower) 为整数这个条件来找 a

from gmpy2 import iroot

n = 3
a = 30

while a < 10^n:
    upper = 10^n - a
    lower = int(str(upper)[::-1])
    tmp = a^2 - 4*lower
    if tmp > 0:
        if iroot(tmp, int(2))[1]:
            print (int(str(upper) + str(lower)))
            break
    a += 1
```

    906609



```python
# 5. Smallest multiple
# 最小公倍数，参考：https://doc.sagemath.org/html/en/reference/rings_standard/sage/arith/functions.html
# 求 LCM 相关算法：GCD 法

from sage.arith.functions import LCM_list

LCM_list(list(range(1, 21)))
```




    232792560




```python
# 6. Sum square difference
# 平方和公式

n = 100
sum_of_squares = n*(n+1)*(2*n+1)/6
square_of_sum = ((1+n)*n/2)^2
diff = square_of_sum - sum_of_squares
diff
```




    25164150




```python
# 7. 10001st prime
# 思路一：参考：https://en.wikipedia.org/wiki/Prime_number_theorem，素数定理给出了第 i 个素数的上下界
# 然而我知道上下界也没什么用啊，上下界之间有好多个，project euler 是有验证码的，总不能一个个尝试吧
# 翻了一下 SageMath 的文档，有个 prime_pi 可以算出小于某个数的素数个数。那直接看还少多少个，再 next_prime 不就行了
# 文档：https://doc.sagemath.org/html/en/reference/functions/sage/functions/prime_pi.html
# 具体怎么实现的可以去看官网的源代码
# 思路二：直接 next_prime 加一个 for 循环算了，很快跑出结果
# 思路三：突然想起来，SageMath 有自带的啊，直接 Primes().unrank(10000)，注意这里是 10000，因为第一个被记作 0

# 第一次尝试，成功
# from gmpy2 import next_prime

# n = 10001
# lower = n*(log(n) + log(log(n))) - n
# upper = lower + n
# ans = []

# tmp = int(lower)
# for i in range(n - prime_pi(lower)):
#     tmp = next_prime(tmp)
# tmp

# 第二次尝试，成功
# from gmpy2 import next_prime
# from tqdm import tqdm

# tmp = 2
# n = 10001
# for i in tqdm(range(n-1)):
#     tmp = next_prime(tmp)
# tmp

# 第三次尝试，成功
P = Primes()
P.unrank(10000)
```




    mpz(104743)




```python
# 8. Largest product in a series
# 遍历一遍就完事了，怎么改进懒得想了

from functools import reduce

f = open(r"C:\Users\13672\Desktop\Project Euler\1 - 50\Largest product in a series.txt", "r")
givenstr = ""
tmp = f.readline().strip()
while tmp != "":
    givenstr += tmp
    tmp = f.readline().strip()
    
ans = []
for i in range(len(givenstr)-12):
    tmp = [int(j) for j in givenstr[i:i+13]]
    ans.append(reduce((lambda x, y: x * y), tmp))
max(ans)
```




    23514624000




```python
# 9. Special Pythagorean triplet
# 首先画出图像，显然是有解的，但不知道有没有整数解
# 尝试 Sage 自带的，貌似非齐次的并不能解，NotImplementedError: No solver has been written for inhomogeneous_general_quadratic.
# 尝试 sympy，得到解析解；观察发现，a、b 都是用 c 表示，而 a、b 根号里的内容一样；因此代入数值，只要其中一个没有虚部，那另一个必然也是正整数
# 其他解法：（1）直接爆破，由于数据量不大，很容易；（2）利用毕达哥拉斯三元组的性质来优化爆破，参考：https://radiusofcircle.blogspot.com/2016/04/problem-9-project-euler-solution-with-python.html；
# 毕达哥拉斯三元组在 Sage 中的相关文档：http://math.gordon.edu/ntic/

# Sage 部分的代码，失败
# x, y, z = var('x, y, z')
# g = Graphics()

# f1 = x^2 + y^2 - z^2
# f2 = x + y + z - 1000

# g += implicit_plot3d(f1, (x, 0, 500), (y, 0, 500), (z, 0, 500))
# g += implicit_plot3d(f2, (x, 0, 500), (y, 0, 500), (z, 0, 500))
# print (g.show())

# assume(x, 'integer')
# assume(y, 'integer')
# assume(z, 'integer')
# solve([x^2 + y^2 - z^2 == x + y + z - 1000], [x, y, z])


# sympy 部分的代码，成功
import sympy as sym

a, b, c = sym.symbols('a b c', integer=True, positive=True)

eq1 = sym.Eq(a^2 + b^2 - c^2, 0)
eq2 = sym.Eq(a + b + c - 1000, 0)

f = sym.solve([eq1, eq2], [a, b, c])[0]

res_a = f[0]
res_b = f[1]
res_c = f[2]

for i in range(1, 1001):
    tmp = str(res_b.evalf(subs={c:i}))
    if '+' not in tmp:
        if int(tmp.split('.')[1]) == 0:
            final_a = int(res_a.evalf(subs={c:i}))
            final_b = int(res_b.evalf(subs={c:i}))
            final_c = int(res_c.evalf(subs={c:i}))
            print ("a = {}".format(final_a))
            print ("b = {}".format(final_b))
            print ("c = {}".format(final_c))
            print ("abc = {}".format(final_a * final_b * final_c))
            break

```

    a = 200
    b = 375
    c = 425
    abc = 31875000



```python
# 10. Summation of primes
# 我比较懒，two million 并不大，直接 next_prime 跑一分钟就完事了
# 正规的做法貌似是筛法，不知道有没有比筛法更好的

import time

start_time = time.time()

P = Primes()
upper = 2000000
ans = 0

i = 0
tmp = P.unrank(i)
while tmp < upper:
    ans += tmp
    i += 1
    tmp = P.unrank(i)
    
end_time = time.time()

print ("Time used: {} s.".format(end_time - start_time))
print (ans)
```

    Time used: 84.27182841300964 s.
    142913828922



```python
# 11. Largest product in a grid
# 对矩阵的操作，不管哪个方向，坐标改一改就行了，然后 for 循环，做法都一样
# 看到矩阵，第一反应当然是 numpy
# 懒得写了，直接用的这个人的代码：https://stackoverflow.com/users/13873980/joe-ferndz

import numpy as np

f = open(r"C:\Users\13672\Desktop\Project Euler\1 - 50\Largest product in a grid.txt", "r")
s = f.read().strip()

ss = np.array([[int(x) for x in s.split()][i:i+20] for i in range(0,400,20)]).reshape(20, 20)

#process 4 elements to right, down, left-down-diagonal, right-down-diagonal

#check right: row: 1 thru 20 (index 0:19); columns: 1 thru 16 (index 0:15)
mr = max(np.prod(ss[i,j:j+4]) for i in range(20) for j in range(16))

#check down: row: 1 thru 16 (index 0:15); columns: 1 thru 20 (index 0:19)
mc = max(np.prod(ss[i:i+4,j]) for i in range(16) for j in range(20))

#check right-down-diagonal: row: 1 thru 16 (index 0:15); columns: 1 thru 16 (index 0:15). row,col increments by 1 to go right-down-diagonal 
mx = max(np.prod([ss[i+k,j+k] for k in range(4)]) for i in range(16) for j in range(16))

#check left-down-diagonal: row: 1 thru 20 (index 0:19); columns: 4 thru 20 (index 3:19). row increments by 1 and col decrements by 1 to go right-down-diagonal
my = max(np.prod([ss[i+k,j-k] for k in range(4)]) for i in range(16) for j in range(3,20))

ans = max([mr,mc,mx,my])

print (ans)
```

    70600674



```python
# 12. Highly divisible triangular number
# 查了一下 SageMath 文档，果然有求 divisors 数量的，应该是用了乘性函数的性质来算的
# https://doc.sagemath.org/html/en/constructions/number_theory.html
# 试了几个，大致确定了范围，然后二分法？不对！二分法不行，因为因子数量并不是数字越大就越多的！
# 初等数论的乘性函数部分就专门讲过因子个数函数，我在凝聚招新还出过题，居然下意识当单调增函数来做了，第一时间没想起来
# 解法应该是直接爆破，因为 500 个因子并不是很多

from tqdm import tqdm

for i in tqdm(range(1, 1000000)):
    res = (i+1)*i/2
    tmp = number_of_divisors(res)
    if tmp > 500:
        print (res)
        break
```

      1%|          | 12374/999999 [00:00<00:08, 120986.81it/s]

    76576500


    



```python
# 13. Large sum
# 应该是想让你写一个高精度大数加法，不过 python 自带任意精度，直接加就完事了

f = open(r"C:/Users/13672/Desktop/Project Euler/1 - 50/Large sum.txt", "r")

res = 0
for i in range(100):
    res += int(f.readline().strip())
str(res)[:10]
```




    '5537376230'




```python
# 14. Longest Collatz sequence
# 没什么巧妙的解法，穷举就行；如何加速代码？可以把之前走过的路放 cache 里，之后用到就可以直接用
# 代码懒得写，直接用的 https://codereview.stackexchange.com/questions/122273/longest-collatz-sequence-in-python
# 用了 python 的装饰器
# 其实 wikipedia 的 Collatz conjecture 页面直接有答案

def memo(f):
    f.cache = {}
    def _f(*args):
        if args not in f.cache:
            f.cache[args] = f(*args)
        return f.cache[args]
    return _f

@memo
def collatz(n):
    if n == 1:
        return 1
    if n % 2 == 0:
        return 1 + collatz(n / 2)
    if n % 2 == 1:
        return 1 + collatz(3 * n + 1)

print (max(range(1, 10**6), key=collatz))
```

    837799



```python
# 15. Lattice paths
# 小学低年级时印象很深的奥赛题，这应该是我最早接触到递归思想？

dimension = 20
dimension += 1 # 格点的维度变成矩阵会多一
M = Matrix(ZZ, dimension, dimension)

for i in range(dimension):
    M[0, i] = 1
    M[i, 0] = 1

for i in range(1, dimension):
    for j in range(1, dimension):
        M[i, j] = M[i-1, j] + M[i, j-1]

print (M[dimension-1, dimension-1])
```

    137846528820



```python
# 16. Power digit sum
# 可能是想模 10 、100、1000 得到每一位，考察快速幂？
# 但 SageMath 不用这么麻烦，随便求的

sum([int(i) for i in str(2^1000)])
```




    1366




```python
# 17. Number letter counts
# 很没意思的题目，数就完事了
# 可以直接用 num2words

from num2words import num2words
    
res = 0
for i in range(1,1001):
    res += len(''.join(''.join(num2words(i).split('-')).split(' ')))
res
```




    21124




```python
# 18. Maximum path sum I
# 动态规划，递归就完事了
# 更简单的思路：bottom up，选最大的往上加

tri = [[int(number) for number in row.split()] for row in open(r"C:\Users\13672\Desktop\Project Euler\1 - 50\Maximum path sum I.txt", "r")]

for i in range(len(tri)-1, 0, -1):
    for j in range(len(tri[i])-1): # 前一行的每一个，加上下一行的走上来的最大值
        tri[i-1][j] += max(tri[i][j], tri[i][j+1])
        
tri[0][0]
```




    1074




```python
# 19. Counting Sundays
# 初等数论书里有过一个万年历的算法，不过这道题显然简单得多，用不上那个方法
# 查万年历知，1901.01.01 是周二

months = {"January": 31, "February": 28, "March": 31, "April": 30, "May": 31, "June": 30, "July": 31, "August": 31, "September": 30, "October": 31, "November": 30, "December": 31}

start = 2
cnt = 0

for year in range(1901, 2001):
    for month in months:
        start += months[month]
        if year%4 == 0 and month == "February":
            start += 1
        if start%7 == 0:
            cnt += 1

cnt
```




    171




```python
# 20. Factorial digit sum
# SageMath 可以直接求
# 我想这个问题主要考察的应该是空间，时间复杂度并不高，100 次乘法，但阶乘后数字很大，如果是用 C 等语言，会超出精度
# 所以高精度乘法怎么做的，这题就应该怎么做
# 参考：https://www.geeksforgeeks.org/find-sum-digits-factorial-number/
# 另一个想法：https://www.xarg.org/puzzle/project-euler/problem-20/，这个想法我一开始也想到了，看到阶乘第一反应威尔逊定理，但这个思路并不能求出结果

# 思路一，正常解法
# def multiply(v, x):
#     carry = 0
#     size = len(v)
#     for i in range(size):
          
#         # Calculate res + prev carry
#         res = carry + v[i] * x
  
#         # updation at ith position
#         v[i] = res % 10
#         carry = res // 10
  
#     while (carry != 0):
#         v.append(carry % 10)
#         carry //= 10

# def findSumOfDigits( n):
#     v = []     # create a vector of type int
#     v.append(1) # adds 1 to the end
  
#     # One by one multiply i to current 
#     # vector and update the vector.
#     for i in range(1, n + 1):
#         multiply(v, i)
  
#     # Find sum of digits in vector v[]
#     sum = 0
#     size = len(v)
#     for i in range(size):
#         sum += v[i]
#     return sum
  
# if __name__ == "__main__":
      
#     n = 100
#     print(findSumOfDigits(n))


# 思路二，SageMath 直接求
sum([int(i) for i in str(factorial(100))])
```




    648




```python
# 21. Amicable numbers
# 因子和函数是乘性函数，可以打表来做
# 如何优化？参考：https://stackoverflow.com/questions/38094818/what-is-the-most-efficient-way-to-find-amicable-numbers-in-python
# SageMath 比较方便，因子和可以直接算，注意，不算本身！

from tqdm import tqdm

tmp = [0]
cnt = 0
for i in tqdm(range(1, 10000)):
    tmp.append(sum(divisors(i)[:-1]))
    if tmp[i] != i:
        if len(tmp)-1 > tmp[i]:
            if tmp[tmp[i]] == i:
                cnt += i
                cnt += tmp[i]
cnt
```

    100%|██████████| 9999/9999 [00:00<00:00, 116493.56it/s]





    31626




```python
# 22. Names scores
# 直接算即可

f = open(r"C:\Users\13672\Desktop\Project Euler\1 - 50\Names scores.txt", "r")
t = [i.strip("\"") for i in f.read().split(",")]
t.sort()
sum(sum(ord(j)-64 for j in t[i]) * (i+1) for i in range(len(t)))
```




    871198282




```python
# 23. Non-abundant sums
# 思路一：找出所有 abundant，然后找出所有它们可以表示的数，用总和减掉这些数的和；一开始跑了 14 秒，加了那个 break 之后变成了 9 秒
# 思路二：不到 1 秒！用了 not in，大大减少开销；参考 https://stackoverflow.com/questions/42499002/project-euler-23-python

# 思路一，成功
# from tqdm import tqdm

# upper = 28123
# tmp = []
# ans = []

# for i in tqdm(range(1, upper+1)):
#     if sum(divisors(i)[:-1]) > i:
#         tmp.append(i)
#         for j in range(len(tmp)):
#             tmp_sum = i + tmp[j]
#             if tmp_sum > upper:
#                 break
#             ans.append(tmp_sum)

# ans = list(set(ans))
# res = upper*(upper+1)/2 - sum(ans)
# res

# 思路二，成功
import time

if __name__ == '__main__':
    start_time = time.time()

    abundant = set()
    s = sum(list(range(12)))
    for i in range(12, 28124):
        if not any(i-a in abundant for a in abundant):
            s += i
        if sum(divisors(i)[:-1]) > i:
            abundant.add(i)
    print(s)
    print("--- {} seconds ---".format(time.time() - start_time))
```

    4179871
    --- 0.6846120357513428 seconds ---



```python
# 24. Lexicographic permutations
# 因为是按顺序来的，所以多少位的数就那么多个，模这个数目，就可以知道在某一位上对应的是哪个数了
# 注意，python 是从 0 开始的，因此 target 要设置成 999999，而非 1000000！

target = 1000000-1
number = list(range(10))
tmp = []
res = []
result = ""

for i in range(9):
    tmp.append(factorial(i+1))
tmp.sort(reverse = True)

for i in tmp:
    res.append(target//i)
    target = target%i
    
for i in res:
    result += str(number[i])
    del(number[i])

result += str(number[0])
result
```




    '2783915460'




```python
# 25. 1000-digit Fibonacci number
# 思路一：可以根据 Fibonacci 的通项公式得到一个大概的 digits 的公式，然后求一下反函数；参考：https://codereview.stackexchange.com/questions/224518/project-euler-25-the-1000-digit-fibonacci-index
# 注意，SageMath 的 log 默认以 e 为底
# 思路二：既然已经知道通项公式，那么二分查找即可

# 思路一，成功
# def euler25(k):
#     if k < 2:
#         return 1
#     phi = (1 + 5^0.5) / 2
#     return ceil((k + log(5, 10) / 2 - 1) / log(phi, 10))

# euler25(1000)


# 思路二，成功
target = 1000
upper = 5000
lower = 3000

while upper-lower > 1:
    mid = (lower+upper)//2
    tmp = len(str(fibonacci(mid)))
    if tmp == 1000:
        break
    elif tmp > 1000:
        upper = mid
    else:
        lower = mid

while tmp >= 1000:
    mid -= 1
    tmp = len(str(fibonacci(mid)))

print (mid+1)
```

    4782



```python
# 26. Reciprocal cycles
# Full Reptend Prime，参考：https://mathworld.wolfram.com/FullReptendPrime.html

def f(N):
    if N < 8: return 3
    for d in eratosthenes(N)[::-1]:
        k = d//2
        while 10^k%d != 1:
            k += 1
        if d-1 == k:
            return d

N = int(input('The longest recurring cycle for 1/d where d < '))
d = f(N)
print ("The longest repeating decimal for d < %d is 1/%d with %d repeating digits" % (N, d, (1 if N<8 else d-1)))
```

    The longest recurring cycle for 1/d where d < 1000
    The longest repeating decimal for d < 1000 is 1/983 with 982 repeating digits



```python
# 27. Quadratic primes
# 问题的背景和素数公式、乌岚螺旋、黑格纳数有关，可以自行搜索相关资料，但本题的解法和它们并没有什么关系
# 解题思路：n=0，所以 b 必须是素数；b 基本不可能是 2，b 是奇数，那么 n=1 有 a 是奇数；而 a 是素数时，明显比 a 是合数时生成的序列长；
# 这三点可以大大减少穷举时间
# 注意 a、b 可以是负数

from tqdm import tqdm

N = 1000
tmp = eratosthenes(N)
tmp += [-i for i in tmp]
ans = [0, 0]
for b in tqdm(tmp):
    for a in tmp:
        i = 0
        while is_prime(i^2+a*i+b):
            i += 1
        if ans[0] < i:
            ans[0] = i
            ans[1] = a*b
ans[1]
```

    100%|██████████| 336/336 [00:00<00:00, 1724.72it/s]





    -59231




```python
# 28. Number spiral diagonals
# 问题的背景是乌岚螺旋，就是他听讲座时闲着无聊画出来的东西...
# 不过这题和背景知识没什么关系，找规律直接算就行

upper = 1001
step = 2
start = 1
dimension = 1
ans = 1

while dimension < upper:
    for i in range(4):
        start += step
        ans += start
    step += 2
    dimension += 2

ans
```




    669171001




```python
# 29. Distinct powers
# 数字不大，直接穷举就行
# 怎么改进？可以只算素数的某某次方，然后用这些去表示那些合数的

ans = []
for a in range(2, 101):
    for b in range(2, 101):
        ans.append(a^b)

len(list(set(ans)))
```




    9183




```python
# 30. Digit fifth powers
# 问题背景是初中 oi 里的水仙花数，不一样的是这里并没有限制数字位数
# 显然，999999 > 6*9^5，因此最多五位，那么循环次数不多，穷举就好了，没什么好的改进，改进的话可以缩小穷举范围
# 如果用 i.digits() 会莫名其妙报错 AttributeError: 'int' object has no attribute 'digits'

sum(i for i in range(2, 1000000) if i == sum(int(d)^5 for d in str(i)))
```




    443839




```python
# 31. Coin sums
# 典型的动态规划
# 懒得写了，直接用答案给的代码
# 这个代码比起一般的做法，因为是按币种数来递归，因此减少了递归深度
# 可以理解成 200 元在 i 个币种下的表示方法 = i-1 个币种表示方法 + 新币种加入后的表示方法，新币种每次多加一张，剩余的用以前的币种表示

import time

def problem_31_a(money, coin_index, depth=1):
    global glob_depth
    glob_depth = max(glob_depth, depth)
    count = 0
    if coin_index <= 0:
        return 1
    m = money
    if memoiz_list[m][coin_index] > 0:
        return memoiz_list[m][coin_index]
    while money >= 0:
        count += problem_31_a(money, coin_index - 1, depth=depth+1)
        money -= coin_list[coin_index]
    memoiz_list[m][coin_index] = count
    return count

coin_list = [1, 2, 5, 10, 20, 50, 100, 200]

func = problem_31_a
glob_depth = 0
start = time.time()
memoiz_list = [[0, 0, 0, 0, 0, 0, 0, 0] for i in range(201)]
print(func(200, 7))
elapsed = time.time() - start
print("Result found in %f seconds - depth:%d" % (elapsed, glob_depth))

```

    73682
    Result found in 0.004504 seconds - depth:8



```python
# 32. Pandigital products
# 思路：总共 9 位十进制数，三个部分应该怎么分？显然，a 和 b 位数加起来应该和 c 差不多，那么 a 和 b 加起来不超过五位
# 而为了避免重复，我们可以假设 a < b，那么 a 最多两位，b最多四位，大大缩小穷举范围
# 注意，重复的不能算两次！

from tqdm import tqdm

ans = []

for a in tqdm(range(1, 100)):
    for b in range(a, 9999//a):
        c = a*b
        if  "".join(sorted(str(a)+str(b)+str(c))) == "123456789":
            ans.append(c)

sum(list(set(ans)))
```

    100%|██████████| 99/99 [00:00<00:00, 900.36it/s]





    45228




```python
# 33. Digit cancelling fractions
# 参考：https://mathworld.wolfram.com/AnomalousCancellation.html
# 可以理解成：\frac{xb+y}{yb+z} = \frac{x}{z}，b = 10，x < y
# 简单推导可得 y(10x-z) = 9xz；由于 y 最后会被消去，所以并不重要，x/z 是不同的形式就不是重复解
# 注意，SageMath 中的 Integer 和 Python 的 int 不是一回事！所以使用 python 的模块 Fraction 需要类型转换
# 参考：https://ask.sagemath.org/question/39717/pythons-fraction-incompatibility/

from fractions import Fraction

ans = int(1)

for z in range(1, 10):
    for x in range(1, z):
        tmp = divmod(9*x*z, 10*x-z)
        if not tmp[1] and tmp[0] in range(1, 10):
            ans *= Fraction(int(x), int(z))
            
ans.denominator
```




    100




```python
# 34. Digit factorials
# 和之前那题类似，首先确定范围，7*factorial(9) = 2540160，所以最多 6 位
# 还可以更进一步缩小范围，比如 6 位不可能含有 9

from tqdm import tqdm

fac = []
ans = []

for i in range(10):
    fac.append(factorial(i))

for i in tqdm(range(3, 999999)):
    if sum(fac[int(j)] for j in str(i)) == i:
        ans.append(i)
        
sum(ans)
```

    100%|██████████| 999996/999996 [00:02<00:00, 339264.85it/s]





    40730




```python
# 35. Circular primes
# 显然，应当从筛法得到的素数中，再去掉任何一位出现偶数或 5 的素数

from tqdm import tqdm

tmp = eratosthenes(1000000)
cnt = 0

def rotate(string):
    return string[1:] + string[:1]

def is_circular(i):
    int_i = i
    str_i = str(int_i)
    times = len(str_i) - 1
    if times == 0:
        return i
    digit = i.digits()
    if 5 in digit or 0 in digit or 2 in digit or 4 in digit or 6 in digit or 8 in digit:
        return 0
    while times:
        times -= 1
        int_i = int(rotate(str_i))
        str_i = str(int_i)
        while not is_prime(int_i):
            return 0
    return i


for i in tqdm(tmp):
    if is_circular(i):
        cnt += 1
        
cnt
```

    100%|██████████| 78498/78498 [00:00<00:00, 247413.65it/s]





    55




```python
# 36. Double-base palindromes
# 穷举就完事了

from tqdm import tqdm

def isDecPalindrome(x):
    tmp = str(x)
    return tmp[::-1] == tmp

def isBinPalindrome(x):
    tmp = bin(x)[2:]
    return tmp[::-1] == tmp

cnt = 0

for i in tqdm(range(1000000)):
    if isDecPalindrome(i):
        if isBinPalindrome(i):
            cnt += i

cnt
```

    100%|██████████| 1000000/1000000 [00:00<00:00, 1153083.09it/s]





    872187




```python
# 37. Truncatable primes
# 一开始想的是能否左右分别生成取交集，因为都是素数限制了某些位的位数只能取特定的数；
# 这种方法确实有一定效果，得到了十个解，但最大的那个数恰好逃出了我的限制，我也发现我这种做法有漏洞
# 没办法，只能穷举了，穷举也确实很快得到了结果

from tqdm import tqdm

primes = eratosthenes(1000000)
primelist = [str(x) for x in primes[::-1]]
primeset = set(primelist)

ans = []

for n in tqdm(primelist):
    if int(n) < 10:
        continue
    left = set(n[i:] for i in range(len(n)))
    right = set([n[:i+1] for i in range(len(n))])
    if not left.issubset(primeset):
        continue
    if not right.issubset(primeset):
        continue
    ans.append(int(n))

assert len(ans) == 11
sum(ans)
```

    100%|██████████| 78498/78498 [00:00<00:00, 109918.02it/s]





    748317




```python
# 38. Pandigital multiples
# 显然要让数字最大，原来的数字必然 9 开头，如果答案不是 9，那么乘上的东西必然最多是 4
# 而原来的数字必然不超过 4 位，否则 n = 2 都会越界，所以只需要爆破后三位
# 而且后三位必然不相同，不过爆破的数字已经足够小了，这个限制没什么必要

ans = []

for i in range(999):
    i = int('9'+str(i))
    tmp = str(i)
    for j in range(3):
        grow = tmp + str((j+2)*i)
        if len(grow) > 9:
            break
        tmp = grow
    if "".join(sorted(tmp)) == "123456789":
        ans.append(int(tmp))

max(ans)
```




    932718654




```python
# 39. Integer right triangles
# 问题背景是毕达哥拉斯三角形
# 思路是由于 a+b+c = p，a^2+b^2 = c^2，于是有 b = \frac{p^2-2ap}{2p-2a}
# 而 a+b > c，于是可知 a < p/2
# 此外，根据奇偶性质可知，p 一定是偶数

upper = 1000
ans = [0, 0]

for p in range(upper, 2, -2):
    tmp = [p, 0]
    for a in range(1, p//2):
        b = divmod(p^2-2*a*p, 2*(p-a))
        if b[1]:
            continue
        tmp[1] = tmp[1] + 1
    if tmp[1] > ans[1]:
        ans = tmp

ans[0]
```




    840




```python
# 40. Champernowne's constant
# 问题背景是钱伯瑙恩数，是一个超越数，可以表示为无穷级数
# 不过这里用不上无穷级数，只要利用钱伯瑙恩数的特殊性质即可

def champernowne(length):
    ans = ""
    cnt = 0
    while len(ans) < length:
        ans += str(cnt)
        cnt += 1
    return ans

tmp = champernowne(1000000)
ans = 1
for i in range(6):
    ans *= int(tmp[10^i])

ans
```




    210




```python
# 41. Pandigital prime
# 利用 itertools 的 permutations 即可

from itertools import permutations
from tqdm import tqdm


class success(Exception):
    pass

try:
    for n in range(9, 0, -1):
        for p in tqdm(permutations(range(n, 0, -1))):
            s = reduce(lambda b, a : 10 * b + a, p)
            if is_prime(s):
                raise success
except success:
    print (s)
```

    362880it [00:02, 162431.39it/s]
    40320it [00:00, 188237.58it/s]
    13it [00:00, 74898.29it/s]

    7652413


    



```python
# 42. Coded triangle numbers
# 按要求算就行

f = open(r"C:\Users\13672\Desktop\Project Euler\1 - 50\Coded triangle numbers.txt", "r")
tmp = f.read().strip("\"").split("\",\"")

length = max(len(i) for i in tmp)
tri_num = [1]
n = 1
t = 1
cnt = 0

while t < length*26:
    t = n*(n+1)//2
    tri_num.append(t)
    n += 1

for i in tmp:
    if sum(ord(j)-64 for j in i) in tri_num :
        cnt += 1
        
cnt
```




    162




```python
# 43. Sub-string divisibility
# 思路一：不需要写太多代码，直接根据整除的性质确定每一位的数，参考：http://mijkenator.github.io/2016/03/15/project-euler-problem-43/
# 思路二：穷举，其实可以先确定后面几位，再穷举前面的

from itertools import permutations
from tqdm import tqdm

p = permutations('0123456789')

solution = 0

for i in tqdm(p):
    if (int(''.join(i[7:10])) % 17 == 0):
        if (int(''.join(i[6:9])) % 13 == 0):
            if (int(''.join(i[5:8])) % 11 == 0):
                if (int(''.join(i[4:7])) % 7 == 0 and
                    int(''.join(i[3:6])) % 5 == 0 and
                    int(''.join(i[2:5])) % 3 == 0 and
                    int(''.join(i[1:4])) % 2 == 0):
                    solution += int(''.join(i))

solution
```

    3628800it [00:07, 460616.03it/s]





    16695334890




```python
# 44. Pentagon numbers
# 思路一：解一元二次方程可判断是否为五边形数，生成一定范围内所有这种数，再加减得到的结果穷举判断，非常慢
# 思路二：大约要跑十秒，参考：https://codereview.stackexchange.com/questions/93232/speedup-for-project-euler-44-pentagon-numbers
# 思路三：两个五边形数之间的差是 3n+1，利用这一点，我们可以避免乘法的情况下，用生成器生成五边形数列；一共四个数，b、d、b+d、b+2d，判断是否在生成的数列里即可
# 思路三目前最快，参考：http://louistiao.me/posts/project-euler/problem-44-pentagon-numbers/

import time

start = time.time()

def is_pentagon(k):
    n = (1+sqrt(1+24*k))/6
    return bool(n == int(n))

def polygonal(s):
    c = s - 2
    a = b = 1
    while True:
        yield a
        b += c
        a += b

def sum_diff_polygonal(s):
    seen = set()
    for i in polygonal(s):
        for j in seen:
            if i-j in seen and i-2*j in seen:
                yield i-j, j, i-2*j  
        seen.add(i)

it = sum_diff_polygonal(5)
end = time.time()
print ("Used {}s.".format(end-start))
next(it)[2]
```

    Used 0.0005259513854980469s.





    5482660




```python
# 45. Triangular, pentagonal, and hexagonal
# 和上一道题一样的做法，用生成器

def polygonal(s):
    c = s - 2
    a = b = 1
    while True:
        yield a
        b += c
        a += b

tri = set()
pen = set()
hex = set()

def search(times):
    cnt = 0
    t = polygonal(3)
    p = polygonal(5)
    h = polygonal(6)
    while cnt < times:
        i = next(t); j = next(p); k = next(h);
        if i in pen and i in hex:
            cnt += 1
            if cnt == times:
                yield i
        tri.add(i); pen.add(j); hex.add(k)

it = search(2)
next(it)
```




    1533776805




```python
# 46. Goldbach's other conjecture
# 穷举即可

upper = 10000
prime = eratosthenes(upper)

start = 3

while start <= upper:
    conjecture = True
    if not is_prime(start):
        conjecture = False
        for i in range(1, int(sqrt(start))+1):
            p = start - 2*i^2
            if p in prime:
                conjecture = True
                break
    if not conjecture:
        break
    start += 2
    
start
```




    5777




```python
# 47. Distinct primes factors
# SageMath 自带 prime_factors()，所以只要自己实现一个连续四个数的函数即可
# 参考：https://doc.sagemath.org/html/en/reference/rings_standard/sage/arith/misc.html
# 一开始自己写的代码有点丑陋，之后参考这篇文章修改了代码：http://louistiao.me/posts/project-euler/problem-47-distinct-primes-factors/
# itertools 文档：https://docs.python.org/zh-cn/3/library/itertools.html
# 由于 SageMath 的 prime_factors() 不会给 factor 的次数，因此对 i 用 prod 并不一定能得到原来的数，所以引入一个 cnt 来表示开头的元素

from itertools import count, tee

def nwise(iterable, n=2):
    iters = tee(iterable, n)
    for i, it in enumerate(iters):
        for _ in range(i):
            next(it, None)
    return zip(*iters)

n = 4
m = 4
cnt = 1

for i in nwise(map(prime_factors, count(2)), n):
    cnt += 1
    if all(map(lambda x: len(x) == m, i)):
        print (cnt)
        break
```

    134043



```python
# 48. Self powers
# 经典快速幂，python 从 3.x 开始，pow 就是快速幂，所以不需要我们自己实现了

mod = 10^10
ans = 0

for i in range(1, 1001):
    ans += pow(i, i, mod)
    
ans
```




    9110846700




```python
# 49. Prime permutations
# 穷举

from itertools import permutations, tee, combinations
from collections import defaultdict
from tqdm import tqdm

def nwise(iterable, n=2):
    iters = tee(iterable, n)
    for i, it in enumerate(iters):
        for _ in range(i):
            next(it, None)
    return zip(*iters)

primelist = list(primes(1000, 9999))
res = defaultdict(list)
ans = 0

for i in primelist:
    res["".join(sorted(str(i)))].append(i)

class success(Exception):
    pass

try:
    for j in res.values():
        tmp = list(combinations(j, int(3)))
        for k in tmp:
            if k[0] == 1487:
                continue
            if k[2]+k[0] == 2*k[1]:
                ans = str(k[0]) + str(k[1]) + str(k[2])
                raise success
except success:
    print (ans)
```

    296962999629



```python
# 50. Consecutive prime sum
# 穷举就完事了

from tqdm import tqdm

upper = 1000000
primelist = list(primes(upper))
primesum = [0]

for p in primelist:
    tmp = primesum[-1]+p
    if tmp > upper:
        break
    primesum.append(tmp)

length = len(primesum)
tmp = 0

for i in tqdm(range(length)):
    for j in range(length-1, i-1, -1):
        if j-i <= tmp:
            break
        num = primesum[j] - primesum[i]
        if is_prime(num):
            tmp = j-i
            max_num = num
            break

max_num
```

    100%|██████████| 547/547 [00:00<00:00, 397485.15it/s]





    997651


***

> **声明：本文采用 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/) 授权。**
> 
> **This work is licensed under a [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/).**
