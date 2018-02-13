This post includes my answers to the questions in the unofficial contest that was held in [Codeforces](http://codeforces.com/) by [METU Computer Engineering Department](https://ceng.metu.edu.tr/). There were ten questions in total.

I will be providing the link to the question first and then my solution to it. I solved eight out of ten questions so there will be eight explanations.



[A-) Declined Finalists](http://codeforces.com/problemset/problem/859/A)

The complexity of my solution is O(nlog(n)). Firstly, I sorted the input array and then iterated over the array. Secondly, in every step, I checked how many people declined the invitation by considering the size of the input array.

```python
K = input()
arr = map(int, raw_input().split())
arr.sort()

unknowns = 25 - K
mox = 0
cost = 0
for elem in arr:
    if elem > mox+1:
        missing = elem-mox-1
        if unknowns < missing:
            cost += missing - unknowns
            unknowns = 0
        else:
            unknowns -= missing

        mox = elem
    elif elem == mox+1:
        mox += 1

print cost
```



[B-) The Contest](http://codeforces.com/problemset/problem/813/A)

The complexity of my solution is O(n). First I initalized an array by considering input range and then I marked the start and end points of intervals on the array finally I filled between each mark inclusively on the array. After doing that the first marked cell at index greater or equal to the sum of the times of solving problems is our answer and if there is no cell marked answer is -1.



```python
n = input()
ll = map(int, raw_input().split())
m = input()

tt = [0 for i in xrange(100001)]

for i in xrange(m):
    s, e = map(int, raw_input().split())
    tt[s] = 1
    tt[e] = -1

now = None
for i in xrange(len(tt)):
    if tt[i] == 1:
        now = 1
    elif tt[i] == 0:
        tt[i] = now
    else:
        tt[i] = 1
        now = 0

s = sum(ll)
if s >= len(tt):
    print -1
else:
    if tt[s] == 1:
        print s
    else:
        check = False
        for i in xrange(s+1, len(tt)):
            if tt[i] == 1:
                check = True
                print i
                break

        if not check:
            print -1
```



[C-) Dishonest Sellers](http://codeforces.com/problemset/problem/779/C)

The complexity of my solution is O(nlog(n)) at worst case and O(n) at best case, it depends on the number of entries having decreased costs after a week. We need to buy at least k items so we buy all the items whose price is lower this week. Later we continue buying from items whose price will be decreased next week and to minimize our cost we take from items whose change in the price is the lowest. 

```python
n, k = map(int, raw_input().split(" "))

pb = map(int, raw_input().split(" "))
pa = map(int, raw_input().split(" "))

inc = []
dec = []
for i in xrange(n):
    if pa[i] > pb[i]:
        inc.append(i)
    else:
        dec.append(i)

dec.sort(key=lambda x: abs(pa[x]-pb[x]))

som = 0
oops = False
for i in xrange(len(inc)):
    som += pb[inc[i]]

for i in xrange(len(dec)):
    if i+len(inc) >= k:
        oops = True

    if oops:
        som += pa[dec[i]]
    else:
        som += pb[dec[i]]

print som
```



[D-) Dijkstra?](http://codeforces.com/problemset/problem/20/C)

The complexity of my solution is O(Elog(V)) where E is the number of edges and V is the number of vertices. The tricky part of this question is graph can have multiple edges between two vertices and can have loops. To overcome them we simply take the minimum weight edge in case of multiple edges occur and for the loops we simply do not care because if a node is already visited we will not be visiting again. After applying dijkstra with min-heap, we traverse back from destination to the source vertex by using our dictionary holding from-to relation.



```python
from heapq import heappush, heappop

n, m = map(int, raw_input().split(" "))

d = {}
for i in xrange(m):
    a, b, w = map(int, raw_input().split(" "))
    if a not in d:
        d[a] = {}

    if b not in d[a]:
        d[a][b] = w
    else:
        d[a][b] = min([d[a][b], w])

    if b not in d:
        d[b] = {}

    if a not in d[b]:
        d[b][a] = w
    else:
        d[b][a] = min([d[b][a], w])


q = []
sofar = set()
paths = {}
heappush(q, [0, 1, None])
check = False
while len(q) > 0:
    cost, node, fr = heappop(q)

    if node in sofar:
        continue

    paths[node] = fr
    sofar.add(node)

    if node == n:
        check = True
        break

    if node not in d:
        continue

    for neighbour in d[node]:
        if neighbour not in sofar:
            heappush(q, [cost + d[node][neighbour], neighbour, node])

if check:
    ll = [n]
    while True:
        if paths[ll[-1]] is None:
            break
        else:
            ll.append(paths[ll[-1]])

    print ' '.join(map(str, ll[::-1]))
else:
    print -1
```



[F-) Test](http://codeforces.com/problemset/problem/25/E)

The complexity of my solution is O(n). By combining three strings we need to find minimum length. Combining strategy is intersecting prefix of left and suffix of right strings so this intersection is included once in the final string. To solve this question I simply generated all possible match positions and for every match position I used a modified version of KMP algorithm as you can see below.



```cpp
#include <iostream>
#include <string>
#include <cstring>
#include <algorithm>

using namespace std;

void computeLPSArray(string pat, int M, int *lps);

int KMPSearch(string pat, string txt)
{
    int M = pat.length();
    int N = txt.length();

    int lps[M];

    computeLPSArray(pat, M, lps);

    int i = 0; 
    int j  = 0; 
    for(int i = 0; i<N;i++) {
        while (j > 0 && txt[i] != pat[j])
            j = lps[j - 1];

        if (txt[i] == pat[j])
            j++;

        if (i == N - 1)
            return i - (j - 1);
        else if (j == M)
            j = lps[j - 1];

    }

    return 0;
}

void computeLPSArray(string pat, int M, int *lps)
{
    int len = 0;

    lps[0] = 0; 
    
    int i = 1;
    while (i < M)
    {
        if (pat[i] == pat[len])
        {
            len++;
            lps[i] = len;
            i++;
        }
        else 
        {
            if (len != 0)
            {
                len = lps[len-1];
            }
            else 
            {
                lps[i] = 0;
                i++;
            }
        }
    }
}


int main() {
    string s1, s2, s3;
    cin >> s1;
    cin >> s2;
    cin >> s3;

    string ll[3] = {s1, s2, s3};
    int pp[3] = {0, 1, 2};
    int result = -1;
    do {
        int match1 = KMPSearch(ll[pp[1]], ll[pp[0]]);
        string neu = ll[pp[0]].substr(0, match1) + ll[pp[1]];
        int match11 = KMPSearch(ll[pp[2]], neu);
        int len = match11 + ll[pp[2]].length();
        if (result == -1 || result > len )
            result = len;

        match1 = KMPSearch(ll[pp[2]], ll[pp[1]]);
        neu = ll[pp[1]].substr(0, match1) + ll[pp[2]];
        match11 = KMPSearch(neu, ll[pp[0]]);
        len = match11 + neu.length();
        if (result == -1 || result > len )
            result = len;

    } while (next_permutation(pp, pp+3));

    cout << result << endl;

    return 0;
}
```



[G-) Game Of Stones](http://codeforces.com/problemset/problem/768/E)

The complexity of my solution is O(n). The idea is calculating the grundy numbers for the game and calculating the xor-sum of them. First I found the grundy numbers and then I used these numbers in my calculation.

```python
def calculateMex(s):
    mex = 0

    while mex in s:
        mex += 1

    return mex

def calculateGrundy(n, moves):
    if n == 0:
        return 0

    s = set()

    for i in xrange(n):
        if n-i not in moves:
            result = calculateGrundy(i, moves | {n-i})
            s.add(result)

    return calculateMex(s)
```

```cpp
#include <iostream>

using namespace std;

int main() {
    int gg[66] = {0, 1, 1, 2, 2, 2, 3, 3, 3, 3, 4, 4, 4, 4, 4, 5, 5, 5, 5, 5, 5, 6, 6, 6, 6, 6, 6, 6, 7, 7, 7, 7, 7, 7, 7, 7, 8, 8, 8, 8, 8, 8, 8, 8, 8, 9, 9, 9, 9, 9, 9, 9, 9, 9, 9, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10};

    int piles;
    cin >> piles;
    int now = 0;
    for (int i = 0; i < piles; i++) {
        int stones;
        cin >> stones;
        now ^= gg[stones];
    }

    if (now == 0)
        cout << "YES"<<endl;
    else
        cout << "NO" << endl;

    return 0;
}
```



[H-) Random Query](http://codeforces.com/problemset/problem/846/F)

The complexity of my solution is O(n). I spent 16 hours for this question. I will explain the solution step by step.

To find the expected value, we need to generate all possible cases and sum the number of unique values in these cases and divide by the case count.

After writing O(n^2) brute force algorithm we see that denominator of the final equation, case count number, is equal to the square of the length of the array.

Lets check a two element array case. These two elements can either be same or not. If they are same, we have two cases, range [0-1] and range [1-0] and for each range we have 1 unique number so in total we have 2. If they are unique, ranges are same but this time we have 2 unique numbers in each range so in total we have 4.

Now let's increase the array size. For example consider the arrays [1, 2, 8, 3, 2, 10] and [1, 2, 2, 8, 3, 10]. If we consider every range starting from the beginning [0, x] then we can see that changing the index of the duplicate number changes the total sum proportional with the distance of it from its first occurence. So the idea is to be able to find indexes of each numbers' second, third, fourth,.. etc occurences in linear time and we can do that. To do that we initialize an array with the same length first then we iterate the original array and when we see an element first time we write -1 to our second array to the same index and if it is not the same we write the index of the array in which we saw this element before. Think of it like a linked-list between same valued numbers based on their indexes.

Since we know that each same element decreases the value by two, (or halts the increase by two another way to say it) we can iterate over this newly created array of us and take a cumulative sum to able to find the effect of duplicates to each element in our original array.

To understand the idea visually, you can check the number triangle below. Every number corresponds to the sum of cases generated from indexes covered by its leftmost and rightmost childs.

```
Array: [1, 4, 15, 7, 2, 4, 5]
			  10               [L, L+6]

            8    8             [L, L+5]

          8   6    8           [L, L+4]

       8    6   6   8          [L, L+3]

     6   6   4   6    6        [L, L+2]

   4   4   4   4   4  4        [L, L+1]

1   4   2   7   2   4   5      Original array

```



​              



In our code we simply sum through diagonals. On the first diagonal we have [1, 4, 6, 8, 8, 8, 10] why didn't 8 in index three did not increase? Because second 2 comes in the original array and we can see that since we found before the effects of duplicates.

And finally we add n to the sum because ranges can be [x, x] meaning every number creates a one element range and it is unique for sure.



```cpp
#include <iostream>
#include <vector>
#include <cstring>

using namespace std;


int ll[1000001];
int nll[1000001];
int dp[1000001];

int main() {
    int n;
    cin >> n;

    for(int i=0;i<n;i++)
        cin >> ll[i];

    memset(dp, -1, 1000001*sizeof(int));

    for(int i=0;i<n;i++) {
        nll[i] = -1;
        if (dp[ll[i]] != -1){
            int pastInd = dp[ll[i]];
            nll[pastInd] = i;
        }
        dp[ll[i]] = i;
    }

    nll[n-1] = 0;
    for(int i=n-2;i>-1;i--) {
        if (nll[i] != -1)
            nll[i] = nll[i+1] + n - nll[i];
        else
            nll[i] = nll[i+1];
    }

    float som = 0;
    for(int i=0;i<n;i++)
        som += (n-i) * (n-i + 1) - 2 - nll[i] * 2;

    som += n;
    cout << som / (float)(n*n) << endl;

    return 0;
}
```



[I-) Level generation](http://codeforces.com/problemset/problem/818/F)

The complexity of my solution is O(1). Maximum number of bridges in a graph is a straight line in which every edge is a bridge and if we have n vertices, we have n-1 bridges. we start adding edges and bridge number starts decreasing. We need to keep doing that till number of edges are as close to the number of bridges times two as possible.

Bridge	Edge

N-1		N-1

N-3		N

N-4		N+2

N-5		N+5



So we have a pattern here huh? We can create an equation and check for times two equality and solve the equation accordingly. The tricky part here is after deleting a bridge not adding the max amount of edges because it can create a situation in which times two equality is broken and we fall back to number of optimal bridges+1 thus finding less edges than the answer.

```cpp
#include <iostream>
#include <cmath>

using namespace std;

int main() {
    int T;
    cin >> T;

    for (int i=0;i<T;i++) {
        long long inp;
        cin >> inp;

        long long now = (long long) sqrt((8*inp)+1);
        long long x = (now-5)/2;
        x = x > 0 ? x : 0;


        long long edgeNow = inp - 1 + (x * (x + 1))/2;
        long long bridgeNow;
        if ( x==0 )
            bridgeNow = inp-1;
        else
            bridgeNow = inp - 1 - (x + 1);


        long long edgeNext = inp - 1 + ((x + 1) * (x + 2))/2;

        if ((bridgeNow-1)*2 > edgeNow && (bridgeNow-1)*2 < edgeNext)
            cout << (bridgeNow-1)*2 << endl;
        else
            cout << edgeNow << endl;
    }

    return 0;
}
```



