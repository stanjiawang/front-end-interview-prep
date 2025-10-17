# Java Coding Problems — With Explanations, Complexity & Pitfalls

> Practice these to build fluency. Each problem includes **approach**, **code**, **time/space complexity**, and **pitfalls**.

---

## 1) Reverse Linked List (Iterative & Recursive)
**Approach:** Rewire pointers iteratively; or recurse to end then reverse links.  
**Complexity:** O(n) time, O(1) space (iterative).  
**Pitfalls:** Losing next pointer; stack overflow on deep recursion.

```java
class ListNode { int val; ListNode next; ListNode(int v){val=v;} }
class ReverseList {
  ListNode iterative(ListNode head){
    ListNode prev=null, cur=head;
    while(cur!=null){ ListNode nxt=cur.next; cur.next=prev; prev=cur; cur=nxt; }
    return prev;
  }
  ListNode recursive(ListNode head){
    if(head==null || head.next==null) return head;
    ListNode p = recursive(head.next);
    head.next.next = head; head.next = null; return p;
  }
}
```

---

## 2) Two Sum
**Approach:** HashMap of value→index; for each number, look for complement.  
**Complexity:** O(n) time, O(n) space.  
**Pitfalls:** Duplicates; returning indices not values.

```java
int[] twoSum(int[] a, int target){
  var map = new java.util.HashMap<Integer,Integer>();
  for(int i=0;i<a.length;i++){
    int need = target - a[i];
    if(map.containsKey(need)) return new int[]{map.get(need), i};
    map.put(a[i], i);
  }
  return new int[]{-1,-1};
}
```

---

## 3) Valid Parentheses
**Approach:** Stack on open, check pairs on close.  
**Complexity:** O(n), O(n).  
**Pitfalls:** Empty stack checks; extra opens left.

```java
boolean isValid(String s){
  var st=new java.util.ArrayDeque<Character>();
  for(char c: s.toCharArray()){
    switch(c){
      case '(': case '[': case '{': st.push(c); break;
      case ')': if(st.isEmpty()||st.pop()!='(') return false; break;
      case ']': if(st.isEmpty()||st.pop()!='[') return false; break;
      case '}': if(st.isEmpty()||st.pop()!='{') return false; break;
    }
  }
  return st.isEmpty();
}
```

---

## 4) Group Anagrams (Streams)
**Approach:** Normalize by sorted letters; group in map.  
**Complexity:** O(n * k log k), k = word length.  
**Pitfalls:** Locale/case; using `char[]` vs code points for non‑ASCII.

```java
java.util.List<java.util.List<String>> group(String[] strs){
  return java.util.Arrays.stream(strs)
   .collect(java.util.stream.Collectors.groupingBy(
      s -> s.chars().sorted()
             .collect(StringBuilder::new,(b,c)->b.append((char)c),StringBuilder::append)
             .toString()
   ))
   .values().stream().toList();
}
```

---

## 5) Merge Intervals
**Approach:** Sort by start; merge overlapping windows.  
**Complexity:** O(n log n).  
**Pitfalls:** Off‑by‑one on inclusivity; empty input.

```java
int[][] merge(int[][] in){
  java.util.Arrays.sort(in, java.util.Comparator.comparingInt(a->a[0]));
  java.util.List<int[]> res=new java.util.ArrayList<>();
  int[] cur=in[0];
  for(int i=1;i<in.length;i++){
    if(in[i][0] <= cur[1]) cur[1] = Math.max(cur[1], in[i][1]);
    else { res.add(cur); cur=in[i]; }
  }
  res.add(cur);
  return res.toArray(new int[0][]);
}
```

---

## 6) Top K Frequent Elements (Min‑Heap)
**Approach:** Count with map; maintain min‑heap of size k.  
**Complexity:** O(n log k).  
**Pitfalls:** k > unique elements; comparator correctness.

```java
int[] topK(int[] nums, int k){
  var freq=new java.util.HashMap<Integer,Integer>();
  for(int n:nums) freq.merge(n,1,Integer::sum);
  var pq=new java.util.PriorityQueue<int[]>((a,b)->a[1]-b[1]);
  for(var e:freq.entrySet()){
    pq.offer(new int[]{e.getKey(), e.getValue()});
    if(pq.size()>k) pq.poll();
  }
  int[] out=new int[pq.size()]; int i=0;
  while(!pq.isEmpty()) out[i++]=pq.poll()[0];
  return out;
}
```

---

## 7) LRU Cache (LinkedHashMap)
**Approach:** Access‑order LinkedHashMap overriding `removeEldestEntry`.  
**Complexity:** O(1) amortized.  
**Pitfalls:** Not thread‑safe; use a wrapper or CHM + deque for concurrency.

```java
class LRUCache<K,V> extends java.util.LinkedHashMap<K,V>{
  private final int cap;
  LRUCache(int cap){ super(16,0.75f,true); this.cap=cap; }
  @Override protected boolean removeEldestEntry(java.util.Map.Entry<K,V> e){ return size()>cap; }
}
```

---

## 8) Producer–Consumer (BlockingQueue)
**Approach:** Use `ArrayBlockingQueue` and two threads.  
**Complexity:** O(n).  
**Pitfalls:** Deadlock if capacity 0; forgetting to handle interrupts.

```java
class PC {
  private final java.util.concurrent.BlockingQueue<Integer> q = new java.util.concurrent.ArrayBlockingQueue<>(3);
  void start(){
    Thread p = new Thread(() -> { for(int i=0;i<5;i++) try{ q.put(i);}catch(Exception ignored){} });
    Thread c = new Thread(() -> { for(int i=0;i<5;i++) try{ System.out.println("c:"+q.take()); }catch(Exception ignored){} });
    p.start(); c.start();
    try{ p.join(); c.join(); }catch(Exception ignored){}
  }
}
```

---

## 9) Binary Search
**Approach:** Classic two‑pointer narrowing.  
**Complexity:** O(log n).  
**Pitfalls:** Overflow on `(l+r)/2` (use `l+(r-l)/2`).

```java
int binarySearch(int[] a, int target){
  int l=0,r=a.length-1;
  while(l<=r){
    int m=l+(r-l)/2;
    if(a[m]==target) return m;
    if(a[m]<target) l=m+1; else r=m-1;
  }
  return -1;
}
```

---

## 10) Palindrome Check
**Approach:** Two pointers inward.  
**Complexity:** O(n).  
**Pitfalls:** Case/space handling if required.

```java
boolean isPal(String s){
  int i=0,j=s.length()-1;
  while(i<j){
    if(s.charAt(i++) != s.charAt(j--)) return false;
  }
  return true;
}
```

---

## 11) Rotate Matrix 90°
**Approach:** Transpose + reverse rows.  
**Complexity:** O(n^2).  
**Pitfalls:** In‑place care; rectangular matrices out of scope.

```java
void rotate(int[][] m){
  int n=m.length;
  for(int i=0;i<n;i++) for(int j=i;j<n;j++){ int t=m[i][j]; m[i][j]=m[j][i]; m[j][i]=t; }
  for(int i=0;i<n;i++) for(int j=0;j<n/2;j++){ int t=m[i][j]; m[i][j]=m[i][n-1-j]; m[i][n-1-j]=t; }
}
```

---

## 12) BFS Level Order (Binary Tree)
**Approach:** Queue by levels.  
**Complexity:** O(n).  
**Pitfalls:** Null root.

```java
class TreeNode{ int val; TreeNode left,right; TreeNode(int v){val=v;} }
java.util.List<java.util.List<Integer>> levelOrder(TreeNode root){
  var res=new java.util.ArrayList<java.util.List<Integer>>();
  if(root==null) return res;
  var q=new java.util.ArrayDeque<TreeNode>(); q.add(root);
  while(!q.isEmpty()){
    int sz=q.size(); var lvl=new java.util.ArrayList<Integer>();
    for(int i=0;i<sz;i++){ var n=q.poll(); lvl.add(n.val); if(n.left!=null) q.add(n.left); if(n.right!=null) q.add(n.right); }
    res.add(lvl);
  }
  return res;
}
```

---

## 13) Dijkstra (Adjacency List)
**Approach:** Min‑heap relaxations.  
**Complexity:** O((V+E) log V).  
**Pitfalls:** 1‑based vs 0‑based indices; INF sentinel.

```java
int[] dijkstra(int n, java.util.List<int[]>[] g, int src){
  int INF=1_000_000_000; int[] dist=new int[n];
  java.util.Arrays.fill(dist, INF); dist[src]=0;
  var pq=new java.util.PriorityQueue<int[]>((a,b)->a[1]-b[1]);
  pq.offer(new int[]{src,0});
  while(!pq.isEmpty()){
    var cur=pq.poll(); int u=cur[0], d=cur[1];
    if(d!=dist[u]) continue;
    for(var e: g[u]){
      int v=e[0], w=e[1];
      if(dist[v]>d+w){ dist[v]=d+w; pq.offer(new int[]{v,dist[v]}); }
    }
  }
  return dist;
}
```

---

## 14) Quickselect (Kth Smallest)
**Approach:** Partition around pivot; recurse into side.  
**Complexity:** Avg O(n), worst O(n^2).  
**Pitfalls:** Bad pivot on sorted arrays; randomize or median‑of‑three.

```java
int quickselect(int[] a, int k){ // 0-indexed
  int l=0,r=a.length-1;
  while(l<=r){
    int x=a[(l+r)>>>1];
    int i=l, j=r;
    while(i<=j){
      while(a[i]<x) i++;
      while(a[j]>x) j--;
      if(i<=j){ int t=a[i]; a[i]=a[j]; a[j]=t; i++; j--; }
    }
    if(k<=j) r=j; else if(k>=i) l=i; else return a[k];
  }
  return -1;
}
```

---

## 15) Longest Substring Without Repeating Characters
**Approach:** Sliding window + last‑seen map.  
**Complexity:** O(n).  
**Pitfalls:** Unicode vs `char` edge cases.

```java
int lengthOfLongestSubstring(String s){
  var last = new java.util.HashMap<Character,Integer>();
  int l=0, best=0;
  for(int r=0;r<s.length();r++){
    char c=s.charAt(r);
    if(last.containsKey(c) && last.get(c)>=l) l=last.get(c)+1;
    last.put(c,r);
    best=Math.max(best, r-l+1);
  }
  return best;
}
```

---

## 16) Kth Largest Element in an Array
**Approach:** Min‑heap of size k or quickselect.  
**Complexity:** O(n log k) or avg O(n).  
**Pitfalls:** Duplicates; off‑by‑one in k indexing.

```java
int findKthLargest(int[] nums, int k){
  var pq=new java.util.PriorityQueue<Integer>();
  for(int n:nums){ pq.offer(n); if(pq.size()>k) pq.poll(); }
  return pq.peek();
}
```

---

## 17) Number of Islands (DFS/BFS on Grid)
**Approach:** Flood fill counting components.  
**Complexity:** O(mn).  
**Pitfalls:** Modifying input grid vs visited set.

```java
int numIslands(char[][] g){
  int m=g.length, n=g[0].length, cnt=0;
  boolean[][] vis=new boolean[m][n];
  int[][] dirs={{1,0},{-1,0},{0,1},{0,-1}};
  java.util.function.BiConsumer<Integer,Integer> dfs = new java.util.function.BiConsumer<>(){
    public void accept(Integer i,Integer j){
      vis[i][j]=true;
      for(var d:dirs){
        int x=i+d[0], y=j+d[1];
        if(0<=x && x<m && 0<=y && y<n && g[x][y]=='1' && !vis[x][y]) accept(x,y);
      }
    }
  };
  for(int i=0;i<m;i++) for(int j=0;j<n;j++) if(g[i][j]=='1' && !vis[i][j]){ cnt++; dfs.accept(i,j); }
  return cnt;
}
```

---

## 18) Implement Blocking Bounded Queue (with wait/notify)
**Approach:** Use intrinsic lock; conditions via wait/notifyAll.  
**Complexity:** O(1) per op.  
**Pitfalls:** Spurious wakeups → use `while` loops; notify vs notifyAll.

```java
class BoundedQueue<T>{
  private final java.util.LinkedList<T> q=new java.util.LinkedList<>();
  private final int cap;
  BoundedQueue(int cap){ this.cap=cap; }
  public synchronized void put(T x) throws InterruptedException {
    while(q.size()==cap) wait();
    q.addLast(x);
    notifyAll();
  }
  public synchronized T take() throws InterruptedException {
    while(q.isEmpty()) wait();
    T v=q.removeFirst();
    notifyAll();
    return v;
  }
}
```

---

## 19) Serialize & Deserialize Binary Tree (Preorder with nulls)
**Approach:** Preorder traversal with sentinel; rebuild recursively.  
**Complexity:** O(n).  
**Pitfalls:** Split/join performance; use StringBuilder.

```java
class Codec {
  public String serialize(TreeNode root){
    var sb=new StringBuilder();
    ser(root,sb);
    return sb.toString();
  }
  void ser(TreeNode n,StringBuilder sb){
    if(n==null){ sb.append("#,"); return; }
    sb.append(n.val).append(',');
    ser(n.left,sb); ser(n.right,sb);
  }
  int idx;
  public TreeNode deserialize(String data){
    String[] a=data.split(",");
    idx=0; return de(a);
  }
  TreeNode de(String[] a){
    if("#".equals(a[idx])){ idx++; return null; }
    TreeNode n=new TreeNode(Integer.parseInt(a[idx++]));
    n.left=de(a); n.right=de(a);
    return n;
  }
}
```

---

## 20) Median of Two Sorted Arrays (Binary Partition)
**Approach:** Binary search on smaller array for correct partition.  
**Complexity:** O(log(min(m,n))).  
**Pitfalls:** Off‑by‑one; handling even/odd total lengths.

```java
double findMedianSortedArrays(int[] A, int[] B){
  if(A.length>B.length) return findMedianSortedArrays(B,A);
  int m=A.length, n=B.length, lo=0, hi=m;
  while(lo<=hi){
    int i=(lo+hi)/2, j=(m+n+1)/2 - i;
    int L1=(i==0?Integer.MIN_VALUE:A[i-1]), R1=(i==m?Integer.MAX_VALUE:A[i]);
    int L2=(j==0?Integer.MIN_VALUE:B[j-1]), R2=(j==n?Integer.MAX_VALUE:B[j]);
    if(L1<=R2 && L2<=R1){
      if(((m+n)&1)==0) return (Math.max(L1,L2)+Math.min(R1,R2))/2.0;
      else return Math.max(L1,L2);
    } else if(L1>R2) hi=i-1; else lo=i+1;
  }
  return 0.0;
}
```

---

**Tip:** Re‑implement these from scratch once without looking to build muscle memory.
