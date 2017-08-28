Project euler question 49 was about finding prime sequences such that this sequence should be an arithmetic one (differences between pairs of consecutive elements must be same) and the digits of the elements in this sequence must be the same, their places should only change.

For example, 1487, 4817, 8147 are prime, their pair differences are 3330 and they share the same digits.

In Project Euler version it gives us the sequence above and asks that there is a second 3 element sequence whose elements are in the range of [1000, 10000).

This problem is really simple, a brute force approach can easily find the solution. For brute force approach one can find all primes in the range and for each prime take permutations of it, check if it is prime and check if the difference of pairs are the same.



On the hackerrank version, it asks us to find 3 or 4 element prime sequences with the same logic with one addition, the first element of the sequence must be less than N and N is in the range [2000, 1000000]. So this makes things complicated because taking permutations takes a lot of time and considering the range this will not be possible.



First thing we need to make is to generate primes up to 1000000, the upper bound, so we can only iterate on primes and we need to hash these as well since we will be needing to reverse check if a number is prime or not after we start generating numbers from same digit set. 

To generate primes less than a spesific number we can use sieve of eranthoses, an ancient algorithm for finding primes up to a spesific limit. Below you can see the code.

``` python
d = {}
def primes_sieve2(limit):
    a = [True] * limit
    a[0] = a[1] = False

    for (i, isprime) in enumerate(a):
        if isprime:
            d[i] = 1
            yield i
            for n in xrange(i**2, limit, i):
                a[n] = False
```



Next thing is to find a way of generating numbers from a digit set and this generated number needs to be higher and higher since we need a constant difference between pairs.

It seems this problem was a question of Google Code Jam in 2009 in round 1B, [Link to question-solution][https://code.google.com/codejam/contest/dashboard?c=186264#s=a&a=1]

This algorithm finds the next highest number by using the digits of that number and algorithm is O(n). Code is below.

``` python
dp = {}
def next_highest(n):
    if n in dp: return dp[n]
    ll = map(int, list(str(n)))

    for i in xrange(len(ll)-1, 0, -1):         
        if ll[i] > ll[i-1]:
            mon, monitr = None, None
            for j in xrange(i, len(ll)):
                if ll[j] > ll[i-1] and (mon is None or mon >= ll[j]):
                    mon = ll[j]
                    monitr = j
                    
            ll[i-1], ll[monitr] = ll[monitr], ll[i-1]
            s, e = i, len(ll)-1
            while s < e:
                ll[s], ll[e] = ll[e], ll[s]
                s += 1
                e -= 1
                
            now = ''.join(map(str, ll))
            
            dp[n] = int(now)
            return int(now)
    
    dp[n] = None
    return None
```



Now we need to be able to check if the differences of the pairs are the same or not when we are provided with a sequence of primes created by using the same digit set. If there is subsequence(s) we need to be able to detect them as well.



``` python
def solve(ll, K):
    sofar = []
    for m in xrange(len(ll)):
        p = ll[m]
        if p >= N: return sofar
        for i in xrange(m+1, len(ll)):
            diff = ll[i] - p
            for j in xrange(i+1, len(ll)):
                diff2 = ll[j] - ll[i]
                if diff2 > diff: break
                if diff2 == diff:
                    if K == 4:
                        for k in xrange(j+1, len(ll)):
                            diff3 = ll[k] - ll[j]
                            if diff3 > diff2: break
                            if diff3 == diff2:
                                sofar.append(str(p) + str(ll[i]) + str(ll[j]) + str(ll[k]))
                    else:
                        sofar.append(str(p) + str(ll[i]) + str(ll[j]))
                
    return sofar
```



Since K was 3 or 4, I hardcoded the loops but one could make it cleaner by creating a list holding K iterators and using backpropagation when necessary. The important thing here is the start of the sequence must be less than N but other primes can be larger actually.



Finally we put it all together and solve the question.



``` python
N, K = map(int, raw_input().split())
    
notgo = {}
final = []
for p in primes_sieve2(1000000)::
    if p in notgo: continue 
    if p < 1000: continue
    if p >= N: break
    
    kp = [p] # Holds the sequence
    res = p
    while True:
        res = next_highest(res)
        if res is None: 
            break
        if res not in d: continue  # Check if the number is prime
        notgo[res] = 1 # Never visit that number again
        kp.append(res)

    if len(kp) >= K:
        res = solve(kp, K)
        final.extend(res)

print '\n'.join(map(str, sorted(map(int, final))))
```



If you have any questions, you can ask me from: sukru[at]sukrubezen.com