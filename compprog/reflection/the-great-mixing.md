# The Great Mixing (A Problem Reflection)

By **Gabee De Vera**

*March 2, 2025*

---

[Read the Problem Statement here](https://codeforces.com/problemset/problem/788/C)

## Solution

### Basic Insights

Let's start with a basic insight:
> There are only 1001 unique coke types, corresponding to the concentrations $$\frac{0}{1000}, \frac{1}{1000}, \frac{2}{1000}, \cdots, \frac{1000}{1000}$$. Therefore, we effectively have $$k \le 1001$$.

### Reinterpreting the Average

Concentration is defined as the volume of carbon dioxide divided by the total volume of coke. Therefore, we have the formula:

$$\dfrac{\sum_i a_i V_i}{\sum_i V_i} = n$$

Notice that the left-hand side is essentially a weighted average.

We want to minimize $$\sum_i V_i$$ while satisfying the formula. Let's eliminate the division operation by multiplying both sides by $$\sum_i V_i$$:

$$\sum_i a_i V_i = n \sum_i V_i$$

$$\sum_i a_i V_i = \sum_i nV_i$$

$$\sum_i (a_i - n) V_i = 0$$

Let $$b_i = a_i - n$$, then we have:

$$\sum_i b_i V_i = 0$$

Where $$-1000 \le b_i \le 1000$$.

Thus, we have ["reinterpreted the average"](https://cp-algorithms.com/num_methods/binary_search.html).

Now, we want to find an assignment of $$V_i$$ such that $$\sum_i b_i V_i = 0$$ and $$\sum_i V_i$$ is minimized.

### Reframing as a Shortest Paths Problem

The problem could be reframed as follows:

> You are given an array of values $$B$$ with elements $$b_0, b_1, b_2, \cdots b_{k - 1}$$. You start with a sum $$s$$ of $$0$$. A move consists of setting $$s \leftarrow s + b_i$$ for some $$b_i$$. What is the minimum number of moves needed so that $$s$$ is $$0$$ once again?

The simplest solution would be to search through all possibilities: At each point, store all the reachable values after $$n$$ moves.

We can model the space of all possibilities as a directed graph $$G$$: Each node represents a possible sum, while each edge represents a possible move. Then, the optimal solution is the shortest cycle starting and ending at node $$0$$.

We *know* how to find the shortest cycle in a directed graph: We simply use [breadth-first search](https://cp-algorithms.com/graph/breadth-first-search.html)!

The time complexity of breadth-first search is $$O(\lvert V \rvert + \lvert E \rvert)$$.

### A Tiny Worry

One worry you may have is that the number of nodes and edges used is too large since we need to account for abritrarily large sums.

Thankfully, it can be shown that if an optimal solution exists, then we can rearrange the terms so that the partial sums are always in the interval $$I \in [-1000, 1000]$$. This implies that we only need to consider the nodes $$[-1000, 1000]$$. The proof for this is shown in [Appendix A](#appendix-a).

Therefore, the time complexity of this solution is $$O(I^2 + K)$$, where $$I$$ is the magnitude of the denominator of the concentration values (in this case, $$I = 1000$$).

## Reflection
---

### What Didn't Work

### Motivation Behind the Solution

## Appendices
---

### Appendix A

**Setup:** Let $$s_i$$ be the sequence of values in the array $$b_0, b_1, b_2, \cdots$$ such that $$\sum_{i = 0}^{V^{*} - 1} s_i = 0$$.

Note that $$V^{*}$$ is the minimum volume needed to achieve a sum of zero. 

**Claim:** There is some permutation of $$s_0, s_1, s_2, \cdots$$, say $$s'_0, s'_1, s'_2, \cdots$$, such that $$-1000 \le \sum_{i = 0}^{n} s'_i \le 1000$$ for all $$0 \le n \le V^{*} - 1$$.

**Proof:** The proof follows by construction.

Call the integers in the range $$[-1000, 1000]$$ as "good" and everything else as "bad". We want all partial sums $$-1000 \le \sum_{i = 0}^{n} s'_i \le 1000$$ to be good.

Let the current sum be $$k$$. Initially, $$k = 0$$.

If $$k > 0$$, there is always an element $$s_i$$ in $$S$$ such that $$s_i < 0$$. To see this, suppose, by contradiction, that all $$s_i$$ in $$S$$ is nonnegative and that $$k > 0$$. Then, adding all of the elements in $$S$$ to $$k$$ results in a positive value, which is a contradiction since adding all elements to $$S$$ must result in 0.

*Mutatis mutandis*, if $$k < 0$$, there is always an element $$s_i$$ in $$S$$ such that $$s_i > 0$$.

Now that we have the proper set-up, we can look at the construction.

We will keep adding elements $$s_i$$ from the set $$S$$ as follows:

When $$k = 0$$, pick any element of $$S$$ (say, $$s_i$$) then remove it from $$S$$. Then, the new sum is $$k = s_i$$, which is still good since $$s_i$$ is good.

When $$k > 0$$, pick an element of $$s_i$$ that's negative. note that since $$-1000 \le s_i < 0$$ and $$0 < k \le 1000$$, $$-1000 < s_i + k < 1000$$, so the sum is still good.

When $$k < 0$$, pick an element of $$s_i$$ that's positive. note that since $$0 < s_i <= 1000$$ and $$-1000 <= k < 0$$, $$-1000 < s_i + k < 1000$$, so the sum is still good.

Since we have found a construction such that the running sums are all good, we have successfully proved the claim $$\blacksquare$$.