I like mathematics and  competitive programming challenges a lot. Nowadays I am into combinatorics and one of my favourite topics is Nim games. In combinatorial games there are some predetermined rules :

- No random factor (Like throwing a dice)
- 2 players only
- No ties (One of the players will win eventually)
- Finite Moves (Game will not take forever in a loop)



Nim is a combinatorial game too. It is played by two players, there are piles of stones and every turn the player must choose a pile and take one or more stones from it. Eventually, the player who can not make a move since all piles are empty loses the game.

What if I tell you that the result of the Nim game can be determined by just looking into the starting positions of the piles of stones? First time I heard this, it was like a magic to me, to determine the result of the game by just looking into the starting positions.. Wow! 

It is really simple actually, you just take the number of stones in each pile and XOR(eXclusive OR) them all. If the result is zero second player wins if not first player is the winner.



## The Theory

If you want to learn more about the logic behind this method, we need to visit Grundy numbers. Grundy numbers, also called Nimbers, can be represented as the values of nim piles. *In addition, mex (minumum excludant) is a function such that when provided a set, returns the minimum natural number that is not in the provided set.* Grundy number of a pile is calculated as follows: 



Assume G(n) is the grundy number of a pile with n stones in it.

> G(0) = 0

> G(n) = mex(G(n-1), G(n-2), G(n-3), … G(0))



In standard Nim game, if there are n stones in a pile, the player can take [1, n] number of stones (range is inclusive). Every different number of stone the player can take leads to a different state of the game and that is why the player can create n different game states (actually it is different in terms of the stone numbers but results may be same). 

**Grundy value of a pile with  n stones is the minumum excludant of the possible states that can be reached from our current state.**

Let's calculate Grundy number of a pile with 3 stones in it.

`

> G(0) = 0

> G(1) = mex(G(0)) = mex(0) = 1

> G(2) = mex(G(1), G(0)) = mex(1, 0) = 2

> G(3) = mex(G(2), G(1), G(0)) = mex(2, 1, 0) = 3



As you can see above, in a standard Nim game, the grundy number of a pile is equal to the number of stones in it.



Now consider the case that a player is allowed to not make a move on a pile resulting with passing his/her turn. Furthermore, consider that either of the players can make this **zero move** only once meaning if a player used zero move on a pile, neither of the players can use zero move on that pile anymore. How would our Grundy numbers would change with this slight change?

If a zero move is applied to a pile we will underline it in terms of identification. If a pile is underline, since one can not make zero move on it, it will behave just like an ordinary pile.

> G(0) = 0

> G(1) = mex(G(<u>1</u>), G(0)) = mex(1, 0) = 2

> G(2) = mex(G(<u>2</u>), G(1), G(0)) = mex(2, 2, 0) = 1

> G(3) = mex(G(<u>3</u>), G(2), G(1), G(0)) = mex(3, 1, 2, 0) = 4

> G(4) = mex(G(<u>4</u>), G(3), G(2), G(1), G(0)) = mex(4, 4, 1, 2, 0) = 3



Can you spot the pattern?

Nevermind, I will tell you now. 

> N is even

> > G(n) = n - 1

> N is odd

> > G(n) = n + 1



Instead of memorization, if one learns the theory, s/he can extend it and apply to various different extensions of the same game.

The above extension was a [programming question](https://www.hackerrank.com/contests/w27/challenges/zero-move-nim) asked in the Week of Code 27 in hackerrank.