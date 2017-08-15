Recently I started solving Project Euler questions through hackerrank site. Hackerrank versions of the questions challenge the user more when compared with the actual question in terms of larger input ranges thus complexity of the solution needs to be a lot less.

I plan to provide my approaches to questions that are making me struggle on hackerrank version here.

This post's question will be 29th question in Project Euler. It wants us to find the number of different numbers  of a^b in the range of 2 <= a <= N  and 2 <= b  <= N

Project Euler version asks the result when N equals 1000. This is a fairly simple range because brute force works.

**Brute force:**

```python
numbers = {}

for a in xrange(2, N+1):

    for b in xrange(2, N+1):

        number = a**b

        numbers[number] = 1

print len(numbers)
```



In the Hackerrank version of the question N can go up to 10^5 and brute force fails in that case. 

First thing we need to find is the definition of duplication. Just get some paper and pencil and start writing the numbers, soon you will be realizing there is a pattern. If two numbers share the same prime factors with different counts then there is a chance these two numbers can have duplicates, others we can just ignore. To find these pairs we can simply search for numbers which have an integer as a result when taken the powers like 1/2, 1/3, 1/4, … From the result we can just make a power series. Let me give you an example.

If we need to check 16:

- 16^(1/4) = 2
  - 2^1, 2^2, 2^3, 2^4
- 16^(1/2) = 4
  - 4^1, 4^2

These are the numbers we need to cross check while taking powers on both sides. 
Below code calculates all possible duplicate holding numbers.

```python
from math import log

N = 100000
upper = int(log(N, 2)) + 1
d = {}
for a in xrange(2, N+1):
    for i in xrange(2, upper):
        now = round(a**(1.0/i))

        if int(now)**i != a: continue
        if a not in d: d[a] = []
        d[a].append(now)


print str(d)
```



This code generates all the numbers in the range [2, 10^5] and we can directly assign this precomputed dictionary of numbers in our code. You may wonder why I chose upper to be equal to the log of N in base 2. That is because 2 is the smallest prime and finding the logarithm enables us to find maximum possible power a number can have.

Now we have the numbers and we need to iterate them and count possible duplicates. But iterating for every number in the range [2, 10^5] is very costly and we need to be able to jump in this range.

Consider the case 4^6 and 8^4 duplication. they both have 2 as their prime factor and 4 is 2^2, 8 is 2^3. If we write a of their duplications cases, 4^3 - 8^2, 4^6 - 8^4, 4^9 - 8^6… , we can see a pattern. It is the least common multiple (lca) rule. 4 was 2^2 and 8 was 2^3 and their count of primes were 2 and 3 and lca(2, 3) = 6. So for every 6th power of 2 we will have a duplication occuring.That is true unless the lca of these two numbers is equal to one of the numbers.

You can see the code below.

``` python
from math import log
from fractions import gcd

N = input()
MAX = (N-1)**2
res = 0    
dp = {}
upper = int(log(N, 2)) + 1
for a in precomputed:
    if a > N: continue
    for now in precomputed[a]:
        i = int(round(log(a, now)))
        for us in xrange(1, i):
            smallest_common_multiple = (i * us) / gcd(i, us)
            
            if smallest_common_multiple != i:
                for j in xrange(smallest_common_multiple, us*N+1, smallest_common_multiple):
                    dp[(a, j/i)] = 1
            else:
                for j in xrange(2*i, us*N+1, smallest_common_multiple):
                    dp[(a, j/i)] = 1
    res += len(dp)
    dp = {}
    
print MAX - res
```



This code works on all inputs in hackerrank under 10 seconds. precomputed is the result of the first program we mentioned here.

We need to re-initialize the dp in every outer loop because otherwise we face with a Memory error, this is important.

You can email me if you have any questions. My mail: sukru[at]sukrubezen.com

