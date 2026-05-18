# 📊 Graph Data Structures & Algorithms — Complete Tutorial & Roadmap

> **A production-grade, interview-ready deep dive into Graphs for DSA mastery.**  
> Covers theory, implementation, patterns, problems, and a structured learning roadmap.

---

## 📚 Table of Contents

1. [What is a Graph?](#1-what-is-a-graph)
2. [Graph Terminology](#2-graph-terminology)
3. [Types of Graphs](#3-types-of-graphs)
4. [Graph Representations](#4-graph-representations)
5. [Graph Traversals](#5-graph-traversals)
6. [Cycle Detection](#6-cycle-detection)
7. [Topological Sort](#7-topological-sort)
8. [Shortest Path Algorithms](#8-shortest-path-algorithms)
9. [Minimum Spanning Tree (MST)](#9-minimum-spanning-tree-mst)
10. [Disjoint Set Union (DSU)](#10-disjoint-set-union-dsu)
11. [Advanced Graph Algorithms](#11-advanced-graph-algorithms)
12. [Graph Patterns for Interviews](#12-graph-patterns-for-interviews)
13. [Problem Bank with Solutions](#13-problem-bank-with-solutions)
14. [Complexity Cheat Sheet](#14-complexity-cheat-sheet)
15. [Learning Roadmap](#15-learning-roadmap)

---

## 1. What is a Graph?

A **Graph** is a non-linear data structure consisting of **nodes (vertices)** and **edges** that connect them.

```
    (0)-----(1)
     |  \    |
     |   \   |
    (3)  (2)-(4)
```

> **Real-world examples:**  
> Social networks (users = nodes, friendships = edges),  
> Maps (cities = nodes, roads = edges),  
> Internet routing, dependency trees, circuit design.

### Mathematical Definition

```
G = (V, E)

where:
  V = set of vertices  → {0, 1, 2, 3, 4}
  E = set of edges     → {(0,1), (0,2), (0,3), (1,4), (2,4)}
```

---

## 2. Graph Terminology

| Term | Definition | Example |
|---|---|---|
| **Vertex / Node** | A point in the graph | City in a map |
| **Edge** | Connection between two vertices | Road between cities |
| **Degree** | Number of edges connected to a vertex | Node 0 has degree 3 |
| **In-degree** | Edges coming INTO a vertex (directed) | Vertex with 2 incoming arrows |
| **Out-degree** | Edges going OUT from a vertex (directed) | Vertex with 3 outgoing arrows |
| **Path** | Sequence of vertices connected by edges | 0 → 1 → 4 → 2 |
| **Cycle** | Path that starts and ends at the same vertex | 0 → 1 → 2 → 0 |
| **Connected** | Every vertex reachable from every other (undirected) | One piece graph |
| **Strongly Connected** | Every vertex reachable from every other (directed) | Two-way reachable |
| **Weighted Edge** | Edge with an associated cost/weight | Road with distance |
| **Self-loop** | Edge from vertex to itself | Node → itself |
| **Sparse Graph** | Few edges: E ≈ V | Trees |
| **Dense Graph** | Many edges: E ≈ V² | Complete graphs |

---

## 3. Types of Graphs

```
┌─────────────────────────────────────────────────────────────────┐
│                         GRAPH TYPES                             │
├─────────────────┬───────────────────┬───────────────────────────┤
│  Undirected     │    Directed (DAG) │    Weighted               │
│                 │                   │                           │
│  A ─── B        │  A ──► B          │  A ──5── B                │
│  |     |        │  |     ▼          │  |        |               │
│  C ─── D        │  C ──► D          │  3        2               │
│                 │                   │  C ──1── D                │
├─────────────────┴───────────────────┴───────────────────────────┤
│  Tree (connected,     │  Bipartite             │  Complete      │
│  acyclic undirected)  │  (2-colorable)         │  (all pairs)   │
│                       │                        │                │
│       A               │  {A,B} --- {C,D}       │  A ─── B       │
│      / \              │  A─C, A─D              │  |\ /|         │
│     B   C             │  B─C, B─D              │  | X |         │
│    / \                │  (no edge within set)  │  |/ \|         │
│   D   E               │                        │  C ─── D       │
└───────────────────────┴────────────────────────┴────────────────┘
```

### 3.1 Undirected Graph
- Edges have **no direction**
- `(u, v)` is the same as `(v, u)`
- Example: Facebook friendships

### 3.2 Directed Graph (Digraph)
- Edges have a **direction**: `u → v`
- Example: Twitter follows, web links

### 3.3 Weighted Graph
- Each edge has a **cost/weight**
- Example: Road distances, network bandwidth

### 3.4 DAG (Directed Acyclic Graph)
- Directed + **no cycles**
- Example: Task scheduling, build systems, git commits

### 3.5 Bipartite Graph
- Vertices divided into **two disjoint sets**
- Edges only go **between** sets, never within
- Example: Job-applicant matching

### 3.6 Tree
- Connected, undirected, **acyclic** graph
- N vertices, N-1 edges

---

## 4. Graph Representations

### 4.1 Adjacency Matrix

```
Graph:  0─1, 0─2, 1─2, 2─3

     0  1  2  3
  0 [0, 1, 1, 0]
  1 [1, 0, 1, 0]
  2 [1, 1, 0, 1]
  3 [0, 0, 1, 0]
```

```python
# Adjacency Matrix Implementation
class GraphMatrix:
    def __init__(self, V):
        self.V = V
        self.matrix = [[0] * V for _ in range(V)]

    def add_edge(self, u, v, weight=1):
        self.matrix[u][v] = weight
        self.matrix[v][u] = weight  # Remove for directed graph

    def has_edge(self, u, v):
        return self.matrix[u][v] != 0

    def neighbors(self, u):
        return [v for v in range(self.V) if self.matrix[u][v] != 0]

# Usage
g = GraphMatrix(4)
g.add_edge(0, 1)
g.add_edge(0, 2)
g.add_edge(1, 2)
g.add_edge(2, 3)
```

**When to use:** Dense graphs (E ≈ V²), O(1) edge lookup needed

| Operation | Time | Space |
|---|---|---|
| Add Edge | O(1) | O(V²) |
| Remove Edge | O(1) | |
| Check Edge | O(1) | |
| Find Neighbors | O(V) | |

---

### 4.2 Adjacency List ⭐ (Most Common)

```
Graph:  0─1, 0─2, 1─2, 2─3

0: [1, 2]
1: [0, 2]
2: [0, 1, 3]
3: [2]
```

```python
from collections import defaultdict

class Graph:
    def __init__(self):
        self.adj = defaultdict(list)

    def add_edge(self, u, v, weight=None):
        if weight:
            self.adj[u].append((v, weight))
            self.adj[v].append((u, weight))  # Remove for directed
        else:
            self.adj[u].append(v)
            self.adj[v].append(u)

    def add_directed_edge(self, u, v):
        self.adj[u].append(v)

    def neighbors(self, u):
        return self.adj[u]

    def display(self):
        for node, neighbors in self.adj.items():
            print(f"{node} → {neighbors}")

# Usage
g = Graph()
g.add_edge(0, 1)
g.add_edge(0, 2)
g.add_edge(1, 2)
g.add_edge(2, 3)
g.display()
```

**When to use:** Sparse graphs, most real-world problems

| Operation | Time | Space |
|---|---|---|
| Add Edge | O(1) | O(V + E) |
| Remove Edge | O(degree) | |
| Check Edge | O(degree) | |
| Find Neighbors | O(degree) | |

---

### 4.3 Edge List

```python
edges = [(0,1), (0,2), (1,2), (2,3)]

# Weighted
edges = [(0,1,5), (0,2,3), (1,2,2), (2,3,7)]
```

**When to use:** Kruskal's MST, sorting edges by weight

---

### 4.4 Comparison Table

| Representation | Space | Add Edge | Check Edge | Iterate Neighbors | Best For |
|---|---|---|---|---|---|
| Adjacency Matrix | O(V²) | O(1) | O(1) | O(V) | Dense graphs |
| Adjacency List | O(V+E) | O(1) | O(deg) | O(deg) | Sparse graphs ✅ |
| Edge List | O(E) | O(1) | O(E) | O(E) | MST algorithms |

---

## 5. Graph Traversals

### 5.1 Breadth-First Search (BFS)

**Idea:** Explore level by level using a **Queue**

```
Graph:       BFS from node 0:
  0          Level 0: [0]
 /|\         Level 1: [1, 2, 3]
1 2 3        Level 2: [4, 5]
|   |
4   5
```

```python
from collections import deque

def bfs(graph, start):
    visited = set()
    queue = deque([start])
    visited.add(start)
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return order


# BFS with level tracking
def bfs_levels(graph, start):
    visited = set([start])
    queue = deque([(start, 0)])  # (node, level)
    result = {}

    while queue:
        node, level = queue.popleft()
        result[node] = level

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, level + 1))

    return result


# BFS Shortest Path (unweighted)
def bfs_shortest_path(graph, start, end):
    visited = set([start])
    queue = deque([(start, [start])])

    while queue:
        node, path = queue.popleft()
        if node == end:
            return path

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))

    return []  # No path found
```

**Time:** O(V + E) | **Space:** O(V)

**Use BFS when:**
- Finding **shortest path** in unweighted graph
- Level-order traversal
- Checking bipartiteness
- Finding connected components

---

### 5.2 Depth-First Search (DFS)

**Idea:** Go as deep as possible using **Recursion** or **Stack**

```
Graph:       DFS from node 0:
  0          Visit: 0 → 1 → 4 → (backtrack) → 2 → 5 → 3
 / \
1   2
|   |\ 
4   5 3
```

```python
# DFS - Recursive
def dfs_recursive(graph, node, visited=None):
    if visited is None:
        visited = set()

    visited.add(node)
    print(node, end=" ")  # Process node

    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)

    return visited


# DFS - Iterative (using explicit stack)
def dfs_iterative(graph, start):
    visited = set()
    stack = [start]
    order = []

    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            order.append(node)
            # Push neighbors (reversed for consistent order)
            for neighbor in reversed(graph[node]):
                if neighbor not in visited:
                    stack.append(neighbor)

    return order


# DFS with pre/post timestamps (useful for many algorithms)
def dfs_timestamps(graph, start):
    visited = set()
    timer = [0]
    pre = {}   # discovery time
    post = {}  # finish time

    def dfs(node):
        visited.add(node)
        timer[0] += 1
        pre[node] = timer[0]

        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)

        timer[0] += 1
        post[node] = timer[0]

    dfs(start)
    return pre, post
```

**Time:** O(V + E) | **Space:** O(V)

**Use DFS when:**
- Detecting cycles
- Topological sort
- Finding connected components
- Path existence problems
- Backtracking problems

---

### 5.3 BFS vs DFS — Decision Guide

```
┌─────────────────────────────────────────────┐
│           BFS vs DFS — When to Use          │
├────────────────────┬────────────────────────┤
│        BFS         │          DFS           │
├────────────────────┼────────────────────────┤
│ Shortest path      │ Cycle detection        │
│ (unweighted)       │ Topological sort       │
│                    │                        │
│ Level-order        │ Connected components   │
│ processing         │ (can use either)       │
│                    │                        │
│ Nearest node       │ Backtracking/maze      │
│ in graph           │ solving                │
│                    │                        │
│ Bipartite check    │ Finding all paths      │
│                    │                        │
│ Multi-source BFS   │ DFS tree properties    │
│ problems           │ (bridges, articulation)│
└────────────────────┴────────────────────────┘
```

---

## 6. Cycle Detection

### 6.1 Undirected Graph — Cycle Detection (DFS)

```python
def has_cycle_undirected(graph, V):
    visited = set()

    def dfs(node, parent):
        visited.add(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                if dfs(neighbor, node):
                    return True
            elif neighbor != parent:
                # Back edge found — cycle exists!
                return True
        return False

    for v in range(V):
        if v not in visited:
            if dfs(v, -1):
                return True
    return False
```

### 6.2 Undirected Graph — Cycle Detection (DSU)

```python
def has_cycle_dsu(edges, V):
    parent = list(range(V))

    def find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])  # Path compression
        return parent[x]

    def union(x, y):
        px, py = find(x), find(y)
        if px == py:
            return False  # Cycle detected
        parent[px] = py
        return True

    for u, v in edges:
        if not union(u, v):
            return True
    return False
```

### 6.3 Directed Graph — Cycle Detection (DFS + Color)

**Key insight:** Use 3 colors:
- `WHITE (0)`: Not visited
- `GRAY (1)`: Currently in DFS stack (in-progress)
- `BLACK (2)`: Fully processed

```python
def has_cycle_directed(graph, V):
    # 0 = white, 1 = gray, 2 = black
    color = [0] * V

    def dfs(node):
        color[node] = 1  # Mark gray (in progress)

        for neighbor in graph[node]:
            if color[neighbor] == 1:
                return True   # Back edge → cycle!
            if color[neighbor] == 0:
                if dfs(neighbor):
                    return True

        color[node] = 2  # Mark black (done)
        return False

    for v in range(V):
        if color[v] == 0:
            if dfs(v):
                return True
    return False
```

---

## 7. Topological Sort

> **Only applicable to DAGs (Directed Acyclic Graphs)**  
> Ordering such that for every edge u→v, u comes before v

### 7.1 DFS-based Topological Sort (Kahn's is next)

```python
def topo_sort_dfs(graph, V):
    visited = set()
    stack = []

    def dfs(node):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(node)  # Append AFTER all neighbors are processed

    for v in range(V):
        if v not in visited:
            dfs(v)

    return stack[::-1]  # Reverse for topological order
```

### 7.2 BFS-based Topological Sort (Kahn's Algorithm) ⭐

**Key insight:** Repeatedly remove nodes with in-degree 0

```python
from collections import deque

def topo_sort_kahn(graph, V):
    in_degree = [0] * V

    # Calculate in-degrees
    for u in range(V):
        for v in graph[u]:
            in_degree[v] += 1

    # Start with all nodes having in-degree 0
    queue = deque([v for v in range(V) if in_degree[v] == 0])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node)

        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    # If result has fewer nodes than V, there's a cycle
    if len(result) != V:
        return []  # Cycle detected — no valid topo order

    return result
```

### 7.3 Topo Sort — Real Example

```
Tasks: A→C, B→C, C→D, C→E
       (A and B must finish before C; C before D and E)

In-degrees: A=0, B=0, C=2, D=1, E=1
Step 1: Queue=[A,B] → process A → [B,C] (C's in-degree: 2→1)
Step 2: process B → [C]   (C's in-degree: 1→0)
Step 3: process C → [D,E]
Step 4: process D, E

Result: A → B → C → D → E  (valid topo order)
```

---

## 8. Shortest Path Algorithms

### 8.1 BFS — Unweighted Shortest Path

Already covered in Section 5.1. Time: O(V + E)

---

### 8.2 Dijkstra's Algorithm ⭐ (Non-negative weights)

**Idea:** Greedily pick the closest unvisited node using a **min-heap**

```
     (0)
    / | \
  4/  |2  \7
  /   |    \
(1)  (2)──3──(3)
 \   /
  6\ /1
   (4)

Dijkstra from 0:
dist = [0, ∞, ∞, ∞, ∞]
→ Visit 0: dist=[0, 4, 2, 7, ∞]
→ Visit 2: dist=[0, 4, 2, 5, 3]  (0→2→3=5, 0→2→4=3)
→ Visit 4: dist=[0, 4, 2, 5, 3]
→ Visit 1: dist=[0, 4, 2, 5, 3]
→ Visit 3: final
```

```python
import heapq

def dijkstra(graph, start, V):
    """
    graph: adjacency list with weights  {node: [(neighbor, weight), ...]}
    Returns: dist[] = shortest distance from start to all nodes
    """
    dist = [float('inf')] * V
    dist[start] = 0
    min_heap = [(0, start)]  # (distance, node)

    while min_heap:
        d, u = heapq.heappop(min_heap)

        # Skip if we already found a better path
        if d > dist[u]:
            continue

        for v, weight in graph[u]:
            new_dist = dist[u] + weight
            if new_dist < dist[v]:
                dist[v] = new_dist
                heapq.heappush(min_heap, (new_dist, v))

    return dist


# Dijkstra with path reconstruction
def dijkstra_with_path(graph, start, end, V):
    dist = [float('inf')] * V
    dist[start] = 0
    prev = [-1] * V
    min_heap = [(0, start)]

    while min_heap:
        d, u = heapq.heappop(min_heap)
        if d > dist[u]:
            continue
        if u == end:
            break

        for v, weight in graph[u]:
            if dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight
                prev[v] = u
                heapq.heappush(min_heap, (dist[v], v))

    # Reconstruct path
    path = []
    node = end
    while node != -1:
        path.append(node)
        node = prev[node]
    return dist[end], path[::-1]
```

**Time:** O((V + E) log V) | **Space:** O(V)  
**Constraint:** ❌ Does NOT work with **negative weights**

---

### 8.3 Bellman-Ford Algorithm (Handles negative weights)

**Idea:** Relax all edges V-1 times

```python
def bellman_ford(edges, V, start):
    """
    edges: list of (u, v, weight)
    Returns: dist[] or None if negative cycle exists
    """
    dist = [float('inf')] * V
    dist[start] = 0

    # Relax all edges V-1 times
    for _ in range(V - 1):
        for u, v, w in edges:
            if dist[u] != float('inf') and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    # Check for negative cycles (V-th relaxation)
    for u, v, w in edges:
        if dist[u] != float('inf') and dist[u] + w < dist[v]:
            return None  # Negative cycle detected

    return dist
```

**Time:** O(V × E) | **Space:** O(V)

---

### 8.4 Floyd-Warshall (All-pairs shortest path)

**Idea:** Dynamic programming — all pairs in one go

```python
def floyd_warshall(V, edges):
    """Returns shortest path between ALL pairs of vertices"""
    INF = float('inf')
    dist = [[INF] * V for _ in range(V)]

    for i in range(V):
        dist[i][i] = 0  # Distance to self is 0

    for u, v, w in edges:
        dist[u][v] = w
        dist[v][u] = w  # Remove for directed

    # Core DP: try every node k as intermediate
    for k in range(V):
        for i in range(V):
            for j in range(V):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]

    # Check for negative cycles
    for i in range(V):
        if dist[i][i] < 0:
            return None  # Negative cycle

    return dist
```

**Time:** O(V³) | **Space:** O(V²)

---

### 8.5 Shortest Path Algorithm Decision Guide

```
┌─────────────────────────────────────────────────────┐
│         Shortest Path — Algorithm Selection         │
├─────────────────────────────────────────────────────┤
│  Unweighted graph?  ──────────► BFS  O(V+E)        │
│                                                     │
│  Weighted, no negative weights?                     │
│    Single source  ────────────► Dijkstra O((V+E)lgV)│
│    All pairs      ────────────► Floyd-Warshall O(V³)│
│                                                     │
│  Negative weights (no neg cycle)?                   │
│    Single source  ────────────► Bellman-Ford O(VE)  │
│    All pairs      ────────────► Floyd-Warshall O(V³)│
│                                                     │
│  DAG?             ────────────► Topo sort + relax   │
│                                 O(V+E)              │
└─────────────────────────────────────────────────────┘
```

---

## 9. Minimum Spanning Tree (MST)

> An MST of a weighted undirected graph is a **spanning tree** (connects all vertices) with **minimum total edge weight**. Has exactly V-1 edges.

### 9.1 Kruskal's Algorithm

**Idea:** Sort edges by weight, add smallest edge that doesn't form a cycle (use DSU)

```python
def kruskal(V, edges):
    """
    edges: list of (weight, u, v)
    Returns: MST edges, total weight
    """
    parent = list(range(V))
    rank = [0] * V

    def find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])  # Path compression
        return parent[x]

    def union(x, y):
        px, py = find(x), find(y)
        if px == py:
            return False  # Would create cycle
        # Union by rank
        if rank[px] < rank[py]:
            parent[px] = py
        elif rank[px] > rank[py]:
            parent[py] = px
        else:
            parent[py] = px
            rank[px] += 1
        return True

    edges.sort()  # Sort by weight
    mst_edges = []
    total_weight = 0

    for weight, u, v in edges:
        if union(u, v):
            mst_edges.append((u, v, weight))
            total_weight += weight
            if len(mst_edges) == V - 1:
                break

    return mst_edges, total_weight
```

**Time:** O(E log E) | **Space:** O(V)

---

### 9.2 Prim's Algorithm

**Idea:** Grow MST from a start vertex, always pick the cheapest edge to an unvisited vertex

```python
import heapq

def prim(graph, V, start=0):
    """
    graph: adjacency list {node: [(neighbor, weight), ...]}
    Returns: MST total weight, MST edges
    """
    in_mst = [False] * V
    min_heap = [(0, start, -1)]  # (weight, node, parent)
    total_weight = 0
    mst_edges = []

    while min_heap:
        weight, u, parent = heapq.heappop(min_heap)

        if in_mst[u]:
            continue

        in_mst[u] = True
        total_weight += weight

        if parent != -1:
            mst_edges.append((parent, u, weight))

        for v, w in graph[u]:
            if not in_mst[v]:
                heapq.heappush(min_heap, (w, v, u))

    return total_weight, mst_edges
```

**Time:** O((V + E) log V) | **Space:** O(V)

---

### 9.3 Kruskal vs Prim

| Feature | Kruskal | Prim |
|---|---|---|
| Approach | Edge-based | Vertex-based |
| Data Structure | DSU + sorted edges | Min-heap |
| Best for | Sparse graphs | Dense graphs |
| Time | O(E log E) | O((V+E) log V) |
| Works on | Disconnected graphs | Connected graphs |

---

## 10. Disjoint Set Union (DSU)

> Also called **Union-Find**. Efficiently tracks connected components.

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.size = [1] * n
        self.components = n  # Number of connected components

    def find(self, x):
        """Find root with path compression"""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x, y):
        """Union by rank. Returns False if already same component."""
        px, py = self.find(x), self.find(y)
        if px == py:
            return False  # Already connected

        if self.rank[px] < self.rank[py]:
            px, py = py, px  # Ensure px has higher rank

        self.parent[py] = px
        self.size[px] += self.size[py]

        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1

        self.components -= 1
        return True

    def connected(self, x, y):
        return self.find(x) == self.find(y)

    def component_size(self, x):
        return self.size[self.find(x)]


# Usage Example: Number of Islands / Components
dsu = DSU(5)
dsu.union(0, 1)
dsu.union(1, 2)
dsu.union(3, 4)

print(dsu.components)           # 2
print(dsu.connected(0, 2))      # True
print(dsu.connected(0, 3))      # False
print(dsu.component_size(0))    # 3
```

**Operations:** O(α(n)) ≈ O(1) amortized (inverse Ackermann)

---

## 11. Advanced Graph Algorithms

### 11.1 Connected Components

```python
def count_components(graph, V):
    """Count connected components in undirected graph"""
    visited = set()
    count = 0

    def dfs(node):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)

    for v in range(V):
        if v not in visited:
            dfs(v)
            count += 1

    return count
```

---

### 11.2 Bipartite Check (2-Coloring)

```python
from collections import deque

def is_bipartite(graph, V):
    color = [-1] * V

    for start in range(V):
        if color[start] != -1:
            continue

        queue = deque([start])
        color[start] = 0

        while queue:
            node = queue.popleft()
            for neighbor in graph[node]:
                if color[neighbor] == -1:
                    color[neighbor] = 1 - color[node]  # Opposite color
                    queue.append(neighbor)
                elif color[neighbor] == color[node]:
                    return False  # Same color = not bipartite

    return True
```

---

### 11.3 Bridges and Articulation Points (Tarjan's Algorithm)

> **Bridge:** An edge whose removal disconnects the graph  
> **Articulation Point:** A vertex whose removal disconnects the graph

```python
def find_bridges(graph, V):
    visited = [False] * V
    disc = [0] * V      # Discovery times
    low = [0] * V       # Lowest reachable disc time
    bridges = []
    timer = [0]

    def dfs(u, parent):
        visited[u] = True
        disc[u] = low[u] = timer[0]
        timer[0] += 1

        for v in graph[u]:
            if not visited[v]:
                dfs(v, u)
                low[u] = min(low[u], low[v])

                # v cannot reach back to u or above → bridge!
                if low[v] > disc[u]:
                    bridges.append((u, v))
            elif v != parent:
                low[u] = min(low[u], disc[v])

    for v in range(V):
        if not visited[v]:
            dfs(v, -1)

    return bridges


def find_articulation_points(graph, V):
    visited = [False] * V
    disc = [0] * V
    low = [0] * V
    ap = set()
    timer = [0]

    def dfs(u, parent):
        children = 0
        visited[u] = True
        disc[u] = low[u] = timer[0]
        timer[0] += 1

        for v in graph[u]:
            if not visited[v]:
                children += 1
                dfs(v, u)
                low[u] = min(low[u], low[v])

                # Root with 2+ children is AP
                if parent == -1 and children > 1:
                    ap.add(u)
                # Non-root: if low[v] >= disc[u], it's an AP
                if parent != -1 and low[v] >= disc[u]:
                    ap.add(u)
            elif v != parent:
                low[u] = min(low[u], disc[v])

    for v in range(V):
        if not visited[v]:
            dfs(v, -1)

    return ap
```

---

### 11.4 Strongly Connected Components (SCC) — Kosaraju's Algorithm

```python
def kosaraju_scc(graph, V):
    """
    Step 1: DFS on original graph, push to stack by finish time
    Step 2: Transpose graph
    Step 3: DFS on transposed in reverse finish order
    """
    # Step 1: DFS and fill stack
    visited = [False] * V
    stack = []

    def dfs1(v):
        visited[v] = True
        for u in graph[v]:
            if not visited[u]:
                dfs1(u)
        stack.append(v)

    for v in range(V):
        if not visited[v]:
            dfs1(v)

    # Step 2: Transpose graph
    transposed = [[] for _ in range(V)]
    for v in range(V):
        for u in graph[v]:
            transposed[u].append(v)

    # Step 3: DFS on transposed in reverse finish order
    visited = [False] * V
    sccs = []

    def dfs2(v, component):
        visited[v] = True
        component.append(v)
        for u in transposed[v]:
            if not visited[u]:
                dfs2(u, component)

    while stack:
        v = stack.pop()
        if not visited[v]:
            component = []
            dfs2(v, component)
            sccs.append(component)

    return sccs
```

---

### 11.5 Flood Fill (Matrix as Graph)

```python
def flood_fill(grid, sr, sc, new_color):
    """
    Grid problems: each cell is a node, adjacent cells are edges
    """
    rows, cols = len(grid), len(grid[0])
    old_color = grid[sr][sc]

    if old_color == new_color:
        return grid

    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols:
            return
        if grid[r][c] != old_color:
            return

        grid[r][c] = new_color
        dfs(r+1, c)
        dfs(r-1, c)
        dfs(r, c+1)
        dfs(r, c-1)

    dfs(sr, sc)
    return grid


# BFS version (avoids recursion depth issues)
def flood_fill_bfs(grid, sr, sc, new_color):
    from collections import deque
    rows, cols = len(grid), len(grid[0])
    old_color = grid[sr][sc]

    if old_color == new_color:
        return grid

    queue = deque([(sr, sc)])
    grid[sr][sc] = new_color

    while queue:
        r, c = queue.popleft()
        for dr, dc in [(1,0),(-1,0),(0,1),(0,-1)]:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == old_color:
                grid[nr][nc] = new_color
                queue.append((nr, nc))

    return grid
```

---

### 11.6 Multi-Source BFS

```python
def multi_source_bfs(grid):
    """
    Example: Find distance from any 0 to nearest 1 in grid (01 Matrix)
    Start BFS from ALL sources simultaneously
    """
    from collections import deque

    rows, cols = len(grid), len(grid[0])
    dist = [[float('inf')] * cols for _ in range(rows)]
    queue = deque()

    # Add all sources to queue at distance 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 0:
                dist[r][c] = 0
                queue.append((r, c))

    while queue:
        r, c = queue.popleft()
        for dr, dc in [(1,0),(-1,0),(0,1),(0,-1)]:
            nr, nc = r+dr, c+dc
            if 0 <= nr < rows and 0 <= nc < cols:
                if dist[r][c] + 1 < dist[nr][nc]:
                    dist[nr][nc] = dist[r][c] + 1
                    queue.append((nr, nc))

    return dist
```

---

## 12. Graph Patterns for Interviews

### Pattern 1: Grid as Graph
```
Directions for 4-connected: [(0,1),(0,-1),(1,0),(-1,0)]
Directions for 8-connected: [(0,1),(0,-1),(1,0),(-1,0),(1,1),(1,-1),(-1,1),(-1,-1)]

Template:
  for r, c in current_cells:
      for dr, dc in directions:
          nr, nc = r+dr, c+dc
          if valid(nr, nc) and condition(grid[nr][nc]):
              process and enqueue
```

### Pattern 2: Multi-Source BFS
```
Use when: "Find minimum distance from ANY of multiple sources"
Example: Walls and gates, Nearest 1 to each cell, Rotten oranges

Template:
  Initialize dist[][] = INF
  Add ALL sources with dist=0 to queue
  Run standard BFS
```

### Pattern 3: Bidirectional BFS
```
Use when: Finding shortest path between two specific nodes in large graph
Speedup: O(b^(d/2)) instead of O(b^d)

Template:
  front_visited = {start}
  back_visited  = {end}
  front_queue, back_queue = deque([start]), deque([end])
  steps = 0
  while both queues non-empty:
      Expand smaller frontier
      If intersection found → return steps
      steps += 1
```

### Pattern 4: Dijkstra on 2D Grid
```
Use when: Grid with weighted cells and need minimum cost path
Example: Minimum effort path, cheapest path

Template:
  dist[r][c] = INF, dist[start] = 0
  heap = [(0, start_r, start_c)]
  while heap:
      cost, r, c = heappop(heap)
      if (r,c) == end: return cost
      for each neighbor:
          new_cost = compute(cost, neighbor)
          if new_cost < dist[nr][nc]:
              update and push
```

### Pattern 5: Topo Sort for Dependencies
```
Use when: Tasks with prerequisites, build order, course schedule

Template:
  Build in_degree[]
  Kahn's BFS from all 0-in-degree nodes
  If result length < total tasks → CYCLE (impossible)
```

### Pattern 6: Union-Find for Dynamic Connectivity
```
Use when: Incrementally add edges, count components, check connectivity

Template:
  dsu = DSU(n)
  for each new edge (u, v):
      if dsu.union(u, v):
          components -= 1
  answer queries with dsu.connected(u, v)
```

---

## 13. Problem Bank with Solutions

### 🟢 Easy Problems

| Problem | Pattern | Algorithm |
|---|---|---|
| Number of Islands | Grid DFS/BFS | DFS/BFS + visited |
| Flood Fill | Grid DFS | DFS |
| Find if Path Exists | BFS/DFS | BFS |
| Find Center of Star Graph | Math | Degree check |
| Clone Graph | BFS + HashMap | BFS |

### 🟡 Medium Problems

| Problem | Pattern | Algorithm |
|---|---|---|
| Course Schedule | Cycle in DAG | Topo Sort / DFS |
| Course Schedule II | Topo Order | Kahn's BFS |
| Number of Provinces | Connected Components | DFS/DSU |
| Rotting Oranges | Multi-source BFS | BFS |
| 01 Matrix | Multi-source BFS | BFS |
| Pacific Atlantic Water Flow | Reverse DFS from coasts | DFS |
| Surrounded Regions | Border DFS | DFS |
| Word Ladder | BFS shortest path | BFS |
| Cheapest Flights K Stops | Dijkstra variant | BFS + DP |
| Network Delay Time | Single source shortest | Dijkstra |
| Path with Min Effort | Dijkstra on grid | Dijkstra |
| Redundant Connection | Cycle detection | DSU |
| Graph Valid Tree | Connected + Acyclic | DSU |

### 🔴 Hard Problems

| Problem | Pattern | Algorithm |
|---|---|---|
| Word Ladder II | BFS + DFS backtrack | BFS + DFS |
| Alien Dictionary | Topo sort | Kahn's |
| Critical Connections | Bridges | Tarjan's |
| Swim in Rising Water | Binary search + BFS | Dijkstra |
| Bus Routes | BFS on hypergraph | Multi-level BFS |
| Minimum Cost to Connect Points | MST | Kruskal/Prim |
| Sequence Reconstruction | Unique topo sort | Topo Sort |

---

### Selected Problem Walkthrough: Course Schedule

```python
"""
LeetCode 207: Course Schedule
Given numCourses and prerequisites [[a,b]] meaning "must take b before a",
determine if it is possible to finish all courses.
→ Check if directed graph has a cycle
"""
from collections import defaultdict, deque

def canFinish(numCourses, prerequisites):
    graph = defaultdict(list)
    in_degree = [0] * numCourses

    for course, prereq in prerequisites:
        graph[prereq].append(course)
        in_degree[course] += 1

    # Kahn's algorithm
    queue = deque([c for c in range(numCourses) if in_degree[c] == 0])
    completed = 0

    while queue:
        course = queue.popleft()
        completed += 1

        for next_course in graph[course]:
            in_degree[next_course] -= 1
            if in_degree[next_course] == 0:
                queue.append(next_course)

    return completed == numCourses  # If cycle, some courses never reach 0 in-degree
```

---

### Selected Problem Walkthrough: Number of Islands

```python
"""
LeetCode 200: Number of Islands
Grid of '1' (land) and '0' (water). Count number of islands.
"""
def numIslands(grid):
    if not grid:
        return 0

    rows, cols = len(grid), len(grid[0])
    count = 0

    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != '1':
            return
        grid[r][c] = '#'  # Mark visited
        dfs(r+1, c); dfs(r-1, c)
        dfs(r, c+1); dfs(r, c-1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                dfs(r, c)
                count += 1

    return count
```

---

## 14. Complexity Cheat Sheet

```
┌────────────────────────────────────────────────────────────────────┐
│                    GRAPH ALGORITHM COMPLEXITIES                    │
├──────────────────────┬─────────────────┬──────────────────────────┤
│  Algorithm           │   Time          │   Space                  │
├──────────────────────┼─────────────────┼──────────────────────────┤
│  BFS                 │  O(V + E)       │  O(V)                    │
│  DFS                 │  O(V + E)       │  O(V)                    │
│  Topological Sort    │  O(V + E)       │  O(V)                    │
│  Dijkstra (heap)     │  O((V+E) log V) │  O(V)                    │
│  Bellman-Ford        │  O(V × E)       │  O(V)                    │
│  Floyd-Warshall      │  O(V³)          │  O(V²)                   │
│  Kruskal's MST       │  O(E log E)     │  O(V)                    │
│  Prim's MST          │  O((V+E) log V) │  O(V)                    │
│  DSU Find/Union      │  O(α(V)) ≈ O(1) │  O(V)                    │
│  Tarjan's (SCC)      │  O(V + E)       │  O(V)                    │
│  Kosaraju's (SCC)    │  O(V + E)       │  O(V + E)                │
│  Bridges/AP          │  O(V + E)       │  O(V)                    │
├──────────────────────┼─────────────────┼──────────────────────────┤
│  Adj Matrix          │  —              │  O(V²)                   │
│  Adj List            │  —              │  O(V + E)                │
└──────────────────────┴─────────────────┴──────────────────────────┘
```

---

## 15. Learning Roadmap

```
╔═══════════════════════════════════════════════════════════════════╗
║              GRAPH DSA LEARNING ROADMAP                          ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  PHASE 1 — FOUNDATIONS (Week 1-2)                                ║
║  ─────────────────────────────────                               ║
║  □ What is a graph, types of graphs                              ║
║  □ Adjacency Matrix and Adjacency List                           ║
║  □ BFS — implementation + shortest path on unweighted graph      ║
║  □ DFS — recursive + iterative                                   ║
║  □ Problems: Number of Islands, Flood Fill, Clone Graph          ║
║                                                                   ║
║  PHASE 2 — INTERMEDIATE (Week 3-4)                               ║
║  ──────────────────────────────────                              ║
║  □ Cycle Detection (Undirected + Directed)                       ║
║  □ Topological Sort (DFS + Kahn's BFS)                           ║
║  □ Connected Components (DFS + DSU)                              ║
║  □ Bipartite Check                                               ║
║  □ Problems: Course Schedule I & II, Redundant Connection,       ║
║             Graph Valid Tree, Rotting Oranges, 01 Matrix         ║
║                                                                   ║
║  PHASE 3 — SHORTEST PATHS (Week 5)                               ║
║  ─────────────────────────────────                               ║
║  □ Dijkstra's Algorithm                                          ║
║  □ Bellman-Ford Algorithm                                        ║
║  □ Floyd-Warshall                                                ║
║  □ Multi-source BFS                                              ║
║  □ Problems: Network Delay Time, Cheapest Flights,               ║
║             Path with Min Effort, Pacific Atlantic Water Flow     ║
║                                                                   ║
║  PHASE 4 — MST & DSU (Week 6)                                    ║
║  ────────────────────────────                                    ║
║  □ Disjoint Set Union (Union-Find) with optimizations            ║
║  □ Kruskal's Algorithm                                           ║
║  □ Prim's Algorithm                                              ║
║  □ Problems: Min Cost to Connect All Points, Kruskal's MST,      ║
║             Min Spanning Tree, Number of Operations to Make      ║
║             Network Connected                                    ║
║                                                                   ║
║  PHASE 5 — ADVANCED (Week 7-8)                                   ║
║  ─────────────────────────────                                   ║
║  □ Bridges and Articulation Points (Tarjan's)                    ║
║  □ Strongly Connected Components (Kosaraju's / Tarjan's)         ║
║  □ Euler Path/Circuit                                            ║
║  □ Bidirectional BFS                                             ║
║  □ Dijkstra on 2D grid                                           ║
║  □ Problems: Critical Connections, Word Ladder I & II,           ║
║             Alien Dictionary, Sequence Reconstruction            ║
║                                                                   ║
║  PHASE 6 — MASTERY (Ongoing)                                     ║
║  ─────────────────────────                                       ║
║  □ Review all patterns (Grid, Multi-source, Topo, DSU etc.)      ║
║  □ Solve 3-5 problems per pattern                                ║
║  □ Aim for 60+ graph problems on LeetCode                        ║
║  □ Participate in contests to apply under time pressure          ║
║  □ Review NeetCode 150 / Blind 75 graph section                  ║
║                                                                   ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  RECOMMENDED PROBLEM SEQUENCE ON LEETCODE                        ║
║  ─────────────────────────────────────────                       ║
║  #200  Number of Islands               🟢 Easy                   ║
║  #733  Flood Fill                      🟢 Easy                   ║
║  #133  Clone Graph                     🟡 Medium                 ║
║  #695  Max Area of Island              🟡 Medium                 ║
║  #207  Course Schedule                 🟡 Medium                 ║
║  #210  Course Schedule II              🟡 Medium                 ║
║  #547  Number of Provinces             🟡 Medium                 ║
║  #994  Rotting Oranges                 🟡 Medium                 ║
║  #542  01 Matrix                       🟡 Medium                 ║
║  #417  Pacific Atlantic Water Flow     🟡 Medium                 ║
║  #130  Surrounded Regions              🟡 Medium                 ║
║  #721  Accounts Merge                  🟡 Medium                 ║
║  #261  Graph Valid Tree                🟡 Medium                 ║
║  #684  Redundant Connection            🟡 Medium                 ║
║  #127  Word Ladder                     🔴 Hard                   ║
║  #743  Network Delay Time              🟡 Medium                 ║
║  #787  Cheapest Flights K Stops        🟡 Medium                 ║
║  #1631 Path with Min Effort            🟡 Medium                 ║
║  #1584 Min Cost Connect All Points     🟡 Medium                 ║
║  #1192 Critical Connections            🔴 Hard                   ║
║  #269  Alien Dictionary                🔴 Hard                   ║
║  #126  Word Ladder II                  🔴 Hard                   ║
║                                                                   ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## 🔑 Key Takeaways

```
1. ALWAYS clarify graph properties first:
   → Directed or undirected?
   → Weighted or unweighted?
   → Can there be cycles?
   → Connected or potentially disconnected?

2. REPRESENTATION default: Use adjacency list for most problems.

3. VISITED SET is essential: Always track visited nodes in BFS/DFS
   to avoid infinite loops.

4. GRID problems = graph problems:
   Each cell is a node, adjacent cells are edges.

5. SHORTEST PATH algorithm selection:
   Unweighted → BFS
   Non-negative weights → Dijkstra
   Negative weights → Bellman-Ford

6. CYCLE detection:
   Undirected → DFS (check back edge != parent) or DSU
   Directed → DFS with 3-color marking (white/gray/black)

7. TOPOLOGICAL sort only works on DAGs.
   Kahn's BFS is often cleaner and detects cycles naturally.

8. DSU is ideal for:
   → Dynamic connectivity
   → Kruskal's MST
   → Redundant connections
   → Grouping/merging problems

9. THINK in terms of PATTERNS:
   Multi-source BFS, Grid DFS, Bidirectional BFS, etc.
```

---

## 📖 Resources

| Resource | Type | Notes |
|---|---|---|
| [NeetCode Graph Playlist](https://neetcode.io) | Video | Best structured walkthrough |
| [LeetCode Graph Tag](https://leetcode.com/tag/graph/) | Problems | 200+ problems |
| [CP-Algorithms](https://cp-algorithms.com/graph/) | Theory | Deep algorithmic detail |
| [CLRS — Chapter 22-25](https://mitpress.mit.edu/9780262046305/) | Book | Gold standard textbook |
| [Visualgo](https://visualgo.net/en/graphds) | Visual | Interactive graph animations |
| [Graph Theory — William Fiset (YouTube)](https://www.youtube.com/c/WilliamFiset-videos) | Video | Comprehensive theory + code |

---

*Last updated: May 2026 | Author: Akash | Version: 1.0*

> ⭐ **Star this guide** if you found it helpful. Happy Coding!
