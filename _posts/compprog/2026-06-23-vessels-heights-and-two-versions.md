---
layout: post
title: Vessels, Heights, and Two Versions

author: Gabee De Vera
excerpt: 'I was recently solving a Codeforces problem for practice...'
---

# Introduction

I was recently solving [a Codeforces problem](https://codeforces.com/problemset/problem/2234/C) for practice. The statement is as follows:

> You are given an array $$h$$ with $$n$$ positive integers, numbered $$h_0, ..., h_{n - 1}$$. Consider an arbitrary array $$w$$ with $$n$$ nonnegative integers. We say that $$w$$ is good iff for all $$0 \le i \lt n$$, $$\max(w_{i}, w_{(i + 1 \mod n)}) > h_i \to w_{i} = w_{(i + 1 \mod n)}$$.

> Define $$a_i$$ to be the maximum value of $$\sum_{j = 0}^{n - 1} w_j$$ subject to the constraint $$w_i = 0$$. Output the sequence $$a_0, a_1, a_2, ..., a_{n - 1}$$

Here, $$1 \le h_i \le 10^9$$ and $$n \le 2 \cdot 10^5$$.

# An $$O(n^2)$$ Solution

WLOG, let us consider how to compute $$a_0$$. The other values are computed similarly after we perform some cyclic shifts.

One nice way to deal with problems on circular arrays is to "linearize" it---first remove the cyclic connection, then reintroduce it later somehow. This trick turned out to be useful for this problem.

I first thought of "constraint propagation". Starting from the first element, go from left to right, keeping track of the maximum possible value I could set an element to. Since this problem is on a cyclic array, I also have to do the same thing from right to left.

The end result is the following identity for $$a_i$$ (here, we assume the summation indices wrap around $$\mod n$$):

$$ a_i = \sum_{l = i}^{i - 2} \min(\max(a_{i, i + 1, ..., l}), \max(a_{l + 1, l + 2, ..., i - 1})) $$

Which one may compute in $$O(n)$$ with prefix maxima. This results in an $$O(n^2)$$ solution.

Of course, this won't pass all test cases. This is where we'll need a crucial observation for this problem.

# An Important Observation

The main thing blocking us from computing the sum above quickly is the outer $$\min$$ function. Could we remove it?

It turns out that we could! Let us fix $$i$$. If we knew more about when the left argument of the $$\min$$ is strictly smaller, we can split the values of $$l$$ between those where the left argument of min is strictly smaller, and those where the right argument of min is strictly smaller.

Notice that this removes the $$\min$$---we may essentially replace it with two sums with a certain cutoff point. But what is this cutoff point? Phrased differently, at what point does the left argument cease to be strictly smaller than the right argument?

This is where we may apply the [Extremal Principle](https://brilliant.org/wiki/extremal-principle/). Consider the largest possible element (call it $$a_j$$). Notice that each element appears exactly once within the two range maxima $$\max(a_{i, i + 1, ..., l}), \max(a_{l + 1, l + 2, ..., i - 1})$$. Hence, at least one of these must be greater than or equal to $$a_k$$. However, since $$a_k$$ is the maximum value, we know that neither $$\max$$ may exceed $$a_k$$. Hence, at least one of the two arguments is equal to $$a_k$$.

Thus, $$k$$ is the cutoff point, and the sum simplifies to:

$$ a_i = \sum_{l = i}^{k - 1} \max(a_{i, i + 1, ..., l}) + \sum_{l = k}^{i - 2} \max(a_{l + 1, l + 2, ..., i - 1}) $$

But the value $$k$$ is constant for all $$i$$! Hence, if we can precompute $$\sum_{l = i}^{k - 1} \max(a_{i, i + 1, ..., l})$$ and $$\sum_{l = k}^{i - 2} \max(a_{l + 1, l + 2, ..., i - 1})$$ in $$O(f(n))$$, we can solve the full problem in $$O(\max(n, f(n)))$$!

# An $$O(n \log^2 (n))$$ Solution

For brevity, I will only discuss how to precompute $$\sum_{l = 0}^{k - 1} \max(a_{0, 1, ..., l})$$. Both sums above reduce to this sum after some cyclic shifts and reversals.

For some reason, the first thought I had to solve this was a **lazy propagating segment tree** combined with **binary search**. Specifically, consider a range sum segment tree with range assignment. Then, I consider the values of $$a_i$$ from left to right, starting from $$a_0$$. As I insert $$a_i$$, I set all the values strictly less than $$a_i$$ to $$a_i$$ within the segment tree.

Since we maintain the invariant that the values in the segment tree are nonincreasing, we may perform binary search to find the exact cutoff point at which the values begin to exceed $$a_i$$. This runs in $$O(\log^2 (n))$$---one log factor from the binary search, another from querying a value of the segment tree.

The code is found below:

```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
#define INF(type) numeric_limits<type>::max()

/*
A lazy-propagating segment tree over a monoidal structure
*/
template<typename T, typename U>
struct Tree {
    ll n;
    vector<T> v;
    vector<U> lazy;

    Tree(ll a_n): n(a_n), v(a_n << 2ll, T::identity()), lazy(a_n << 2ll, U::identity()) {}

    void _build(ll i, ll l, ll r, const vector<T>& a) {
        if(l == r) {
            v[i] = a[l];
            lazy[i] = U::identity();
            return;
        }

        ll m = (l + r) >> 1ll;

        _build(i << 1ll, l, m, a);
        _build((i << 1ll) | 1ll, m + 1ll, r, a);

        v[i] = v[i << 1ll] + v[(i << 1ll) | 1ll];
        lazy[i] = U::identity();
    }

    void _build(ll i, ll l, ll r, const T& a_v) {
        if(l == r) {
            v[i] = a_v;
            lazy[i] = U::identity();
            return;
        }

        ll m = (l + r) >> 1ll;

        _build(i << 1ll, l, m, a_v);
        _build((i << 1ll) | 1ll, m + 1ll, r, a_v);

        v[i] = v[i << 1ll] + v[(i << 1ll) | 1ll];
        lazy[i] = U::identity();
    }

    void _build(ll i, ll l, ll r, const function<T(ll)>& init) {
        if(l == r) {
            v[i] = init(l);
            lazy[i] = U::identity();
            return;
        }

        ll m = (l + r) >> 1ll;

        _build(i << 1ll, l, m, init);
        _build((i << 1ll) | 1ll, m + 1ll, r, init);

        v[i] = v[i << 1ll] + v[(i << 1ll) | 1ll];
        lazy[i] = U::identity();
    }

    void build(const vector<T>& a) {
        _build(1ll, 0ll, n - 1ll, a);
    }

    void build(const T& a_v) {
        _build(1ll, 0ll, n - 1ll, a_v);
    }

    void build(const function<T(ll)>& init) {
        _build(1ll, 0ll, n - 1ll, init);
    }

    void push(ll i, ll l, ll r) {
        if(lazy[i].is_identity()) {
            return;
        }

        if(l == r) {
            lazy[i] = U::identity();
            return;
        }

        ll m = (l + r) >> 1ll;

        lazy[(i << 1ll)] = lazy[(i << 1ll)] + lazy[i];
        v[(i << 1ll)] = lazy[i].upd(l, m, v[(i << 1ll)]);

        lazy[(i << 1ll) | 1ll] = lazy[(i << 1ll) | 1ll] + lazy[i];
        v[(i << 1ll) | 1ll] = lazy[i].upd(m + 1ll, r, v[(i << 1ll) | 1ll]);

        v[i] = v[(i << 1ll)] + v[(i << 1ll) | 1ll];
        lazy[i] = U::identity();
    }

    T _qry(ll i, ll l, ll r, ll ql, ll qr) {
        if(ql > r || qr < l) return T::identity();
        push(i, l, r);
        if(ql == l && qr == r) return v[i];

        ll m = (l + r) >> 1ll;

        T res = _qry(i << 1ll, l, m, ql, min(qr, m)) + _qry((i << 1ll) | 1ll, m + 1ll, r, max(ql, m + 1ll), qr);
        v[i] = v[(i << 1ll)] + v[(i << 1ll) | 1ll];
        return res;
    }

    T qry(ll ql, ll qr) {
        return _qry(1ll, 0ll, n - 1ll, ql, qr);
    }

    void _upd(ll i, ll l, ll r, ll ql, ll qr, const U& updfn) {
        if(ql > r || qr < l) return;
        push(i, l, r);

        if(ql == l && qr == r) {
            lazy[i] = lazy[i] + updfn;
            v[i] = updfn.upd(l, r, v[i]);
            return;
        }
        ll m = (l + r) >> 1ll;

        _upd(i << 1ll, l, m, ql, min(m, qr), updfn);
        _upd((i << 1ll) | 1ll, m + 1ll, r, max(ql, m + 1ll), qr, updfn);

        v[i] = v[i << 1ll] + v[(i << 1ll) | 1ll];
    }

    void upd(ll ql, ll qr, const U& updfn) {
        _upd(1ll, 0ll, n - 1ll, ql, qr, updfn);
    }
};


/*
(T, +) forms a monoid
+ is closed, associative, and has an identity in T
in this case, i'm doing range add
! if Mono is not associative, this fails
*/
struct Mono {
    ll v;

    Mono(ll a_v): v(a_v) {};

    inline Mono operator+(const Mono& o) const {
        return Mono(v + o.v);
    }
    
    static inline Mono identity() {
        return Mono(0ll);
    }
};

/*
(U, +) forms a unital magma.
+ is closed and has an identity in U

range assignment
set range to lazy

v -> (r - l + 1ll) * lazy
*/
struct Upd {
    ll lazy;
    bool id;
    Upd(ll a_lazy, bool is_id): lazy(a_lazy), id(is_id) {};

    // [l, r] is the interval over which we are applying this lazy update
    Mono upd(ll l, ll r, const Mono& old) const {
        if(id) return Mono(old.v);
        return Mono((r - l + 1ll) * lazy);
    }

    Upd operator+(const Upd& o) const {
        if(o.is_identity()) return Upd(lazy, id);
        return Upd(o.lazy, o.id);
    }
    
    bool is_identity() const {
        return id;
    }

    static inline Upd identity() {
        return Upd(0ll, true);
    }
};

// trim off the largest element before passing into this function
vector<ll> find_sum_maxima(const vector<ll>& a) {
    ll n = a.size();

    Tree<Mono, Upd> tr(n);

    tr.build(Mono(0ll));

    vector<ll> ans;

    for(ll i = 0ll; i < n; i++) {
        ll v = a[i];

        if(i == 0ll) {
            tr.upd(i, i, Upd(v, false));
        } else {
            /*
            find the largest index < i such that 
            its value is > v
            */
            ll l = -1ll, r = i;

            while(r - l > 1ll) {
                ll m = (l + r) >> 1ll;

                if(tr.qry(m, m).v > v) l = m;
                else r = m;
            }

            // update the suffix to the new maximum
            tr.upd(r, i, Upd(v, false));
        }

        ans.push_back(tr.qry(0ll, n - 1ll).v);
    }

    return ans;
}

void printv(const vector<ll>& a) {
    for(ll v : a) cerr << v << " ";
    cerr << endl;
}

int main() {
    ios_base::sync_with_stdio(false); cin.tie(0); cout.tie(0);

    ll t;
    cin >> t;

    while(t--) {
        ll n;
        cin >> n;

        vector<ll> h(n, 0ll);

        for(ll& v : h) cin >> v;

        // find largest and index of such
        ll maxi = 0ll;

        for(ll i = 1ll; i < n; i++) {
            if(h[i] > h[maxi]) {
                maxi = i;
            }
        }

        vector<ll> new_arr(n - 1ll, 0ll);

        for(ll i = 1ll; i < n; i++) {
            new_arr[i - 1ll] = h[(maxi + i) % n];
        }

        vector<ll> pref = find_sum_maxima(new_arr);

        reverse(new_arr.begin(), new_arr.end());

        vector<ll> suff = find_sum_maxima(new_arr);

        // printv(pref);
        // printv(suff);
        // cerr << "---" << endl;

        vector<ll> ans;

        for(ll i = 0ll; i < n; i++) {
            ll i1 = i - 1ll, i2 = n - 2ll - i;
            ans.push_back((i1 >= 0ll ? pref[i1] : 0ll) + (i2 >= 0ll ? suff[i2] : 0ll));
        }

        for(ll i = 0ll; i < n; i++) {
            cout << ans[(i - maxi + n - 1ll) % n] << " ";
        }
        cout << "\n";
    }

    cout << flush;

    return 0;
}
```

# I've Won, but at What Cost?

![AC1]({{site.baseurl}}/assets/images/vessels-heights-two-versions-1.png "AC1")

Indeed, this solution passes! I went to read the editorial and... wait what, $$O(n)$$ is possible??

At this point, I realized that my segtree was overkill. In fact, it seems to be a common pattern for me to immediately use these heavy machinery when something simpler suffices. And so, I asked myself:

![won-but-at-what-cost]({{site.baseurl}}/assets/images/won-but-at-what-cost.png)

So, I set out to implement the $$O(n)$$ solution instead of remaining reliant on segment trees for everything.

# An $$O(n)$$ Solution

Instead of using a segment tree for this particular problem, it turns out to be a *lot* simpler to use something known as a **monotonic stack**[^1].

[^1]: See page 40 of [this handout](https://redblazerflame.github.io/reboot-materials/compprog-materials/noiph-modules/ds3.pdf) for more information.

We may maintain a struct of running maxima in a stack. Then, when we insert a new element, we pop off all smaller elements in the stack first, then push the new element. This works because the monotonic stack (at least, for this problem) maintains the *invariant* that the *elements of the stack are nondecreasing from left to right*. Further, each element is pushed at most once and popped at most once. Even if we pop off multiple elements in one go, the number of operations remains at most $$O(n)$$ by amortization[^1].

For this problem in particular, we do have to maintain the current sum. This turns out to be doable if we augment the stack with information about the indices of its elements.

With that, witness as the code magically shortens *and* becomes 16 times faster! 🔥🔥🔥

```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
#define INF(type) numeric_limits<type>::max()

void printv(const vector<ll>& a) {
    for(ll v : a) cerr << v << " ";
    cerr << endl;
}

struct Stack {
    ll v;
    ll i;
    stack<pair<ll, ll>> s;

    Stack(): v(0ll), i(0ll) {};

    ll top_ind() {
        if(s.empty()) return -1ll;
        else return s.top().second;
    }

    void insert(ll x) {
        while((!s.empty()) && (s.top().first <= x)) {
            // remove topmost
            auto [tv, ti] = s.top();
            s.pop();

            v -= (ti - top_ind()) * tv;
        }

        v += (i - top_ind()) * x;
        s.push({x, i});
        i++;
    }
};

vector<ll> solve(const vector<ll>& a) {
    vector<ll> ans;

    Stack s;

    for(ll v : a) {
        s.insert(v);

        ans.push_back(s.v);
    }

    return ans;
}

int main() {
    ios_base::sync_with_stdio(false); cin.tie(0); cout.tie(0);

    ll t;
    cin >> t;

    while(t--) {
        ll n;
        cin >> n;

        vector<ll> h(n, 0ll);

        for(ll& v : h) cin >> v;

        ll maxi = distance(h.begin(), max_element(h.begin(), h.end()));

        vector<ll> hp(n - 1ll, 0ll);

        for(ll i = 0ll; i < n - 1ll; i++) {
            hp[i] = h[(maxi + i + 1ll) % n];
        }

        // printv(hp);
        vector<ll> pref = solve(hp);
        reverse(hp.begin(), hp.end());

        vector<ll> suff = solve(hp);

        // printv(pref);
        // printv(suff);

        vector<ll> ans;

        for(ll i = 0ll; i < n; i++) {
            ll i1 = i - 1ll, i2 = n - 2ll - i;
            ans.push_back((i1 == -1ll ? 0ll : pref[i1]) + (i2 == -1ll ? 0ll : suff[i2]));
        }

        for(ll i = 0ll; i < n; i++) {
            cout << ans[(i - maxi + n - 1ll) % n] << " ";
        }
        cout << "\n";
    }

    cout << flush;

    return 0;
}
```

![AC2]({{site.baseurl}}/assets/images/vessels-heights-two-versions-2.png "AC2")

# Reflection

One thing that I did well for this problem was to use functions such as `printv` to print the state. This allowed for easy debugging and removed the need for me to derive some formulae for the indices---I simply printed the results and changed the formulae until the results were correct.

This problem served, once again, as a stark reminder of searching for simpler solutions when possible. Sure, if I did encounter this problem in a contest, perhaps going for the segment tree was the right thing to do---especially since I had a template ready.

On the other hand, I have experienced problems with a tight time limit before, such as [Sanae, Cross, and Color](https://codeforces.com/problemset/problem/2228/D). In such cases, log factors begin to matter.

Simpler solutions lead to less mental overhead. I had to think about how the states and updates combine within the segment tree, as well as the relevant indices for binary search. Had I implemented a monotonic stack, I could have simply abstracted away the operations into a simple struct then not have to think about how the stack contents change.

Succinctly, one must **identify the best abstraction and technique** for a problem. The best way to build this intuition is, as it usually is, through practice.