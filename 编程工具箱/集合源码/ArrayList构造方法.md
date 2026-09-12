#集合源码 #Java #ArrayList
# ArrayList 构造方法
> 学习法：只读主干（字段/构造/add 链路/grow/remove/get），画调用链图，口述面试题；看一段消化一段，其余都是噪音。
> 第 1 遍 2026.08.31-09.06；第 2 遍（隔几天关书默写调用链）待做。

## 面试题进度（口述版）
1. 底层数据结构是数组 ✅——`Object[] elementData`，真正存数据的变量
2. 初始容量：**空**（懒加载），add 第一个元素时才分配 ✅（grow 待看）
3. 扩容机制 / 均摊 O(1)：待看 grow
4. remove / get：待看
5. vs LinkedList：一句话对比，待补

## 调用链图（带参构造）

```
new ArrayList(c)  [c 是某个 Collection]
  └─ elementData = c.toArray()          ← 动态分派：执行 c 自己那个类的 toArray！
       └─ (c 恰好是 ArrayList 时) Arrays.copyOf(elementData, size)
            └─ copyOf(original, newLength, original.getClass())   ← 三参重载
                 ├─ 判断 newType 是否 Object[].class
                 │    ├─ 是 → new Object[newLength]                （快速路径）
                 │    └─ 否 → Array.newInstance(类型, 新长度)      （反射造数组）
                 └─ System.arraycopy(original, 0, copy, 0, min(原长, 新长))
  └─ size = elementData.length
  └─ 非空 && elementData.getClass() != Object[].class → 重建 Object[] 副本（类型保险）
  └─ 空 → elementData = EMPTY_ELEMENTDATA（共享空数组）
```

## 我的理解（逐层拆解）

### 第 1 层：构造方法本体
`elementData = c.toArray()` 先拿到数组；`(size = elementData.length) != 0` 是**先给 size 赋值再判断**的复合表达式，size 是**新 ArrayList** 的字段。

### 第 2 层：c.toArray() 是动态分派 ⭐
toArray 是 c 的**实例方法**，里面访问的 elementData/size 是 **c 自己**的，不是新 ArrayList 的。只有当 c 恰好也是 ArrayList 时才会走进 `Arrays.copyOf(elementData, size)`；c 是 HashSet/LinkedList 时 toArray 实现完全不同。容易混的点，标记住。

### 第 3 层：Arrays.copyOf 两参 → 三参
两参版 `copyOf(original, newLength)` 内部直接转调三参版，第三个参数 = `original.getClass()`（原数组运行时类型）。

三参版 = **造新数组 + 搬数据**两步：
- 造：判断 `newType` 是否恰好 `Object[].class`（此链中 = 原数组是不是 Object[]）
  - 是 → `new Object[newLength]`（快速路径，注意是**数组语法** `new Object[n]`）
  - 否 → `Array.newInstance(newType.getComponentType(), newLength)`——运行时按类型造数组
- 搬：`System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength))`——native 批量复制（JVM intrinsic，比 for 循环快），min 决定"多退少补"：新数组更长 → 原数据全拷、多出的位置留 null；更短 → 只拷前 newLength 个

一句话记忆：**copyOf = 造新数组 + 搬数据**。

### 第 4 层：为什么 Array.newInstance？——泛型擦除
`new E[n]` 编译不过（泛型擦除后运行时不知道 E 是什么），但数组运行期带类型（String[] 和 Object[] 是不同类），所以用反射 `Array.newInstance(String.class, 5)` 造 String[5]。这是 copyOf 保留元素类型的手段。

### 第 5 层：为什么构造里"再判断一次类型"？——6260652 类型保险 ⭐
源码注释：`c.toArray might (incorrectly) not return Object[] (see 6260652)`。某些集合的 toArray() 返回**具体类型**数组（如 String[]），而 ArrayList 内部数组必须是 Object[]（要能 add 任意类型）：
```java
String[] 只能存 String；
elementData = c.toArray();               // 若返回 String[]，协变赋值语法通过
elementData[0] = Integer.valueOf(1);     // 以后 add 非 String → ArrayStoreException 炸掉
```
所以拿到数组后检查运行时类型，**不是 Object[] 就重建一个 Object[] 副本**，保证之后 add 任何类型都安全。
> 纠正过的一个误解：赋值其实在第 1 步就完成了，这个 if 里只有"非 Object[] 才重建"一个动作。

### 版本差异（不用背，知道思路一致即可）
- JDK 8：判断 `elementData.getClass() != Object[].class`（看**数组类型**）
- JDK 11+：判断 `c.getClass() == ArrayList.class`（看**集合类型**，c 是 ArrayList 直接复用其数组副本）——思路一样，都是类型保险

## 复杂度 / 反哺 408
- 带参构造：O(n)（copy 一次数组）
- 408 联系：**顺序表扩容 = copyOf 干的事**（分配新容量数组 + 拷贝旧数据 O(n)）；顺序表插入/删除移动元素 = System.arraycopy 干的事。看完集合源码，回来默写顺序表这两点，两条线互相咬合。

## 待补（第 2 遍时）
- [ ] 关书默写整条调用链图
- [ ] 口述 5 面试题（目前能答 1、2 两条）
