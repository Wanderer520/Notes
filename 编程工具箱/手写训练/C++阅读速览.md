#C++ #阅读 #手写训练

# C++ 阅读速览（网课/题解代码扫盲）

> 目的：**会读不写**。网课代码标 "C/C++"、洛谷题解是 C++，认得出、跟得上即可；408 手写与训练仍用 C。
> 学完标准：给你一段"标 C/C++"的网课代码，能说出每处 C++ 便利对应 C 的写法。

# 主要思想

- "C/C++" 标法的真相：90% 是 **C 语法 + 三样 C++ 便利**（引用 `&`、`new`、`bool`），再加偶尔的 STL。
- 你已会 C 和 Java：**引用 ≈ C 指针的语法糖，STL ≈ Java 集合改名**。用 Java 对照学 STL 最快。
- 以下全部是"认得"级别，不用背；遇到不认识的就回来查这张表。

# 速查表

## 1. 引用 &（最重要的一个）

| C++（网课写法） | C（手写转换） | 含义 |
|---|---|---|
| `void f(int &x)` | `void f(int *x)` | 形参就是实参本体，改形参 = 改实参 |
| `f(a)` | `f(&a)` | 调用处不用加 `&` |
| `void f(int *&p)` | `void f(int **p)` | 指针的引用 = 二级指针 |

- 读法：看到形参带 `&`，就知道"函数想修改调用者的变量 / 避免拷贝"。
- 手写 408 时的替代：**全部用指针**即可，效果一样。

## 2. new / delete

| C++ | C | 说明 |
|---|---|---|
| `p = new BiTNode;` | `p = (BiTNode*)malloc(sizeof(BiTNode));` | 申请单个节点 |
| `p = new BiTNode[10];` | `p = (BiTNode*)malloc(10 * sizeof(BiTNode));` | 申请数组 |
| `delete p;` | `free(p);` | 释放 |

- 读法：看到 `new` 就是 malloc，看到 `delete` 就是 free。

## 3. bool 与 true / false
- C++ 内置 `bool`/`true`/`false`；C 里用 `int` + `0/1`（或 `#include <stdbool.h>`）。
- 网课函数返回 `bool` 时，手写 C 写成 `int`：`return true;` → `return 1;`。

## 4. cin / cout（认得即可）
- `cin >> x;` = `scanf(...)`；`cout << x << endl;` = `printf(...)`。
- 408 手写继续用 printf/scanf，不用管它们。

## 5. 结构体省略 typedef
- C++ 里 `struct BiTNode {…};` 定义后可直接用 `BiTNode` 当类型名，无需 `typedef`。
- 看网课代码觉得"少写了什么"？不是缺，是 C++ 允许省略。

## 6. STL 容器 → Java 对照（速记核心）

| C++ 容器 | 读法 | ≈ Java |
|---|---|---|
| `vector<int> v` | 动态数组，`v[i]` 取元素 | ArrayList\<Integer\> |
| `stack<int> s` | `s.push() / top() / pop()` | Stack / ArrayDeque |
| `queue<int> q` | `q.push() / front() / pop()` | LinkedList（队列用法） |
| `priority_queue<int> pq` | 默认大根堆，`top()` 取最大 | PriorityQueue（默认小根堆，**方向相反**） |
| `unordered_map<K,V>` | 哈希表，`m[k]` 取值 | HashMap |
| `map<K,V>` | 红黑树，key 有序 | TreeMap |
| `unordered_set` / `set` | 哈希集合 / 有序集合 | HashSet / TreeSet |
| `sort(v.begin(), v.end())` | 排序 | Collections.sort |

- 注意：C++ 的 `priority_queue` 是**大根堆**，Java 的 `PriorityQueue` 是**小根堆**——读代码时方向要反过来。

## 7. 竞赛常见外壳（认个脸）
- `#include <bits/stdc++.h>`：万能头文件，竞赛专用，正规工程不用。
- `using namespace std;`：省得写 `std::` 前缀。
- `for (int x : v)`：范围 for，读作"遍历 v 的每个元素放入 x"。

# 实例：一段"标 C/C++"的网课代码 → C 翻译

```cpp
// 网课写法（C/C++）
typedef struct { int data[100]; int top; } SqStack;

bool Push(SqStack &S, int x) {          // & 引用 + bool
    if (S.top == 99) return false;
    S.data[++S.top] = x;
    return true;
}

int Pop(SqStack &S) {                   // 出栈
    return S.data[S.top--];
}
```

```c
// 408 手写对应（C）
typedef struct { int data[100]; int top; } SqStack;

int Push(SqStack *S, int x) {           // & → 指针，bool → int
    if (S->top == 99) return 0;         // . → ->
    S->data[++S->top] = x;
    return 1;
}

int Pop(SqStack *S) {
    return S->data[S->top--];
}
```

- 翻译口诀：**`&` → `*`、`.` → `->`、调用处 `S` → `&S`、`bool` → `int`**。
- 其余部分（数组、结构体字段、算法本体）一字不改——这就是"标 C/C++"的真相。

# 例题

- [[每日温度]]：回看该题评论区/洛谷的 C++ 题解，用本表逐行读通，复述思路。

# 铁律

- 手写训练、408 考场：**继续用 C**，不跟着网课写 C++。
- 这张表记不住没关系，卡住时回来看一眼即可；见得多了自然就认得了。
