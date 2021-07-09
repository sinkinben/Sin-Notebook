## [202103] Interview Summary

整理 2021 March「偷」到的算法题。

题目：

- 阿里：https://codeforces.com/contest/1465/problem/C
- 字节：输出 LCS
- Jump Trading：给出若干个任意坐标的点，判断是否能被 2 条直线覆盖。
- 美团：给定 n 个数字，k 为滑动窗口大小，输出每个窗口内的众数（如果有多个，输出最小的一个）。
- 微软：4 道题（亲自偷题）。



## 阿里

Code Force 上的题目。

首先，输入必然是保证每个棋子不在同一行且不在同一列的， `n x n` 的棋盘具有对称性，所以只需要考虑每个棋子的**横向移动**即可。

以下面的输入为例：

```text
n = 4, m = 3
2 3
3 1
1 2

+---+---+---+---+
|   | X |   |   |
+---+---+---+---+
|   |   | X |   |
+---+---+---+---+
| X |   |   |   |
+---+---+---+---+
|   |   |   |   |
+---+---+---+---+
```

把棋子所在的行作为棋子的 ID，要把每个棋子放在主对角线上，它们的位置冲突存在这样的依赖关系：`1 -> 3 -> 2 -> 1` ，显然这种情况最少需要移动 4 次（把 1 移动到其他位置打破依赖环），即 **依赖环的长度加 1**。

但同时存在这么一种可能，只存在一个依赖链，不存在环，比如：

```
+---+---+---+---+
|   | X |   |   |
+---+---+---+---+
|   |   | X |   |
+---+---+---+---+
|   |   |   | X |
+---+---+---+---+
|   |   |   |   |
+---+---+---+---+
```

上述情况存在一个依赖链 `3 -> 2 -> 1` ，那么只需要移动  3 次（1 可以直接移动到正确的位置）。

综合上述 2 种情况，假设输入有 **`m` 个不在主对角线的棋子**，如果它们形成了 `circle` 个环，那么至少需要移动 `m + circle` 次。

判断成环问题，可以使用并查集来解决。

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

vector<int> root;
int find(int x) { return root[x] == -1 ? x : root[x] = find(root[x]); }
void reset(int n) { root.clear(), root.resize(n, -1); }

int main()
{
    int t;
    cin >> t;
    cin.ignore();
    int n, m, x, y;

    while (t--)
    {
        cin >> n >> m;
        cin.ignore();
        reset(n + 1); // IDs range from 1 to n
        int circle = 0, match = 0;
        for (int i = 0; i < m; i++)
        {
            cin >> x >> y;
            cin.ignore();
            if (x == y)
            {
                match++;
                continue;
            }
            int rx = find(x), ry = find(y);
            if (rx != ry) root[ry] = rx;
            else circle++;
        }
        cout << (m - match) + circle << endl;
    }
}

/* Test Case
4
3 1
2 3
Output: 1
3 2
2 1
1 2
Output: 3
5 3
2 3
3 1
1 2
Output: 4
5 4
4 5
5 1
2 2
3 3
Output: 2
*/
```



## 字节

人已经麻了，但还是总结一下吧。

以前也写过：https://www.cnblogs.com/sinkinben/p/11512710.html

**输出最长公共子序列**

先默写一个求长度：

```cpp
int lcs(const string &a, const string &b)
{
    int alen = a.length(), blen = b.length();
    vector<vector<int>> dp(alen + 1, vector<int>(blen + 1, 0));
    for (int i = 1; i <= alen; i++)
    {
        for (int j = 1; j <= blen; j++)
        {
            if (a[i - 1] == b[j - 1]) dp[i][j] = dp[i - 1][j - 1] + 1;
            else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    return dp[alen][blen];
}
```

下面考虑如何输出，其关键点在于记住 `dp` 方程的转移「路径」，当前状态 `dp[i, j]` 是由哪一个状态转移而来的：`dp[i-1, j], dp[i, j-1], dp[i-1, j-1]` ?

开辟 `table` 数组记录即可，`dp[i-1, j], dp[i, j-1], dp[i-1, j-1]` 分别用字符 `|, -, \` 表示。

然后从 `dp[n1][n2]` 开始使用回溯法，求出完整的转移路径（使用回溯递归倒序输出），即完整的 LCS 串。

```cpp
void printlcs(const vector<vector<char> > &table, int i, int j, const string &astr)
{
    if (i < 0 || j < 0)
    {
        printf("\n");
        return;
    }
    char x = table[i][j];
    if (x == '-')  printlcs(table, i, j - 1, astr);
    if (x == '|')  printlcs(table, i - 1, j, astr);
    if (x == '\\') printlcs(table, i - 1, j - 1, astr), printf("%c", astr[i - 1]);
}
int lcs(const string &a, const string &b)
{
    int alen = a.length(), blen = b.length();
    vector<vector<int> > dp(alen + 1, vector<int>(blen + 1, 0));
    vector<vector<char> > table(alen + 1, vector<char>(blen + 1, 0));
    for (int i = 1; i <= alen; i++)
    {
        for (int j = 1; j <= blen; j++)
        {
            if (a[i - 1] == b[j - 1])
            {
                dp[i][j] = dp[i - 1][j - 1] + 1;
                table[i][j] = '\\';
            }
            else
            {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                table[i][j] = (dp[i - 1][j] > dp[i][j - 1]) ? '|' : '-';
            }
        }
    }
    printlcs(table, alen, blen, a);
    cout << endl;
    return dp[alen][blen];
}
int main()
{
    string a = "cnblog", b = "belong";
    // a = "xyxxzxyzxy", b = "zxzyyzxxyxxz";
    cout << lcs(a, b) << endl;
}
```



**输出最长公共子串**

这就更简单了.jpg

因为是连续的，只需要记住最长的长度，和最长公共子串的结束位置即可。

```cpp
int lcsubstring(const string &a, const string &b)
{
    int alen = a.length(), blen = b.length();
    vector<vector<int> > dp(alen + 1, vector<int>(blen + 1, 0));
    int maxlen = 0, maxidx = -1;
    for (int i = 1; i <= alen; i++)
    {
        for (int j = 1; j <= blen; j++)
        {
            if (a[i - 1] == b[j - 1]) dp[i][j] = dp[i - 1][j - 1] + 1;
            else dp[i][j] = 1;
            if (dp[i][j] >= maxlen) maxlen = dp[i][j], maxidx = i;
        }
    }
    cout << a.substr(maxidx - maxlen + 1, maxlen) << endl;
    return maxlen;
}
int main()
{
    string a = "xxxyxxxaaabcaaa";
    string b = "yyxxxyxxxaayabcaaa";
    cout << lcsubstring(a, b) << endl;
}
```



## 美团

给定 n 个数字，k 为滑动窗口大小，输出每个窗口内的众数（如果有多个，输出最小的一个）。

数据范围：$1 \le k \le n \le 10^5$ .

**Sample**

```
Input: n=5, k=3, nums=[1,2,2,1,3]
Output: [2,2,1]
```

在 Google 找到了洛谷上的讨论帖：https://www.luogu.com.cn/discuss/show/238648 

但代码被美团的神秘力量盗走了 😅 。

首先，很自然的想法，对于窗口内的数字 `val`，建立哈希表记录它出现的次数：`val -> times`。

但如果通过遍历哈希表的方法找到最大次数，那么算法复杂度为 $O(nk)$ ，显然会超时。

考虑再用一个 `map<int, set<int>>` 把窗口内出现相同次数的 `val` 放在一起，即：`times -> [vals, ...]` . 由于 `map` 的有序性，窗口内的最小众数就是：`*((map.rbegin()->second).begin())` ，该做法的时间复杂度为 $O(n\log{k})$ .

但好像还是超时了，只通过 91% 。

```cpp
int main()
{
    int n, m;
    cin >> n >> m; cin.ignore();
    vector<int> v(n);
    for (int i = 0; i < n; i++)  cin >> v[i];
    
    unordered_map<int, int> table; // val -> times
    map<int, set<int>> rtable;     // rev-table: times -> [vals, ...]
    int i = 0;
    while (i < m)
    {
        int times = table[v[i]]+1;
        table[v[i]]++;
        rtable[times].insert(v[i]);
        rtable[times - 1].erase(v[i]);
        rtable.erase(times-1);
        i++;
    }
    cout << *((rtable.rbegin()->second).begin()) << endl;
    int j = i;
    i = 0;
    while (j < n)
    {
        // 窗口左端
        int times = table[v[i]];
        rtable[times].erase(v[i]);
        if (rtable[times].empty()) rtable.erase(times);
        rtable[--table[v[i]]].insert(v[i]);
        i++;
        // 窗口右端
        times = table[v[j]];
        rtable[times].erase(v[j]);
        if (rtable[times].empty()) rtable.erase(times);
        rtable[++table[v[j]]].insert(v[j]);
        j++;
        cout << *((rtable.rbegin()->second).begin()) << endl;
    }
}
```

做法好像有点像 LFU 😅 。



## Jump Trading

给定若干个点的坐标 $(x_i, y_i)$ ，判断这些点是否能被 2 条直线覆盖。

一道面试题（非机考），还有个边界不确定：所有点都在一条直线上，结果是否为 True ？此处暂且默认这种情况返回 True 。

又发现了这是 Codeforces 上的原题：https://codeforces.com/contest/961/problem/D 

### 暴力

考虑暴力解法，枚举任意 2 个点（确定一条直线），把点集分为 2 个集合，一个是在该直线上的点的集合，另一个是不在该直线上的点的集合 `notmatch`，然后判断 `notmatch` 的点是否在同一直线，若是，说明原集合可被 2 直线覆盖。

```cpp
#include <iostream>
#include <vector>
#include <set>
using namespace std;
typedef pair<int, int> point;
typedef pair<point, point> line;
inline bool sameLine(point &p0, point &p1, point &p2)
{
    int x0, y0, x1, y1, x2, y2;
    tie(x0, y0) = p0, tie(x1, y1) = p1, tie(x2, y2) = p2;
    return (x1 - x0) * (y2 - y0) == (y1 - y0) * (x2 - x0);
}
inline bool sameLine(line &l, point &p) { return sameLine(l.first, l.second, p); }
inline bool coveredBy2Lines(vector<point> &p, line l)
{
    vector<point> notmatch;
    for (auto &x : p)
        if (!sameLine(l, x)) notmatch.emplace_back(x);

    int len = notmatch.size();
    for (int i = 2; i < len; i++)
    {
        if (!sameLine(notmatch[0], notmatch[1], notmatch[i]))
            return false;
    }
    return true;
}
inline bool solve(vector<point> &p, int n)
{
    if (n <= 4) return true;
    for (int i = 0; i < n; i++)
    {
        for (int j = i + 1; j < n; j++)
            if (coveredBy2Lines(p, line(p[i], p[j]))) return true;
    }
    return false;
}
int main()
{
    int n, x, y;
    cin >> n;
    vector<point> points;
    for (int i = 0; i < n; i++)
    {
        cin >> x >> y;
        points.emplace_back(x, y);
    }
    cout << std::boolalpha << solve(points, n) << endl;
}
```

### 改进

在某个懒狗 🐶 帮助下，发现可有 $O(n)$ 的解法。

算法思想：任选一个点 `x` ，与其余的 n-1 个点分别求出斜率，把具有相同斜率的点放在同一集合；然后，找出**元素数目最多的集合 `s`**，那么 `x` 与 `s` 所在的直线必然是所求的 2 个直线之一；最后判断 $s \cup \{x\}$ 集合之外的点集（即补集）是否在同一直线即可。

为什么呢？可以通过反证法证明，如果该直线并非所求，那么要覆盖这个直线上的所有点，必然需要 m 条不同的直线，m 为上述集合 $s \cup \{x\}$ 的大小。`m >= 3` 时，矛盾是显然的，下面考虑 `m = 2` 的情况。

先看一个例子：

```text
y
^
|    0
| 1  2  3 
|                 4
+-------------------->x
```

显然，这里只有 1 种方案去覆盖所有的点：`[1,2,3], [0,4]` .

如果我们开始选择的点是 0，那么根据「斜率相同」来对剩下的点进行分类，会得到：

```text
selected point: x = 0
k1 -> [1]
k2 -> [2]
k3 -> [3]
k4 -> [4]
```

可以发现，这里 4 个集合的元素数目均为 1 ，即 $size(s \cup \{x\}) = 2$ 。如果我们选择 $s = [1]$ ，显然，直线 `[0, 1]` 不是所求之一，剩下的点 `2,3,4` 也不在同一直线上，所以我们的算法会返回 False ，与预期的 True 相悖。

那么这种情况下，我们考虑枚举，把 $s$ 分别取 `[1], [2], [3], [4]` ，判断 $s \cup \{x\}$ 的补集是否在同一直线，这样的做法时间复杂度是 $O(n^2)$ .

但似乎没有必要这样做？我们可以尝试 3 次（加常数级的复杂度，至少 3 次，不能 2 次）， `x` 分别取 `0, 1, 2` ，重复上述算法，只要有一次成功，则返回 True 。

> 个人的直觉，引起上述情况的原因是：某个输入有且只有 1 种方案覆盖，并且其中的一条直线只有 2 个点在上面，如果我们选择了这 2 个点之一，那么这种边界情况就会发生。

上述过程看起来很复杂，但是分析下来其实只有 2 点：

- 任选一个点，找到与它共线的 2 个点，该直线是所求之一，判断该直线外的点是否共线即可
- 如果找不到与之共线的 2 个点，那么换一个点重做一遍（最多做 3 次）



### 实现

表示点和线的数据结构：

```cpp
typedef pair<int, int> point_t;
typedef pair<point_t, point_t> line_t;
```

表示斜率的数据结构：

```cpp
// 分数 up/down 表示斜率
// up = 0 && down = 0 表示斜率不存在
// up = 0 && down = 1 表示斜率为 0 
// 若斜率为负数，默认 up 为负，down 为正
struct slope_t
{
    int up, down;
    slope_t(int _up, int _down)
    {
        if (_down == 0) up = down = 0;
        else if (_up == 0) up = 0, down = 1;
        else
        {
            int k = gcd(max(_up, _down), min(_up, _down));
            up = _up / k;
            down = _down / k;
            if (((up >> 31) & 1) == 0 && ((down >> 31) & 1) == 1)
                up = -up, down = -down;
        }
    }
    bool operator==(const slope_t &slope) const { return slope.up == up && slope.down == down; }
    int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }
};
```

为了把 `slope_t` 作为 `unordered_map` 的 key，我们还需要自定义哈希函数：

```cpp
struct hash_slope
{
    size_t operator()(const slope_t &slope) const
    { return (hash<int>()(slope.up) << 1) ^ (hash<int>()(slope.down)); }
};
```

2 个 API，`getslope` 通过 2 个点得到一个斜率，`sameLine` 判断三点是否共线。

```cpp
slope_t getslope(point_t &p1, point_t &p2)
{ return slope_t(p1.second - p2.second, p1.first - p2.first); }

bool sameLine(point_t &p0, point_t &p1, point_t &p2)
{
    int x0, y0, x1, y1, x2, y2;
    tie(x0, y0) = p0, tie(x1, y1) = p1, tie(x2, y2) = p2;
    return (x1 - x0) * (y2 - y0) == (y1 - y0) * (x2 - x0);
}
```

最后是代码的主逻辑：

```cpp
bool solve(vector<point_t> &p, int selected)
{
    int n = p.size();
    if (n <= 4) return true;
    unordered_map<slope_t, unordered_set<int>, hash_slope> table;
    // itor->second 指向元素数目最多的 unordered_set
    auto itor = table.end();
    for (int i = 0; i < n; i++)
    {
        if (i == selected) continue;
        slope_t slope = getslope(p[selected], p[i]);
        table[slope].insert(i);
        if (itor == table.end() || itor->second.size() <= table[slope].size())
            itor = table.find(slope);
    }
    auto s(itor->second);
    s.insert(selected);
    vector<point_t> notmatch;
    for (int i = 0; i < n; i++)
    {
        if (s.count(i) == 0)
            notmatch.emplace_back(p[i]);
    }
    int len = notmatch.size();
    for (int i = 2; i < len; i++)
    {
        if (!sameLine(notmatch[0], notmatch[1], notmatch[i]))
            return false;
    }
    return true;
}

int main()
{
    int n, x, y;
    cin >> n;
    vector<point_t> points;
    for (int i = 0; i < n; i++)
    {
        cin >> x >> y;
        points.emplace_back(x, y);
    }
    cout << std::boolalpha
         << (solve(points, 0) || solve(points, 1) || solve(points, 2))
         << endl;
}
```



### 简化

过去 2 天了，突然又想到了更简洁的解法，而且不用定义 `slope_t` 这个反人类的结构。

继续化简一下思路，其实就是：

- 先判断是否一条直线覆盖所有的点
- 如果一条直线不能覆盖所有，找到不共线的 3 个点，那么会有 3 条直线，3 条直线其中之一是所求的直线。对于每个直线，判断该直线的补集是否均在同一直线。

应该没有比这更简洁的解法了吧（嗯，普通却自信.jpg）

代码实现：

```cpp
#include <iostream>
#include <vector>
#include <tuple>
using namespace std;
typedef pair<int, int> point_t;
typedef pair<point_t, point_t> line_t;
inline bool sameline(point_t &p0, point_t &p1, point_t &p2)
{
    int64_t x0, y0, x1, y1, x2, y2;
    tie(x0, y0) = p0, tie(x1, y1) = p1, tie(x2, y2) = p2;
    return (x1 - x0) * (y2 - y0) == (y1 - y0) * (x2 - x0);
}
inline bool sameline(line_t &l, point_t &p)
{ return sameline(l.first, l.second, p); }

inline int oneline(vector<point_t> &p, int n)
{
    int i = 2;
    while (i < n && sameline(p[0], p[1], p[i])) i++;
    return i;
}
inline bool twolines(vector<point_t> &p, line_t &&l)
{
    vector<point_t> notmatch;
    for (auto &x : p)
        if (!sameline(l, x))
            notmatch.emplace_back(x);
    int size = notmatch.size();
    for (int i = 2; i < size; i++)
        if (!sameline(notmatch[0], notmatch[1], notmatch[i]))
            return false;
    return true;
}
inline bool solve(vector<point_t> &p, int n)
{
    if (n <= 4) return true;
    int idx = oneline(p, n);
    return idx == n ||
           twolines(p, line_t(p[0], p[1])) ||
           twolines(p, line_t(p[0], p[idx])) ||
           twolines(p, line_t(p[1], p[idx]));
}
int main()
{
    int n, x, y;
    cin >> n;
    vector<point_t> p;
    for (int i = 0; i < n; i++)
    {
        cin >> x >> y;
        p.emplace_back(x, y);
    }
    if (solve(p, n)) cout << "YES" << endl;
    else cout << "NO" << endl;
}
```



## 微软

2 轮面试，一共 4 道题。我太蒻了。

给 Microsoft 的面试官好评，拿到题目上来写不出最优解，面试官会给出适当的提示，然后提点一步一步写出来。然而我太蒻了，2 道题，一般第二道时间不够，只是口述和写伪代码了。此外，微软的面试会要求看写代码的过程，代码习惯、风格应该也是考核范围之内。

后知后觉，我才发现二面的 2 道题都是单调栈/队列的题目（然而写得不熟练，不说了赶紧背几道题了）。

面试心得：写题还是不利索，基础不牢，地动山摇。

### 强众数

好像是 leetcode 原题，找出的出现次数大于 $n/2$ 的众数。

要求时间 $O(n)$，空间 $O(1)$ .

题目：https://leetcode-cn.com/problems/majority-element/

原本是用哈希表做的，在提示下最终还好写出来了。

```cpp
class Solution {
public:
    const int nil = 0x80000000;
    int majorityElement(vector<int>& nums) {
        int cur = nil, cnt = 0;
        for (int x: nums)
        {
            if (cur == nil) cur = x;
            if (x == cur) cnt++;
            else cnt--;
            if (cnt == 0) cur = nil;
        }
        return cnt > 0 ? cur : nil;
    }
};
```



### 建立水库覆盖城市

给定一个 `height[n, m]` 矩阵，每个元素表示高度，水流只能从高向低流。

假设我们需要从 `height[0]` 这一行中选取一些点，建立水库，要求在 `height[n-1]` 的 `m` 个城市都要被水流覆盖到，解决下面 2 个问题：

- 输出 true/false，表示 `m` 个城市是否都能被覆盖。
- 如果为 true，输出选取哪些点（要尽可能少）；如果为 false，输出哪些城市不能被覆盖。

要求时间为 $O(mn)$ 的解法。

当时的回答：使用 BFS，对每个 `height[0][i]` 做一次 BFS，然后使用 `vis[n][m]` 记录哪些位置是可达的。最后，在 `vis[n-1]` 这一行中，如果 `vis[n-1][j] == false` 说明该城市不能覆盖（水流不可达）。但这样的做法无法解决「选取哪些点建立水库」这个问题。

感觉可以用贪心来做，在 `height[0]` 这一行中，从高度最高的点开始做 BFS ，如果本次 BFS 没有覆盖到新的城市，说明这个点不用取。

没找到原题，作罢。



### 求每个窗口最大值

给定 `nums[n], k` ，`k` 是滑动窗口大小，返回每个窗口的最大值（一共 `n-k+1` 个）。

题目：https://leetcode-cn.com/problems/sliding-window-maximum/

我发现是个 Hard 题 😅 。

写了一个 $O(n\log{n})$ 的解法，用 `map` 记录维护窗口内的值。面试官说没问题（对方好像不懂 C++ ，所以就没仔细解释代码）。当时没考虑 `k >= n` 的情况，还被对方揪了一下小辫子 😅 。

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        vector<int> res;
        map<int, int> table;
        int l = 0, r = 0, n = nums.size();
        if (k > n) k = n;
        while (r < k) table[nums[r++]]++;
        res.push_back(table.rbegin()->first);
        while (r < n)
        {
            table[nums[l]]--;
            if (table[nums[l]] == 0) table.erase(nums[l]);
            l++;
            table[nums[r++]]++;
            res.push_back(table.rbegin()->first);
        }
        return res;
    }
};
```

单调队列解法，时间复杂度是 $O(n)$ 。

参考：https://www.cnblogs.com/sinkinben/p/14616524.html





### 两端的更小元素

给定 `nums[n]`，输出每个元素，离它最近且比它小的下标（左端和右端各一个）。

要求只能扫描一遍。

当时回答：

- 用 `map<int, int>` 记录每个元素下标，时间复杂度是 $O(n\log{n})$，空间是 $O(n)$ . （有没有更好的？）
- 单调栈，第一次从左往右扫描，找出每个元素的「左端」，第二次从右往左扫描，找出每个元素的「右端」。（能不能只扫描一遍？）

好吧，这时候我已经卡壳了。最后对面直接给我说怎么做我才想明白，伪代码也没时间写。

题解：https://www.cnblogs.com/sinkinben/p/14616524.html
