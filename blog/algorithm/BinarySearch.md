## 二分查找

代码框架:

```cpp
int len = v.size();
int l=0, r=len-1;
while (l<=r)
{
    // m = (l+r)/2 会出现整型溢出
    int m = l + (r-l)/2;
    if (l == r)
    {
        // do something here
        // 防止死循环
        break;
    }
    if (...) r = m;
    else     l = m+1;
}
```

