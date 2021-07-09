## ByteDance 2019 春招题目

牛客网字节跳动笔试真题：https://www.nowcoder.com/test/16516564/summary

分了 2 次做，磕磕碰碰才写完，弱鸡悲鸣。

## 1. 聪明的编辑

题目：[Link](https://www.nowcoder.com/questionTerminal/42852fd7045c442192fa89404ab42e92?answerType=1&f=discussion) .

### 两遍扫描

两遍扫描：第一次处理情况 1 ，第二次处理情况 2 。

```cpp
// 万万没想到之聪明的编辑
// https://www.nowcoder.com/test/question/42852fd7045c442192fa89404ab42e92?pid=16516564&tid=40818118
#include <string>
#include <iostream>
using namespace std;
string stupid_check(string &s)
{
    for (int i = 0; i + 1 < (int)s.length(); i++)
    {
        if (s[i] == s[i + 1])
        {
            if (i + 2 < (int)s.length() && s[i + 1] == s[i + 2])
                s.erase(s.begin() + i + 2), i--;
        }
    }
    for (int i = 0; i + 3 < (int)s.length(); i++)
    {
        if (s[i] == s[i + 1] && s[i + 2] == s[i + 3])
            s.erase(s.begin() + i + 3);
    }
    return s;
}
int main()
{
    int n;
    cin >> n;
    cin.ignore();
    while (n--)
    {
        string s;
        cin >> s;
        cin.ignore();
        cout << stupid_check(s) << endl;
    }
}
```

### 自动机

由题意可得以下有限自动机：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20210130193648.png" style="width:40%;" />

每个状态的转移条件为：当前字符与上一个字符是否相等。

```cpp
// 万万没想到之聪明的编辑
// https://www.nowcoder.com/test/question/42852fd7045c442192fa89404ab42e92?pid=16516564&tid=40818118
#include <string>
#include <iostream>
using namespace std;
string stupid_check(const string &s)
{
    int len = s.length();
    string result = "";
    char cur = s[0], last = s[0];
    int state = 0;
    result.append(1, cur);
    for (int i = 1; i < len; i++)
    {
        cur = s[i];
        switch (state)
        {
        case 0:
        {
            if (cur == last) state = 1;
            else state = 0;
            result.append(1, cur);
            break;
        }
        case 1:
        {
            if (cur == last) state = 1;
            else state = 2, result.append(1, cur);
            break;
        }
        case 2:
        {
            if (cur == last) state = 2;
            else state = 0, result.append(1, cur);
            break;
        }
        }
        last = cur;
    }
    return result;
}
int main()
{
    int n;
    cin >> n;
    cin.ignore();
    while (n--)
    {
        string s;
        cin >> s;
        cin.ignore();
        cout << stupid_check(s) << endl;
    }
}
```



## 2. 抓捕孔连顺

题目：[Link.](https://www.nowcoder.com/questionTerminal/c0803540c94848baac03096745b55b9b?answerType=1&f=discussion)

滑动窗口的思想，$O(n^2)$ 的解法超时。

```cpp
#include <iostream>
#include <vector>
using namespace std;
const uint64_t mod = 99997867;
int main()
{
    ios::sync_with_stdio(0);
    int n, d;
    cin >> n >> d;
    cin.ignore();
    vector<int> pos(n, 0);
    for (int i = 0; i < n; i++)
        cin >> pos[i];
    cin.ignore();

    uint64_t ans = 0;
    int i = 0;
    while (i + 2 < n)
    {
        int j = i + 2;
        while (j < n && ((pos[j] - pos[i]) <= d))
            j++;
        uint64_t t = j - i - 1;
        ans = (ans + t * (t - 1) / 2) % mod;
        i++;
    }
    cout << ans << endl;
}
```

优化一下，变成 $O(n)$ . 注意有的地方需要用 `uint64_t`，用 `int` 会溢出导致结果错误。

```cpp
#include <iostream>
#include <vector>
using namespace std;
const uint64_t mod = 99997867;
int main()
{
    ios::sync_with_stdio(0);
    int n, d;
    cin >> n >> d;
    cin.ignore();
    vector<int> pos(n, 0);
    uint64_t ans = 0;
    for (int i = 0, j = 0; i < n; i++)
    {
        cin >> pos[i];
        while (i >= 2 && pos[i] - pos[j] > d) j++;
        uint64_t t = i - j;
        if (t >= 2)
            ans = (ans + t * (t - 1) / 2) % mod;
        
    }
    cout << ans << endl;
}
```

## 3. 雀魂

题目：[Link.](https://www.nowcoder.com/questionTerminal/448127caa21e462f9c9755589a8f2416?answerType=1&f=discussion)

考虑暴力枚举方法（模拟法）：

- 利用哈希表 `table` 计算 13 个数字的频率
- 枚举 `1 - 9` 加入 `table`，检查 14 张牌是否能和牌

检查和牌的函数 `check(table)`：

- 在 `table` 选取次数大于等于 2 的作为雀头
- 检查剩下的 12 张牌是否能组成 4 对顺子/刻子

检查顺子/刻子的函数 `sub_check(table, n)`，`n` 表示剩下多少张牌可以检查。枚举 `1 - 9`：

- 如果该牌数目大于等于 3 ，说明可以组成刻子，继续检查 `sub_check(table, n-3)` .
- 如果 `table[i], table[i+1], table[i+2] ` 的数量都大于 1 ，说明可以组成顺子，继续检查 `sub_check(table, n-3)` .

代码：

```cpp
// 雀魂启动！
// https://www.nowcoder.com/question/next?pid=16516564&qid=362291&tid=40818118
#include <iostream>
#include <array>
#include <vector>
using namespace std;

// 剩下的 n 张是否能组成顺子或者刻子
bool sub_check(array<int, 10> &table, int n)
{
    if (n == 0) return true;
    for (int i = 1; i <= 9; i++)
    {
        if (table[i] >= 3)
        {
            table[i] -= 3;
            if (sub_check(table, n - 3)) return true;
            table[i] += 3;
        }
        if (i + 2 <= 9 && table[i] >= 1 && table[i + 1] >= 1 && table[i + 2] >= 1)
        {
            table[i]--, table[i + 1]--, table[i + 2]--;
            if (sub_check(table, n - 3)) return true;
            table[i]++, table[i + 1]++, table[i + 2]++;
        }
    }
    return false;
}
// 任意选取次数 >= 2 的牌作为雀头
bool check(array<int, 10> table)
{
    for (int i = 1; i <= 9; i++)
    {
        if (table[i] >= 2)
        {
            table[i] -= 2;
            if (sub_check(table, 12)) return true;
            table[i] += 2;
        }
    }
    return false;
}
int main()
{
    vector<int> result;
    array<int, 10> table = {0};
    int val;
    for (int i = 0; i < 13; i++)
    {
        cin >> val;
        table[val]++;
    }
    for (int x = 1; x <= 9; x++)
    {
        if (table[x] >= 4) continue;
        table[x]++;
        if (check(table)) result.push_back(x);
        table[x]--;
    }
    if (result.size() == 0) result.push_back(0);
    for (int x : result) cout << x << ' ';
    cout << endl;
}
```



## 4. 特征提取

题目：[Link.](https://www.nowcoder.com/questionTerminal/5afcf93c419a4aa793e9b325d01957e2?answerType=1&f=discussion)

用 `map` 记录连续出现的次数。

具体实现细节：`set` 记录本次出现的 `(x, y)` 。每次来一帧，如果 `map` 中的 `feature` 不在 `set` 中出现，说明该 `feature` 不是连续出现的，置为 0 。

代码：

```cpp
// 特征提取
// https://www.nowcoder.com/question/next?pid=16516564&qid=362292&tid=40818118
#include <iostream>
#include <map>
#include <set>
using namespace std;
struct feature
{
    int x, y;
    feature(int _x, int _y) : x(_x), y(_y) {}
    bool operator<(const feature &f) const { return x < f.x || (x == f.x && y < f.y); }
};
int main()
{
    ios::sync_with_stdio(0);
    int n;
    cin >> n;
    cin.ignore();
    map<feature, int> a;
    set<feature> s;
    a.clear(), s.clear();
    int ans = 1;
    while (n--)
    {
        int frames;
        cin >> frames;
        cin.ignore();
        while (frames--)
        {
            int k, x, y;
            cin >> k;
            for (int i = 0; i < k; i++)
            {
                cin >> x >> y;
                a[feature(x, y)]++;
                s.insert(feature(x, y));
            }
            for (auto &[key, val] : a)
            {
                ans = max(ans, val);
                if (s.count(key) == 0)
                    a[key] = 0;
            }
            s.clear();
        }
    }
    cout << ans << endl;
}
```

## 5. 毕业旅行问题

题目：[Link.](https://www.nowcoder.com/questionTerminal/3d1adf0f16474c90b27a9954b71d125d?answerType=1&f=discussion)

居然是 TSP 问题（离散数学中称之为哈密顿回路），我记得上课学的时候只会写贪心 😅 。

看了题解，可以用状态压缩的 DP 求解。[此处](https://www.cnblogs.com/sinkinben/p/14264235.html)的最后一道题也是状压 DP 。

状态定义 $dp[s,v]$ ，$s$ 表示没去过的城市集合，$v$ 表示当前所在城市。因此，$dp[0,0]$ 表示所有城市去过，并在所在地为 0 城市，即结束状态；$dp[2^n-1, 0]$ 表示所有城市都没去过，当前在 0 号城市，即开始状态。

定义 `is_visited(s, u)` 为集合 `s` 是否包含了城市 `u`, 即当前状态是否已经访问过 `u` .

定义 `set_zero(s, k)` 把 `s` 的从左往右数的第 `k` 比特置为 1 ，表示城市 `k` 已访问。

时间复杂度 $O(n^2\cdot2^n)$，空间复杂度 $O(n \cdot 2^n)$ .

代码：

```cpp
#include <iostream>
#include <vector>
using namespace std;
const int N = 21;
int graph[N][N] = {{0}};
int tsp(int n)
{
    vector<vector<int>> dp((1 << n), vector<int>(n, 0x3f3f3f3f));
    auto is_visited = [](int s, int u) { return ((s >> u) & 1) == 0; };
    auto set_zero = [](int s, int k) { return (s & (~(1 << k))); };
    dp[(1 << n) - 1][0] = 0;
    // double 'for' loop to fill the dp
    for (int s = (1 << n) - 1; s >= 0; s--)
    {
        for (int v = 0; v < n; v++)
        {
            // for current 's', try all the cities
            for (int u = 0; u < n; u++)
            {
                if (!is_visited(s, u))  // if 'u' has not been visited
                {
                    int state = set_zero(s, u);
                    dp[state][u] = min(dp[state][u], dp[s][v] + graph[v][u]);
                }
            }
        }
    }
    return dp[0][0];
}
int main()
{
    int n;
    cin >> n;
    cin.ignore();
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
            cin >> graph[i][j];
        cin.ignore();
    }
    cout << tsp(n) << endl;
}
```



## 6. 找零

题目：[Link.](https://www.nowcoder.com/questionTerminal/944e5ca0ea88471fbfa73061ebe95728?answerType=1&f=discussion)

老水题了。不过要注意的是：这里没有 2 和 8 面值的硬币（原来写了个 `for` 循环，浪费了一次提交 😅 ）。

```cpp
// 硬币找零
// https://www.nowcoder.com/question/next?pid=16516564&qid=362294&tid=40818118
#include <iostream>
using namespace std;
int main()
{
    ios::sync_with_stdio(0);
    int n;
    cin >> n;
    cin.ignore();
    n = 1024 - n;
    int ans = n / 64;
    n %= 64;
    ans += n / 16;
    n %= 16;
    ans += n / 4;
    n %= 4;
    ans += n;
    cout << ans << endl;
}
```



## 7. 机器人跳跃

题目：[Link](https://www.nowcoder.com/questionTerminal/7037a3d57bbd4336856b8e16a9cafd71) .

首先观察规律：

```
H: 0   h1  h2  h3  h4 ...
E: x0  x1  x2  x3  x4 ...
```

显然有：

```
x0 = 1 * x0 - h0
x1 = x0 + (x0 - h1) = 2 * x0 - h1
x2 = x1 + (x1 - h2) = 2 * x1 - h2
...
x[i] = 2 * x[i-1] - h[i]
```

显然对于每个 `x[i]` 都要求 `x[i] >= 0` .

所以有第一版的代码：

```cpp
#include <iostream>
using namespace std;
int main()
{
    ios::sync_with_stdio(0);
    int n;
    cin >> n;
    cin.ignore();
    int h = 0;
    int ans = 1;
    int64_t a = 1, b = 0;
    int x = 1;
    for (int i = 0; i < n; i++)
    {
        cin >> h;
        a = 2 * a, b = 2 * b - h;
        x = (-b) / a + 1;
        ans = max(x, ans);
    }
    cout << x << endl;
}
```

但不知道为什么，居然发生浮点错误 🤔 ？我寻思 `a` 始终都不能为 0 啊。

当我写下这句话的时候，突然想到 `a` 的最大值为 `2^n` ，如果 `n >= 64` 就会发生整型溢出，所以 `a, b` 换成 `double` 就 AC 了，我佛🌶️。

题解代码一则，已知 $x_{i} = 2x_{i-1} - h_{i}$，因此 $x_{i-1} = \lceil \frac{x_{i} + h_{i}}{2} \rceil$ .

```cpp
int main()
{
    ios::sync_with_stdio(0);
    int n;
    cin >> n;
    cin.ignore();
    vector<int> height(n, 0);
    for (int i = 0; i < n; i++)
        cin >> height[i];
    reverse(height.begin(), height.end());
    int e = 0;
    for (int h : height)
        e = (e + h + 1) / 2;
    cout << e << endl;
}
```

