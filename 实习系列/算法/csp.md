### 前缀合 & 差分

**1. 知识点：一维差分（区间修改优化）**
*   **核心逻辑**：通过 `D[L] += v` 和 `D[R+1] -= v` 将 $O(N)$ 的区间操作降为 $O(1)$，最后通过前缀和还原。
*   **对应题目**：[洛谷 P2367 语文成绩](https://www.luogu.com.cn/problem/P2367)

**2. 知识点：二维前缀和（子矩阵求和）**
*   **核心逻辑**：基于容斥原理，预处理 $O(N^2)$，查询 $O(1)$。
*   **对应题目**：[洛谷 P1719 最大加权矩形](https://www.luogu.com.cn/problem/P1719)

**3. 知识点：二维差分（子矩阵修改）**

*   **核心逻辑**：操作 4 个关键点，通过一次二维前缀和还原，将子矩阵批量修改优化至 $O(1)$。
*   **对应题目**：[洛谷 P3397 地毯](https://www.luogu.com.cn/problem/P3397)

**4. 知识点：差分与实际场景建模（综合应用）**
*   **核心逻辑**：将路线旅行转化为区间覆盖次数统计，结合贪心决策求最优解。
*   **对应题目**：[洛谷 P3406 海底高铁](https://www.luogu.com.cn/problem/P3406)

**5. 知识点：前缀和与同余定理（进阶统计）**

*   **核心逻辑**：利用 $(S[R] - S[L-1]) \pmod K = 0 \Rightarrow S[R] \equiv S[L-1] \pmod K$ 进行 $O(N)$ 优化。
*   **对应题目**：[洛谷 P3131 [USACO16JAN] Subsequences Summing to Sevens S](https://www.luogu.com.cn/problem/P3131)

**6. 知识点：二分答案 + 差分验证（复合应用）**
*   **核心逻辑**：利用答案的单调性，将“求最优解”转化为“判定可行性”。在 `check` 函数中使用差分数组 $O(N)$ 快速验证区间覆盖是否溢出。
*   **对应题目**：[洛谷 P1083 [NOIP2012 提高组] 借教室](https://www.luogu.com.cn/problem/P1083)

**7. 知识点：差分数组的状态平衡（逻辑深挖）**
*   **核心逻辑**：理解差分操作对全局状态的影响。通过统计差分数组中正数总和 $P$ 与负数绝对值总和 $Q$，最少步数为 $\max(P, Q)$，最终序列种数为 $|P-Q|+1$。
*   **对应题目**：[洛谷 P4552 [Poetize6] IncDec Sequence](https://www.luogu.com.cn/problem/P4552)

**8. 知识点：异或前缀和（位运算抵消）**
*   **核心逻辑**：利用异或自反性 $x \oplus x = 0$，实现区间异或和查询：$XorSum(L, R) = S[R] \oplus S[L-1]$。常用于统计出现奇偶次数或处理二进制特性。
*   **对应题目**：[洛谷 P1469 找出现一次的数](https://www.luogu.com.cn/problem/P1469)

**9. 知识点：桶统计与离散化前缀和（真题高频）**
*   **核心逻辑**：针对不连续的数据，先排序，再利用前缀和分别统计不同类别（如 result 为 0 或 1）的频率，从而在 $O(N)$ 内扫描出最优阈值。
*   **对应题目**：[CCF-CSP 202012-2 期末预测之最佳阈值](https://www.luogu.com.cn/problem/P7107)（注：此为对应逻辑的 CSP 真题）

**10. 知识点：状态偏移与负数下标处理**
*   **核心逻辑**：在处理“数量相等”或“和为 0”的问题时，前缀和可能出现负数。通过 `Offset`（给下标加一个大常数）或 `HashMap` 记录该和第一次出现的位置。
*   **对应题目**：[洛谷 P2697 宝石串](https://www.luogu.com.cn/problem/P2697)

---

### 二分

**1. 知识点：整数二分边界（寻找起始与终止位置）**
*   **核心逻辑**：区分“寻找第一个满足条件的左边界”和“最后一个满足条件的右边界”。关键在于 `mid` 的取法（是否 +1）以及 `l` 或 `r` 的更新，防止死循环。
*   **对应题目**：[AcWing 789 数的范围](https://www.acwing.com.cn/problem/content/791/)（注：此题逻辑同洛谷 P2249）

**2. 知识点：二分答案 - 基础计数/切割模型**

*   **核心逻辑**：给定资源总量，求满足需求的最大单位尺寸。`check` 函数通常为线性累加。
*   **对应题目**：[洛谷 P1873 砍树](https://www.luogu.com.cn/problem/P1873)、[洛谷 P2440 木材加工](https://www.luogu.com.cn/problem/P2440)

**3. 知识点：二分答案 - 最大化最小值（Max-Min）**
*   **核心逻辑**：典型的“跳石头”模型。通过二分尝试“最小距离”，在 `check` 函数中使用**贪心策略**：凡是小于 `mid` 的间隔一律通过操作（如移走石头）消除，看操作次数是否超限。
*   **对应题目**：[洛谷 P2678 跳石头](https://www.luogu.com.cn/problem/P2678)

**4. 知识点：二分答案 - 最小化最大值（Min-Max）**
*   **核心逻辑**：用于均衡负载或分段问题。`check` 函数逻辑：只要当前段累加和超过 `mid`，就必须强制开启新段。注意左边界 `left` 应初始化为序列中的最大值。
*   **对应题目**：[洛谷 P1182 数列分段 Section II](https://www.luogu.com.cn/problem/P1182)

**5. 知识点：二分答案 - 插入类构造模型**
*   **核心逻辑**：在给定的不均匀间隙中插入元素，使最大间距最小。关键点在于 `check` 函数中计算两个原始点 $D$ 之间需补发的元素个数。
*   **对应题目**：[洛谷 P3853 路标设置](https://www.luogu.com.cn/problem/P3853)

**6. 知识点：浮点数二分（精度控制与金融/几何建模）**
*   **核心逻辑**：取消 `+1/-1` 的边界处理，使用 `l = mid` 和 `r = mid`。终止条件采用固定循环次数（如 `for i < 100`）以获得超越 `double` 极限的稳健精度。常用于求解方程根、月利率或几何极值。
*   **对应题目**：[洛谷 P1024 一元三次方程求解](https://www.luogu.com.cn/problem/P1024)、[洛谷 P1163 银行贷款](https://www.luogu.com.cn/problem/P1163)

---

### 双指针与滑动窗口

**1. 知识点：单向/对撞指针（基础线性扫描）**

*   **核心逻辑**：利用序列的单调性，通过两个指针同向或对撞移动，将 $O(N^2)$ 的枚举优化至 $O(N)$。关键在于指针不回退。
*   **对应题目**：[洛谷 P1147 连续自然数和](https://www.luogu.com.cn/problem/P1147)

**2. 知识点：排序 + 多指针维护（区间计数模型）**

*   **核心逻辑**：在处理 $A-B=C$ 等等值匹配时，先排序，再通过两个不回退的右指针 `r1, r2` 分别维护“第一个 $\ge$ 目标值”和“第一个 $>$ 目标值”的边界，实现 $O(N)$ 统计重复元素个数。
*   **对应题目**：[洛谷 P1102 A-B 数对](https://www.luogu.com.cn/problem/P1102)

**3. 知识点：变长滑动窗口（最窄覆盖模型）**

*   **核心逻辑**：固定右端点 `r` 主动向右扩展，当窗口满足条件（如包含所有类别）时，不断收缩左端点 `l` 以寻找局部最优解。
*   **对应题目**：[洛谷 P1638 逛画展](https://www.luogu.com.cn/problem/P1638)

**4. 知识点：排序映射与原序恢复（索引绑定技巧）**

*   **核心逻辑**：当双指针匹配需要基于排序数组，但输出需遵循原始次序时，通过 `Node` 对象或 `long` 位运算绑定 `(value, original_index)`。匹配成功后利用 `boolean[]` 标记原始下标。
*   **对应题目**：[洛谷 P1571 眼红的 FXXZ](https://www.luogu.com.cn/problem/P1571)

**5. 知识点：双指针预处理 + 后缀最大值（区间组合优化）**

*   **核心逻辑**：利用双指针 $O(N)$ 预处理出每个点出发的最长合规区间，结合后缀最大值数组 `sufMax[i]`，在 $O(1)$ 时间内找到与当前区间不重叠的最优第二个区间。
*   **对应题目**：[洛谷 P3143 [USACO16OPEN] Diamond Collector S](https://www.luogu.com.cn/problem/P3143)

**6. 知识点：数据合流与大规模滑动窗口（综合性能优化）**

*   **核心逻辑**：针对多列表分散数据，先“合流”并按坐标排序。在 $N=10^6$ 级别下，利用字节流 Fast I/O 和基本类型数组避开 Java 对象开销与 GC 压力。
*   **对应题目**：[洛谷 P2564 [SCOI2009] 生日礼物](https://www.luogu.com.cn/problem/P2564)

---

### 单调栈（Monotonic Stack）

**1. 知识点：单调栈基础（寻找单侧第一个大/小）**

*   **核心逻辑**：维护一个栈内元素单调的逻辑，通过“破坏单调性即弹出”的机制，在 $O(N)$ 时间内找到每个元素左侧或右侧第一个比其大/小的位置。**栈内通常存储下标**。
*   **对应题目**：[洛谷 P5788 【模板】单调栈](https://www.luogu.com.cn/problem/P5788)

**2. 知识点：双向单调栈（贡献接收模型）**

*   **核心逻辑**：通过正反两次单调栈扫描，确定每个元素左右两边最近的“更高点”。常用于模拟信号发射、雨水接纳或区间最值贡献判定。
*   **对应题目**：[洛谷 P1901 发射站](https://www.luogu.com.cn/problem/P1901)

**3. 知识点：单调栈与贡献度统计（视线覆盖模型）**

*   **核心逻辑**：在元素出栈时统计其对答案的贡献，或者利用类似 DP 的思想 `C[i] += C[index] + 1` 跳跃式累加被覆盖的区间长度。
*   **对应题目**：[洛谷 P2866 [USACO06NOV] Bad Hair Day S](https://www.luogu.com.cn/problem/P2866)

**4. 知识点：辐射范围最大化（直方图/最小值贡献模型）**

*   **核心逻辑**：与其枚举区间，不如**固定元素 $a_i$ 为区间最小值**。利用单调栈 $O(N)$ 寻找 $a_i$ 向左、向右能辐射到的最远边界（即左右第一个比它小的数）。结合前缀和，可在 $O(1)$ 内计算出以该元素为最小值的最大区间贡献。
*   **对应题目**：[洛谷 P2422 良好的感觉](https://www.luogu.com.cn/problem/P2422)、[洛谷 P2659 美丽的序列](https://www.luogu.com.cn/problem/P2659)

**5. 知识点：二维矩阵单调栈优化（压缩直方图/悬线法思想）**

*   **核心逻辑**：将二维矩阵问题“降维打击”。逐行处理，维护每一列向上连续空地的“高度” $H[j]$，将每一行看作一个直方图的底部，利用单调栈求解该行对应的最大矩形。最终在 $O(N \times M)$ 时间内解决二维最值问题。
*   **对应题目**：[洛谷 P4147 玉蟾宫](https://www.luogu.com.cn/problem/P4147)

---

### 单调队列（Monotonic Queue）

**1. 知识点：滑动窗口最值（单调队列模板）**

*   **核心逻辑**：使用双端队列维护下标。新元素入队前弹出队尾不如其“优秀”的元素；获取答案前弹出队头已过期的下标（滑出窗口）。队头始终为当前窗口最值。
*   **对应题目**：[洛谷 P1886 滑动窗口](https://www.luogu.com.cn/problem/P1886)、[洛谷 P2032 扫描](https://www.luogu.com.cn/problem/P2032)

**2. 知识点：单调队列优化前缀和（限制长度的最大子段和）**

*   **核心逻辑**：求长度 $\le M$ 的最大子段和即求 $Max(S[R] - S[L-1])$，其中 $R - (L-1) \le M$。固定 $R$ 时，利用单调队列维护窗口内 $S$ 的最小值。
*   **对应题目**：[洛谷 P1714 切前缀和](https://www.luogu.com.cn/problem/P1714)

*暂时跳过*

---

### 位运算

**1. Tricks**
   *   lowbit(n) = `n & -n`：获取二进制中最低位的 1（例如：$6(0110) \rightarrow 2(0010)$）。
   *   **去掉最低位的 1**：`n = n & (n - 1)`。例如 `n & (n-1)` 可以用来统计一个数中 1 的个数，循环几次就是几个 1。
   *   **判断奇偶**：`n & 1 == 1` (奇数) 或 `0` (偶数)。
   *   **判断 2 的幂**：`(n > 0) && ((n & (n - 1)) == 0)`。
   *   **Java 内置方法**：`Integer.bitCount(n)`（统计 1 的个数）、`Integer.numberOfTrailingZeros(n)`（尾部 0 的个数）。

**2. 知识点：二进制拆分（快速幂运算）**
*   **核心逻辑**：将指数 `b` 看作二进制数。利用 `b & 1` 判断当前位是否为 1，是则乘入底数；利用 `b >>= 1` 实现指数的“折半”处理。将时间复杂度从 $O(b)$ 优化至 $O(\log b)$。
*   **对应题目**：[洛谷 P1226 (快速幂/取模幂)](https://www.luogu.com.cn/problem/P1226)

**3. 知识点：状态压缩枚举（子集/组合问题）**

*   **核心逻辑**：利用整数的二进制位 `0` 或 `1` 表示集合中元素的“不选”或“选中”。遍历 `0` 到 `(1 << N) - 1` 即可不重不漏地枚举出所有组合情况。利用 `Integer.bitCount(mask) == k` 可快速过滤出大小为 $k$ 的合法组合。
*   **对应题目**：[洛谷 P1036 (选数)](https://www.luogu.com.cn/problem/P1036)

**4. 知识点：状压 DP（状态转移）**

*   **核心逻辑**：将整个局面编码为一个整数 `mask`。定义 `dp[mask][i]` 表示当前局面为 `mask` 且当前处于 `i` 时的最优状态。转移时使用位运算（如 `mask | (1 << j)`）将当前状态推向下一状态，外层循环必须按 `mask` 从小到大遍历以保证“无后效性”。
*   **对应题目**：[洛谷 P1433 (吃奶酪)](https://www.luogu.com.cn/problem/P1433)、[洛谷 P2704 (炮兵阵地 - 进阶)](https://www.luogu.com.cn/problem/P2704)

---

### 图论

**1. 知识点：DFS BFS**

- **对应题目**：[P5318 【深基18.例3】查找文献](https://www.luogu.com.cn/problem/P5318)

  ```java
  // 为什么以下两种写法是错误的？？？
      private static void dfs(List<Integer>[] adj, boolean[] visited, int n, int start, PrintWriter out) {
          Deque<Integer> stack = new ArrayDeque<>();
          stack.push(start);
          visited[start] = true;
          while(!stack.isEmpty()){
              int u = stack.pop();
              out.print(u);
              out.print(" ");
  
              List<Integer> vs = adj[u];
              int len = vs.size();
              for(int i = len - 1;i >= 0;i--){
                  if(!visited[vs.get(i)]){
                      stack.push(vs.get(i));
                      visited[vs.get(i)] = true;
                  }
              }
          }
      }
  
      private static void dfs(List<Integer>[] adj, boolean[] visited, int n, int start, PrintWriter out) {
          Deque<Integer> stack = new ArrayDeque<>();
          stack.push(start);
          visited[start] = true;
          while(!stack.isEmpty()){
              int u = stack.pop();
              out.print(u);
              out.print(" ");
              List<Integer> vs = adj[u];
              int len = vs.size();
              for(int i = len - 1;i >= 0;i--){
                  if(!visited[vs.get(i)]){
                      stack.push(vs.get(i));
                      visited[vs.get(i)] = true;
                  }
              }
          }
      }
  
  // 正确答案
  private static void dfs(List<Integer>[] adj, boolean[] visited, int n, int start, PrintWriter out) {
          Deque<Integer> stack = new ArrayDeque<>();
          stack.push(start);
          // visited[start] = true;
          while(!stack.isEmpty()){
              int u = stack.pop();
              if(!visited[u]){
                  out.print(u);
                  out.print(" ");
                  visited[u] = true;
              }
              List<Integer> vs = adj[u];
              int len = vs.size();
              for(int i = len - 1;i >= 0;i--){
                  if(!visited[vs.get(i)]){
                      stack.push(vs.get(i));
                      // visited[vs.get(i)] = true;
                  }
              }
          }
      }
  // 能区分的样例
  6 6
  1 2
  1 3
  2 4
  2 5
  4 6
  4 5
  ```

**2. 知识点：反向建图（图论的“逆向逻辑”）**

*   **核心逻辑**：将图中所有的边 $u \to v$ 修改为 $v \to u$。
    *   **为什么用它**：当原图中的“可达性”计算是**从前驱到后继**（一对多），而我们需要计算**“哪些点能到达某个点”**（多对一）或**“从所有点出发汇聚信息”**时，反向建图能将复杂的全局搜索简化为单次局部搜索。
    *   **关键点**：反向建图不改变图的强连通分量 (SCC) 结构，但在单源路径问题和依赖传递问题中，它是打破 $O(N^2)$ 超时的唯一路径。
*   **四大高频场景**：
    1.  **多源汇聚（如 P3916）**：求“每个点能到达的最大编号”。原图是“从 $i$ 出发”，反向后变成“所有能走到 $i$ 的点”，配合从大到小的遍历，即可一次性锁定最大值。
    2.  **求所有点到某个点的最短路**：Dijkstra 只能求“点 $S$ 到所有点”。若题目要求“所有点到点 $S$ 的距离”，直接在反向图上从 $S$ 跑一次 Dijkstra，结果与原图等价。
    3.  **强连通分量 (SCC) 的 Kosaraju 算法**：该算法的核心步骤就是通过“反向建图 + 二次 DFS”来划分强连通分量。
    4.  **博弈论或依赖消除**：当一个状态 $A$ 依赖于后继状态 $B$ 和 $C$ 时，如果在原图上难以按顺序计算，反向建图可以将依赖链反转，变为从已知状态向未知状态的“推导”。

*   **对应题目**：
    *   [洛谷 P3916 (图的遍历)](https://www.luogu.com.cn/problem/P3916) **这题不赖**
    *   [洛谷 P1629 (邮递员送信 - 最短路应用)](https://www.luogu.com.cn/problem/P1629)（此题非常经典，要求所有点到某点的往返，利用反向建图可以极大地减少计算量）

**3. 知识点：Dijkstra 算法**

*   **核心逻辑**：基于贪心策略，每次选取当前距离源点最近的节点进行松弛。使用 `PriorityQueue` 优化后，复杂度为 $O(M \log N)$。在 Java 实现中，必须使用 `long[]` 存储距离以防止 `Integer.MAX_VALUE` 相加导致的溢出。**优化关键**：在 `while` 循环中加入 `if (dist[u] < d) continue;` 进行惰性删除，避免无效计算。
*   **对应题目**：[洛谷 P4779 (单源最短路径-标准版)](https://www.luogu.com.cn/problem/P4779)

**4. 知识点：0-1 BFS (双端队列最短路)**

*   **核心逻辑**：当边权仅为 0 或 1 时，无需使用 `PriorityQueue`。利用 `ArrayDeque` 实现双端操作：权为 0 的边入队头，权为 1 的边入队尾。复杂度优化为 **$O(V + E)$**，是解决网格图、矩阵搜索类问题的“降维打击”算法。
*   **对应题目**：[洛谷 P4667 (Switch the Lamp)](https://www.luogu.com.cn/problem/P4667)

**5. 知识点：Floyd-Warshall 算法 (任意两点间最短路)**

*   **核心逻辑**：基于动态规划，`dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`。外层循环必须是“中转点 `k`”，内存循环为 `i` 和 `j`。
*   **适用场景**：$N \le 500$ 的稠密图，或题目要求查询任意两点间距离。
*   **CSP 陷阱**：时间复杂度 $O(N^3)$。禁止在 $N \ge 1000$ 的题目中使用，且注意 `dist[i][k] + dist[k][j]` 加法操作可能导致的溢出。
*   **对应题目**：[洛谷 P2910 (旅行路线)](https://www.luogu.com.cn/problem/P2910)

**6. 知识点：分层图最短路**

*   **核心逻辑**：当路径搜索带有“机会次数 $K$”或“状态切换”限制时，构建 $N \times (K+1)$ 个节点的新图。每一层代表一次状态改变，在层间通过 0 权边实现跳转。本质是状态空间的图构建。
*   **对应题目**：[洛谷 P4568 (飞行路线)](https://www.luogu.com.cn/problem/P4568)

**7. 知识点：Bellman-Ford 算法**

*   **核心逻辑**：基于动态规划（状态转移）思想，核心动作是对所有边进行“松弛”。算法的底层原理在于其**逐层递推**的过程：外层循环的每一轮 $K$（$1 \le K \le N-1$），其真实的物理含义是——**计算出从起点出发，最多经过 $K$ 条边到达各个顶点的最短路径**。因为在一个包含 $N$ 个顶点的图中，任意两点间的最短简单路径（不兜圈子）最多只包含 $N-1$ 条边，所以强制执行 $N-1$ 轮遍历，就能像水波一样把最短路径的信息推导至极限。这种“按边的跳数全局复盘”的机制，彻底摒弃了 Dijkstra 的局部贪心，因此能完美处理**负权边**。进一步地，如果进入第 $N$ 轮遍历（即允许走 $N$ 条边）时依然能发生松弛，根据鸽巢原理，走 $N$ 条边必然重复经过了某个顶点，且总权重还在缩小，这必定意味着图中存在**负权环**（无限倒贴钱的死循环）。时间复杂度为 $O(N \times M)$。**优化关键**：在 $N-1$ 轮循环中加入 `boolean updated;` 标记，若第 $K$ 轮没有任何距离被更新，说明“最多经过 $K$ 条边”的结果已经是最优解，全局最短路已提前确立，直接 `break` 结束循环，这是避免 TLE 的黄金剪枝策略。
*   **对应题目**：[洛谷 P3385 【模板】负环](https://www.google.com/url?sa=E&q=https%3A%2F%2Fwww.luogu.com.cn%2Fproblem%2FP3385)

**8. 知识点：并查集 (DSU)**
*   **核心逻辑**：通过路径压缩优化查询效率，利用树形结构管理不相交集合，实现连通性判断及动态连通块维护，是处理复杂图论合并问题的利器。
*   **对应题目**：
    *   [洛谷 P3367 【模板】并查集](https://www.luogu.com.cn/problem/P3367)
    *   [洛谷 P1551 亲戚](https://www.luogu.com.cn/problem/P1551)
    *   [洛谷 P1197 [JSOI2008] 星球大战](https://www.luogu.com.cn/problem/P1197)

---

### 提醒点

- 在涉及同余的题目中，请养成使用 (sum % 7 + 7) % 7 的习惯，因为java中对负数取余会产生负余数
- `S[i] / mid - (S[i] % mid == 0 && S[i] != 0 ? 1 : 0);` 和 `(S[i] - 1) / mid;` 是一样的
- 处理 10<sup>5</sup> 的数据的时候，最好不要使用 HashMap，内存可能会炸
- **内存优化技巧（long 压缩）**：若需存储两个 int（如坐标和类型、值和索引）并进行排序，可使用 long 数组替代对象数组。通过 (long)v << 32 | (id & 0xFFFFFFFFL) 压缩，Arrays.sort(long[]) 比对象排序更快且内存占用仅为 1/3 到 1/4。
- 数组容量心算预估：128MB 限制下 int[] 的安全上限约 1.5 * 10<sup>8</sup> ~ 2 * 10
- <sup>8</sup>（long[] 减半，256MB 翻倍），请时刻预留 30MB 左右的 JVM 基础开销。
- 警惕 Java 内存刺客（防 MLE）：① 杜绝包装类（Integer[] 占用是 int[] 的 5 倍以上）；② 二维数组对象头开销大，行数极多且内存紧张时，务必用一维数组模拟二维（arr[i * cols + j]）。
- 清空栈的代价：在多组数据或多次扫描时，使用 `top = -1` 即可实现 $O(1)$ 清空手写栈，严禁使用 $O(N)$ 的 `Arrays.fill(arr, 0)`，因为反正把 top 改成了 -1 之后也会覆盖。

- 记得 ans 的初始化可能会导致某些边界出错，比如下面的代码有什么问题？
  ```java
      public static void main(String[] args){
          FastReader fr = new FastReader(System.in);
          PrintWriter out = new PrintWriter(System.out);
  
          int a = fr.nextInt(), b = fr.nextInt(), p = fr.nextInt(), bb = b;
          long ans = 1, pow = a;
          while(b > 0){
              if((b & 0x1) == 1){
                  ans *= pow;
                  ans %= p;
              }
              pow *= pow;
              pow %= p;
              b >>= 1;
          }
  
          out.printf("%d^%d mod %d=%d", a, bb, p, ans);
  
          out.flush();
          out.close();
      }
  ```
- 小数学

  - GCD
    ```java
    int gcd(int a, int b) {
        return b == 0 ? a : gcd(b, a % b);
    }
    ```

  - LCM
    ```java
    long lcm(int a, int b) {
        return (long)a / gcd(a, b) * b; // 先除后乘防止溢出
    }
    ```

  - 判定素数

    - 单个
      ```java
      bool isPrime(int n) {
          if (n < 2) return false;
          for (int i = 2; i * i <= n; i++) 
              if (n % i == 0) return false;
          return true;
      }
      ```

    - 线性筛（找到1 - N的所有素数，O(N)）
      ```java
      int primes[N], cnt = 0;
      bool st[N]; // st[i] 为 true 表示 i 是合数
      void get_primes(int n) {
          for (int i = 2; i <= n; i++) {
              if (!st[i]) primes[cnt++] = i;
              for (int j = 0; primes[j] * i <= n; j++) {
                  st[primes[j] * i] = true;
                  if (i % primes[j] == 0) break; // 保证每个合数只被最小质因子筛掉
              }
          }
      }
      ```





