## 树状数组

树状数组 (Binary Index Tree, BIT) 用于解决这样一个问题：给定数组 `a[n]`， 并且要求 `w` 次修改数组，现有 `q` 次区间查询，区间查询要求返回任意给定区间之和。

如果采用暴力方法，一次修改需要 $O(1)$ 的时间复杂度，一次查询需要 $O(n)$ 的时间复杂度，总的时间复杂度为 $O(qn+w)$ .

如果使用前缀和的方法，即维护数组 $sum[i] = \sum_{j=0}^{i}a[j]$ 。那么一次修改操作的时间复杂度为 $O(n)$ ，一次查询的时间复杂度为 $O(1)$，总的时间复杂度为 $O(wn+q)$ ，空间复杂度为 $O(n)$ .

树状数组可以实现一次修改和一次查询的时间复杂度为 $O(\log{n})$ . 最简单的树状数组可以支持 2 种操作：

- **单点修改**：更改数组中一个元素的值
- **区间查询**：查询一个区间内所有元素的和

树状数组与前缀和数组有点类似：用一个数组 $C$ 维护若干个小区间，单点修改时，只更新包含这一元素的区间。

为了适应树状数组这一数据结构，把数组 $a$ 的下标改为 $[1, n]$ .

定义`lowbit(i) = x & (-x)` ，这一操作实际上是：保留 `i` 的二进制表示中最右边的 1 。例如二进制数 $i=(1010)_2$, 那么 $lowbit(i) = (0010)_2$ . 数组 $C$ 中的一个元素 $C[i]$ 代表的是数组 $a$ 的 $(i-lowbit(i), i]$ 的区间之和（这是把 $a$ 的下标改为从 1 开始的原因）。

那么如何求出区间 `[a, b]` 之和？这一操作可以通过 `sum([1, b]) - sum([1, a-1])` 实现，那么现在需要解决如何实现求区间 `[1, i]` 之和。以 `i = 11` 为例子：

```c
// How to get the sum of a[1, ..., 11]
k = 0;
lowbit(11) = 10, k += sum{ (10, 11] } = C[11];
lowbit(10) =  8, k += sum{ ( 8, 10] } = C[10];
lowbit( 8) =  0, k += sum{ ( 0,  8] } = C[8];
// Thus
sum([1,11]) = C[11] + C[10] + C[8];
```



**代码实现**

```cpp
const int n = 1024;
vector<int> bitree(n, 0);
int lowbit(int x) { return x & (-x); }
void update(int i, int k)
{
    while (i <= n) bitree[i] += k, i += lowbit(i);
}
// get sum of [1, i]
int getsum(int i)
{
    int k = 0;
    while (i >= 1) k += bitree[i], i -= lowbit(i);
    return k;
}
// get sum of [x, y]
int getsum(int x, int y) { return getsum(y) - getsum(x-1); }
```

例题：https://loj.ac/p/130

代码：

```cpp
#include <iostream>
#include <vector>
using namespace std;
int n, q;
vector<int64_t> bitree, arr;
int lowbit(int64_t x) { return x & (-x); }
void update(int i, int64_t k) { while (i <= n) bitree[i] += k, i += lowbit(i); }
int64_t getsum(int i)
{
    int64_t k = 0;
    while (i) k += bitree[i], i -= lowbit(i);
    return k;
}
int main()
{
    int64_t op, x, y;
    cin >> n >> q;
    cin.ignore();
    bitree.resize(n + 1, 0), arr.resize(n + 1, 0);
    for (int i = 1; i <= n; i++) cin >> arr[i];
    cin.ignore();
    for (int i = 1; i <= n; i++) update(i, arr[i]);
    while (q--)
    {
        cin >> op >> x >> y;
        cin.ignore();
        if (op == 1) update(x, y);
        else cout << getsum(y) - getsum(x - 1) << endl;
    }
}
```



