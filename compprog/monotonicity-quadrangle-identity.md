# On the Monotonicity of Row Minima in DP Recurrences Involving Cost Functions Satisfying the Quadrangle Identity

By **Gabee De Vera**

*February 16, 2025*

---

## Intro

Consider a function $$f_d(x)$$ and a cost function $$C(x,y)$$. Here, $$f$$ usually contains the DP state.

In general, we want to consider how to compute recurrences of the form:

$$f_{d + 1}(i) = \min_{0 \le j \lt i} f_{d} (j) + C(j, i)$$

Over the values $$f_{d + 1} (0), f_{d + 1}(1), f_{d + 1}(2), ..., f_{d + 1}(n - 1)$$.

This is a common DP pattern in Competitive Programming. To see this, we will consider an example problem.

## A Sample Problem

*Read the problem statement here: [https://atcoder.jp/contests/dp/tasks/dp_z](https://atcoder.jp/contests/dp/tasks/dp_z)*

We will consider a *variant* of this problem: What is the minimum cost assuming you do *exactly* $$d$$ hops?

Suppose that $$n = N$$ and that $$1 \le n, d \le 5000$$.

In short, the problem asks you to compute $$f_d (n)$$, where $$f$$ satisfies:

$$f_d (i) = \min_{0 \le j \lt i} f_{d - 1} (j) + C(j, i)$$

Here, $$C(j, i) = (h_i - h_j)^2$$, and $$i \lt j \implies h_i \le h_j$$.

Naive computation of this recurrence gives an $$O(dn^2)$$ algorithm. Could we do better?

It turns out that we can do better! There is a certain property of $$C$$ we can exploit that allows us to solve this problem in $$O(d n \log n)$$. I won't discuss it here for now, since it's outside the scope of this blog, but if you're curious, look up "Divide and Conquer DP".

What is this property of $$C$$, and why does it matter?

## The Quadrangle Inequality

Suppose that $$L \le l \le r \le R$$. Then, we say that $$C$$ satisfies the Quadrangle Inequality iff:

$$C(L, r) + C(l, R) \le C(l, r) + C(L, R)$$

For example, when $$C(j, i) = (h_i - h_j)^2$$, you can show that it satisfies the quadrangle inequality as follows. First, let's start with the trivial inequalities $$h_l \ge h_L$$ and $$h_R \ge h_r$$. Thus, we have $$h_l - h_L \ge 0$$ and $$h_R - h_r \ge 0$$. Multiplying both identities together, we get:

$$\left(h_l - h_L\right) \left(h_R - h_r\right) \ge 0$$

Expanding...

$$h_L \left(h_r - h_R\right) + h_l \left(h_R - h_r\right) \ge 0$$

$$h_L h_r + h_l h_R \ge h_l h_r + h_L h_R$$

$$- 2h_L h_r - 2h_l h_R \le - 2h_l h_r - 2h_L h_R$$

$$h_L^2 - 2h_L h_r + h_r^2 + h_l^2 - 2h_l h_R + h_R^2 \le h_l^2 - 2h_l h_r + h_r^2 + h_L^2 - 2h_L h_R + h_R^2$$

$$\left(h_L - h_r\right)^2 + \left(h_l - h_R\right)^2 \le \left(h_l - h_r\right)^2 + \left(h_L - h_R\right)^2$$

Finally, substituting $$C(i, j) = \left(h_i, h_j\right)^2$$,

$$C(L, r) + C(l, R) \le C(l, r) + C(L, R) \ \blacksquare$$

Thus, the cost function in Frog 3 satisfies the Quadrangle Inequality.



Sure, the cost function satisfies a special property, but why do we *care* about this property?

## Monotonicity of Row Minima

It turns out that the value of $$j$$ that minimizes $$f_d (i) = \min_{0 \le j \lt i} f_{d - 1}(j) + C(j, i)$$ satisfies an interesting property when $$C$$ satisfies the Quadrangle Inequality. (Try printing the values yourself for some concrete set of inputs and see if you can spot the pattern!)

Denote $$x_i$$ as the value of $$j$$ that minimizes $$f_{d - 1}(j) + C(j, i)$$ for a fixed $$i$$. Then, I claim that if $$k \lt k'$$, then $$x_k \le x_{k'}$$.

In other words, the row minima move to the right as you move down the rows. Since the minima continuously move to the right, we say that they are *monotonically increasing*.

Why does this phenomenon occur?

## Proof of Monotonicity

Greedy insights can only take you so far---if left unproven, it may compromise the correctness of your algorithm. Therefore, we will *prove* that, if $$C$$ satisfies the Quadrangle Inequality, we have $$x_k \le x_{k'}$$.

Let's simply denote $$f_d$$ as $$f$$---We're only going to consider a single value of $$d$$ anyways. First, we know that $$j = x_k$$ minimizes $$f(j) + C(j, k)$$ for a fixed $$k$$. Therefore,

$$f(x_k) + C(x_k, k) \le f(x) + C(x, k)$$

Now, we want to show that $$x_{k'} \ge x_{k}$$. Equivalently, we want to show that all $$x \lt x_k$$ *cannot* be optimal for $$i = k'$$. Notice that if we can show the following:

$$x \lt x_k \implies f(x_k) + C(x_k, k') \le f(x) + C(x, k')$$

... we immediately prove that all $$x \lt x_k$$ cannot be optimal since $$j = x_k$$ achieves a smaller sum than $$j = x$$.



Now, our problem reduces to the following: Show that

$$x \lt x_k \implies f(x_k) + C(x_k, k') \le f(x) + C(x, k')$$

First, note that $$x \lt x_k \lt k \lt k'$$. Therefore, by the Quadrangle Identity, we have:

$$C(x, k) + C(x_k, k') \le C(x_k, k) + C(x, k')$$

Equivalently,

$$C(x_k, k') - C(x, k') \le C(x_k, k) - C(x, k) \text{  (Eq. 1)}$$

Adding $$f(x_k) - f(x)$$ to both sides of Equation 1,

$$f(x_k) + C(x_k, k') - f(x) - C(x, k') \le f(x_k) + C(x_k, k) - f(x) - C(x, k) \text{  (Eq. 2)}$$

Also, we know that:

$$f(x_k) + C(x_k, k) \le f(x) + C(x, k)$$

Equivalently,

$$f(x_k) + C(x_k, k) - f(x) - C(x, k) \le 0 \text{  (Eq. 3)}$$

Using Equation 3, we can rewrite Equation 2 as follows:

$$f(x_k) + C(x_k, k') - f(x) - C(x, k') \le f(x_k) + C(x_k, k) - f(x) - C(x, k) \le 0$$

$$f(x_k) + C(x_k, k') - f(x) - C(x, k') \le 0$$

$$f(x_k) + C(x_k, k') \le f(x) + C(x, k')$$

Notice: This is exactly what we wanted to prove. Therefore, $$x \lt x_k$$ cannot be optimal for $$i = k'$$, which implies that $$x_{k'} \ge x_k$$, as needed. $$\blacksquare$$

## Conclusion

When working with recurrences, it is always helpful to note the special properties of the sequences and functions involved. These special properties may allow one to optimize certain DP transitions. In this article, we were introduced to one such special property: The Quadrangle Inequality. Furthermore, we learned how this property enforces monotonicity in the minima of the DP transition, which we can then exploit with Divide and Conquer to optimize $$O(n^2)$$ DP to $$O(n \log n)$$.

## Acknowledgements

I would like to acknowledge the NOI.PH trainers for help in answering some of my queries regarding this topic.