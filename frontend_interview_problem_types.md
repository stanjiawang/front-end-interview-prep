# Frontend Interview Problem Types – 中英双语优化版
A bilingual, GitHub-ready study guide for 25 essential frontend interview problem types.  
前端算法题型中英对照精华总结，共 25 类常考问题。

---
## 📘 Table of Contents
1. Frequency Counting and Deduplication 频率统计与去重
2. Two-Sum / Three-Sum / Four-Sum Families 求和问题
3. Sliding Window 滑动窗口
4. Stack and Queue 栈与队列
5. Monotonic Stack / Deque 单调栈与单调队列
6. Sorting and Selection 排序与选择
7. Prefix Sum and Difference Array 前缀和与差分数组
8. Interval Scheduling and Meeting Rooms 区间调度与会议室问题
9. Binary Search Patterns 二分查找模式
10. Greedy Patterns 贪心算法
11. Heap and Priority Queue 堆与优先队列
12. Graph Traversal 图的遍历
13. Tree Traversal 树的遍历
14. Union-Find 并查集
15. Trie 字典树
16. Topological Sorting 拓扑排序
17. Dynamic Programming 动态规划
18. Divide and Conquer 分治算法
19. Segment Tree 线段树
20. String Matching 字符串匹配
21. Palindromic Problems 回文问题
22. Caching and LRU/LFU 缓存与淘汰策略
23. Bloom Filter 布隆过滤器
24. Backtracking 回溯算法
25. Combination and Permutation 组合与排列

---

## 1. Frequency Counting and Deduplication  
**When it appears:** Remove duplicates, count occurrences, first unique, majority element.  
**出现场景：** 去重、统计出现次数、找出唯一或多数元素。  
**Data structures:** HashMap / Set  
**Algorithm:** Traverse input, record frequency, query result by rule.  
**Complexity:** O(n) time, O(k) space.  
**Frontend example:** Track duplicate usernames or tags in form validation.  

---

## 2. Two-Sum / Three-Sum / Four-Sum Families  
**When it appears:** Find pairs or tuples summing to target.  
**Data structures:** HashMap (Two-Sum), sorting + two pointers (Three/Four).  
**Complexity:** O(n) / O(n²) / O(n³).  
**Pitfalls:** Duplicate combinations.  
**Frontend example:** Price combinations meeting a given total budget.  

---

## 3. Sliding Window  
**When it appears:** Longest substring without repeat, minimum window substring.  
**Algorithm:** Expand right pointer; shrink left when constraint breaks; track max/min window.  
**Complexity:** O(n) time, O(k) space.  
**Frontend example:** User activity tracking within time window.  

---

## 4. Stack and Queue  
**When it appears:** Valid parentheses, evaluate expression, BFS level traversal.  
**Stack:** LIFO, simulate recursion, matching brackets.  
**Queue:** FIFO, used in BFS, message processing.  
**Complexity:** O(1) push/pop; O(n) traversal.  
**Frontend example:** Task queue handling in event loop simulation.  

---

## 5. Monotonic Stack / Deque  
**When it appears:** Next greater element, sliding window maximum.  
**Algorithm:** Maintain monotonic decreasing/increasing stack; remove invalid elements.  
**Complexity:** O(n) total since each element enters/exits once.  
**Frontend example:** Real-time charting – maintain recent max values.  

---

## 6. Sorting and Selection  
**When it appears:** Sort data, find Kth largest/smallest.  
**Algorithms:** QuickSort, MergeSort, QuickSelect.  
**Complexity:** O(n log n) typical, O(n²) worst for QuickSort.  
**Frontend example:** Client-side data sorting for tables.  

---

## 7. Prefix Sum and Difference Array  
**When it appears:** Range sum queries, subarray sums.  
**Algorithm:** Compute prefix array; for range queries use diff = prefix[r] - prefix[l-1].  
**Complexity:** O(n) build, O(1) query.  
**Frontend example:** Cumulative metrics in analytics dashboards.  

---

## 8. Interval Scheduling and Meeting Rooms  
**When it appears:** Meeting rooms, merge intervals, maximum non-overlapping intervals.  
**Algorithm:** Sort by start or end time; greedy selection of earliest finish.  
**Complexity:** O(n log n).  
**Frontend example:** Scheduling UI components, Gantt chart timeline management.  

---

## 9. Binary Search Patterns  
**When it appears:** Search in sorted array, first/last position, min/max threshold.  
**Algorithm:** Repeated halving until condition satisfied.  
**Complexity:** O(log n).  
**Frontend example:** Search feature optimization (pagination indices).  

---

## 10. Greedy Patterns  
**When it appears:** Coin change, interval cover, task scheduling.  
**Algorithm:** Always choose local optimal candidate; verify global validity.  
**Complexity:** O(n log n) due to sorting.  
**Frontend example:** Optimize asset loading order by priority.  

---

## 11. Heap and Priority Queue  
**When it appears:** Kth largest, top K elements, merge K sorted lists.  
**Algorithm:** Maintain heap property via push/pop; smallest/largest at top.  
**Complexity:** O(log n) per operation.  
**Frontend example:** Manage animation frame priorities or task throttling.  

---

## 12. Graph Traversal  
**When it appears:** Find connectivity, shortest path, cycles.  
**Algorithms:** BFS, DFS, Dijkstra.  
**Complexity:** O(V + E).  
**Frontend example:** Component dependency graphs or route planners.  

---

## 13. Tree Traversal  
**When it appears:** Binary tree problems (inorder, preorder, postorder).  
**Algorithm:** Recursive or iterative with stack.  
**Complexity:** O(n).  
**Frontend example:** Virtual DOM traversal.  

---

## 14. Union-Find  
**When it appears:** Detect connected components, redundancy in edges.  
**Algorithm:** Union by rank + path compression.  
**Complexity:** ~O(1) amortized per operation.  
**Frontend example:** Social graph connection checks.  

---

## 15. Trie  
**When it appears:** Prefix search, autocomplete, word dictionary.  
**Algorithm:** Tree where each node stores next char reference.  
**Complexity:** O(L) insert/search, L = word length.  
**Frontend example:** Autocomplete search box.  

---

## 16. Topological Sorting  
**When it appears:** Dependency resolution (tasks, modules).  
**Algorithm:** Kahn’s algorithm (BFS) or DFS postorder.  
**Complexity:** O(V + E).  
**Frontend example:** Module bundling dependency order.  

---

## 17. Dynamic Programming  
**When it appears:** Fibonacci, knapsack, edit distance, path sum.  
**Algorithm:** Define subproblem, recurrence, base case, store results.  
**Complexity:** O(n²) typical; varies by dimension.  
**Frontend example:** Optimize UI transitions or animation sequences.  

---

## 18. Divide and Conquer  
**When it appears:** Merge sort, binary search, matrix multiplication.  
**Algorithm:** Split → Solve recursively → Combine.  
**Complexity:** O(n log n).  
**Frontend example:** Layout computation (split views).  

---

## 19. Segment Tree  
**When it appears:** Range query and update (sum/min/max).  
**Algorithm:** Build tree with segment info, query recursively.  
**Complexity:** O(log n) per query/update.  
**Frontend example:** Canvas area updates or large data visualization.  

---

## 20. String Matching  
**When it appears:** Find substring pattern.  
**Algorithms:** KMP, Rabin-Karp, sliding window.  
**Complexity:** O(n + m).  
**Frontend example:** Search bar keyword highlighting.  

---

## 21. Palindromic Problems  
**When it appears:** Longest palindrome, valid palindrome.  
**Algorithms:** Expand around center, DP.  
**Complexity:** O(n²) time, O(1)/O(n) space.  
**Frontend example:** Text pattern validation.  

---

## 22. Caching and LRU/LFU  
**When it appears:** Cache eviction, performance optimization.  
**Algorithm:** HashMap + Doubly Linked List.  
**Complexity:** O(1) per operation.  
**Frontend example:** Browser cache simulation or in-memory caching.  

---

## 23. Bloom Filter  
**When it appears:** Space-efficient membership check with small error.  
**Algorithm:** Multiple hash functions, bit array.  
**Complexity:** O(k) hash ops, O(m) space.  
**Frontend example:** Detect duplicate requests quickly.  

---

## 24. Backtracking  
**When it appears:** Generate all valid configurations.  
**Algorithm:** Recursive DFS with pruning.  
**Complexity:** Exponential in depth.  
**Frontend example:** UI layout generation or puzzle solvers.  

---

## 25. Combination and Permutation  
**When it appears:** Choose K elements, generate permutations.  
**Algorithm:** DFS with state tracking and backtracking.  
**Complexity:** O(n!) worst.  
**Frontend example:** Generate all color/style combinations for UI testing.  

---

**End of File**
