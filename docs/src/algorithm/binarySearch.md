# 二分算法

## 思想概述

****

二分算法是算法竞赛最经典的算法，它几乎随处可见，请看下面的例子：

> 猜数字：给定某个数 $x, (1 \le x \le 100)$，每次猜数都会明确告诉你这个数相较 $x$ 是大还是小，你该如何快速猜对这个数是多少？
>
> 法一：从 $1$ 开始一个一个地猜，直到猜对为止
>
> 法二：先猜 $50$，如果小于 $x$ 则下次猜 $75$，否则猜 $25$，每次都取剩下一半区间的数，直到猜对为止

对于上述示例，法一没有用到题目的全部条件（每次询问会指明当前猜的数和 $x$ 的大小），相较之下，法二则是效率极高的。

因为法二是基于二分的思想来实现的：

- 第一次猜数取范围的中间值 $50$，即取二分范围的中间值
- 第二次猜数根据第一次的判断再次确定范围，即将二分范围的中间值和目标值比较后确定下一次查找的区间

综上：每次查找取范围的中间值，然后根据区间的单调性来缩小查询范围，将查找范围减小为上一次查找的一半大小，直至找到目标值，显然，二分算法的时间复杂度一般为 $\mathcal{O}(log_2^N)$。

由此可见，二分算法是一种**高效查询某一区间内指定值**的算法，这里的**区间必须满足某种单调性质**——递增、递减或某种唯一性质的有序组合。

## 算法实现

****

由于二分查找取区间中间值的特殊性质，可以将其划分为整数二分和浮点数二分：

- 整数二分：满足绝大部分的二分应用场景，不针对区间进行限制
- 浮点数二分：针对区间范围要求精确到浮点数，很少见

****

### 整数二分

****

在**保证数组为从小到大升序排列**的情况下，这里我们给出 `Java` 的 `Arrays.binarySearch()` 方法的实现，该方法调用了 `binarySearch0` 方法：

```java
// Like public version, but without range checks.
private static int binarySearch0(int[] a, int fromIndex, int toIndex,
                                 int key) {
    int low = fromIndex;
    int high = toIndex - 1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        int midVal = a[mid];
        if (midVal < key)
            low = mid + 1;
        else if (midVal > key)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found.
}
```

但为了适应写题的需求，我们针对上述实现做一些改进，以一个**单调递增的区间**为例：

```java
private static int bs (int l, int r, int target, int[] nums) {
	while (l <= r) {
        int mid = (l + r) >> 1;               // 计算区间中点
    	if (nums[mid] == target) return mid;  // 找到直接返回
    	if (nums[mid] < target) l = mid + 1;  // target 大，则 target 在左半部分，二分区间的左端点 l 要向右更新
    	else r = mid - 1;                     // target 小，则 target 在右半部分，二分区间的右端点 r 要向左更新
    }
    return nums[r] == target ? r : -1;
} 
```

- 若 `int mid = (l + r) >> 1` 其中 `l + r` 在某些条件下可能会发生溢出，因此可以改为 `int mid = l + (r - l) >> 1`
- 如果结束时仍没有找到目标值：
  - `l` 会落在最后一次不满足 `target` 的右边，因此 `l` 可能会出右边界
  - `r` 会落在 `l - 1`，也是最后一次不满足 `target` 的位置，因此 `r` 可能会出左边界
- 对于 `return` 具体的返回值可以根据题目条件灵活修改，这里我们找不到目标值直接返回了 $-1$

****

### 浮点数二分

****

对于浮点数二分的应用场景十分局限，可能只有求根的场景会使用到

```java
private static double bs (double l, double r, double target, double eps) {
    while (r - l > eps) {
        double mid = (l + r) / 2;
        if (mid < target) l = mid;
        else r = mid;
    }
    return l;
}
```

## 经典例题

****

### 搜索插入位置

****

[原题链接](https://leetcode.cn/problems/search-insert-position/)

- 整数二分的板子题
- 利用二分找到指定的目标值即可
- 对于我们的算法模板来说，在结束时仍没有找到目标值：
  - `l` 会落在最后一次不满足 `target` 的右边，即应当插入的数的位置
  - `r` 会落在 `l - 1`，也就是最后一次不满足 `target` 的位置，即应当插入的数的前一个位置
- 因此，需要插入的位置应该是 `l` 或者 `r + 1`

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int l = 0, r = nums.length - 1;
        while (l <= r) {
            int mid = (l + r) >> 1;
            if (nums[mid] == target) return mid;
            if (nums[mid] < target) l = mid + 1;
            else r = mid - 1;
        }
        return l;  // r + 1
    }
}
```

****

### 在排序数组中查找元素的第一个和最后一个位置

****

[原题链接](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

- 数组已从小到大排序，满足单调性，由于要找到两个位置，故需要进行两次二分：
  - 第一遍二分寻找第一个小于 `target` 的数
  - 第二遍二分寻找第一个大于 `target` 的数
- 注意要如何更新区间才能找到满足上面条件的数：
  - 对于第一个位置，需要更新的区间必然在 `target` 左边，故当 `nums[mid] >= target` 时，更新 `r = mid - 1`
  - 对于最后的位置，需要更新的区间必然在 `target` 右边，故当 `nums[mid] <= target` 时，更新 `l = mid + 1`
- 显然，按照上述更新区间的方式：
  - 第一种情况下，当 `nums[mid] == target` 时，区间更新后会不断向左半部分收缩，最终找到第一个数
  - 第二种情况下，当 `nums[mid] == target` 时，区间更新后会不断向右半部分收缩，最终找到第二个数

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int n = nums.length - 1;
        int[] ans = new int[2];
        // 查找最小位置
        int l = 0, r = n;
        while (l <= r) {
            int mid = (l + r) >> 1;
            // 只要大于等于 target 就收缩右边界
            if (nums[mid] < target) l = mid + 1;
            else r = mid - 1;
        }
        // 找不到
        if (l > n || nums[l] != target) {
            ans[0] = ans[1] = -1;
            return ans;
        } else ans[0] = l;
        // 查找最大位置
        // 重置
        l = 0; r = n;
        while (l <= r) {
            int mid = (l + r) >> 1;
            // 只要小于等于 target 就收缩左边界
            if (nums[mid] <= target) l = mid + 1;
            else r = mid - 1;
        }
        ans[1] = l - 1; // l 会落在最后一次不满足条件的位置的右边
        return ans;
    }
}
```

****

### 搜索二维矩阵 II

****

[原题链接](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

- 由于矩阵的每一行都是单调递增的
- 因此我们遍历矩阵的每一行做二分：
  - 对于每一行，设置好边界条件，`l = matrix.length, r = matrix[i].length`
  - 如果在二分中间找到了 `matrix[i][mid] == target` 直接返回即可

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int n = matrix.length, m = matrix[0].length;
        for (int i = 0; i < n; i ++) {   // 遍历矩阵
            int l = 0, r = m - 1;
            while (l <= r) {
                int mid = (l + r) >> 1;  // 二分的中点
                if (matrix[i][mid] == target) return true;
                if (matrix[i][mid] < target) l = mid + 1;
                else r = mid - 1;
            }
        }
        return false;  // 找不到，返回 false
    }
}
```

****

### 搜索二维矩阵

****

[原题链接](https://leetcode.cn/problems/search-a-2d-matrix/)

- 这道题是上一道题目的加强版本
- 由于二维矩阵的每一行是升序，每一列的最后的值也是升序，因此分别做二分
- 第一次二分：
  - 对矩阵的每一行最后一列的值进行二分，确定是否存在某一行的区间包含 `target`
  - 由于我们模板最后一次不满足的条件是 `nums[mid][nums[0].length - 1] < target`
  - 因此最终可能包含 `target` 的区间应为 `nums[l]`
- 第二次二分:
  - 先判断 `l` 是否在合法的区间内
  - 之后对 `nums[l]` 进行二分查找即可


```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int n = matrix.length - 1, m = matrix[0].length - 1;
        // 搜索可能所在的行
        int u = 0, d = n;
        while (u <= d) {
            int mid = (u + d) >> 1;
            if (matrix[mid][m] == target) return true;
            if (matrix[mid][m] < target) u = mid + 1;
            else d = mid - 1;
        }
        if (u > n) return false;
        // 搜索改行可能存在的位置
        int l = 0, r = m;
        while (l <= r) {
            int mid = (l + r) >> 1;
            if (matrix[u][mid] == target) return true;
            if (matrix[u][mid] < target) l = mid + 1;
            else r = mid - 1;
        }
        return false;
    }
}
```