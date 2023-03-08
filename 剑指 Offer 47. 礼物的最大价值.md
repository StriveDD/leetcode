# [剑指 Offer 47. 礼物的最大价值](https://leetcode.cn/problems/li-wu-de-zui-da-jie-zhi-lcof/)

### 题目介绍

在一个 `m * n` 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0

你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角

给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

<br>

示例1：

```
输入: 
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 12
解释: 路径 1→3→5→2→1 可以拿到最多价值的礼物
```

<br>

提示：

-   `0 < grid.length <= 200`
-   `0 < grid[0].length <= 200`

<br>

### 参考解法

#### 方法一：暴力递归

对棋盘上的任意一个位置而言，都有两种选择，分别是**向右**和**向下**，所以我们考虑用递归来完成这种重复任务

对于每个位置(状态)而言，需要记录的变量就是行号和列号(状态变量)，结合题目意思思考可知：

$dfs(row, col)$ 表示从 $(row, col)$ 位置开始到棋盘右下角可以得到的最大礼物价值

现在我们来处理一下递归函数的一些细节问题：

- 递归出口：这里的出口有两个
    - 当位置越界时，直接返回系统最小值，表示该路径无效(因为题目要求最大价值)
    - 当位置来到了右下角，说明找到了一条路径，此时返回右下角礼物的价值即可
- 递归的内容：根据题目意思，我们只能向右和向下走，所以可知：
    - $right$ 表示向右走可以获取的最大价值
    - $down$ 表示向下走可以获取的最大价值
- 递归返回值：根据递归函数的定义，返回值为 $max(right, down) + 本身的价值$



#### **C++**

```C++
class Solution {
public:
    int dfs(vector<vector<int>> &grid, int row, int col) {
        if (row < 0 || row >= grid.size() || col < 0 || col >= grid[0].size()) 
            return INT_MIN;
        // 到达右下角之后，直接返回它的礼物价值即可
        if (row == grid.size() - 1 && col == grid[0].size() - 1)
            return grid[row][col];
        int right = dfs(grid, row, col + 1);        // 向右移动一格
        int down = dfs(grid, row + 1, col);         // 向左移动一格
        return max(right, down) + grid[row][col];       // 不管怎么移动都要加上自身的礼物价值
    }

    int maxValue(vector<vector<int>>& grid) {
        return dfs(grid, 0, 0);
    }
};
```



### **Java**

```Java
class Solution {
    public int dfs(int[][] grid, int row, int col) {
        if (row < 0 || row >= grid.length || col < 0 || col >= grid[0].length) 
            return Integer.MIN_VALUE;
        if (row == grid.length - 1 && col == grid[0].length - 1)    // 到达右下角之后，直接返回它的礼物价值即可
            return grid[row][col];
        int right = dfs(grid, row, col + 1);        // 向右移动一格
        int down = dfs(grid, row + 1, col);         // 向左移动一格
        return Math.max(right, down) + grid[row][col];      // 不管怎么移动都要加上自身的礼物价值
    }

    public int maxValue(int[][] grid) {
        return dfs(grid, 0, 0);
    }
}
```

时间复杂度: $O(2^n)$，其中 $n$ 是二维数组行的长度 + 列的长度

空间复杂度: $O(1)$，不算系统栈空间



<br>

### 方法二：记忆化搜索

根据上面的递归方法可知，在递归过程中存在重复计算，比如，如果从某个节点 $(row, col)$ 开始
- 先向右走，再向下走，那么当前位置为 $(row + 1, col + 1)$

- 先向下走，再向右走，那么当前位置为 $(row + 1, col + 1)$

  由此可以说明，从 $(row, col)$ 开始，用可以用不同的方式走到 $(row + 1, col + 1)$，然后都会调用 

  $dfs(row + 1, col + 1)$，说明 $dfs(row + 1, col + 1)$ 会被重复计算，导致时间复杂度激增，为了防

  止重复计算，我们可以使用**记忆化搜索**来记录每个位置的值

- 对于每个位置(状态)而言，需要记录的变量就是行号和列号(状态变量)

- 那么我们用一个二维数组 $cache$ 来保存每个位置计算的值，这样当某个位置需要计算时，先去 $cache$ 查看
    
    该位置是否被计算过
    
    - 如果没有被计算过，那么进行计算，并把计算结果保存到 $cache$ 中
    - 否则就直接从 $cache$ 拿出答案，防止重复计算



### **C++**

```C++
class Solution {
public:
    int dfs(vector<vector<int>> &grid, int row, int col, vector<vector<int>> &cache) {
        if (row < 0 || row >= grid.size() || col < 0 || col >= grid[0].size()) 
            return INT_MIN;
        if (row == grid.size() - 1 && col == grid[0].size() - 1) 
            return grid[row][col];
        // 当前状态被计算过了，直接使用答案
        if (cache[row][col] != -1) return cache[row][col];
        int right = dfs(grid, row, col + 1, cache);
        int down = dfs(grid, row + 1, col, cache);
        // 当前状态没有被计算过，就把计算结果保存到 cache 中
        cache[row][col] = max(right, down) + grid[row][col];
        return max(right, down) + grid[row][col];
    }

    int maxValue(vector<vector<int>>& grid) {
        // 创建二维数组 cache，初始化 -1，表示每个位置都没被计算
        vector<vector<int>> cache(grid.size(), vector<int>(grid[0].size(), -1));
        return dfs(grid, 0, 0, cache);
    }
};
```



### **Java**

```Java
class Solution {
    public int dfs(int[][] grid, int row, int col, int[][] cache) {
        if (row < 0 || row >= grid.length || col < 0 || col >= grid[0].length) return Integer.MIN_VALUE;
        if (row == grid.length - 1 && col == grid[0].length - 1) return grid[row][col];
        // 当前状态被计算过了，直接使用答案
        if (cache[row][col] != -1) return cache[row][col];
        int right = dfs(grid, row, col + 1, cache);
        int down = dfs(grid, row + 1, col, cache);
        // 当前状态没有被计算过，就把计算结果保存到 cache 中
        cache[row][col] = Math.max(right, down) + grid[row][col];
        return Math.max(right, down) + grid[row][col];
    }

    public int maxValue(int[][] grid) {
        // 创建二维数组 cache，初始化 -1，表示每个位置都没被计算
        int[][] cache = new int[grid.length][grid[0].length];
        for (int i = 0; i < grid.length; i++)
            Arrays.fill(cache[i], -1);
        return dfs(grid, 0, 0, cache);
    }
}
```

时间复杂度: $O(n)$，其中 $n$ 是二维数组中所有元素的个数

空间复杂度: $O(n)$，使用了 $cache$ 来保存结构



<br>

### 方法三：动态规划

之前我们在写暴力递归时，强调了：**对于每个位置(状态)而言，需要记录的变量就是行号和列号(状态变量)**
1. 所以每个位置就是状态，而状态由 $(row, col)$ 表示

2. 在每个状态下的选择就是**向右**或者**向下**

  由此我们可以改出动态规划

3. 递归中的可变参数就是 $dp$ 数组的维度

4. 递归出口就是 $dp$ 数组的初值

5. 递归内容就是状态转移方程

6. 根据递归的方向来确定遍历顺序
    - 递归中是向右或向下进行递归，相当于当前状态需要向右或向下的值
    - 所以我们需要从下向上，从右向左进行遍历



### **C++**

```C++
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        int rowSize = grid.size(), colSize = grid[0].size();
        // 数组长度开大一个，这样可以根据 dfs 的递归出口初始化 dp 数组
        int dp[rowSize + 1][colSize + 1];
        dp[rowSize - 1][colSize - 1] = grid[rowSize - 1][colSize - 1];
        for (int i = 0; i < colSize; i++) dp[rowSize][i] = INT_MIN;
        for (int i = 0; i < rowSize; i++) dp[i][colSize] = INT_MIN;
        for (int row = rowSize - 1; row >= 0; row--) {
            for (int col = colSize - 1; col >= 0; col--) {
                // 右下角已被初始化，所以直接跳过
                if (row == rowSize - 1 && col == colSize - 1) continue;
                dp[row][col] = max(dp[row + 1][col], dp[row][col + 1]) + grid[row][col];
            }
        }
        return dp[0][0];
    }
};
```



### **Java**

```Java
class Solution {
    public int maxValue(int[][] grid) {
        int rowSize = grid.length, colSize = grid[0].length;
        // 数组长度开大一个，这样可以根据 dfs 的递归出口初始化 dp 数组
        int[][] dp = new int[rowSize + 1][colSize + 1];
        dp[rowSize - 1][colSize - 1] = grid[rowSize - 1][colSize - 1];
        for (int i = 0; i < colSize; i++) dp[rowSize][i] = Integer.MIN_VALUE;
        for (int i = 0; i < rowSize; i++) dp[i][colSize] = Integer.MIN_VALUE;
        for (int row = rowSize - 1; row >= 0; row--) {
            for (int col = colSize - 1; col >= 0; col--) {
                // 右下角已被初始化，所以直接跳过
                if (row == rowSize - 1 && col == colSize - 1) continue;
                dp[row][col] = Math.max(dp[row + 1][col], dp[row][col + 1]) + grid[row][col];
            }
        }
        return dp[0][0];
    }
}
```

时间复杂度: $O(n)$，其中 $n$ 是二维数组中所有元素的个数

空间复杂度: $O(n)$
