# 算法速查手册：遇到什么题用什么算法

> 本手册按**题目特征**分类，每类列出「识别关键词 → 选择算法 → 例题 → 完整解析 + 代码」。
> 所有代码使用 **C++17**，适用于 CCF CSP、力扣（LeetCode）、洛谷等 OJ 平台。

---

## 目录

| 编号 | 题型关键词 | 算法 |
|------|-----------|------|
| 1 | 按规则操作、逐步推导 | [暴力枚举 / 模拟](#1-暴力枚举--模拟) |
| 2 | 有序序列中找值、最小化最大值 | [二分查找 / 二分答案](#2-二分查找--二分答案) |
| 3 | 局部最优→全局最优、区间选择 | [贪心算法](#3-贪心算法) |
| 4 | 所有可能、路径、子集、排列 | [DFS / 回溯](#4-dfs--回溯) |
| 5 | 最短步数、层次扩展、感染扩散 | [BFS](#5-bfs) |
| 6 | 重叠子问题、最优子结构 | [动态规划（DP）](#6-动态规划dp) |
| 7 | 有向/无向图、路径、联通性 | [图论](#7-图论) |
| 8 | 区间求和/最值、频繁单点更新 | [前缀和 / 差分 / 线段树](#8-前缀和--差分--线段树) |
| 9 | 下一个更大元素、括号匹配 | [单调栈 / 单调队列](#9-单调栈--单调队列) |
| 10 | 集合合并、判断联通 | [并查集](#10-并查集) |
| 11 | 模式串在文本中的出现位置 | [KMP 字符串匹配](#11-kmp-字符串匹配) |
| 12 | 第 k 小/大、动态最值 | [堆（优先队列）](#12-堆优先队列) |

---

## 1. 暴力枚举 / 模拟

### 识别关键词

- 题目描述像「规则说明书」，一步一步按规则执行
- 数据范围很小（n ≤ 1000 甚至 n ≤ 100）
- 题目要求输出某种状态经过若干步后的结果

### 使用场景

直接按题目描述逐步模拟，不需要优化技巧。

---

### 例题 1：报数游戏（模拟）

**题目描述**

n 个人围成一圈，从第 1 个人开始，每次数到第 m 个人就淘汰，求最后剩下的人的编号（约瑟夫问题，小数据模拟版）。

**输入**：两个整数 n、m（1 ≤ n ≤ 100，1 ≤ m ≤ 100）

**输出**：最后剩余的人的编号（编号从 1 开始）

**样例**
```
输入：5 3
输出：4
```

**解析**

数据范围极小，直接用链表或 vector 模拟淘汰过程：
1. 用 vector 保存所有编号
2. 维护当前报数指针，数到第 m 个就 erase
3. 注意指针越界时取模回绕

**代码**

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;
    vector<int> v;
    for (int i = 1; i <= n; i++) v.push_back(i);

    int cur = 0; // 当前指针（下一个报1的人）
    while (v.size() > 1) {
        cur = (cur + m - 1) % v.size(); // 第m个人的下标
        v.erase(v.begin() + cur);
        if (cur == (int)v.size()) cur = 0; // 越界回绕
    }
    cout << v[0] << endl;
    return 0;
}
```

**复杂度**：O(n·m)，n ≤ 100 时完全够用。

---

### 例题 2：字符串替换（模拟）

**题目描述**（改编自 CSP 第 1 题常见类型）

给定字符串 s，将所有连续的空格替换为单个下划线 `_`，并将小写字母转大写。

**输入**：一行字符串（长度 ≤ 1000）

**输出**：处理后的字符串

**样例**
```
输入：hello   world  cpp
输出：HELLO_WORLD_CPP
```

**解析**

按字符逐一处理：
1. 遇到字母 → 转大写输出
2. 遇到空格 → 标记「上一个是空格吗」，若不是则输出 `_`，是则跳过

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s;
    getline(cin, s);
    string res;
    bool lastSpace = false;
    for (char c : s) {
        if (c == ' ') {
            if (!lastSpace) res += '_';
            lastSpace = true;
        } else {
            res += toupper(c);
            lastSpace = false;
        }
    }
    cout << res << endl;
}
```

---

## 2. 二分查找 / 二分答案

### 识别关键词

- 在**有序数组**中找某个值或满足条件的边界
- 题目问「最小的最大值」「最大的最小值」（二分答案）
- 时间复杂度需要从 O(n) 降到 O(log n)

### 使用场景

- 有序序列查找：直接 `lower_bound` / `upper_bound`
- 答案具有单调性（可行/不可行的分界点）→ 二分答案

---

### 例题 3：在排序数组中查找元素（LeetCode 704）

**题目描述**

给定升序整数数组 nums 和目标值 target，返回 target 的下标；不存在则返回 -1。

**样例**
```
输入：nums = [-1,0,3,5,9,12], target = 9
输出：4
```

**解析**

经典二分查找。维护 `[lo, hi]` 闭区间：
- `mid = lo + (hi - lo) / 2`（防溢出）
- `nums[mid] == target` → 返回 mid
- `nums[mid] < target` → `lo = mid + 1`
- `nums[mid] > target` → `hi = mid - 1`

```cpp
#include <bits/stdc++.h>
using namespace std;

int search(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

**复杂度**：O(log n)

---

### 例题 4：砍树（二分答案）

**题目描述**（经典二分答案题型）

有 n 棵树，高度分别为 h[i]。用电锯把所有超过 H 的树都截为 H 米，收集截下的木材，问 H 最大是多少，使得收集到的木材总量 ≥ M。

**输入**：n M，然后 n 个整数 h[i]（1 ≤ n ≤ 10^6，0 ≤ M ≤ 2×10^9）

**输出**：最大的 H

**样例**
```
输入：
4 7
8 3 14 5

验证（H=7）：(8-7)+(14-7) = 1+7 = 8 ≥ 7 ✓
验证（H=8）：(8-8)+(14-8) = 0+6 = 6 < 7 ✗
输出：7
```

**解析**

答案 H 具有单调性：H 越小，收集到的木材越多。
- 二分 H 的范围 `[0, max(h)]`
- `check(H)` = 计算截到高度 H 后总木材量是否 ≥ M

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n; long long M;
    cin >> n >> M;
    vector<long long> h(n);
    long long maxH = 0;
    for (int i = 0; i < n; i++) { cin >> h[i]; maxH = max(maxH, h[i]); }

    auto check = [&](long long H) {
        long long total = 0;
        for (long long x : h) if (x > H) total += x - H;
        return total >= M;
    };

    long long lo = 0, hi = maxH, ans = 0;
    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (check(mid)) { ans = mid; lo = mid + 1; } // H 越大越难满足，尽量大
        else hi = mid - 1;
    }
    cout << ans << endl;
}
```

**复杂度**：O(n log maxH)

---

## 3. 贪心算法

### 识别关键词

- 每次做出**局部最优**选择，最终得到全局最优
- 「尽可能多」「最少操作次数」「区间覆盖/选取」
- 排序后从一端开始选取

### 常见模型

| 模型 | 贪心策略 |
|------|---------|
| 区间调度（选最多不重叠区间） | 按右端点升序排序，每次选结束最早的 |
| 区间覆盖（最少区间覆盖一段） | 按左端点排序，每次在能到达的区间中选右端点最远的 |
| 活动安排 | 同区间调度 |
| 找零钱（硬币面额有包含关系） | 每次选最大面额 |

---

### 例题 5：区间调度（LeetCode 435 / 经典贪心）

**题目描述**

给定若干区间 `[start, end]`，去掉最少数量的区间，使剩余区间互不重叠。返回需要去掉的最少区间数。

**样例**
```
输入：[[1,2],[2,3],[3,4],[1,3]]
输出：1（去掉 [1,3]）
```

**解析**

等价于**最多能选多少互不重叠区间**（答案 = 总数 - 最多选数）。

贪心策略：按**右端点**升序排序，维护当前已选区间的最大右端点 `end`：
- 若下一个区间 `start >= end`，则选它，更新 `end`
- 否则跳过（即去掉它）

```cpp
#include <bits/stdc++.h>
using namespace std;

int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(),
         [](auto& a, auto& b){ return a[1] < b[1]; }); // 按右端点升序
    int count = 0; // 已选数量
    int end = INT_MIN;
    for (auto& iv : intervals) {
        if (iv[0] >= end) { // 不重叠，选它
            end = iv[1];
            count++;
        }
        // 否则跳过（去掉）
    }
    return (int)intervals.size() - count;
}
```

**复杂度**：O(n log n)

---

### 例题 6：分发糖果（LeetCode 135）

**题目描述**

n 个孩子站成一排，每个孩子有评分 ratings[i]。规则：
1. 每个孩子至少 1 颗糖
2. 相邻孩子中评分高的必须比评分低的多

求最少需要多少颗糖。

**样例**
```
输入：[1,0,2]
输出：5（糖果分配：[2,1,2]）
```

**解析**

两次贪心扫描：
1. **从左到右**：若 `ratings[i] > ratings[i-1]`，则 `candy[i] = candy[i-1] + 1`，否则 `candy[i] = 1`
2. **从右到左**：若 `ratings[i] > ratings[i+1]`，则 `candy[i] = max(candy[i], candy[i+1] + 1)`

```cpp
#include <bits/stdc++.h>
using namespace std;

int candy(vector<int>& ratings) {
    int n = ratings.size();
    vector<int> c(n, 1);
    // 左→右
    for (int i = 1; i < n; i++)
        if (ratings[i] > ratings[i-1]) c[i] = c[i-1] + 1;
    // 右→左
    for (int i = n-2; i >= 0; i--)
        if (ratings[i] > ratings[i+1]) c[i] = max(c[i], c[i+1] + 1);
    return accumulate(c.begin(), c.end(), 0);
}
```

**复杂度**：O(n)

---

## 4. DFS / 回溯

### 识别关键词

- 枚举所有**路径**、**子集**、**全排列**
- 走迷宫、找联通分量
- 「所有可能的方案」

### 回溯三要素

1. **选择**：本次递归做什么选择
2. **约束**：什么情况下剪枝（不合法）
3. **目标**：什么情况下记录答案

---

### 例题 7：全排列（LeetCode 46）

**题目描述**

给定不含重复数字的数组 nums，返回其所有全排列。

**样例**
```
输入：[1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

**解析**

经典回溯框架：
- 维护 `path`（当前路径）和 `used`（是否已使用）
- 每次从未使用的数字中选一个加入 path
- path 长度等于 n 时记录答案
- 回溯：撤销选择

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> res;
vector<int> path;
vector<bool> used;

void dfs(vector<int>& nums) {
    if (path.size() == nums.size()) {
        res.push_back(path);
        return;
    }
    for (int i = 0; i < (int)nums.size(); i++) {
        if (used[i]) continue;
        used[i] = true;
        path.push_back(nums[i]);
        dfs(nums);
        path.pop_back();     // 撤销
        used[i] = false;
    }
}

vector<vector<int>> permute(vector<int>& nums) {
    used.assign(nums.size(), false);
    dfs(nums);
    return res;
}
```

**复杂度**：O(n × n!)

---

### 例题 8：岛屿数量（LeetCode 200）

**题目描述**

给定 m×n 的网格，'1' 表示陆地，'0' 表示水，计算岛屿数量。

**样例**
```
输入：
11110
11010
11000
00000
输出：1
```

**解析**

DFS 标记联通分量：
- 遍历每个格子，遇到未访问的 '1' → 岛屿数+1，DFS 将整个岛屿标记为访问
- DFS 向四个方向扩展

```cpp
#include <bits/stdc++.h>
using namespace std;

int dx[] = {0,0,1,-1}, dy[] = {1,-1,0,0};

void dfs(vector<vector<char>>& grid, int x, int y) {
    int m = grid.size(), n = grid[0].size();
    if (x < 0 || x >= m || y < 0 || y >= n || grid[x][y] != '1') return;
    grid[x][y] = '0'; // 标记已访问
    for (int d = 0; d < 4; d++) dfs(grid, x+dx[d], y+dy[d]);
}

int numIslands(vector<vector<char>>& grid) {
    int cnt = 0;
    for (int i = 0; i < (int)grid.size(); i++)
        for (int j = 0; j < (int)grid[0].size(); j++)
            if (grid[i][j] == '1') { dfs(grid, i, j); cnt++; }
    return cnt;
}
```

**复杂度**：O(m × n)

---

## 5. BFS

### 识别关键词

- 求**最短步数**（无权图）
- **层次遍历**（树的层序）
- 感染/扩散问题（从多个源点同时扩散）
- 迷宫最短路径

### BFS 模板

```cpp
queue<状态> q;
visited[起点] = true;
q.push(起点);
int step = 0;
while (!q.empty()) {
    int sz = q.size();
    while (sz--) {
        auto cur = q.front(); q.pop();
        for (auto next : 邻居(cur)) {
            if (!visited[next]) {
                visited[next] = true;
                q.push(next);
            }
        }
    }
    step++;
}
```

---

### 例题 9：迷宫最短路（经典 BFS）

**题目描述**

n×m 的网格，'.' 可走，'#' 是墙，从 S 出发找到 E 的最短步数（上下左右移动）。

**样例**
```
输入：
4 4
S..#
.##.
....
.#.E
输出：7
```

**解析**

BFS 每扩展一层 = 走一步，第一次到达终点时的步数即最短。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;
    vector<string> grid(n);
    for (auto& row : grid) cin >> row;

    int sx, sy, ex, ey;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++) {
            if (grid[i][j] == 'S') { sx = i; sy = j; }
            if (grid[i][j] == 'E') { ex = i; ey = j; }
        }

    int dx[] = {0,0,1,-1}, dy[] = {1,-1,0,0};
    vector<vector<int>> dist(n, vector<int>(m, -1));
    queue<pair<int,int>> q;
    dist[sx][sy] = 0;
    q.push({sx, sy});

    while (!q.empty()) {
        auto [x, y] = q.front(); q.pop();
        for (int d = 0; d < 4; d++) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx >= 0 && nx < n && ny >= 0 && ny < m
                && grid[nx][ny] != '#' && dist[nx][ny] == -1) {
                dist[nx][ny] = dist[x][y] + 1;
                q.push({nx, ny});
            }
        }
    }
    cout << dist[ex][ey] << endl;
}
```

**复杂度**：O(n × m)

---

### 例题 10：腐烂的橘子（LeetCode 994，多源 BFS）

**题目描述**

网格中 0=空，1=新鲜橘子，2=腐烂橘子。每分钟腐烂橘子感染四邻新鲜橘子，问所有橘子腐烂需要几分钟，不能则返回 -1。

**解析**

多源 BFS：将所有腐烂橘子同时入队作为第 0 层，BFS 扩展即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

int orangesRotting(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    queue<pair<int,int>> q;
    int fresh = 0;
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 2) q.push({i, j});
            if (grid[i][j] == 1) fresh++;
        }
    if (fresh == 0) return 0;

    int dx[] = {0,0,1,-1}, dy[] = {1,-1,0,0};
    int minutes = 0;
    while (!q.empty() && fresh > 0) {
        minutes++;
        int sz = q.size();
        while (sz--) {
            auto [x, y] = q.front(); q.pop();
            for (int d = 0; d < 4; d++) {
                int nx = x+dx[d], ny = y+dy[d];
                if (nx>=0 && nx<m && ny>=0 && ny<n && grid[nx][ny]==1) {
                    grid[nx][ny] = 2;
                    fresh--;
                    q.push({nx, ny});
                }
            }
        }
    }
    return fresh == 0 ? minutes : -1;
}
```

**复杂度**：O(m × n)

---

## 6. 动态规划（DP）

### 识别关键词

- 「最多/最少/最长/最大」且有**重叠子问题**
- 当前状态依赖之前的状态
- 「能否达到」「方案数」

### DP 三步法

1. **定义状态** `dp[i]` 或 `dp[i][j]` 的含义
2. **写出状态转移方程**
3. **确定初始值和遍历顺序**

### 常见 DP 类型

| 类型 | 状态定义 | 转移方程举例 |
|------|---------|------------|
| 线性DP | `dp[i]` = 考虑前i个元素的答案 | `dp[i] = max(dp[i-1]+a[i], a[i])` |
| 背包DP | `dp[i][j]` = 前i件物品容量j | `dp[i][j] = max(dp[i-1][j], dp[i-1][j-w]+v)` |
| 区间DP | `dp[l][r]` = 区间[l,r]的答案 | `dp[l][r] = min(dp[l][k]+dp[k+1][r]+cost)` |
| 树形DP | `dp[u]` = 以u为根的子树答案 | 在DFS中合并子节点 |

---

### 例题 11：最长递增子序列（LIS，LeetCode 300）

**题目描述**

给定整数数组 nums，找最长严格递增子序列的长度。

**样例**
```
输入：[10,9,2,5,3,7,101,18]
输出：4（[2,3,7,101]）
```

**解析（O(n²) DP）**

`dp[i]` = 以 `nums[i]` 结尾的最长递增子序列长度。

对于每个 i，枚举 j < i，若 `nums[j] < nums[i]`，则 `dp[i] = max(dp[i], dp[j]+1)`。

```cpp
#include <bits/stdc++.h>
using namespace std;

int lengthOfLIS(vector<int>& nums) {
    int n = nums.size();
    vector<int> dp(n, 1);
    int ans = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++)
            if (nums[j] < nums[i]) dp[i] = max(dp[i], dp[j]+1);
        ans = max(ans, dp[i]);
    }
    return ans;
}
```

**O(n log n) 贪心+二分优化**：维护一个辅助数组 `tails`，用 `lower_bound` 替换。

```cpp
int lengthOfLIS_fast(vector<int>& nums) {
    vector<int> tails;
    for (int x : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) tails.push_back(x);
        else *it = x;
    }
    return tails.size();
}
```

---

### 例题 12：0/1 背包（经典 DP）

**题目描述**

有 n 件物品，背包容量 W。第 i 件物品重量 w[i]，价值 v[i]，每件只能选一次。求最大价值。

**解析**

`dp[j]` = 容量为 j 时能装的最大价值。

**逆序遍历**保证每件物品只用一次（若正序则等同于完全背包）。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, W;
    cin >> n >> W;
    vector<int> w(n), v(n);
    for (int i = 0; i < n; i++) cin >> w[i] >> v[i];

    vector<long long> dp(W+1, 0);
    for (int i = 0; i < n; i++)
        for (int j = W; j >= w[i]; j--) // 逆序！
            dp[j] = max(dp[j], dp[j - w[i]] + v[i]);

    cout << dp[W] << endl;
}
```

**复杂度**：O(n × W)

---

### 例题 13：最长公共子序列（LCS，LeetCode 1143）

**题目描述**

给定两个字符串 text1 和 text2，求最长公共子序列的长度。

**样例**
```
输入：text1 = "abcde", text2 = "ace"
输出：3（"ace"）
```

**解析**

二维 DP：`dp[i][j]` = `text1[0..i-1]` 与 `text2[0..j-1]` 的最长公共子序列长度。

$$dp[i][j] = \begin{cases} dp[i-1][j-1]+1 & \text{if } text1[i-1]==text2[j-1] \\ \max(dp[i-1][j],\; dp[i][j-1]) & \text{otherwise} \end{cases}$$

```cpp
#include <bits/stdc++.h>
using namespace std;

int longestCommonSubsequence(string text1, string text2) {
    int m = text1.size(), n = text2.size();
    vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (text1[i-1] == text2[j-1]) dp[i][j] = dp[i-1][j-1] + 1;
            else dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
        }
    return dp[m][n];
}
```

**复杂度**：O(m × n)

---

## 7. 图论

### 识别关键词

- 节点+边的关系网络
- 求最短路、判断联通、检测环、拓扑排序
- 「任意两点距离」「能否到达」

---

### 例题 14：拓扑排序（课程表，LeetCode 207）

**题目描述**

n 门课，给定先修关系 `[a, b]` 表示先修 b 才能修 a。判断是否能完成所有课程。

**样例**
```
输入：n=2, prerequisites=[[1,0]]
输出：true（先修0，再修1）
```

**解析**

检测有向图是否有环 → 拓扑排序（Kahn 算法）：
1. 计算所有节点的入度
2. 将入度为 0 的节点加入队列
3. 每次弹出一个节点，将其邻居入度减 1，若入度变 0 则入队
4. 若最终处理了 n 个节点 → 无环（返回 true）

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canFinish(int n, vector<vector<int>>& pre) {
    vector<int> indegree(n, 0);
    vector<vector<int>> adj(n);
    for (auto& p : pre) {
        adj[p[1]].push_back(p[0]);
        indegree[p[0]]++;
    }
    queue<int> q;
    for (int i = 0; i < n; i++) if (indegree[i] == 0) q.push(i);
    int cnt = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        cnt++;
        for (int v : adj[u])
            if (--indegree[v] == 0) q.push(v);
    }
    return cnt == n;
}
```

**复杂度**：O(V + E)

---

### 例题 15：Dijkstra 单源最短路

**题目描述**

有 n 个城市，m 条有向带权边，求从城市 1 到所有城市的最短距离。

**解析**

Dijkstra 算法（贪心+最小堆）：
1. `dist[src] = 0`，其他为 ∞
2. 每次从堆中取出距离最小的节点 u
3. 松弛 u 的所有出边：若 `dist[u] + w < dist[v]`，则更新并入堆

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef pair<long long,int> pli;

int main() {
    int n, m;
    cin >> n >> m;
    vector<vector<pair<int,long long>>> adj(n+1);
    for (int i = 0; i < m; i++) {
        int u, v; long long w;
        cin >> u >> v >> w;
        adj[u].push_back({v, w});
    }

    vector<long long> dist(n+1, LLONG_MAX);
    priority_queue<pli, vector<pli>, greater<pli>> pq;
    dist[1] = 0;
    pq.push({0, 1});

    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue; // 过期条目
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }

    for (int i = 1; i <= n; i++)
        cout << (dist[i] == LLONG_MAX ? -1 : dist[i]) << "\n";
}
```

**复杂度**：O((V + E) log V)

---

## 8. 前缀和 / 差分 / 线段树

### 识别关键词

- **前缀和**：频繁查询某区间的和/均值，数组静态不变
- **差分数组**：频繁对某区间整体加减一个值，最后查询每个位置的值
- **线段树**：既有区间更新又有区间查询（动态）

---

### 例题 16：区间求和（前缀和）

**题目描述**

给定数组 a，多次查询区间 [l, r] 的和。

**解析**

预处理 `prefix[i] = a[0] + a[1] + ... + a[i-1]`，查询 [l,r]（0-indexed）= `prefix[r+1] - prefix[l]`。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, q;
    cin >> n >> q;
    vector<long long> a(n), prefix(n+1, 0);
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        prefix[i+1] = prefix[i] + a[i];
    }
    while (q--) {
        int l, r; cin >> l >> r; // 0-indexed
        cout << prefix[r+1] - prefix[l] << "\n";
    }
}
```

**复杂度**：预处理 O(n)，每次查询 O(1)

---

### 例题 17：区间批量加值（差分数组）

**题目描述**

给定数组 a，m 次操作每次对 [l, r] 区间所有元素加 v，最后输出数组。

**解析**

差分数组 `d[i] = a[i] - a[i-1]`：对 [l,r] 加 v 等价于 `d[l] += v; d[r+1] -= v`。
最后前缀和还原：`a[i] = d[0] + d[1] + ... + d[i]`。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;
    vector<long long> d(n+2, 0);
    for (int i = 0; i < n; i++) {
        long long x; cin >> x;
        d[i] += x; d[i+1] -= x; // 建差分
    }
    while (m--) {
        int l, r; long long v;
        cin >> l >> r >> v; // 0-indexed
        d[l] += v; d[r+1] -= v;
    }
    long long cur = 0;
    for (int i = 0; i < n; i++) {
        cur += d[i];
        cout << cur << " \n"[i==n-1];
    }
}
```

**复杂度**：O(n + m)

---

## 9. 单调栈 / 单调队列

### 识别关键词

- 「下一个更大/更小元素」
- 「每个位置能看多远」
- 「滑动窗口最值」（单调队列）
- 直方图最大矩形

---

### 例题 18：每日温度（LeetCode 739，单调栈）

**题目描述**

给定每天温度 T[i]，求每天需要等几天才能遇到更高温度。

**样例**
```
输入：[73,74,75,71,69,72,76,73]
输出：[1,1,4,2,1,1,0,0]
```

**解析**

维护一个**单调递减栈**（存下标）：
- 当前温度 > 栈顶温度 → 栈顶元素的答案确定 → 弹出、计算差值

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> dailyTemperatures(vector<int>& T) {
    int n = T.size();
    vector<int> ans(n, 0);
    stack<int> stk; // 存下标，栈内温度单调递减
    for (int i = 0; i < n; i++) {
        while (!stk.empty() && T[i] > T[stk.top()]) {
            int j = stk.top(); stk.pop();
            ans[j] = i - j;
        }
        stk.push(i);
    }
    return ans;
}
```

**复杂度**：O(n)

---

### 例题 19：滑动窗口最大值（LeetCode 239，单调队列）

**题目描述**

给定数组 nums 和窗口大小 k，求每个滑动窗口中的最大值。

**解析**

用双端队列维护单调递减序列（存下标）：
- 队头过期（下标 ≤ i-k）→ 弹出队头
- 新元素 ≥ 队尾元素 → 弹出队尾（队尾不可能成为最大值）
- 队头即为当前窗口最大值

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq; // 存下标，对应值单调递减
    vector<int> res;
    for (int i = 0; i < (int)nums.size(); i++) {
        while (!dq.empty() && dq.front() <= i-k) dq.pop_front(); // 过期
        while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k-1) res.push_back(nums[dq.front()]);
    }
    return res;
}
```

**复杂度**：O(n)

---

## 10. 并查集

### 识别关键词

- 「合并集合」「判断是否同属一组」
- 动态连通性
- 关系网络（朋友的朋友）

---

### 例题 20：朋友圈（LeetCode 547）

**题目描述**

n 个学生，`isConnected[i][j] = 1` 表示 i 和 j 直接认识，求朋友圈总数。

**解析**

用并查集合并所有直接认识的学生，最终不同的根节点数即为朋友圈数。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, rank_;
    int components;
    DSU(int n) : parent(n), rank_(n, 0), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }
    void unite(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return;
        if (rank_[px] < rank_[py]) swap(px, py);
        parent[py] = px;
        if (rank_[px] == rank_[py]) rank_[px]++;
        components--;
    }
};

int findCircleNum(vector<vector<int>>& isConnected) {
    int n = isConnected.size();
    DSU dsu(n);
    for (int i = 0; i < n; i++)
        for (int j = i+1; j < n; j++)
            if (isConnected[i][j]) dsu.unite(i, j);
    return dsu.components;
}
```

**复杂度**：O(n² × α(n))，α 为阿克曼函数的反函数，近似 O(1)

---

## 11. KMP 字符串匹配

### 识别关键词

- 在长文本 T 中查找模式串 P 的**所有出现位置**
- 判断字符串是否是另一个字符串的子串
- 字符串的**最短循环节**

---

### 例题 21：实现 strStr（LeetCode 28）

**题目描述**

给定文本 haystack 和模式 needle，返回 needle 在 haystack 中第一次出现的位置，不存在返回 -1。

**解析**

KMP 算法分两步：
1. 构建失配函数 `next[]`（`next[i]` = 模式串 P[0..i] 的最长真前缀等于真后缀的长度）
2. 利用 `next[]` 避免重复比较

```cpp
#include <bits/stdc++.h>
using namespace std;

// 构建 next 数组
vector<int> buildNext(const string& p) {
    int m = p.size();
    vector<int> next(m, 0);
    int k = 0;
    for (int i = 1; i < m; i++) {
        while (k > 0 && p[k] != p[i]) k = next[k-1];
        if (p[k] == p[i]) k++;
        next[i] = k;
    }
    return next;
}

int strStr(string haystack, string needle) {
    if (needle.empty()) return 0;
    string s = needle + "#" + haystack; // 拼接技巧（简化版）
    auto next = buildNext(s);
    int m = needle.size(), n = haystack.size();
    for (int i = m+1; i < (int)s.size(); i++)
        if (next[i] == m) return i - 2*m; // 找到
    return -1;
}
```

**复杂度**：O(n + m)

---

## 12. 堆（优先队列）

### 识别关键词

- 动态维护「最大值」「最小值」
- 「第 k 大/小」
- 多路归并、贪心中的最优选择

---

### 例题 22：数组中第 K 个最大元素（LeetCode 215）

**题目描述**

给定整数数组 nums，返回第 k 个最大元素（不是第 k 个不同元素）。

**解析**

维护大小为 k 的**小顶堆**：
- 遍历数组，将元素入堆
- 若堆大小 > k，弹出堆顶（最小值）
- 最终堆顶即为第 k 大元素

```cpp
#include <bits/stdc++.h>
using namespace std;

int findKthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> minHeap; // 小顶堆
    for (int x : nums) {
        minHeap.push(x);
        if ((int)minHeap.size() > k) minHeap.pop();
    }
    return minHeap.top();
}
```

**复杂度**：O(n log k)

---

### 例题 23：合并 K 个升序链表（LeetCode 23）

**题目描述**

合并 k 个升序链表，返回合并后的升序链表。

**解析**

用小顶堆（按节点值排序）：
- 将每个链表的头节点入堆
- 每次取出最小节点，将其下一个节点（若存在）入堆

```cpp
#include <bits/stdc++.h>
using namespace std;

struct ListNode { int val; ListNode* next; ListNode(int v): val(v), next(nullptr){} };

ListNode* mergeKLists(vector<ListNode*>& lists) {
    auto cmp = [](ListNode* a, ListNode* b){ return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
    for (auto node : lists) if (node) pq.push(node);

    ListNode dummy(0);
    ListNode* cur = &dummy;
    while (!pq.empty()) {
        auto node = pq.top(); pq.pop();
        cur->next = node;
        cur = cur->next;
        if (node->next) pq.push(node->next);
    }
    return dummy.next;
}
```

**复杂度**：O(n log k)，n 为所有节点总数

---

## 快速题型判断流程图

```
拿到题目
    │
    ├─ 数据范围极小（n≤100）或按规则一步步操作？ ──→ 暴力模拟
    │
    ├─ 有序序列中找边界，或答案具有单调性？ ──→ 二分查找 / 二分答案
    │
    ├─ 求最多/最少，局部最优可推全局最优？ ──→ 贪心
    │
    ├─ 网格/图/树，求联通分量或路径？
    │     ├─ 求最短步数（无权）？ ──→ BFS
    │     └─ 枚举所有路径/状态？ ──→ DFS / 回溯
    │
    ├─ 最优解，且有重叠子问题？ ──→ 动态规划
    │     ├─ 序列问题 ──→ 线性DP
    │     ├─ 选/不选问题 ──→ 背包DP
    │     └─ 区间问题 ──→ 区间DP
    │
    ├─ 节点+边，求最短路？
    │     ├─ 非负权 ──→ Dijkstra
    │     ├─ 含负权 ──→ Bellman-Ford / SPFA
    │     └─ 所有点对 ──→ Floyd-Warshall
    │
    ├─ 有向图无环，求顺序？ ──→ 拓扑排序
    │
    ├─ 区间求和/更新，静态 ──→ 前缀和 / 差分
    │            动态 ──→ 线段树 / 树状数组
    │
    ├─ 下一个更大/更小元素？ ──→ 单调栈
    ├─ 滑动窗口最值？ ──→ 单调队列
    ├─ 集合合并/联通判断？ ──→ 并查集
    ├─ 字符串子串匹配？ ──→ KMP
    └─ 动态最值 / 第k大 ──→ 堆（优先队列）
```

---

## 各算法时间复杂度速查

| 算法 | 平均复杂度 | 适用数据规模（参考） |
|------|-----------|-------------------|
| 暴力枚举 | O(n²) ~ O(n³) | n ≤ 1000 |
| 二分查找 | O(log n) | n ≤ 10^9 |
| 二分答案 | O(n log V) | n ≤ 10^6 |
| 贪心（排序后） | O(n log n) | n ≤ 10^6 |
| DFS / BFS | O(V + E) | V,E ≤ 10^6 |
| 线性DP | O(n) ~ O(n²) | n ≤ 10^4 |
| 背包DP | O(n × W) | n×W ≤ 10^7 |
| Dijkstra（堆） | O((V+E) log V) | V,E ≤ 10^5 |
| Floyd-Warshall | O(V³) | V ≤ 500 |
| 拓扑排序 | O(V + E) | V,E ≤ 10^5 |
| 前缀和 | O(n) 预处理，O(1) 查询 | n ≤ 10^7 |
| 线段树 | O(log n) 更新/查询 | n ≤ 10^5 |
| 并查集 | O(α(n)) ≈ O(1) | n ≤ 10^6 |
| KMP | O(n + m) | n,m ≤ 10^6 |
| 堆操作 | O(log n) | n ≤ 10^6 |

---

*本手册持续更新，涵盖 CCF CSP 常见题型。建议配合 [300+ 分方案](README.md) 一起使用。*
