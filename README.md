# protondecay314-blog

A blog for my random musings. It mostly includes CompProg and Math, but I might write about other stuff occasionally

Hello World!

$$g(n) = \sum_{d | n} \mu(d) \cdot $$

```c++
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

int main() {
    ll n;
    cin >> n;

    ll l = 0ll, r = n;

    while(r - l > 1ll) {
        ll m = (l + r) >> 1ll;

        if(m * m <= n) {
            l = m;
        } else {
            r = m;
        }
    }

    cout << "floor(sqrt(n)) = " << n << endl;

    return 0;
}
```