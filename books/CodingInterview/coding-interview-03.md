## 0301 三合一

题目：[面试题 03.01. 三合一](https://leetcode-cn.com/problems/three-in-one-lcci/)。

搞不懂这道题的意义何在。

```cpp
class TripleInOne {
public:
    const int n = 3;
    vector<int> data;
    int idx[3];
    int size;
    TripleInOne(int stackSize) {
        size = stackSize;
        data.resize(size * n, 0);
        for (int i=0; i<n; i++)
            idx[i] = i * stackSize;
    }
    
    void push(int stackNum, int value) {
        if (idx[stackNum] == (stackNum+1) * size) return;
        data[idx[stackNum]++] = value;
    }
    
    int pop(int stackNum) {
        if (isEmpty(stackNum)) return -1;
        return data[--idx[stackNum]];
    }
    
    int peek(int stackNum) {
        if (isEmpty(stackNum)) return -1;
        return data[idx[stackNum] - 1];
    }
    
    bool isEmpty(int stackNum) {
        return idx[stackNum] == stackNum * size;
    }
};
```

## 0302 栈的最小值

题目：[面试题 03.02. 栈的最小值](https://leetcode-cn.com/problems/min-stack-lcci/)。

剑指 Offer 的题目：https://www.cnblogs.com/sinkinben/p/13842997.html

```cpp
class MinStack {
public:
    const int INTMAX = 0x7fffffff;
    stack<int> data, minbuf;
    int minval = INTMAX;
    /** initialize your data structure here. */
    MinStack() {}

    void push(int x) {
        data.push(x);
        minbuf.push(minval = min(minval, x));
    }
    
    void pop() {
        if (data.empty()) return;
        int x = data.top();
        data.pop(), minbuf.pop();
        if (x == minval)
            minval = minbuf.empty() ? INTMAX : minbuf.top();
    }
    
    int top() {
        if (!data.empty()) return data.top();
        return -1;
    }
    
    int getMin() {
        return minval;
    }
};
```

## 0303 堆盘子

题目：[面试题 03.03. 堆盘子](https://leetcode-cn.com/problems/stack-of-plates-lcci/)。

一个边界条件： `cap` 可能输入小于等于 0  的值。

使用 `vector` 实现：

+ push：当前“行”容量已满，开启新一行输入数据；
+ pop：从最后一行 `pop` 一个数据，如果删除后当前行为空，删除当前行；
+ popAt：与 pop 同理。

代码实现：

```cpp
class StackOfPlates {
public:
    int size;
    vector<vector<int>> data;
    StackOfPlates(int cap) { size = cap; }
    
    void push(int val) {
        if (size <= 0) return;
        if (data.empty() || data.back().size() == size) data.push_back({});
        data.back().push_back(val);
    }
    
    int pop() {
        if (data.empty()) return -1;
        int x = data.back().back();
        data.back().pop_back();
        if (data.back().size() == 0) data.pop_back();
        return x;
    }
    
    int popAt(int index) {
        if (index < 0 || index >= data.size()) return -1;
        int x = data[index].back();
        data[index].pop_back();
        if (data[index].size() == 0) data.erase(data.begin() + index);
        return x;
    }
};
```

## 0304 化栈为队

题目：[面试题 03.04. 化栈为队](https://leetcode-cn.com/problems/implement-queue-using-stacks-lcci/)

剑指Offer的题目：https://www.cnblogs.com/sinkinben/p/13826614.html

代码：

```cpp
class MyQueue {
public:
    stack<int> s1, s2;
    /** Initialize your data structure here. */
    MyQueue() {}
    
    /** Push element x to the back of queue. */
    void push(int x) {
        s1.push(x);
    }
    /** Removes the element from in front of queue and returns that element. */
    int pop() {
        if (empty()) return -1;
        if (s2.empty())
        {
            while (!s1.empty())
                s2.push(s1.top()), s1.pop();
        }
        int x = s2.top();
        s2.pop();
        return x;
    }
    
    /** Get the front element. */
    int peek() {
        if (empty()) return -1;
        if (s2.empty())
        {
            while (!s1.empty())
                s2.push(s1.top()), s1.pop();
        }
        return s2.top();
    }
    
    /** Returns whether the queue is empty. */
    bool empty() {
        return s1.empty() && s2.empty();
    }
};
```

## 0305 栈排序

题目：[面试题 03.05. 栈排序](https://leetcode-cn.com/problems/sort-of-stacks-lcci/)

**方法一：暴力**

借助一个额外的栈，每次将 `val` 插入到合适的位置。

代码：

```cpp
class SortedStack {
public:
    stack<int> data, buf;
    SortedStack() {}
    
    void push(int val) {
        while (!data.empty() && data.top() < val)
            buf.push(data.top()), data.pop();
        data.push(val);
        while (!buf.empty())
            data.push(buf.top()), buf.pop();
    }
    
    void pop() {
        if (isEmpty()) return;
        data.pop();
    }
    
    int peek() {
        if (isEmpty()) return -1;
        return data.top();
    }
   
    bool isEmpty() {
        return data.empty();
    }
};
```

**方法二：根据插入的 val 划分 2 个栈**

这里的优化主要是针对：操作序列中，出现多次连续 push 的情况。

```
|   ^    |        |    |    |
|   |    |        |    |    |
|  val   |        |    V    |
| bigger |        | smaller |
+--------+        +---------+
|  data  |        |   buf   |
+--------+        +---------+
```

每次 push 前，把 data 中比 val 小的数字移动到 buf 当中，把 buf 中比 val 大的移动到 data 中。

当多次 push 操作时，可减少数据挪动的次数。

```cpp
class SortedStack {
public:
    stack<int> data, buf;
    SortedStack() {}
    
    void push(int val) {
        while (!data.empty() && data.top() < val)
            buf.push(data.top()), data.pop();
        while (!buf.empty() && buf.top() > val)
            data.push(buf.top()), buf.pop();
        data.push(val);
    }
    
    void pop() {
        if (isEmpty()) return;
        while (!buf.empty()) data.push(buf.top()), buf.pop();
        data.pop();
    }
    
    int peek() {
        if (isEmpty()) return -1;
        while (!buf.empty()) data.push(buf.top()), buf.pop();
        return data.top();
    }
    
    bool isEmpty() {
        return data.empty();
    }
};
```

## 0306 动物收容所

🐒 题目：[面试题 03.06. 动物收容所](https://leetcode-cn.com/problems/animal-shelter-lcci/)。

水题，意义何在？

如果动物编号 `animal[0]` 不一定是递增的情况，那么就需要一个内部 `id` ，标记动物的到达次序 `map[animal[0]] = id++` ，出队的时候，根据 `map[cat.front()], map[dog.front()]` 的大小来判断该领养猫🐱还是狗🐶。

```cpp
class AnimalShelf {
public:
    queue<int> cats, dogs;
    AnimalShelf() { }
    
    void enqueue(vector<int> animal) {
        if (animal[1] == 0) cats.push(animal[0]);
        else dogs.push(animal[0]);
    }
    
    vector<int> dequeueAny() {
        if (cats.empty() && dogs.empty()) return {-1, -1};
        if (cats.empty()) return dequeueDog();
        if (dogs.empty()) return dequeueCat();
        if (cats.front() < dogs.front()) return dequeueCat();
        else return dequeueDog();
    }
    
    vector<int> dequeueDog() {
        if (dogs.empty()) return {-1, -1};
        int dog = dogs.front();
        dogs.pop();
        return {dog, 1};
    }
    
    vector<int> dequeueCat() {
        if (cats.empty()) return {-1, -1};
        int cat = cats.front();
        cats.pop();
        return {cat, 0};
    }
};
```

