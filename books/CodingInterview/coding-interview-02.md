

## 0201 删除重复节点

题目：[面试题 02.01. 移除重复节点](https://leetcode-cn.com/problems/remove-duplicate-node-lcci/) .

### 哈希计数

基于 `unordered_set` 记录该值是否出现过，若已经出现过则删除这个节点。

```cpp
class Solution {
public:
    ListNode* removeDuplicateNodes(ListNode* head) {
        if (head == nullptr || head->next == nullptr) return head;
        auto pre = head, p = head->next;
        unordered_set<int> s;
        s.insert(head->val);
        while (p != nullptr)
        {
            if (s.count(p->val))
            {
                pre->next = p->next;
                p = p->next;
            }
            else
            {
                s.insert(p->val);
                pre = pre->next;
                p = p->next;
            }
        }
        return head;
    }
};
```

### 双循环

吴宇森：为什么盗用我的暴力美学？

```cpp
class Solution {
public:
    ListNode* removeDuplicateNodes(ListNode* head) {
        if (head == nullptr || head->next == nullptr) return head;
        for (auto x = head; x != nullptr; x = x->next)
        {
            auto pre = x, p = x->next;
            while (p != nullptr)
            {
                if (p->val == x->val)
                    pre->next = p->next, p = p->next;
                else
                    pre = pre->next, p = p->next;
            }
        }
        return head;
    }
};
```

## 0202 倒数第 k 节点

题目：[面试题 02.02. 返回倒数第 k 个节点](https://leetcode-cn.com/problems/kth-node-from-end-of-list-lcci/) 。

双指针，快慢指针，做烂了。

```cpp
class Solution {
public:
    int kthToLast(ListNode* head, int k) {
        auto p = head;
        while (p != nullptr && k) p = p->next, k--;
        if (p == nullptr) return head->val;
        auto q = head;
        while (p != nullptr) p = p->next, q = q->next;
        return q->val; 
    }
};
```

## 0203 删除中间节点

题目：[面试题 02.03. 删除中间节点](https://leetcode-cn.com/problems/delete-middle-node-lcci/)

题目没读懂，原来是这么回事：

```
given 'a->b->c->d', delete b
a->b->c->d ==>
a->c'->c->d ==>
a->c'->d
```

代码：

```cpp
class Solution {
public:
    void deleteNode(ListNode* node) {
        node->val = node->next->val;
        node->next = node->next->next;
    }
};
```

## 0204 分割链表

题目：[面试题 02.04. 分割链表](https://leetcode-cn.com/problems/partition-list-lcci/)。

所有小于 x 的节点排在大于或等于 x 的节点之前，而大于或等于 x 的节点在后半部分的顺序不作要求。

双指针。小于 x 的节点放入 `small`，大于或等于 x 的节点放入 `large`，最后合并起来。

```cpp
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        ListNode * const large = new ListNode(-1);
        ListNode * const small = new ListNode(-1);
        auto l1=small, l2=large;
        for (auto p = head; p != nullptr; p = p->next)
        {
            if (p->val < x) l1->next = p, l1 = l1->next;
            else l2->next = p, l2 = l2->next;
        }
        l1->next = large->next;
        l2->next = nullptr;
        return small->next;
    }
};
```



## 0205 链表求和

题目：[面试题 02.05. 链表求和](https://leetcode-cn.com/problems/sum-lists-lcci/) 。

模拟整数加法，十分无聊。

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* const head = new ListNode(-1);
        uint8_t carry = 0, cur = 0;
        auto p = l1, q = l2, r = head;
        while (p && q)
        {
            cur = p->val + q->val + carry;
            carry = (cur >= 10);
            r->next = new ListNode(cur % 10), r = r->next;
            p = p->next, q = q->next;
        }
        auto handle = [&cur, &carry, &r](const ListNode *x) {
            while (x)
            {
                cur = x->val + carry;
                carry = (cur >= 10);
                r->next = new ListNode(cur % 10), r = r->next;
                x = x->next;
            }
        };
        handle(p);
        handle(q);
        if (carry) r->next = new ListNode(carry);
        return head->next;
    }
};
```

简化一下代码，如果两个链表长度不等，`null` 节点取值为 0 。

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        const auto head = new ListNode(-1);
        auto p = l1, q = l2, r = head;
        uint8_t cur = 0, carry = 0, a = 0, b = 0;
        while (p != nullptr || q != nullptr || carry)
        {
            a = (p == nullptr) ? 0 : p->val;
            b = (q == nullptr) ? 0 : q->val;
            cur = (a + b + carry);
            carry = (cur >= 10);
            r->next = new ListNode(cur % 10), r = r->next;
            if (p) p = p->next;
            if (q) q = q->next;
        }
        return head->next;
    }
};
```

## 0206 回文链表

题目：[面试题 02.06. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list-lcci/) 。

没有灵魂的缓冲区解法。

```cpp
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if (head == nullptr || head->next == nullptr) return true;
        auto p = head;
        vector<int> v;
        while (p) v.push_back(p->val), p = p->next;
        int k = v.size() / 2, size = v.size();
        for (int i=0; i<k; i++)
            if (v[i] != v[size - 1 - i]) return false;
        return true;
    }
};
```

考虑不使用额外的空间。先找到中间节点 `mid`，比较 `[head, mid]` 与 `reverse([mid+1, tail])` 是否相同。

经过 `getmid` 和 `getrev` 操作后的链表结构，给出 2 个例子如下：

```
case-1:
before = 1->2->[3]->4
after  = 1->2->[3]<-4
mid=3, rev=4

case-2:
before = 1->2->[3]->4->5
after  = 1->2->[3]<-4-<5
mid=3, rev=5
```

代码实现：

```cpp
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if (head == nullptr || head->next == nullptr) return true;
        auto mid = getmid(head);
        auto rev = getrev(mid);
        auto p = head, q = rev;
        while (p != q)
        {
            if (p->val != q->val) return false;
            p = p->next, q = q->next;
            // e.g. [1,2,2,1]
            if (q == nullptr) break;
        }
        return true;
    }

    ListNode *getmid(ListNode *head)
    {
        auto slow = head, fast = head;
        while (fast && fast->next)
        {
            slow = slow->next;
            fast = fast->next;
            if (fast) fast = fast->next;
        }
        return slow;
    }

    ListNode* getrev(ListNode *head)
    {
        if (head == nullptr || head->next == nullptr) return head;
        auto pre = head, p = head->next;
        while (p)
        {
            auto t = p->next;
            p->next = pre;
            pre = p, p = t;
        }
        head->next = nullptr;
        return pre;
    }
};
```

## 0207 链表相交

题目：[面试题 02.07. 链表相交](https://leetcode-cn.com/problems/intersection-of-two-linked-lists-lcci/)。

如果两个相交，那么二者的「形状」一定是 “Y” 型或者 “|” 型。

### 栈

通过  2 个栈缓存 2 个链表的节点，分别比较栈顶元素，直到栈顶不想等，那么 `s.top()->next` 就是第一个相交节点。

边界情况：

+ 链表长度不等，循环结束时可能其中一个栈为空。
+ 相交节点就是第一个节点。

代码实现：

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) 
    {
        stack<ListNode*> s1, s2;
        for (auto p=headA; p!=nullptr; p=p->next) s1.push(p);
        for (auto p=headB; p!=nullptr; p=p->next) s2.push(p);
        while (!s1.empty() && !s2.empty() && s1.top() == s2.top())
            s1.pop(), s2.pop();
        if (s1.empty()) return s2.empty() ? headA : s2.top()->next;    
        return s1.top()->next;
    }
};
```

### 双指针

空间复杂度 $O(1)$ 的解法。

如果存在相交节点：

+ 如果长度相等，那么两个指针每次走一步，总会汇聚到第一个相交点。
+ 如果长度不等，设 2 链表长度分别为 `a+l, b+l (a<b)`，`l` 是共同部分的长度，而 `a,b` 是非共同部分的长度。在这种情况下，`q` 每遍历一次链表，回到 `headB` 时，`p` 会领先「一步」。即 `q` 在第 `b-a+2`  次遍历时，就会与 `p` 在相交节点汇聚。

如果不存在相交节点，那么 `p, q` 会因为 `p == q == nullptr` 退出循环。

```cpp
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        auto p = headA, q = headB;
        while (p != q)
        {
            p = (p == nullptr) ? headA : p->next;
            q = (q == nullptr) ? headB : q->next;
        }
        return p;
    }
};
```

## 0208 环路检测

题目：[面试题 02.08. 环路检测](https://leetcode-cn.com/problems/linked-list-cycle-lcci/)。

解析可参考：https://www.cnblogs.com/sinkinben/p/13842982.html 。

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        // key node is a node in the ring
        auto node = keyNode(head);
        if (node == nullptr) return nullptr;
        auto p = node->next;
        // k is the length of the ring
        int k = 1;
        while (p != node) p = p->next, k++;
        // p go k step first
        p = head;
        for (int i=0; i<k; i++) p = p->next;
        // then q start at head with p
        auto q = head;
        while (p != q) p = p->next, q = q->next;
        return p;
    }
    ListNode *keyNode(ListNode *head)
    {
        if (head == nullptr || head->next == nullptr) return nullptr;
        auto p = head, q = head->next;
        while (p && q)
        {
            if (p == q) return q;
            p = p->next, q = q->next;
            if (q) q = q->next;
        }
        return nullptr;
    }
};
```

