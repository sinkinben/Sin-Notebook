## 基姆拉尔森公式

基姆拉尔森公式 (Kim Larsen Calculation Formula) 用于给定年份 $y$ , 月份 $m$ 和日子 $d$ 的条件下，计算该天是星期几。

初始条件：从公元 0 年 1 月 1 日，星期日开始计算（PS：公元 0 年不是闰年）。

输入： $y, m, d$ 三个整数表示年月日。

输出：$w \in [0, 6]$ 分别表示星期日到星期六。

下面为推导过程。

对于公元 0 年的第一个月：
$$
w = (d-1) \mod 7
$$
考虑公元 0 年后的年份，**不考虑闰年（即假设每年均为 365 天）**，因为 `365 % 7 == 1`，所以公元 1 年第一天是星期一，公元 2 年第二天是星期二，以此类推。在不考虑闰年的情况下，对于公元 $y$ 年的 1 月：
$$
w = (d-1+y) \mod 7
$$
考虑闰年，在 $[0,y-1]$ 区间内假设有 $k$ 个闰年，那么公元 $y$ 年第一个月的 $w$ 需要加上 $k$ 进行修正。

闰年的条件是：整除 400 或者整除 100 但不能整除 4 。所以 $k = (y-1)/400 + ((y-1)/4 - (y-1)/100)$ ，其中 `/` 表示 C 语言的整型除法。

所以，在考虑闰年的情况下，公元 $y$ 年的 **1 月**的星期几：
$$
w = [d-1 + y + (y-1)/400 + ((y-1)/4 - (y-1)/100)] \mod 7
$$
接下来，将上述公式推广到公元 $y$ 年的任意月份。

公元 $y$ 年的第 29 天的 $w$ 值总是与第一天相同的。以 28 为基准，某些月份会多出几天的偏移，比如日期 `0/1/1` 是星期日，`w = 0`，1 月的偏移为 3 ，所以 日期 `0/2/1` 是星期三，`w = 3` 。

现在我们需要找出每个月份基于 1 月的偏移量（暂不考虑闰年）。

| 月份 | 累计偏移 | 该月偏移 | 累计偏移模 7 |
| :--: | :------: | :------: | :----------: |
|  1   |    0     |    3     |      0       |
|  2   |    3     |    0     |      3       |
|  3   |    3     |    3     |      3       |
|  4   |    6     |    2     |      6       |
|  5   |    8     |    3     |      1       |
|  6   |    11    |    2     |      4       |
|  7   |    13    |    3     |      6       |
|  8   |    16    |    3     |      2       |
|  9   |    19    |    2     |      5       |
|  10  |    21    |    3     |      0       |
|  11  |    24    |    2     |      3       |
|  12  |    26    |    3     |      5       |



以数组记录表格的第 4 列：`disp[] = {0, 0, 3, 3, 6, 1, 4, 6, 2, 5, 0, 3, 5}` .

所以，不考虑考虑闰年的情况下，对于任意的 $m$ 有：
$$
w = \{d-1 + y + (y-1)/400 + ((y-1)/4 - (y-1)/100) + disp[m]\} \mod 7
$$
如果考虑闰年，那么 2 月之后的月份的 $w$ 值都需要加 1 处理。

代码表示即为：

```cpp
#define isleap(y) (((y) % 400 == 0) || (((y) % 4 == 0) && ((y) % 100 != 0)))
int week(int y, int m, int d)
{
    int disp[] = {0, 0, 3, 3, 6, 1, 4, 6, 2, 5, 0, 3, 5};
    int w = ((d - 1) + y + ((y - 1) / 400 + (y - 1) / 4 - (y - 1) / 100) + disp[m]) % 7;
    if (m > 2 && y != 0 && isleap(y))
        w = (w + 1) % 7;
    return w;
}
```

-----

下面考虑如何优化，因为 `disp` 数组每次使用都要手动算一遍。当然，下面很多技巧都是根据已有的结果反推过程而已。

假设把 `n` 年的 1 月和 2 月「划分」到 `n-1` 年，作为 `n-1` 年的第 13 月和 14 月。而**每年的第一天是 3 月 1 日**，即每年的月份范围是 $[3, 14]$ .

公元 0 年 3 月 1 日是星期三，所以对于公元 0 年 3 月有：
$$
w = (d+2) \mod 7
$$
考虑闰年，对于公元 $y$ 年的 3 月，这时候需要考虑的是 $[0,y]$ 范围内的闰年个数，`/` 表示 C 语言整型除法：
$$
w = [d+2+y+(y/400+y/4-y/100)] \mod 7
$$
偏移量表格：

| 月份 | 累计偏移 | 该月偏移 | 累计偏移模7 |
| :--: | :------: | :------: | :---------: |
|  3   |    0     |    3     |      0      |
|  4   |    3     |    2     |      3      |
|  5   |    5     |    3     |      5      |
|  6   |    8     |    2     |      1      |
|  7   |    10    |    3     |      3      |
|  8   |    13    |    3     |      6      |
|  9   |    16    |    2     |      2      |
|  10  |    18    |    3     |      4      |
|  11  |    21    |    2     |      0      |
|  12  |    23    |    3     |      2      |
|  13  |    26    |    3     |      5      |
|  14  |    29    |    0     |      1      |

暂时引入 `disp[m] (3 <= m <= 14)` 记录累计偏移模 7 .

考虑闰年，推广到任意月份 $m$ ：
$$
w = \{d+2+y+(y/400+y/4-y/100)+disp[m]\} \mod 7 \quad (3 \le m \le 14)
$$
这时候 2 月是该年的最后一个月，不需要考虑上述「如果考虑闰年，那么 2 月之后的月份的 $w$ 值都需要加 1 处理」的情况。

站在巨人的肩膀上，可以发现（`/` 表示整型除法）：
$$
disp[m] = (2m+3(m+1)/5-1) \mod 7, \quad (3 \le m \le 14)
$$
所以：
$$
w = [d + y + 1 + (2m+3(m+1)/5) + (y/400+y/4-y/100)] \mod 7 \quad (3 \le m \le 14)
$$

```cpp
int calcDayOfWeek(int y, int m, int d)
{
    int w;
    if (m == 1 || m == 2)
        m += 12, y--;
    w = (d + 2 * m + 3 * (m + 1) / 5 + y + y / 4 - y / 100 + y / 400 + 1) % 7;
    return w;
}
```

---

完整带测试代码：

```cpp
#include <iostream>
#include <cassert>
#include <cstdlib>
#include <ctime>
#define isleap(y) (((y) % 400 == 0) || (((y) % 4 == 0) && ((y) % 100 != 0)))
#define YEARLIM (10000)
using namespace std;
int week(int y, int m, int d)
{
    int disp[] = {0, 0, 3, 3, 6, 1, 4, 6, 2, 5, 0, 3, 5};
    int w = ((d - 1) + y + ((y - 1) / 400 + (y - 1) / 4 - (y - 1) / 100) + disp[m]) % 7;
    if (m > 2 && y != 0 && isleap(y))
        w = (w + 1) % 7;
    return w;
}
int calcDayOfWeek(int y, int m, int d)
{
    //Sun=0, Mon=1, Tue=2, ...,
    //Sat=6, w的取值如上
    int w;
    if (m == 1 || m == 2)
        m += 12, y--;
    w = (d + 2 * m + 3 * (m + 1) / 5 + y + y / 4 - y / 100 + y / 400 + 1) % 7;
    return w;
}
const char *format = "%4d/%02d/%02d\n";
void test(int y, int m, int d)
{
    printf(format, y, m, d);
    int t1 = week(y, m, d), t2 = calcDayOfWeek(y, m, d);
    // cout << t1 << ' ' << t2 << endl;
    assert(t1 == t2);
}
int main()
{
    srand(time(nullptr));

    // common years test
    for (int i = 0; i < 1000; i++)
    {
        int y = rand() % YEARLIM;
        int m = (rand() % 12) + 1;
        int d = (rand() % 31) + 1;
        if (m == 2)
            d = (rand() % 28) + 1;
        if (y != 0 && isleap(y))
            y++;
        test(y, m, d);
    }
    cout << "-------------------" << endl;

    // leap years test
    for (int i = 0; i < 1000; i++)
    {
        int y = rand() % YEARLIM;
        while (!(y != 0 && isleap(y)))
            y = rand() % YEARLIM;
        int m = (rand() % 12) + 1;
        int d = (rand() % 31) + 1;
        if (m == 2)
            d = (rand() % 29) + 1;
        test(y, m, d);
    }

    // Feb. test
    cout << "-------------------" << endl;
    for (int i = 0; i < 1000; i++)
    {
        int y = rand() % YEARLIM;
        int m = 2;
        int d = (rand() % 28) + 1;
        if (y != 0 && isleap(y))
            d = (rand() % 29) + 1;
        test(y, m, d);
    }

    // 29,Feb test
    cout << "-------------------" << endl;
    for (int i = 0; i < 1000; i++)
    {
        int m = 2, d = 29;
        int y = rand() % YEARLIM;
        while (!(y != 0 && isleap(y)))
            y = rand() % YEARLIM;
        test(y, m, d);
    }
}
```
