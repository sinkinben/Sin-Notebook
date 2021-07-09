## ByteDance 2018 后端题目

题目：https://www.nowcoder.com/test/8537209/summary

容我吐槽一句：居然有问答题，最后一题考高并发也就算了，还考分布式存储，本科生能有几个人读过 CAP 这篇论文呢（研究生也没几个人），卷不卷啊，太卷了.jpg

## 用户喜好

题目：[Link](https://www.nowcoder.com/questionTerminal/66b68750cf63406ca1db25d4ad6febbf?answerType=1&f=discussion)

通过 `unordered_map<int, vector<int>>` 把具有相同喜好度的用户分类在一起，保证在 `vector` 中，用户 ID 是有序的。

对于特定的 `<l,r,k>` 在 `map[k]` 这个 `vector` 中用二分查找即可。

`lower_bound(begin, end, x)` 是找到区间 `[begin, end)` 中第一个大于等于 `x` 的位置。

`upper_bound(begin, end, y)` 是找到区间 `[begin, end)` 中第一个大于 `y` 的位置。

两个库函数均通过二分查找实现。

```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
using namespace std;
int main()
{
    ios::sync_with_stdio(0);
    int n;
    cin >> n;
    cin.ignore();
    int val;
    unordered_map<int, vector<int>> table;
    for (int i = 1; i <= n; i++)
    {
        cin >> val;
        table[val].emplace_back(i);
    }
    int q;
    cin >> q;
    cin.ignore();
    int l, r, k;
    while (q--)
    {
        cin >> l >> r >> k;
        cin.ignore();
        if (table.count(k) == 0 || table[k].empty())
        {
            cout << 0 << endl;
            continue;
        }
        const auto &v = table[k];
        auto litor = lower_bound(v.begin(), v.end(), l);
        auto ritor = upper_bound(v.begin(), v.end(), r);
        cout << distance(litor, ritor) << endl;
        // cout << (ritor - litor) << endl;
    }
}
```

## 手串

题目：[Link](https://www.nowcoder.com/questionTerminal/429c2c5a984540d5ab7b6fa6f0aaa8b5?answerType=1&f=discussion)

使用 `vector<unordered_set<int>> beads` 去记录每个珠子所包含的颜色。

枚举每一种颜色 `color` ，然后使用**滑动窗口**去检查该颜色是否满足条件。时间复杂度为 $O(c(n+m))$ .

```cpp
#include <iostream>
#include <unordered_map>
#include <unordered_set>
#include <vector>
using namespace std;
int n, m, c;
bool invalidColor(vector<unordered_set<int>> beads, int color)
{
    int l = 0, r = 0;
    int counter = 0;
    for (r = 0; r < m; r++)
    {
        counter += beads[r].count(color);
        if (counter >= 2)
            return true;
    }
    // l=0, r=m, [0, m) has been checked
    while (l < n)
    {
        counter -= beads[l].count(color), l++;
        counter += beads[r].count(color), r = (r + 1) % n;
        if (counter >= 2)
            return true;
    }
    return false;
}
int main()
{
    ios::sync_with_stdio(0);
    cin >> n >> m >> c;
    cin.ignore();
    m = min(n, m);
    vector<unordered_set<int>> beads(n, unordered_set<int>());
    int k, v;
    for (int i = 0; i < n; i++)
    {
        cin >> k;
        while (k--)
        {
            cin >> v;
            beads[i].insert(v);
        }
    }
    int result = 0;
    for (int i = 1; i <= c; i++)
        result += invalidColor(beads, i);
    cout << result << endl;
}
```



## 右边界二分查找

以下函数使用二分查找搜索一个增序的数组，当有多个元素值与目标元素相等时，返回最后一个元素的下标，目标元素不存在时返回-1。请指出程序代码中错误或不符最佳实践的地方（问题不止一处，请尽量找出所有你认为有问题的地方）

```cpp
int BinarySearchMax(const std::vector<int>& data, int target)
{
   int left = 0;
   int right = data.size();   // 搜索区间为 [l, r)
   while (left < right) {
       int mid = (left + right) / 2; //可能溢出
       if (data[mid] <= target)
           left = mid + 1;
       else
           right = mid - 1;   // [l, r) 区间，此处应为 r = m
   }
   // if (right < size && data[right] == target)
   if (data[right] == target)
       return right;
   return -1;
}
```

错误很多，不如直接默写一个双闭区间搜索的模版：

```cpp
int BinarySearchMax(const std::vector<int>& data, int target)
{
   int left = 0;
   int right = data.size() - 1;
   while (left <= right) {
       int mid = left + (right - left) / 2;
       if (data[mid] <= target) left = mid + 1;
       else if (data[mid] > target) right = mid - 1;
   }
   if (right < 0 || data[right] != target)
       return -1;
   return right;
}
```

至于「字符交换」这道诡异的 DP 和最后一题算了，真的没做出来，题解看了也不太懂 🤡 。