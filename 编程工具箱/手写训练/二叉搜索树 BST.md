#BST #二叉搜索树 #树 #手写训练

# 二叉搜索树 BST（插入 / 删除 / 查找 / 判断）

> 手写训练第 2 项（树清单 2/8）：插入 / 删除（三情况）/ 查找（递归 + 非递归）/ 判断 BST（中序递增）
> 三遍法进度：- [ ] 第 1 遍 跟抄　- [ ] 第 2 遍 关书手写　- [ ] 第 3 遍 脱稿默写（__月__日）

# 主要思想

- **BST = 二叉排序树**：左子树 < 根 < 右子树（默认不存重复 key）。**中序遍历递增**是它最重要、最常拿来验证的性质。
- 查找：从根开始比大小，小往左、大往右，遇到 NULL 即失败。递归 / 非递归两种都要会默写。
- 插入 = 在**查找失败的那个空位**上挂新结点。因为要**改父指针本身**，函数必须拿到"指针的地址"：网课 C++ 用引用 `BSTNode*&`，408 手写 C 用二级指针 `BSTNode**`。
- 删除最麻烦，按孩子数分三情况：① 叶子直接删并把父链置 NULL；② 一个孩子让"孩子顶上"；③ 两个孩子用**右子树最小（中序后继）**顶替，抄值后再递归删掉那个后继（代码里用 `FindMin` 找）。
- 复杂度都取决于树高 h：随机序列平均 O(log n)；升/降序插入会退化成单链，最坏 O(n)。
- 判断 BST：**中序遍历看是否严格递增**——记前驱 pre，一旦出现 `pre->data >= 当前` 就提前剪枝判失败（见 JudgeBST）。

## 复杂度

| 操作 | 时间 | 空间 | 说明 |
|---|---|---|---|
| 查找 SearchRec / SearchIter | O(h) | O(h) / O(1) | 递归栈深 vs 迭代无栈 |
| 插入 InsertRec | O(h) | O(h) | 递归版，栈深 = h |
| 删除 DeleteRec | O(h) | O(h) | 递归版 |
| 建树 n 个结点 | O(n·h) | O(h) | 每个结点依次插入 |
| 判断 BST（JudgeBST） | O(n) | O(h) | 中序检查递增，失败剪枝提前返回 |
| 中序遍历（验证有序） | O(n) | O(h) | 见自测 main |

# 代码实现（C；网课 C++ 版对照见文末）

```c
#include <stdio.h>
#include <stdlib.h>

/* ---------- 结构定义 ---------- */
typedef struct BSTNode {
    int data;
    struct BSTNode *lchild, *rchild;
} BSTNode;

/* ---------- 右子树最小 = 中序后继（DeleteRec 要用，必须定义在它前面） ---------- */
BSTNode* FindMin(BSTNode* p) {
    if (p == NULL) return NULL;
    while (p->lchild != NULL)          // 一路向左到底
        p = p->lchild;
    return p;
}

/* ---------- 查找（递归） ---------- */
BSTNode* SearchRec(BSTNode* root, int x) {
    if (root == NULL)
        return NULL;
    else if (x < root->data)
        return SearchRec(root->lchild, x);
    else if (x > root->data)
        return SearchRec(root->rchild, x);
    else
        return root;                   // 相等：命中
}

/* ---------- 查找（非递归） ---------- */
BSTNode* SearchIter(BSTNode* root, int x) {
    while (root != NULL && root->data != x) {
        if (x < root->data) root = root->lchild;
        else                root = root->rchild;
    }
    return root;                       // 命中返回结点；找不到返回 NULL
}

/* ---------- 插入（递归） ----------
   网课 C++ 版形参是 BSTNode*& root；转 C 后形参变 BSTNode** root，
   函数体内凡是"节点指针 root"一律加一星写成 *root */
void InsertRec(BSTNode** root, int x) {
    if (*root == NULL) {               // 找到空位：新结点挂在这里（判空必须带 *）
        *root = (BSTNode*)malloc(sizeof(BSTNode));
        (*root)->data = x;
        (*root)->lchild = (*root)->rchild = NULL;
        return;
    }
    else if (x < (*root)->data)
        InsertRec(&(*root)->lchild, x); // 往左子树插，传"左孩子指针的地址"
    else if (x > (*root)->data)
        InsertRec(&(*root)->rchild, x); // 往右子树插
    /* x == (*root)->data：重复 key，不插入，静默返回（BST 约定无重复） */
}

/* ---------- 删除（递归）：先找，再按孩子数分三情况 ---------- */
void DeleteRec(BSTNode** root, int x) {
    if (*root == NULL)                 // 空树 / 没找到：递归出口（判空必须带 *）
        return;
    else if (x < (*root)->data) {
        DeleteRec(&(*root)->lchild, x);
        return;
    }
    else if (x > (*root)->data) {
        DeleteRec(&(*root)->rchild, x);
        return;
    }
    /* 命中 *root */
    if ((*root)->lchild == NULL && (*root)->rchild == NULL) {
        /* ① 叶子：free 后把父链置 NULL——这就是要二级指针的原因 */
        free(*root);
        *root = NULL;
    }
    else if ((*root)->lchild == NULL) { /* ② 只有右孩子：右孩子顶上 */
        BSTNode* temp = *root;
        *root = (*root)->rchild;
        free(temp);
    }
    else if ((*root)->rchild == NULL) { /* ③ 只有左孩子：左孩子顶上 */
        BSTNode* temp = *root;
        *root = (*root)->lchild;
        free(temp);
    }
    else {                              /* ④ 两个孩子：右子树最小（后继）顶替 */
        BSTNode* next = FindMin((*root)->rchild);
        (*root)->data = next->data;     // 只抄值，别 free 当前结点（孩子会丢）
        DeleteRec(&(*root)->rchild, next->data); // 再去右子树删掉那个后继
    }
}

/* ---------- 判断 BST：中序遍历，检查是否严格递增 ----------
   需要两个全局变量（408 手写常用此风格）：
   pre = 中序遍历的"前驱结点"；flag = 1 表示目前仍是 BST
   ⚠️ 每次调用前必须重置：pre = NULL; flag = 1;   */
BSTNode* pre = NULL;
int flag = 1;

void JudgeBST(BSTNode* root) {
    if (root == NULL || flag == 0)   // 空子树 / 已判失败：剪枝返回
        return;
    JudgeBST(root->lchild);          // ① 左子树
    if (pre == NULL) {               // ② 最左结点：没有前驱，直接记下
        pre = root;
    } else if (pre->data >= root->data) {
        flag = 0;                    // 前驱 ≥ 当前 => 不是严格递增 => 不是 BST
        return;                      // 提前结束，后面子树全部剪掉
    } else {
        pre = root;                  // 递增正常：前驱更新为当前结点
    }
    JudgeBST(root->rchild);          // ③ 右子树
}

/* ---------- 建树：把数组元素依次插入 ---------- */
BSTNode* Create(int val[], int length) {
    BSTNode* root = NULL;
    for (int i = 0; i < length; i++)
        InsertRec(&root, val[i]);      // 传 root 变量本身的地址
    return root;
}
```

# 自测：验证用的 main（随笔记保留，平时删掉也行）

```c
void InOrder(BSTNode* root) {          // 中序递增 = BST 成立
    if (root == NULL) return;
    InOrder(root->lchild);
    printf("%d ", root->data);
    InOrder(root->rchild);
}

int main(void) {
    int a[] = {50, 30, 80, 20, 40, 90, 35};
    BSTNode* root = Create(a, 7);
    InOrder(root); printf("\n");                       // 期望：20 30 35 40 50 80 90

    printf("%s\n", SearchRec(root, 40)  ? "40 found" : "40 missing");    // found
    printf("%s\n", SearchIter(root, 55) ? "55 found" : "55 missing");    // missing

    DeleteRec(&root, 50);              // 删根：两孩子（80 顶替）
    DeleteRec(&root, 20);              // 删叶子
    DeleteRec(&root, 40);              // 删只有一个左孩子(35)的节点
    InOrder(root); printf("\n");                       // 期望：30 35 80 90

    pre = NULL; flag = 1;                              // 判断前必须重置全局状态
    JudgeBST(root);
    printf("%s\n", flag ? "is BST" : "not BST");       // 期望：is BST

    root->lchild->rchild->data = 100;  // 故意破坏（35 -> 100），应能判出 not BST
    pre = NULL; flag = 1;
    JudgeBST(root);
    printf("%s\n", flag ? "is BST" : "not BST");       // 期望：not BST
    return 0;
}
```

# C++ 网课版 & C 手写版对照（本次最大坑）

网课：`void InsertRec(BSTNode*& root, int x)` —— `root` 就是**节点指针本体**，判空直接写 `root == NULL`。
转 C 后形参是 `BSTNode** root`，函数体内**凡是"节点指针"的地方都要补一个 `*`**：

| C++（网课，形参 `BSTNode*& root`） | C（408 手写，形参 `BSTNode** root`） | 说明 |
|---|---|---|
| `if (root == NULL)` | `if (*root == NULL)` | 判空最容易漏星 |
| `root = new BSTNode;` | `*root = (BSTNode*)malloc(sizeof(BSTNode));` | 新建节点 |
| `root->data = x;` | `(*root)->data = x;` | `(*root)` 括号别丢 |
| `InsertRec(root->lchild, x);` | `InsertRec(&(*root)->lchild, x);` | 递归传"子指针的地址" |

- 翻译口诀照旧：`&` → `*`、`.` → `->`、调用处传 `&`（详见 [[C++阅读速览]]）。
- 判空漏星是**典型笔误**：忘加 `*` 后 `root` 是地址永不为 NULL，插入/删除会直接解引用 NULL 崩溃。

# 手写易错点

- InsertRec / DeleteRec 判空必须写 `*root == NULL`；原稿两处都写成 `root == NULL`，会崩。
- `*root == NULL` 在 DeleteRec 里是"空树/没找到"的出口；删叶子时 `free` 后必须 `*root = NULL`，父链靠这个二级指针才能断干净。
- ④ 两个孩子：`FindMin((*root)->rchild)` 抄值后再递归删后继，**不能直接 free 当前结点**。
- 函数必须先声明/先定义再用：`DeleteRec` 调 `FindMin`、`Create` 调 `InsertRec`，顺序别乱（C99 起隐式声明是编译错误）。
- void 函数里别写 `return 函数名(...)`：C 禁止 void 函数带表达式的 return（C++ 才允许）。
- JudgeBST 的全局 `pre`/`flag` 必须定义并初始化（`pre = NULL; flag = 1;`），且**每次调用前都要重置**——上次调用的结果会留在全局变量里串状态。
- JudgeBST 判断条件写 `pre->data >= root->data`（严格递增）；若题目允许重复 key，改成 `>`。
- JudgeBST 判失败后 `flag = 0` 加 `return` 剪枝：后面子树不用再遍历（不剪枝结果也对，只是慢）。
- 写完自测：建树后中序必须递增；删完根后整棵树仍是 BST。

# 相关

- [[二叉树遍历]]：中序遍历是验证 BST 有序性的工具
- [[C++阅读速览]]：`&` → `*` 翻译口诀
- [[知识点联系]]：树 ↔ 二叉树转换与遍历对应
- LC 98 验证二叉搜索树：本题 JudgeBST 的力扣版（面试场景）
