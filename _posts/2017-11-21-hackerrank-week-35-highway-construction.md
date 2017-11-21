[Hackerrank](https://www.hackerrank.com) has this monthly competitive programming contests in which seven programming questions are asked and contest covers seven days. Each day a question is released and complexity of the question increases everyday. ["Highway Construction"](https://www.hackerrank.com/contests/w35/challenges/highway-construction) was this month's sixth question and its complexity level was labeled as "Hard".

The question asks us to find the sum of numbers starting from 2 up to n by taking powers of k. n can go up to 10^18 and k can go up to 1000.

Question: 

* 1 <= n <= 10^18
* 1 <= k <= 10^3
* 2^k + 3^k + 4^k + … + (n-1)k = ? (mod 10^9 + 7)



We can easily see that brute force would not work because looping until 10^18 is unreachable. We need to dive into number theory, that's for sure.

We've already seen how to find this sum when k equals to one or two.

When k = 1

* = 2 + 3 + 4 + 5 + … (n-1) 
* = (1 + 2 + 3 + 4 + 5 + … n) - 1 - n 
* = (n * (n + 1) / 2) - (n+1)

When k = 2

* = 2^2 + 3^2 + 4^2 + 5^2 + .. (n-1)^2 
* = (1^2 + 2^2 + 3^2 + 4^2 + … + n^2) - 1^2 - n^2 
* = (n * (n+1) * (2n+1) / 6) - (n^2 + 1)

So we can assume there has to be a theory behind it. And guess what, indeed there is. Just a little bit complex..

----------------

The main reason we can not calculate by brute force was because n could be quite large. In the theory part, instead of looping through n, it loops through k. By doing that it makes the computation handled easily. Only fallback is it uses [Bernoulli Numbers](http://mathworld.wolfram.com/BernoulliNumber.html) in the calculation. These numbers seem to be used a lot in number theory and they are simply signed rational numbers. So if we can calculate Bernoulli numbers then we could easily put them in our equation and solve the problem. You can find below the code that generates Bernoulli numbers.

``` python
def seidel():
    """
    "...in 1877 Philipp Ludwig von Seidel published an ingenious
    algorithm which makes it extremely simple to calculate Tn."
    (Ibid, Wikipedia)
    
    See OEIS:
    http://www.research.att.com/~njas/sequences/A000111
    """
    row = [1]
    k = 1
    yield 1
    yield 1
    while True:
        newrow = []
        if not k%2:
            t = row[0]
            row.insert(0, t)
            left = 0
            for i in row:
                term = i + left
                left = term
                newrow.append(term)
            row = newrow
        else:
            t = row[-1]
            row.append(t)
            left = 0
            for i in reversed(row):
                term = i + left
                left = term
                newrow.append(term)
            row = list(reversed(newrow))
        yield t
        k += 1

def bernoulli():
    yield Fraction(1,1)
    yield Fraction(-1,2)
    seidelgen = seidel()
    next(seidelgen)
    sign = 1
    n = 2
    while True:
        sign *= -1
        denom = 2**n -4**n
        numer = sign * n * next(seidelgen)
        next(seidelgen)
        n += 2
        yield Fraction(numer, denom)
```



So this code creates a generator named bernoulli and it generates the next Bernoulli number in the sequence.
Bernoulli numbers are rational and its numerator and denominator gets bigger quickly so taking the mod of this rational number dynamically is important in terms of spending less computation power and time.

````python
MOD = 1000000009
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m

def trans(f):
    numerator, denominator = f.numerator, f.denominator
    if denominator < 0:
        denominator = -1*denominator
        numerator = -1*numerator
        
    return ((numerator%MOD) * modinv(denominator, MOD)) % MOD


bs = []
i = 0
for r in bernoulli():
  bs.append(r)
  i += 1
  if i == 502:
    break


````



So we defined some functions to help us. **modinv** function takes the mod of a rational number in the form of 
(1 / number) and **trans** function takes the mod of a rational number, in our case a bernoulli number and returns a single number. Getting rid of rational number is important, less things to calculate.

``` python
import operator as op
def ncr(n, r):
    r = min(r, n-r)
    if r == 0: return 1
    numer = reduce(op.mul, xrange(n, n-r, -1))
    denom = reduce(op.mul, xrange(1, r+1))
    return numer//denom

def getBernoulli(n):
    if n <= 2:
        return bs[n]
    
    if n%2 == 1: 
        return 0
    
    return bs[(n+2)/2]
```



More functions!
**ncr** simply calculates the combination of n with r and **getBernoulli** returns the Bernoulli number with the given index.
The important thing here is Bernoulli numbers having index of greater than 2 and odd are valued 0, that's why we calculated 502 Bernoulli numbers since our loop would execute a maximum of 1000 times and creating 502 Bernoulli numbers covers index range up to 1000.

```python
MOD = 1000000009
def highwayConstruction(n, k):
    n = n-1
    sum_ = 0
    for j in xrange(k+1):
        if j > 2 and j % 2 == 1: 
            continue
            
        sign = (-1)**j
        cc = ncr(k+1, j) % MOD
        sum_ += (sign % MOD) * cc * getBernoulli(j) * pow(n, (k-j+1), MOD)
        sum_ %= MOD
        
    return sum_ * modinv(k+1, MOD)
```



And here comes the main logic, gathering up the helper functions and creating the sum.
You can look into links below to learn more about the formula.

[Sums of Consecutive Powers](https://www.johndcook.com/blog/2016/12/31/sums-of-consecutive-powers/)

[Power Sum](http://mathworld.wolfram.com/PowerSum.html)

