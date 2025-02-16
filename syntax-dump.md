Hello World! I'm just trying out the markdown syntax. Expect this page to make no sense at all! :D

$$g(n) = \sum_{d | n} \mu(d) \cdot f\left(\frac{n}{d}\right)$$

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