# 📊 Graph Data Structures & Algorithms in JavaScript — Complete Tutorial & Roadmap

> **A production-grade, interview-ready deep dive into Graphs using modern JavaScript.**  
> Covers theory, implementation, patterns, problems, and a structured learning roadmap.

---

## 📚 Table of Contents

1. [What is a Graph?](#1-what-is-a-graph)
2. [Graph Terminology](#2-graph-terminology)
3. [Types of Graphs](#3-types-of-graphs)
4. [Graph Representations in JavaScript](#4-graph-representations-in-javascript)
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
> Web crawling, dependency resolution, routing algorithms.

### Mathematical Definition

```
G = (V, E)

where:
  V = set of vertices  → {0, 1, 2, 3, 4}
  E = set of edges     → {(0,1), (0,2), (0,3), (1,4), (2,4)}
```

### Why JavaScript for Graphs?
- `Map` and `Set` are O(1) and ideal for adjacency lists and visited tracking
- `Array` destructuring makes edge processing clean: `const [u, v, w] = edge`
- No built-in `PriorityQueue` — we implement a **MinHeap** class (covered in §8)
- Closures make DFS/BFS implementations concise
- Spread syntax and array methods keep code readable

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
| **Cycle** | Path that starts and ends at same vertex | 0 → 1 → 2 → 0 |
| **Connected** | Every vertex reachable from every other | One-piece graph |
| **Strongly Connected** | Every vertex reachable in directed graph | Two-way reachable |
| **Weighted Edge** | Edge with an associated cost/weight | Road with distance |
| **DAG** | Directed Acyclic Graph | Build dependency tree |
| **Sparse Graph** | Few edges: E ≈ V | Trees |
| **Dense Graph** | Many edges: E ≈ V² | Complete graphs |

---

## 3. Types of Graphs

```
┌─────────────────────────────────────────────────────────────────┐
│                         GRAPH TYPES                             │
├─────────────────┬───────────────────┬───────────────────────────┤
│  Undirected     │  Directed (DAG)   │    Weighted               │
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

---

## 4. Graph Representations in JavaScript

### 4.1 Adjacency Matrix

```
Graph:  0─1, 0─2, 1─2, 2─3

     0  1  2  3
  0 [0, 1, 1, 0]
  1 [1, 0, 1, 0]
  2 [1, 1, 0, 1]
  3 [0, 0, 1, 0]
```

```javascript
class GraphMatrix {
  constructor(v) {
    this.v = v;
    this.matrix = Array.from({ length: v }, () => new Array(v).fill(0));
  }

  addEdge(u, v, weight = 1) {
    this.matrix[u][v] = weight;
    this.matrix[v][u] = weight; // Remove for directed graph
  }

  addDirectedEdge(u, v, weight = 1) {
    this.matrix[u][v] = weight;
  }

  hasEdge(u, v) {
    return this.matrix[u][v] !== 0;
  }

  neighbors(u) {
    return this.matrix[u]
      .map((val, idx) => (val !== 0 ? idx : -1))
      .filter(idx => idx !== -1);
  }

  display() {
    this.matrix.forEach((row, i) => console.log(`${i} → [${row}]`));
  }
}

// Usage
const g = new GraphMatrix(4);
g.addEdge(0, 1);
g.addEdge(0, 2);
g.addEdge(1, 2);
g.addEdge(2, 3);
g.display();
console.log('Has edge (0,1):', g.hasEdge(0, 1)); // true
```

| Operation | Time | Space |
|---|---|---|
| Add Edge | O(1) | O(V²) |
| Check Edge | O(1) | |
| Find Neighbors | O(V) | |

---

### 4.2 Adjacency List ⭐ (Most Common)

```javascript
// Simple unweighted graph
class Graph {
  constructor(v) {
    this.v = v;
    this.adj = new Map();
    for (let i = 0; i < v; i++) this.adj.set(i, []);
  }

  addEdge(u, v) {
    this.adj.get(u).push(v);
    this.adj.get(v).push(u); // Remove for directed graph
  }

  addDirectedEdge(u, v) {
    this.adj.get(u).push(v);
  }

  neighbors(u) {
    return this.adj.get(u) || [];
  }

  display() {
    this.adj.forEach((neighbors, node) => {
      console.log(`${node} → [${neighbors}]`);
    });
  }
}


// Weighted graph
class WeightedGraph {
  constructor(v) {
    this.v = v;
    this.adj = new Map();
    for (let i = 0; i < v; i++) this.adj.set(i, []);
  }

  addEdge(u, v, weight) {
    this.adj.get(u).push({ to: v, weight });
    this.adj.get(v).push({ to: u, weight }); // Remove for directed
  }

  addDirectedEdge(u, v, weight) {
    this.adj.get(u).push({ to: v, weight });
  }

  neighbors(u) {
    return this.adj.get(u) || [];
  }
}


// Functional helper — build graph from edge list
function buildGraph(n, edges, directed = false) {
  const graph = Array.from({ length: n }, () => []);
  for (const [u, v] of edges) {
    graph[u].push(v);
    if (!directed) graph[v].push(u);
  }
  return graph;
}

// Weighted version
function buildWeightedGraph(n, edges, directed = false) {
  const graph = Array.from({ length: n }, () => []);
  for (const [u, v, w] of edges) {
    graph[u].push([v, w]);
    if (!directed) graph[v].push([u, w]);
  }
  return graph;
}

// Usage
const graph = buildGraph(4, [[0,1],[0,2],[1,2],[2,3]]);
console.log(graph); // [ [1,2], [0,2], [0,1,3], [2] ]
```

| Operation | Time | Space |
|---|---|---|
| Add Edge | O(1) | O(V + E) |
| Check Edge | O(degree) | |
| Find Neighbors | O(degree) | |

---

### 4.3 Edge List

```javascript
// Simple edge list
const edges = [[0, 1], [0, 2], [1, 2], [2, 3]];

// Weighted
const weightedEdges = [
  [0, 1, 5],
  [0, 2, 3],
  [1, 2, 2],
  [2, 3, 7],
];

// Sort by weight for Kruskal's
const sorted = [...weightedEdges].sort((a, b) => a[2] - b[2]);
```

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

**Idea:** Explore level by level using a **Queue (Array with shift, or a proper Queue)**

```
Graph:       BFS from node 0:
  0          Level 0: [0]
 /|\         Level 1: [1, 2, 3]
1 2 3        Level 2: [4, 5]
|   |
4   5
```

> ⚠️ **JS Performance Note:** `Array.shift()` is O(n). For large graphs, use a proper Queue with a head pointer or a deque. See the `Queue` class below.

```javascript
// Lightweight Queue using head pointer (O(1) dequeue)
class Queue {
  constructor() {
    this.data = [];
    this.head = 0;
  }
  enqueue(val) { this.data.push(val); }
  dequeue() { return this.data[this.head++]; }
  get size() { return this.data.length - this.head; }
  get isEmpty() { return this.head >= this.data.length; }
}


// Basic BFS
function bfs(graph, start) {
  const visited = new Set([start]);
  const queue = new Queue();
  const order = [];

  queue.enqueue(start);

  while (!queue.isEmpty) {
    const node = queue.dequeue();
    order.push(node);

    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.enqueue(neighbor);
      }
    }
  }
  return order;
}


// BFS with level tracking
function bfsLevels(graph, start) {
  const visited = new Set([start]);
  const queue = new Queue();
  const levels = new Map();

  queue.enqueue([start, 0]);

  while (!queue.isEmpty) {
    const [node, level] = queue.dequeue();
    levels.set(node, level);

    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.enqueue([neighbor, level + 1]);
      }
    }
  }
  return levels;
}


// BFS Shortest Path (unweighted)
function bfsShortestPath(graph, start, end) {
  const visited = new Set([start]);
  const queue = new Queue();

  queue.enqueue([start, [start]]);

  while (!queue.isEmpty) {
    const [node, path] = queue.dequeue();

    if (node === end) return path;

    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.enqueue([neighbor, [...path, neighbor]]);
      }
    }
  }
  return []; // No path found
}


// BFS across disconnected components
function bfsAll(graph, v) {
  const visited = new Set();
  const result = [];

  for (let start = 0; start < v; start++) {
    if (visited.has(start)) continue;
    const queue = new Queue();
    queue.enqueue(start);
    visited.add(start);

    while (!queue.isEmpty) {
      const node = queue.dequeue();
      result.push(node);
      for (const neighbor of graph[node] || []) {
        if (!visited.has(neighbor)) {
          visited.add(neighbor);
          queue.enqueue(neighbor);
        }
      }
    }
  }
  return result;
}
```

**Time:** O(V + E) | **Space:** O(V)

---

### 5.2 Depth-First Search (DFS)

```javascript
// DFS - Recursive
function dfsRecursive(graph, node, visited = new Set()) {
  visited.add(node);
  const result = [node];

  for (const neighbor of graph[node] || []) {
    if (!visited.has(neighbor)) {
      result.push(...dfsRecursive(graph, neighbor, visited));
    }
  }
  return result;
}


// DFS - Iterative (using array as stack)
function dfsIterative(graph, start) {
  const visited = new Set();
  const stack = [start];
  const order = [];

  while (stack.length > 0) {
    const node = stack.pop();
    if (visited.has(node)) continue;

    visited.add(node);
    order.push(node);

    // Push in reverse for consistent left-to-right ordering
    const neighbors = [...(graph[node] || [])].reverse();
    for (const neighbor of neighbors) {
      if (!visited.has(neighbor)) stack.push(neighbor);
    }
  }
  return order;
}


// DFS across all components (handles disconnected graphs)
function dfsAll(graph, v) {
  const visited = new Set();
  const result = [];

  function dfs(node) {
    visited.add(node);
    result.push(node);
    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) dfs(neighbor);
    }
  }

  for (let i = 0; i < v; i++) {
    if (!visited.has(i)) dfs(i);
  }
  return result;
}


// DFS with pre/post timestamps
function dfsTimestamps(graph, start) {
  const visited = new Set();
  const pre = new Map();
  const post = new Map();
  let timer = 0;

  function dfs(node) {
    visited.add(node);
    pre.set(node, ++timer);

    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) dfs(neighbor);
    }
    post.set(node, ++timer);
  }

  dfs(start);
  return { pre, post };
}
```

**Time:** O(V + E) | **Space:** O(V)

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
│ Multi-source BFS   │ Bridges, articulation  │
│ problems           │ points, SCC            │
└────────────────────┴────────────────────────┘
```

---

## 6. Cycle Detection

### 6.1 Undirected Graph — DFS

```javascript
function hasCycleUndirected(graph, v) {
  const visited = new Set();

  function dfs(node, parent) {
    visited.add(node);
    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        if (dfs(neighbor, node)) return true;
      } else if (neighbor !== parent) {
        return true; // Back edge → cycle!
      }
    }
    return false;
  }

  for (let i = 0; i < v; i++) {
    if (!visited.has(i) && dfs(i, -1)) return true;
  }
  return false;
}
```

### 6.2 Undirected Graph — DSU

```javascript
function hasCycleDSU(edges, v) {
  const parent = Array.from({ length: v }, (_, i) => i);
  const rank = new Array(v).fill(0);

  function find(x) {
    if (parent[x] !== x) parent[x] = find(parent[x]); // Path compression
    return parent[x];
  }

  function union(x, y) {
    const px = find(x), py = find(y);
    if (px === py) return false; // Cycle!
    if (rank[px] < rank[py]) parent[px] = py;
    else if (rank[px] > rank[py]) parent[py] = px;
    else { parent[py] = px; rank[px]++; }
    return true;
  }

  return edges.some(([u, v]) => !union(u, v));
}
```

### 6.3 Directed Graph — 3-Color DFS

**Key insight:** Use 3 states:
- `0 = WHITE` — Not visited
- `1 = GRAY` — In current DFS stack (in-progress)
- `2 = BLACK` — Fully processed

```javascript
function hasCycleDirected(graph, v) {
  const color = new Array(v).fill(0); // 0=white, 1=gray, 2=black

  function dfs(node) {
    color[node] = 1; // Mark gray

    for (const neighbor of graph[node] || []) {
      if (color[neighbor] === 1) return true;  // Back edge → cycle!
      if (color[neighbor] === 0 && dfs(neighbor)) return true;
    }

    color[node] = 2; // Mark black (done)
    return false;
  }

  for (let i = 0; i < v; i++) {
    if (color[i] === 0 && dfs(i)) return true;
  }
  return false;
}
```

---

## 7. Topological Sort

> **Only applicable to DAGs (Directed Acyclic Graphs)**  
> Ordering such that for every directed edge u→v, u appears before v.

### 7.1 DFS-based Topological Sort

```javascript
function topoSortDFS(graph, v) {
  const visited = new Set();
  const stack = [];

  function dfs(node) {
    visited.add(node);
    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) dfs(neighbor);
    }
    stack.push(node); // Push AFTER all neighbors are processed
  }

  for (let i = 0; i < v; i++) {
    if (!visited.has(i)) dfs(i);
  }

  return stack.reverse(); // Reverse for topological order
}
```

### 7.2 Kahn's BFS Algorithm ⭐ (Preferred for interviews)

```javascript
function topoSortKahn(graph, v) {
  const inDegree = new Array(v).fill(0);

  for (let u = 0; u < v; u++) {
    for (const neighbor of graph[u] || []) {
      inDegree[neighbor]++;
    }
  }

  const queue = new Queue();
  for (let i = 0; i < v; i++) {
    if (inDegree[i] === 0) queue.enqueue(i);
  }

  const result = [];

  while (!queue.isEmpty) {
    const node = queue.dequeue();
    result.push(node);

    for (const neighbor of graph[node] || []) {
      inDegree[neighbor]--;
      if (inDegree[neighbor] === 0) queue.enqueue(neighbor);
    }
  }

  // If result.length < v → cycle exists, no valid topo order
  return result.length === v ? result : [];
}


// Real-world usage: Course Schedule (LC 207)
function canFinish(numCourses, prerequisites) {
  const graph = Array.from({ length: numCourses }, () => []);
  const inDegree = new Array(numCourses).fill(0);

  for (const [course, prereq] of prerequisites) {
    graph[prereq].push(course);
    inDegree[course]++;
  }

  const queue = new Queue();
  for (let i = 0; i < numCourses; i++) {
    if (inDegree[i] === 0) queue.enqueue(i);
  }

  let completed = 0;
  while (!queue.isEmpty) {
    const course = queue.dequeue();
    completed++;
    for (const next of graph[course]) {
      if (--inDegree[next] === 0) queue.enqueue(next);
    }
  }

  return completed === numCourses;
}
```

### 7.3 Topo Sort — Step-by-Step Example

```
Tasks: A→C, B→C, C→D, C→E

In-degrees: A=0, B=0, C=2, D=1, E=1

Step 1: Queue=[A,B]
        Process A → queue=[B,C]  (C in-degree: 2→1)
Step 2: Process B → queue=[C]   (C in-degree: 1→0)
Step 3: Process C → queue=[D,E]
Step 4: Process D, E

Result: A → B → C → D → E  ✅ valid topological order
```

---

## 8. Shortest Path Algorithms

### 8.1 BFS — Unweighted Shortest Path

Already covered in Section 5.1. Time: O(V + E)

---

### 8.2 MinHeap — Required for Dijkstra in JS

> JavaScript has no built-in priority queue. Implement a MinHeap before using Dijkstra.

```javascript
class MinHeap {
  constructor() {
    this.heap = [];
  }

  push(val) {
    this.heap.push(val);
    this._bubbleUp(this.heap.length - 1);
  }

  pop() {
    if (this.size === 0) return null;
    const top = this.heap[0];
    const last = this.heap.pop();
    if (this.heap.length > 0) {
      this.heap[0] = last;
      this._sinkDown(0);
    }
    return top;
  }

  get size() { return this.heap.length; }

  _bubbleUp(i) {
    while (i > 0) {
      const parent = Math.floor((i - 1) / 2);
      if (this.heap[parent][0] <= this.heap[i][0]) break;
      [this.heap[parent], this.heap[i]] = [this.heap[i], this.heap[parent]];
      i = parent;
    }
  }

  _sinkDown(i) {
    const n = this.heap.length;
    while (true) {
      let smallest = i;
      const left = 2 * i + 1, right = 2 * i + 2;
      if (left < n && this.heap[left][0] < this.heap[smallest][0]) smallest = left;
      if (right < n && this.heap[right][0] < this.heap[smallest][0]) smallest = right;
      if (smallest === i) break;
      [this.heap[smallest], this.heap[i]] = [this.heap[i], this.heap[smallest]];
      i = smallest;
    }
  }
}
```

---

### 8.3 Dijkstra's Algorithm ⭐ (Non-negative weights)

```javascript
function dijkstra(graph, start, v) {
  /**
   * graph: adjacency list — graph[u] = [[v, weight], ...]
   * Returns: dist[] = shortest distances from start
   */
  const dist = new Array(v).fill(Infinity);
  dist[start] = 0;

  const heap = new MinHeap();
  heap.push([0, start]); // [distance, node]

  while (heap.size > 0) {
    const [d, u] = heap.pop();

    if (d > dist[u]) continue; // Stale entry

    for (const [neighbor, weight] of graph[u] || []) {
      const newDist = dist[u] + weight;
      if (newDist < dist[neighbor]) {
        dist[neighbor] = newDist;
        heap.push([newDist, neighbor]);
      }
    }
  }
  return dist;
}


// Dijkstra with path reconstruction
function dijkstraWithPath(graph, start, end, v) {
  const dist = new Array(v).fill(Infinity);
  const prev = new Array(v).fill(-1);
  dist[start] = 0;

  const heap = new MinHeap();
  heap.push([0, start]);

  while (heap.size > 0) {
    const [d, u] = heap.pop();
    if (d > dist[u]) continue;
    if (u === end) break;

    for (const [neighbor, weight] of graph[u] || []) {
      if (dist[u] + weight < dist[neighbor]) {
        dist[neighbor] = dist[u] + weight;
        prev[neighbor] = u;
        heap.push([dist[neighbor], neighbor]);
      }
    }
  }

  // Reconstruct path
  const path = [];
  let node = end;
  while (node !== -1) {
    path.push(node);
    node = prev[node];
  }
  return { distance: dist[end], path: path.reverse() };
}


// Dijkstra on 2D Grid (Path with Minimum Effort — LC 1631)
function dijkstraGrid(heights) {
  const rows = heights.length, cols = heights[0].length;
  const dist = Array.from({ length: rows }, () => new Array(cols).fill(Infinity));
  dist[0][0] = 0;

  const heap = new MinHeap();
  heap.push([0, 0, 0]); // [effort, row, col]

  const dirs = [[0,1],[0,-1],[1,0],[-1,0]];

  while (heap.size > 0) {
    const [effort, r, c] = heap.pop();
    if (r === rows - 1 && c === cols - 1) return effort;
    if (effort > dist[r][c]) continue;

    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
        const newEffort = Math.max(effort, Math.abs(heights[nr][nc] - heights[r][c]));
        if (newEffort < dist[nr][nc]) {
          dist[nr][nc] = newEffort;
          heap.push([newEffort, nr, nc]);
        }
      }
    }
  }
  return dist[rows - 1][cols - 1];
}
```

**Time:** O((V + E) log V) | **Space:** O(V)  
**Constraint:** ❌ Does NOT work with **negative weights**

---

### 8.4 Bellman-Ford Algorithm (Handles negative weights)

```javascript
function bellmanFord(edges, v, start) {
  /**
   * edges: [[u, v, weight], ...]
   * Returns: dist[] or null if negative cycle exists
   */
  const dist = new Array(v).fill(Infinity);
  dist[start] = 0;

  // Relax all edges V-1 times
  for (let i = 0; i < v - 1; i++) {
    for (const [u, neighbor, w] of edges) {
      if (dist[u] !== Infinity && dist[u] + w < dist[neighbor]) {
        dist[neighbor] = dist[u] + w;
      }
    }
  }

  // V-th relaxation: if anything still relaxes → negative cycle
  for (const [u, neighbor, w] of edges) {
    if (dist[u] !== Infinity && dist[u] + w < dist[neighbor]) {
      return null; // Negative cycle detected
    }
  }
  return dist;
}
```

**Time:** O(V × E) | **Space:** O(V)

---

### 8.5 Floyd-Warshall (All-pairs shortest path)

```javascript
function floydWarshall(v, edges, directed = false) {
  const INF = Infinity;
  const dist = Array.from({ length: v }, (_, i) =>
    Array.from({ length: v }, (_, j) => (i === j ? 0 : INF))
  );

  for (const [u, neighbor, w] of edges) {
    dist[u][neighbor] = w;
    if (!directed) dist[neighbor][u] = w;
  }

  // DP: try every node k as intermediate
  for (let k = 0; k < v; k++) {
    for (let i = 0; i < v; i++) {
      for (let j = 0; j < v; j++) {
        if (dist[i][k] !== INF && dist[k][j] !== INF) {
          dist[i][j] = Math.min(dist[i][j], dist[i][k] + dist[k][j]);
        }
      }
    }
  }

  // Check for negative cycles
  for (let i = 0; i < v; i++) {
    if (dist[i][i] < 0) return null;
  }
  return dist;
}
```

**Time:** O(V³) | **Space:** O(V²)

---

### 8.6 Shortest Path Algorithm Selection Guide

```
┌─────────────────────────────────────────────────────┐
│         Shortest Path — Algorithm Selection         │
├─────────────────────────────────────────────────────┤
│  Unweighted graph?  ──────────► BFS  O(V+E)        │
│                                                     │
│  Weighted, no negative weights?                     │
│    Single source  ────────────► Dijkstra            │
│                                  O((V+E) log V)     │
│    All pairs      ────────────► Floyd-Warshall O(V³)│
│                                                     │
│  Negative weights (no neg cycle)?                   │
│    Single source  ────────────► Bellman-Ford O(VE)  │
│    All pairs      ────────────► Floyd-Warshall O(V³)│
│                                                     │
│  DAG (no cycles)?  ────────────► Topo + relax O(V+E)│
└─────────────────────────────────────────────────────┘
```

---

## 9. Minimum Spanning Tree (MST)

> An MST connects all V vertices with minimum total edge weight. Always has exactly V-1 edges.

### 9.1 Kruskal's Algorithm

```javascript
function kruskal(v, edges) {
  // edges: [[weight, u, v], ...]
  const parent = Array.from({ length: v }, (_, i) => i);
  const rank = new Array(v).fill(0);

  function find(x) {
    if (parent[x] !== x) parent[x] = find(parent[x]);
    return parent[x];
  }

  function union(x, y) {
    const px = find(x), py = find(y);
    if (px === py) return false; // Cycle
    if (rank[px] < rank[py]) parent[px] = py;
    else if (rank[px] > rank[py]) parent[py] = px;
    else { parent[py] = px; rank[px]++; }
    return true;
  }

  const sortedEdges = [...edges].sort((a, b) => a[0] - b[0]);
  const mstEdges = [];
  let totalWeight = 0;

  for (const [w, u, v] of sortedEdges) {
    if (union(u, v)) {
      mstEdges.push([u, v, w]);
      totalWeight += w;
      if (mstEdges.length === v - 1) break;
    }
  }

  return { mstEdges, totalWeight };
}
```

**Time:** O(E log E) | **Space:** O(V)

---

### 9.2 Prim's Algorithm

```javascript
function prim(graph, v, start = 0) {
  /**
   * graph[u] = [[neighbor, weight], ...]
   */
  const inMST = new Array(v).fill(false);
  const heap = new MinHeap();
  heap.push([0, start, -1]); // [weight, node, parent]

  let totalWeight = 0;
  const mstEdges = [];

  while (heap.size > 0) {
    const [weight, u, parent] = heap.pop();

    if (inMST[u]) continue;
    inMST[u] = true;
    totalWeight += weight;

    if (parent !== -1) mstEdges.push([parent, u, weight]);

    for (const [neighbor, w] of graph[u] || []) {
      if (!inMST[neighbor]) heap.push([w, neighbor, u]);
    }
  }

  return { totalWeight, mstEdges };
}
```

**Time:** O((V + E) log V) | **Space:** O(V)

---

### 9.3 Kruskal vs Prim

| Feature | Kruskal | Prim |
|---|---|---|
| Approach | Edge-based | Vertex-based |
| Data Structure | DSU + sorted edges | MinHeap |
| Best for | Sparse graphs | Dense graphs |
| Time | O(E log E) | O((V+E) log V) |
| Works on | Disconnected graphs | Connected graphs |

---

## 10. Disjoint Set Union (DSU)

> Also called **Union-Find**. Efficiently tracks and merges connected components.

```javascript
class DSU {
  constructor(n) {
    this.parent = Array.from({ length: n }, (_, i) => i);
    this.rank = new Array(n).fill(0);
    this.size = new Array(n).fill(1);
    this.components = n;
  }

  find(x) {
    if (this.parent[x] !== x) {
      this.parent[x] = this.find(this.parent[x]); // Path compression
    }
    return this.parent[x];
  }

  // Returns true if union was performed (different components)
  union(x, y) {
    const px = this.find(x), py = this.find(y);
    if (px === py) return false; // Already connected

    if (this.rank[px] < this.rank[py]) {
      this.parent[px] = py;
      this.size[py] += this.size[px];
    } else if (this.rank[px] > this.rank[py]) {
      this.parent[py] = px;
      this.size[px] += this.size[py];
    } else {
      this.parent[py] = px;
      this.size[px] += this.size[py];
      this.rank[px]++;
    }
    this.components--;
    return true;
  }

  connected(x, y) {
    return this.find(x) === this.find(y);
  }

  componentSize(x) {
    return this.size[this.find(x)];
  }
}


// Usage
const dsu = new DSU(5);
dsu.union(0, 1);
dsu.union(1, 2);
dsu.union(3, 4);

console.log(dsu.components);          // 2
console.log(dsu.connected(0, 2));     // true
console.log(dsu.connected(0, 3));     // false
console.log(dsu.componentSize(0));    // 3


// Practical: Number of Operations to Make Network Connected (LC 1319)
function makeConnected(n, connections) {
  if (connections.length < n - 1) return -1;

  const dsu = new DSU(n);
  for (const [u, v] of connections) dsu.union(u, v);

  return dsu.components - 1; // Move (components - 1) cables
}
```

**Operations:** O(α(n)) ≈ O(1) amortized (inverse Ackermann)

---

## 11. Advanced Graph Algorithms

### 11.1 Connected Components

```javascript
function countComponents(graph, v) {
  const visited = new Set();
  let count = 0;

  function dfs(node) {
    visited.add(node);
    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) dfs(neighbor);
    }
  }

  for (let i = 0; i < v; i++) {
    if (!visited.has(i)) {
      dfs(i);
      count++;
    }
  }
  return count;
}
```

---

### 11.2 Bipartite Check (2-Coloring)

```javascript
function isBipartite(graph, v) {
  const color = new Array(v).fill(-1);

  for (let start = 0; start < v; start++) {
    if (color[start] !== -1) continue;

    const queue = new Queue();
    queue.enqueue(start);
    color[start] = 0;

    while (!queue.isEmpty) {
      const node = queue.dequeue();
      for (const neighbor of graph[node] || []) {
        if (color[neighbor] === -1) {
          color[neighbor] = 1 - color[node]; // Opposite color
          queue.enqueue(neighbor);
        } else if (color[neighbor] === color[node]) {
          return false; // Same color = not bipartite
        }
      }
    }
  }
  return true;
}
```

---

### 11.3 Bridges (Tarjan's Algorithm)

> **Bridge:** An edge whose removal disconnects the graph (LC 1192: Critical Connections)

```javascript
function findBridges(graph, v) {
  const visited = new Array(v).fill(false);
  const disc = new Array(v).fill(0);
  const low = new Array(v).fill(0);
  const bridges = [];
  let timer = 0;

  function dfs(u, parent) {
    visited[u] = true;
    disc[u] = low[u] = timer++;

    for (const neighbor of graph[u] || []) {
      if (!visited[neighbor]) {
        dfs(neighbor, u);
        low[u] = Math.min(low[u], low[neighbor]);
        if (low[neighbor] > disc[u]) {
          bridges.push([u, neighbor]); // Bridge found!
        }
      } else if (neighbor !== parent) {
        low[u] = Math.min(low[u], disc[neighbor]);
      }
    }
  }

  for (let i = 0; i < v; i++) {
    if (!visited[i]) dfs(i, -1);
  }
  return bridges;
}


// LC 1192: Critical Connections in a Network
function criticalConnections(n, connections) {
  const graph = Array.from({ length: n }, () => []);
  for (const [u, v] of connections) {
    graph[u].push(v);
    graph[v].push(u);
  }

  const disc = new Array(n).fill(-1);
  const low = new Array(n).fill(0);
  const result = [];
  let timer = 0;

  function dfs(u, parent) {
    disc[u] = low[u] = timer++;
    for (const v of graph[u]) {
      if (v === parent) continue;
      if (disc[v] === -1) {
        dfs(v, u);
        low[u] = Math.min(low[u], low[v]);
        if (low[v] > disc[u]) result.push([u, v]);
      } else {
        low[u] = Math.min(low[u], disc[v]);
      }
    }
  }

  for (let i = 0; i < n; i++) {
    if (disc[i] === -1) dfs(i, -1);
  }
  return result;
}
```

---

### 11.4 Strongly Connected Components — Kosaraju's Algorithm

```javascript
function kosarajuSCC(graph, v) {
  const visited = new Array(v).fill(false);
  const stack = [];

  // Step 1: DFS on original graph, fill stack by finish time
  function dfs1(node) {
    visited[node] = true;
    for (const neighbor of graph[node] || []) {
      if (!visited[neighbor]) dfs1(neighbor);
    }
    stack.push(node);
  }

  for (let i = 0; i < v; i++) {
    if (!visited[i]) dfs1(i);
  }

  // Step 2: Build transposed graph
  const transposed = Array.from({ length: v }, () => []);
  for (let u = 0; u < v; u++) {
    for (const neighbor of graph[u] || []) {
      transposed[neighbor].push(u);
    }
  }

  // Step 3: DFS on transposed in reverse finish order
  const visitedT = new Array(v).fill(false);
  const sccs = [];

  function dfs2(node, component) {
    visitedT[node] = true;
    component.push(node);
    for (const neighbor of transposed[node] || []) {
      if (!visitedT[neighbor]) dfs2(neighbor, component);
    }
  }

  while (stack.length > 0) {
    const node = stack.pop();
    if (!visitedT[node]) {
      const component = [];
      dfs2(node, component);
      sccs.push(component);
    }
  }

  return sccs;
}
```

---

### 11.5 Flood Fill (Matrix as Graph)

```javascript
// DFS Flood Fill
function floodFill(image, sr, sc, color) {
  const oldColor = image[sr][sc];
  if (oldColor === color) return image;

  const rows = image.length, cols = image[0].length;

  function dfs(r, c) {
    if (r < 0 || r >= rows || c < 0 || c >= cols) return;
    if (image[r][c] !== oldColor) return;

    image[r][c] = color;
    dfs(r + 1, c); dfs(r - 1, c);
    dfs(r, c + 1); dfs(r, c - 1);
  }

  dfs(sr, sc);
  return image;
}


// BFS Flood Fill (safer for large grids)
function floodFillBFS(image, sr, sc, color) {
  const oldColor = image[sr][sc];
  if (oldColor === color) return image;

  const rows = image.length, cols = image[0].length;
  const queue = new Queue();
  queue.enqueue([sr, sc]);
  image[sr][sc] = color;

  const dirs = [[1,0],[-1,0],[0,1],[0,-1]];

  while (!queue.isEmpty) {
    const [r, c] = queue.dequeue();
    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && image[nr][nc] === oldColor) {
        image[nr][nc] = color;
        queue.enqueue([nr, nc]);
      }
    }
  }
  return image;
}
```

---

### 11.6 Multi-Source BFS

```javascript
// Pattern: initialize ALL sources at dist=0, then BFS outward
function multiSourceBFS(grid) {
  const rows = grid.length, cols = grid[0].length;
  const dist = Array.from({ length: rows }, () => new Array(cols).fill(Infinity));
  const queue = new Queue();

  // Enqueue ALL sources simultaneously
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === 0) { // Source condition
        dist[r][c] = 0;
        queue.enqueue([r, c]);
      }
    }
  }

  const dirs = [[1,0],[-1,0],[0,1],[0,-1]];

  while (!queue.isEmpty) {
    const [r, c] = queue.dequeue();
    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
        if (dist[r][c] + 1 < dist[nr][nc]) {
          dist[nr][nc] = dist[r][c] + 1;
          queue.enqueue([nr, nc]);
        }
      }
    }
  }
  return dist;
}
```

---

## 12. Graph Patterns for Interviews

### Pattern 1: Grid as Graph

```javascript
// Direction arrays
const dirs4 = [[0,1],[0,-1],[1,0],[-1,0]];
const dirs8 = [[0,1],[0,-1],[1,0],[-1,0],[1,1],[1,-1],[-1,1],[-1,-1]];

// Bounds check helper
const inBounds = (r, c, rows, cols) => r >= 0 && r < rows && c >= 0 && c < cols;

// Standard grid BFS template
function gridBFS(grid, sr, sc) {
  const rows = grid.length, cols = grid[0].length;
  const visited = Array.from({ length: rows }, () => new Array(cols).fill(false));
  const queue = new Queue();

  queue.enqueue([sr, sc, 0]); // [row, col, steps]
  visited[sr][sc] = true;

  while (!queue.isEmpty) {
    const [r, c, steps] = queue.dequeue();

    // Process cell here
    if (r === rows - 1 && c === cols - 1) return steps; // Example: reach target

    for (const [dr, dc] of dirs4) {
      const nr = r + dr, nc = c + dc;
      if (inBounds(nr, nc, rows, cols) && !visited[nr][nc] && grid[nr][nc] !== '#') {
        visited[nr][nc] = true;
        queue.enqueue([nr, nc, steps + 1]);
      }
    }
  }
  return -1;
}
```

### Pattern 2: Multi-Source BFS Template

```javascript
function multiSourceTemplate(grid) {
  const rows = grid.length, cols = grid[0].length;
  const dist = Array.from({ length: rows }, () => new Array(cols).fill(Infinity));
  const queue = new Queue();

  // Enqueue all sources
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (isSource(grid[r][c])) {
        dist[r][c] = 0;
        queue.enqueue([r, c]);
      }
    }
  }

  while (!queue.isEmpty) {
    const [r, c] = queue.dequeue();
    for (const [dr, dc] of dirs4) {
      const nr = r + dr, nc = c + dc;
      if (inBounds(nr, nc, rows, cols) && dist[r][c] + 1 < dist[nr][nc]) {
        dist[nr][nc] = dist[r][c] + 1;
        queue.enqueue([nr, nc]);
      }
    }
  }
  return dist;
}

const isSource = cell => cell === 0; // Customize per problem
```

### Pattern 3: Dijkstra on Grid

```javascript
function dijkstraGridTemplate(grid) {
  const rows = grid.length, cols = grid[0].length;
  const dist = Array.from({ length: rows }, () => new Array(cols).fill(Infinity));
  dist[0][0] = 0;

  const heap = new MinHeap();
  heap.push([0, 0, 0]); // [cost, row, col]

  while (heap.size > 0) {
    const [cost, r, c] = heap.pop();
    if (r === rows - 1 && c === cols - 1) return cost;
    if (cost > dist[r][c]) continue;

    for (const [dr, dc] of dirs4) {
      const nr = r + dr, nc = c + dc;
      if (inBounds(nr, nc, rows, cols)) {
        const newCost = cost + grid[nr][nc]; // Or Math.max / Math.abs
        if (newCost < dist[nr][nc]) {
          dist[nr][nc] = newCost;
          heap.push([newCost, nr, nc]);
        }
      }
    }
  }
  return dist[rows - 1][cols - 1];
}
```

### Pattern 4: Topo Sort for Dependencies

```javascript
function topoTemplate(n, deps) {
  const graph = Array.from({ length: n }, () => []);
  const inDegree = new Array(n).fill(0);

  for (const [task, prereq] of deps) {
    graph[prereq].push(task);
    inDegree[task]++;
  }

  const queue = new Queue();
  for (let i = 0; i < n; i++) {
    if (inDegree[i] === 0) queue.enqueue(i);
  }

  const order = [];
  while (!queue.isEmpty) {
    const node = queue.dequeue();
    order.push(node);
    for (const next of graph[node]) {
      if (--inDegree[next] === 0) queue.enqueue(next);
    }
  }

  return order.length === n ? order : []; // Empty = cycle
}
```

### Pattern 5: DSU for Dynamic Connectivity

```javascript
function dynamicConnectivity(n, edges) {
  const dsu = new DSU(n);
  const snapshots = [];

  for (const [u, v] of edges) {
    dsu.union(u, v);
    snapshots.push(dsu.components);
  }
  return snapshots;
}
```

### Pattern 6: Word Ladder — BFS on Implicit Graph

```javascript
function wordLadder(beginWord, endWord, wordList) {
  const wordSet = new Set(wordList);
  if (!wordSet.has(endWord)) return 0;

  const queue = new Queue();
  queue.enqueue([beginWord, 1]); // [word, steps]
  const visited = new Set([beginWord]);

  while (!queue.isEmpty) {
    const [word, steps] = queue.dequeue();

    for (let i = 0; i < word.length; i++) {
      for (let c = 97; c <= 122; c++) { // 'a' to 'z'
        const newWord = word.slice(0, i) + String.fromCharCode(c) + word.slice(i + 1);
        if (newWord === endWord) return steps + 1;
        if (wordSet.has(newWord) && !visited.has(newWord)) {
          visited.add(newWord);
          queue.enqueue([newWord, steps + 1]);
        }
      }
    }
  }
  return 0;
}
```

---

## 13. Problem Bank with Solutions

### Problem Categories

| # | Problem | Pattern | Algorithm |
|---|---|---|---|
| 200 | Number of Islands | Grid DFS | DFS + visited |
| 733 | Flood Fill | Grid DFS | DFS |
| 1971 | Find Path Exists | BFS/DFS | BFS |
| 133 | Clone Graph | BFS + Map | BFS |
| 695 | Max Area of Island | Grid DFS | DFS |
| 207 | Course Schedule | Cycle in DAG | Topo / DFS |
| 210 | Course Schedule II | Topo Order | Kahn's BFS |
| 547 | Number of Provinces | Components | DFS / DSU |
| 994 | Rotting Oranges | Multi-source BFS | BFS |
| 542 | 01 Matrix | Multi-source BFS | BFS |
| 417 | Pacific Atlantic | Reverse DFS | DFS |
| 130 | Surrounded Regions | Border DFS | DFS |
| 684 | Redundant Connection | Cycle | DSU |
| 261 | Graph Valid Tree | Connected+Acyclic | DSU |
| 1319 | Make Network Connected | Components | DSU |
| 743 | Network Delay Time | Single-source SP | Dijkstra |
| 787 | Cheapest Flights K Stops | DP + BFS | Bellman-Ford |
| 1631 | Path with Min Effort | Dijkstra grid | Dijkstra |
| 1584 | Min Cost Connect Points | MST | Kruskal/Prim |
| 721 | Accounts Merge | Component groups | DSU |
| 1192 | Critical Connections | Bridges | Tarjan's |
| 269 | Alien Dictionary | Topo sort | Kahn's |
| 127 | Word Ladder | BFS implicit graph | BFS |

---

### ✅ Full Solution: Number of Islands (LC 200)

```javascript
function numIslands(grid) {
  const rows = grid.length, cols = grid[0].length;
  let count = 0;

  function dfs(r, c) {
    if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] !== '1') return;
    grid[r][c] = '#'; // Mark visited
    dfs(r + 1, c); dfs(r - 1, c);
    dfs(r, c + 1); dfs(r, c - 1);
  }

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === '1') {
        dfs(r, c);
        count++;
      }
    }
  }
  return count;
}
```

---

### ✅ Full Solution: Course Schedule II (LC 210)

```javascript
function findOrder(numCourses, prerequisites) {
  const graph = Array.from({ length: numCourses }, () => []);
  const inDegree = new Array(numCourses).fill(0);

  for (const [course, prereq] of prerequisites) {
    graph[prereq].push(course);
    inDegree[course]++;
  }

  const queue = new Queue();
  for (let i = 0; i < numCourses; i++) {
    if (inDegree[i] === 0) queue.enqueue(i);
  }

  const order = [];
  while (!queue.isEmpty) {
    const course = queue.dequeue();
    order.push(course);
    for (const next of graph[course]) {
      if (--inDegree[next] === 0) queue.enqueue(next);
    }
  }

  return order.length === numCourses ? order : [];
}
```

---

### ✅ Full Solution: Rotting Oranges (LC 994)

```javascript
function orangesRotting(grid) {
  const rows = grid.length, cols = grid[0].length;
  const queue = new Queue();
  let fresh = 0;

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === 2) queue.enqueue([r, c]);
      else if (grid[r][c] === 1) fresh++;
    }
  }

  if (fresh === 0) return 0;

  const dirs = [[1,0],[-1,0],[0,1],[0,-1]];
  let minutes = 0;

  while (!queue.isEmpty && fresh > 0) {
    let size = queue.size;
    while (size-- > 0) {
      const [r, c] = queue.dequeue();
      for (const [dr, dc] of dirs) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] === 1) {
          grid[nr][nc] = 2;
          fresh--;
          queue.enqueue([nr, nc]);
        }
      }
    }
    minutes++;
  }

  return fresh === 0 ? minutes : -1;
}
```

---

### ✅ Full Solution: Network Delay Time (LC 743)

```javascript
function networkDelayTime(times, n, k) {
  const graph = Array.from({ length: n + 1 }, () => []);
  for (const [u, v, w] of times) graph[u].push([v, w]);

  const dist = new Array(n + 1).fill(Infinity);
  dist[k] = 0;

  const heap = new MinHeap();
  heap.push([0, k]); // [distance, node]

  while (heap.size > 0) {
    const [d, u] = heap.pop();
    if (d > dist[u]) continue;
    for (const [v, w] of graph[u]) {
      if (dist[u] + w < dist[v]) {
        dist[v] = dist[u] + w;
        heap.push([dist[v], v]);
      }
    }
  }

  const maxDist = Math.max(...dist.slice(1));
  return maxDist === Infinity ? -1 : maxDist;
}
```

---

### ✅ Full Solution: Min Cost to Connect All Points (LC 1584)

```javascript
function minCostConnectPoints(points) {
  const n = points.length;
  const inMST = new Array(n).fill(false);
  const minCost = new Array(n).fill(Infinity);
  minCost[0] = 0;

  let total = 0;

  for (let i = 0; i < n; i++) {
    // Find min cost vertex not yet in MST
    let u = -1;
    for (let j = 0; j < n; j++) {
      if (!inMST[j] && (u === -1 || minCost[j] < minCost[u])) u = j;
    }

    inMST[u] = true;
    total += minCost[u];

    for (let v = 0; v < n; v++) {
      if (!inMST[v]) {
        const dist = Math.abs(points[u][0] - points[v][0]) + Math.abs(points[u][1] - points[v][1]);
        if (dist < minCost[v]) minCost[v] = dist;
      }
    }
  }
  return total;
}
```

---

### ✅ Full Solution: Clone Graph (LC 133)

```javascript
function cloneGraph(node) {
  if (!node) return null;

  const visited = new Map(); // originalNode → clonedNode

  function dfs(n) {
    if (visited.has(n)) return visited.get(n);

    const clone = { val: n.val, neighbors: [] };
    visited.set(n, clone);

    for (const neighbor of n.neighbors) {
      clone.neighbors.push(dfs(neighbor));
    }
    return clone;
  }

  return dfs(node);
}
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
│  Dijkstra (MinHeap)  │  O((V+E) log V) │  O(V)                    │
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

### JavaScript-Specific Performance Notes

| JS Structure | Use Case | Notes |
|---|---|---|
| `Set` | Visited tracking | O(1) add/has — always prefer over arrays |
| `Map` | Adjacency lists, node cloning | O(1) get/set |
| Custom `Queue` | BFS | `Array.shift()` is O(n) — avoid for large graphs |
| Custom `MinHeap` | Dijkstra, Prim's | No built-in PriorityQueue in JS |
| `Array.from({length:n}, ()=>[])` | Adjacency list init | Idiomatic, avoids aliasing bug |
| `[...array].sort(...)` | Kruskal's | Always copy before sort to avoid mutation |
| Spread `[...path, node]` | BFS path tracking | Creates new array — fine for small inputs |
| Closure-based DFS | Recursive DFS | Clean, captures outer scope naturally |

> ⚠️ **Common JS Bug:** `new Array(n).fill([])` creates n references to the **same array**. Always use `Array.from({length: n}, () => [])`.

---

## 15. Learning Roadmap

```
╔═══════════════════════════════════════════════════════════════════╗
║          GRAPH DSA LEARNING ROADMAP — JAVASCRIPT                 ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  PHASE 1 — FOUNDATIONS (Week 1-2)                                ║
║  ─────────────────────────────────                               ║
║  □ Graph types and terminology                                   ║
║  □ Build adjacency list using Map and plain arrays               ║
║  □ Implement the Queue class (head-pointer based)                ║
║  □ BFS with Queue — level traversal + shortest path              ║
║  □ DFS recursive + iterative (array as stack)                    ║
║  □ Handle disconnected graphs (loop over all unvisited nodes)    ║
║  □ Problems: Number of Islands, Flood Fill, Clone Graph,         ║
║             Find Path Exists (LC 1971)                           ║
║                                                                   ║
║  PHASE 2 — INTERMEDIATE (Week 3-4)                               ║
║  ──────────────────────────────────                              ║
║  □ Cycle detection: undirected DFS and DSU                       ║
║  □ Cycle detection: directed graph (3-color)                     ║
║  □ Topological sort: DFS-stack + Kahn's BFS                      ║
║  □ Connected components + bipartite check                        ║
║  □ Build DSU class with path compression + union by rank         ║
║  □ Problems: Course Schedule I & II, Redundant Connection,       ║
║             Graph Valid Tree, Rotting Oranges, 01 Matrix,        ║
║             Number of Provinces, Make Network Connected          ║
║                                                                   ║
║  PHASE 3 — SHORTEST PATHS (Week 5)                               ║
║  ─────────────────────────────────                               ║
║  □ Implement MinHeap class from scratch                          ║
║  □ Dijkstra single-source + path reconstruction                  ║
║  □ Dijkstra on 2D grids                                          ║
║  □ Bellman-Ford for negative weights                             ║
║  □ Floyd-Warshall all-pairs                                      ║
║  □ Multi-source BFS pattern                                      ║
║  □ Problems: Network Delay Time, Cheapest Flights K Stops,       ║
║             Path with Min Effort, Pacific Atlantic Water Flow,   ║
║             Swim in Rising Water                                 ║
║                                                                   ║
║  PHASE 4 — MST & DSU DEEP DIVE (Week 6)                         ║
║  ──────────────────────────────────────                          ║
║  □ Kruskal's with edge sort and DSU                              ║
║  □ Prim's with MinHeap                                           ║
║  □ DSU: component size, dynamic connectivity                     ║
║  □ Problems: Min Cost Connect All Points, Accounts Merge,        ║
║             Optimize Water Distribution, Min Spanning Tree       ║
║                                                                   ║
║  PHASE 5 — ADVANCED (Week 7-8)                                   ║
║  ─────────────────────────────                                   ║
║  □ Bridges and articulation points (Tarjan's)                    ║
║  □ Strongly Connected Components (Kosaraju's)                    ║
║  □ Euler Path/Circuit                                            ║
║  □ Bidirectional BFS                                             ║
║  □ Word Ladder — BFS on implicit graph                           ║
║  □ Problems: Critical Connections, Word Ladder I & II,           ║
║             Alien Dictionary, Sequence Reconstruction,           ║
║             Bus Routes                                           ║
║                                                                   ║
║  PHASE 6 — MASTERY (Ongoing)                                     ║
║  ─────────────────────────                                       ║
║  □ Review all patterns with clean JS idioms                      ║
║  □ Solve 3-5 problems per pattern                                ║
║  □ Aim for 60+ graph problems on LeetCode using JavaScript       ║
║  □ Profile Queue vs Array.shift() on large inputs                ║
║  □ Review NeetCode 150 / Blind 75 graph section in JS            ║
║                                                                   ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  RECOMMENDED PROBLEM SEQUENCE ON LEETCODE (JavaScript)           ║
║  ─────────────────────────────────────────────────────           ║
║  #200  Number of Islands               🟢 Easy                   ║
║  #733  Flood Fill                      🟢 Easy                   ║
║  #1971 Find Path Exists in Graph       🟢 Easy                   ║
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
║  #684  Redundant Connection            🟡 Medium                 ║
║  #261  Graph Valid Tree                🟡 Medium                 ║
║  #1319 Make Network Connected          🟡 Medium                 ║
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
   → Directed or undirected?  Weighted or unweighted?
   → Can there be cycles?  Connected or disconnected?

2. JAVASCRIPT-SPECIFIC must-knows:
   → BFS queue  → Custom Queue class (NOT Array.shift — it's O(n))
   → DFS stack  → Plain array with push/pop (O(1))
   → Min-heap   → Implement MinHeap — JS has no built-in PriorityQueue
   → Visited    → Set (NOT array — O(1) has vs O(n))
   → Adj list   → Array.from({length:n}, ()=>[])  ← avoids aliasing!

3. ARRAY.FILL([]) BUG — the most common JS graph bug:
   ❌ new Array(n).fill([])       // All rows point to SAME array!
   ✅ Array.from({length:n},()=>[]) // Each row is a new array

4. GRID problems = graph problems:
   Each cell is a node; adjacent cells are edges.
   dirs4 = [[0,1],[0,-1],[1,0],[-1,0]] is your best friend.

5. SHORTEST PATH selection:
   Unweighted → BFS
   Non-negative weights → Dijkstra (with MinHeap)
   Negative weights → Bellman-Ford

6. CYCLE detection:
   Undirected → DFS (back edge != parent) or DSU
   Directed   → DFS with 3-color array (0/1/2)

7. TOPOLOGICAL SORT only works on DAGs.
   Kahn's BFS naturally detects cycles (result.length !== v).

8. DSU is ideal for:
   → Dynamic connectivity queries
   → Kruskal's MST
   → Redundant connections
   → Grouping/merging (Accounts Merge)

9. MULTI-SOURCE BFS: enqueue ALL sources at dist=0
   simultaneously. Use for Rotting Oranges, 01 Matrix, etc.

10. IMPLICIT GRAPHS (like Word Ladder):
    Nodes are strings/states, edges are generated on the fly.
    BFS still applies — just generate neighbors dynamically.
```

---

## 📖 Resources

| Resource | Type | Notes |
|---|---|---|
| [NeetCode Graph Playlist](https://neetcode.io) | Video | JS solutions available |
| [LeetCode Graph Tag](https://leetcode.com/tag/graph/) | Problems | Set language to JavaScript |
| [CP-Algorithms](https://cp-algorithms.com/graph/) | Theory | Deep algorithmic detail |
| [MDN — Map & Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) | Docs | Core JS data structures |
| [Visualgo](https://visualgo.net/en/graphds) | Visual | Interactive graph animations |
| [William Fiset — Graph Theory (YouTube)](https://www.youtube.com/c/WilliamFiset-videos) | Video | Theory + code |
| [JavaScript Algorithms GitHub](https://github.com/trekhleb/javascript-algorithms) | Code | Reference implementations |

---

*Last updated: May 2026 | Author: Akash | Language: JavaScript (ES2022+) | Version: 1.0*

> ⭐ **Star this guide** if you found it helpful. Happy Coding in JavaScript! 🚀
