#树 #二叉树 #遍历 #洛谷
# 题干
**题目**：P1827 美国血统 American Heritage（洛谷）
> 题目链接：https://www.luogu.com.cn/problem/P1827

农夫约翰非常认真地对待他的奶牛们的血统。然而他不是一个真正优秀的记帐员。他把他的奶牛们的家谱作成了一棵二叉树，并且把二叉树更线性地表示为"树的中序遍历"和"树的前序遍历"的字符串而不是用图形的方法。

**任务**：给定一棵二叉树的中序遍历和前序遍历，求它的后序遍历。

## 输入格式

两行，第一行中序遍历，第二行前序遍历。

## 输出格式

一行，后序遍历。

## 输入输出样例

**输入 #1**

```
ABEDFCHG
CBADEFGH
```

**输出 #1**

```
AEFDBHGC
```

## 说明/提示

- 字符串由大写字母组成，字符互不相同，长度不超过 26。

# 我的题解
思路：**前序定根，中序分左右**。

- 前序遍历的第一个字符一定是**根**；
- 在**中序**里找到根的位置：左边全是左子树、右边全是右子树（中序的性质）；
- 左子树大小 = rootIndex，据此把前序剩余部分切成"左子树前序"和"右子树前序"；
- 递归还原，**先左、再右、最后追加根**——这个输出顺序就是后序（左→右→根）。

时间复杂度 O(n²)（每层 indexOf 一次 O(n)，共 n 层；n ≤ 26 无所谓，数据大时可用哈希表把下标预存成 O(n)），空间复杂度 O(n)（递归栈深 + 子串）。

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner cin = new Scanner(System.in);
        String in = cin.next();    // 中序遍历
        String pre = cin.next();   // 前序遍历
        StringBuilder post = new StringBuilder();
        getPostOrder(pre, in, post);
        System.out.println(post);
    }

    // 用 前序+中序 还原出后序：先左子树、再右子树、最后根
    public static void getPostOrder(String pre, String in, StringBuilder post) {
        if (pre.length() == 0)
            return;
        char root = pre.charAt(0);          // 前序第一个字符 = 根
        int rootIndex = in.indexOf(root);   // 中序里根的位置：左 = 左子树，右 = 右子树
        // 左子树大小 = rootIndex，据此切分前序
        getPostOrder(pre.substring(1, rootIndex + 1), in.substring(0, rootIndex), post); // 左子树
        getPostOrder(pre.substring(rootIndex + 1), in.substring(rootIndex + 1), post);   // 右子树
        post.append(root);                  // 后序：根在最后
    }
}
```

> 已用 Python 等价验证：原题样例 + 3000 棵随机树全部正确（含单结点、全左链、全右链边界）。
> 同款套路就是 LC 105 前中还原（选题指南黄区），这题是它的递归练手版。

# 官方题解
> 洛谷此题只有通过审核的**民间题解**（题解区），非官方题解。与我的 Java 版完全同构：前序首字符定根、中序切分、左右递归、根最后输出。

```c++
#include <bits/stdc++.h>
using namespace std;
string in, pre;

void dfs(string prePart, string inPart) {
    if (prePart.empty()) return;
    char root = prePart[0];                  // 前序首字符 = 根
    int pos = inPart.find(root);             // 中序切分点
    dfs(prePart.substr(1, pos), inPart.substr(0, pos));     // 左子树
    dfs(prePart.substr(pos + 1), inPart.substr(pos + 1));   // 右子树
    cout << root;                            // 根最后输出 = 后序
}

int main() {
    cin >> in >> pre;                        // 注意输入顺序：中序 前序
    dfs(pre, in);
    return 0;
}
```
来源：洛谷 P1827 题解区
链接：https://www.luogu.com.cn/problem/P1827
