## 快排与堆排

本文复习一下快速排序和堆排序 2 种排序算法（为了多快好省地刷 leetcode ）。

## 快排

主要思想：

> 通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

时间复杂度 $O(n \log n)$ ，空间复杂度取决于是否使用递归实现。

代码实现：

```cpp
int partition(vector<int> &v, int p, int r)
{
    int x = v[r];
    int i = p - 1;
    for (int j = p; j < r; j++)
    {
        if (v[j] < x)
            i = i + 1, swap(v[i], v[j]);
    }
    swap(v[i + 1], v[r]);
    return i + 1;
}
void quickSort(vector<int> &v, int p, int r)
{
    if (p < r)
    {
        int q = partition(v, p, r);
        quickSort(v, p, q - 1);
        quickSort(v, q + 1, r);
    }
}
```

对 `partition` 函数的图解（图源为《算法导论》）：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201021165623.png"  style="width:50%;" />

在这里的实现，我们默认区间的最右侧 `v[r] = 4` 为主元，将上面所示的数组分为 2 部分，左侧小于等于 4，右侧大于 4 。下面是函数中几个临时变量所表示的含义：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201021183712.png"  style="width:50%;" />

## 堆排

堆的性质：

> 子结点的键值总是小于（或者大于）它的父节点。

三个步骤：

+ 建立大顶堆
+ 把最大的的数字放在堆的最末尾 ，即 `j = n-1, ..., 1`
+ 调整堆，使其满足大顶堆的性质

代码实现：

```cpp
class Heap
{
private:
    int heapSize;
    // index start at 0
    inline int getLeft(int x) { return 2 * x + 1; }
    inline int getRight(int x) { return 2 * x + 2; }
    void heapify(vector<int> &v, int idx)
    {
        int l = getLeft(idx), r = getRight(idx);
        int largest = idx;
        if (l < heapSize && v[l] > v[largest]) largest = l;
        if (r < heapSize && v[r] > v[largest]) largest = r;
        // largest 这个位置被影响，所以需要递归调整
        // Of course, 也可以通过迭代实现
        if (largest != idx)
            swap(v[idx], v[largest]), heapify(v, largest);
    }
    void buildHeap(vector<int> &v)
    {
        heapSize = v.size();
        for (int i = v.size() / 2; i >= 0; i--)
            heapify(v, i);
    }

public:
    void heapSort(vector<int> &v)
    {
        buildHeap(v);
        for (int i = v.size() - 1; i >= 1; i--)
        {
            swap(v[i], v[0]);
            heapSize--;
            heapify(v, 0);
        }
    }
};
```

### heapify

`heapify` 函数图解如下图所示。需要注意的是，图中数组下标是从 1 开始的，而上面的代码实现是从 0 开始的。

时间复杂度 $O(\log n)$ .

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201021184011.png"  style="width:67%;" />

### buildHeap

在表示堆的数组中，范围 $\lfloor n/2 \rfloor $ 到 $n-1$ 是叶子节点（下标从 0 开始），对于叶子节点，自然而然会满足堆的性质，对叶子节点调用 `heapify` 丝毫没有影响，因此不需要调整。这就是为什么 `for` 循环的范围是 `size/2 -> 0` 。

时间复杂度为 $O(n)$ .

### heapSort

调用 `buildHeap` 后的数组，是一个大顶堆，所以 `v[0]` 是最大的数字，我们把它交换到数组的最末尾处。然后对 `[0, heapSize)` 范围内的数字进行 `heapify` ，因为影响的只有位置 0 ，所以只需要调用一次 `heapify(v, 0)` 就能使得数组满足堆的性质。

时间复杂度 $O(n\log n)$ .

`heapSort` 的图解如下：

<img src="https://gitee.com/sinkinben/pic-go/raw/master/img/20201021190023.png"  style="width:75%;" />