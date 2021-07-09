## 约瑟夫环问题

问题描述请参考 [🐻度娘](https://baike.baidu.com/item/%E7%BA%A6%E7%91%9F%E5%A4%AB%E9%97%AE%E9%A2%98/3857719?fr=aladdin) .

在这里设一共有 $n$ 个人，标号分别为 $0,...,n-1$，剔除序号为 $m$ ，报数从 1 开始。

一种显而易见的解法是使用链表模拟：

```cpp
int main()
{
    int n = 5, m = 3;
    list<int> people;
    for (int i = 0; i < n; i++)
        people.push_back(i);
    int num = 0;
    auto itor = people.begin();
    while (people.size() > 1)
    {
        auto cur = itor;
        num++;
        if ((++itor) == people.end())
            itor = people.begin();
        if (num == m)
        {
            num = 0;
            people.erase(cur);
        }
    }
    cout << people.front() << endl;
}
```

这样做的算法复杂度是 $O(m*n)$ , 空间复杂度是 $O(n)$ 。

下面考虑数学解法。

（老实说，看了几个证明，都没看懂😅）

先放结论：$p_i = (p_{i-1} + m) \mod i, 2 \le i \le n$ .

初始条件：$p_1 = 0$ .

代码：

```cpp
int math(int n, int m)
{
    int p = 0;
    for (int i = 2; i <= n; i++)  p = (p + m) % i;
    return p;
}
```

