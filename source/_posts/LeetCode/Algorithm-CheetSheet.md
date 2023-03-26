---
title: Algorithm CheetSheet
date: 2019-08-19 19:33:07
categories:
    - LeetCode
tags: 
    - DataStructure
---

# 数据结构

## 数组/顺序表

数组遍历框架，典型的线性迭代结构：

```java
void traverse(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        // 迭代访问 arr[i]
    }
}
```

## 链表

链表遍历框架，兼具迭代和递归结构：

```java
/* 基本的单链表节点 */
class ListNode {
    int val;
    ListNode next;
}

void traverse(ListNode head) {
    for (ListNode p = head; p != null; p = p.next) {
        // 迭代访问 p.val
    }
}

void traverse(ListNode head) {
    // 递归访问 head.val
    traverse(head.next)
}
```

二叉树遍历框架，典型的非线性递归遍历结构：

```java
/* 基本的二叉树节点 */
class TreeNode {
    int val;
    TreeNode left, right;
}

void traverse(TreeNode root) {
    traverse(root.left)
    traverse(root.right)
}
```

你看二叉树的递归遍历方式和链表的递归遍历方式，相似不？再看看二叉树结构和单链表结构，相似不？如果再多几条叉，N 叉树你会不会遍历？

二叉树框架可以扩展为 N 叉树的遍历框架：

```java
/* 基本的 N 叉树节点 */
class TreeNode {
    int val;
    TreeNode[] children;
}

void traverse(TreeNode root) {
    for (TreeNode child : root.children)
        traverse(child)
}
```

## 哈希表

### 构造哈希函数需要考虑的因素：

1. 计算哈希函数所需时间(包括硬件指令因素)
2. 关键字长度
3. 哈希表大小
4. 关键字的分布情况
5. 记录的查找频率

### 常见构造哈希函数的方法：

1. 直接定址法
   - 取关键字或关键字的某个线性函数值为哈希地址，即：`H(key)=key` 或 `H(key)=a*key+b`
2. 数字分析法
   - 假设关键字是以r为基的数(如十进制)，并且哈希表中可能出现的关键字都是事先知道的，则可取关键字的若干数位组成哈希地址
3. 平方取中法
   - 取关键字平方后的中间几位为哈希地址
4. 折叠法(folding)
   - 将关键字分割成位数相同的几部分(最后一部分的位数可以不同)，然后取这几部分的叠加和(舍去进位)作为哈希地址
5. 除留余数法
   - 取关键字被某个不大于哈希表长度m的数p除后所得余数为哈希地址，即：`H(key)=key % p, p<=m`
6. 随机数法
   - 选择一个随机函数，取关键字的随机函数值为它的哈希地址，即：`H(key)=random(key)`，其中random为随机函数。通常关键字长度不等时采用此方法。

### 处理冲突的方法

1. 开放定址法
2. 再哈希法
3. 链地址法
4. 建立一个公共溢出区

```java
/**
 * 采用：
 * 除留余数法+链地址法(拉链法)
 */
public class ZipHashTable {
    // 一般情况下取质数或不包含小于20的质因数的合数
    private static final int N = 1023;

    @SuppressWarnings("unchecked")
    private final List<int[]>[] buckets = new List[N];

    public ZipHashTable() {

    }

    public void put(int key, int value) {
        final int hash = key % N;
        List<int[]> bucket = buckets[hash];
        if (bucket == null) {
            bucket = new LinkedList<>();
            buckets[hash] = bucket;
        }
        boolean contains = false;
        for (int[] slot : bucket) {
            if (slot[0] == key) {
                contains = true;
                slot[1] = value;
                break;
            }
        }
        if (!contains) {
            bucket.add(new int[] {key, value});
        }
    }

    public int get(int key) {
        final int hash = key % N;
        final List<int[]> bucket = buckets[hash];
        if (bucket == null) {
            return -1;
        }
        for (int[] slot : bucket) {
            if (slot[0] == key) {
                return slot[1];
            }
        }
        return -1;
    }

    public void remove(int key) {
        final int hash = key % N;
        final List<int[]> bucket = buckets[hash];
        if (bucket == null) {
            return;
        }
        bucket.removeIf(slot -> slot[0] == key);
    }
}
```

## 二叉树

二叉树算法的设计的总路线：明确一个节点要做的事情，然后剩下的事抛给框架。

DFS(Depth First Search)深度优先搜索：

```java
void traverse(TreeNode root) {
    // root 需要做什么？在这做。
    // 其他的不用 root 操心，抛给框架
    traverse(root.left);
    traverse(root.right);
}
```

BFS(Breadth First Search)广度优先搜索：

```java
// 二叉树每层作为一个数组放到大数组里
// 队列每次全部弹出到临时数组，分别获取该层所有结点的所有孩子并加入队列
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();
        List<Integer> level = new LinkedList<>();
        for (int i = 0; i < size; ++i) {
            TreeNode cur = q.peek();
            q.poll();
            if (cur == null) {
                continue;
            }
            level.add(cur.val);
            q.offer(cur.left);
            q.offer(cur.right);
        }
        if (!level.isEmpty()) {
            res.add(level);
        }
    }
    return res;
}
```


1. 如何把二叉树所有的节点中的值加一？

```java
void plusOne(TreeNode root) {
    if (root == null) return;
    root.val += 1;

    plusOne(root.left);
    plusOne(root.right);
}
```

2. 如何判断两棵二叉树是否完全相同？

```java
boolean isSameTree(TreeNode root1, TreeNode root2) {
    // 都为空的话，显然相同
    if (root1 == null && root2 == null) return true;
    // 一个为空，一个非空，显然不同
    if (root1 == null || root2 == null) return false;
    // 两个都非空，但 val 不一样也不行
    if (root1.val != root2.val) return false;

    // root1 和 root2 该比的都比完了
    return isSameTree(root1.left, root2.left)
        && isSameTree(root1.right, root2.right);
}
```

#### 二叉搜索树

二叉搜索树（Binary Search Tree，简称 BST）是一种很常用的的二叉树。它的定义是：一个二叉树中，任意节点的值要大于等于左子树所有节点的值，且要小于等于右边子树的所有节点的值。

零、判断 BST 的合法性:

```java
boolean isValidBST(TreeNode root) {
    return isValidBST(root, null, null);
}

boolean isValidBST(TreeNode root, TreeNode min, TreeNode max) {
    if (root == null) return true;
    if (min != null && root.val <= min.val) return false;
    if (max != null && root.val >= max.val) return false;
    return isValidBST(root.left, min, root) 
        && isValidBST(root.right, root, max);
}
```

在 BST 中查找一个数是否存在:

```java
boolean isInBST(TreeNode root, int target) {
    if (root == null) return false;
    if (root.val == target)
        return true;
    if (root.val < target) 
        return isInBST(root.right, target);
    if (root.val > target)
        return isInBST(root.left, target);
    // root 该做的事做完了，顺带把框架也完成了，妙
}
```

于是，我们对原始框架进行改造，抽象出一套**针对 BST 的遍历框架**：

```java
void BST(TreeNode root, int target) {
    if (root.val == target)
        // 找到目标，做点什么
    if (root.val < target) 
        BST(root.right, target);
    if (root.val > target)
        BST(root.left, target);
}
```

在 BST 中插入一个数:

```java
TreeNode insertIntoBST(TreeNode root, int val) {
    // 找到空位置插入新节点
    if (root == null) return new TreeNode(val);
    // if (root.val == val)
    //     BST 中一般不会插入已存在元素
    if (root.val < val) 
        root.right = insertIntoBST(root.right, val);
    if (root.val > val) 
        root.left = insertIntoBST(root.left, val);
    return root;
}
```

在 BST 中删除一个数:
1. A 恰好是末端节点，两个子节点都为空，那么它可以当场去世了。
2. A 只有一个非空子节点，那么它要让这个孩子接替自己的位置。
3. A 有两个子节点，麻烦了，为了不破坏 BST 的性质，A 必须找到左子树中最大的那个节点，或者右子树中最小的那个节点来接替自己。我们以第二种方式讲解。

三种情况分析完毕，填入框架，简化一下代码：

```java
TreeNode deleteNode(TreeNode root, int key) {
    if (root == null) return null;
    if (root.val == key) {
        // 这两个 if 把情况 1 和 2 都正确处理了
        if (root.left == null) return root.right;
        if (root.right == null) return root.left;
        // 处理情况 3
        TreeNode minNode = getMin(root.right);
        root.val = minNode.val;
        root.right = deleteNode(root.right, minNode.val);
    } else if (root.val > key) {
        root.left = deleteNode(root.left, key);
    } else if (root.val < key) {
        root.right = deleteNode(root.right, key);
    }
    return root;
}

TreeNode getMin(TreeNode node) {
    // BST 最左边的就是最小的
    while (node.left != null) node = node.left;
    return node;
} 
```

# 动态规划

1. 重叠子问题
2. 最优子结构

状态转移方程

# 查找

## 二分查找

```java
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // 直接返回
            return mid;
        }
    }
    // 直接返回
    return -1;
}

/*
比如说给你有序数组 `nums = [1,2,2,2,3]`，`target` 为 2，此算法返回的索引是 2，没错。但是如果我想得到 `target` 的左侧边界，即索引 1，或者我想得到 `target` 的右侧边界，即索引 3，这样的话此算法是无法处理的。
*/

// 寻找左侧边界的二分搜索
int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定左侧边界
            right = mid - 1;
        }
    }
    // 最后要检查 left 越界的情况
    if (left >= nums.length || nums[left] != target)
        return -1;
    return left;
}

// 寻找右侧边界的二分搜索
int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 别返回，锁定右侧边界
            left = mid + 1;
        }
    }
    // 最后要检查 right 越界的情况
    if (right < 0 || nums[right] != target)
        return -1;
    return right;
}
```
