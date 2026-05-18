# 📊 Graph Data Structures & Algorithms in Kotlin — Complete Tutorial & Roadmap

> **A production-grade, interview-ready deep dive into Graphs using idiomatic Kotlin.**  
> Covers theory, implementation, patterns, problems, and a structured learning roadmap.

---

## 📚 Table of Contents

1. [What is a Graph?](#1-what-is-a-graph)
2. [Graph Terminology](#2-graph-terminology)
3. [Types of Graphs](#3-types-of-graphs)
4. [Graph Representations in Kotlin](#4-graph-representations-in-kotlin)
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
> Internet routing, dependency trees, build systems.

### Mathematical Definition

```
G = (V, E)

where:
  V = set of vertices  → {0, 1, 2, 3, 4}
  E = set of edges     → {(0,1), (0,2), (0,3), (1,4), (2,4)}
```

### Why Kotlin for Graphs?
- `ArrayDeque` replaces Java's `LinkedList` for BFS queues
- `PriorityQueue` with lambda comparators is clean and concise
- `data class` makes edge/node models expressive
- `mutableListOf`, `mutableMapOf`, `mutableSetOf` are idiomatic
- Null safety prevents common visited-check bugs

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

## 4. Graph Representations in Kotlin

### 4.1 Adjacency Matrix

```
Graph:  0─1, 0─2, 1─2, 2─3

     0  1  2  3
  0 [0, 1, 1, 0]
  1 [1, 0, 1, 0]
  2 [1, 1, 0, 1]
  3 [0, 0, 1, 0]
```

```kotlin
class GraphMatrix(private val v: Int) {
    private val matrix = Array(v) { IntArray(v) }

    fun addEdge(u: Int, v: Int, weight: Int = 1) {
        matrix[u][v] = weight
        matrix[v][u] = weight  // Remove for directed graph
    }

    fun addDirectedEdge(u: Int, v: Int, weight: Int = 1) {
        matrix[u][v] = weight
    }

    fun hasEdge(u: Int, v: Int): Boolean = matrix[u][v] != 0

    fun neighbors(u: Int): List<Int> =
        (0 until v).filter { matrix[u][it] != 0 }

    fun display() {
        matrix.forEachIndexed { i, row ->
            println("$i → ${row.toList()}")
        }
    }
}

// Usage
fun main() {
    val g = GraphMatrix(4)
    g.addEdge(0, 1)
    g.addEdge(0, 2)
    g.addEdge(1, 2)
    g.addEdge(2, 3)
    g.display()
    println("Has edge (0,1): ${g.hasEdge(0, 1)}")
}
```

**When to use:** Dense graphs (E ≈ V²), O(1) edge lookup needed

| Operation | Time | Space |
|---|---|---|
| Add Edge | O(1) | O(V²) |
| Check Edge | O(1) | |
| Find Neighbors | O(V) | |

---

### 4.2 Adjacency List ⭐ (Most Common in Kotlin)

```
Graph:  0─1, 0─2, 1─2, 2─3

0: [1, 2]
1: [0, 2]
2: [0, 1, 3]
3: [2]
```

```kotlin
// Simple unweighted graph
class Graph(private val v: Int) {
    val adj: MutableMap<Int, MutableList<Int>> =
        mutableMapOf<Int, MutableList<Int>>().apply {
            (0 until v).forEach { put(it, mutableListOf()) }
        }

    fun addEdge(u: Int, v: Int) {
        adj[u]?.add(v)
        adj[v]?.add(u)  // Remove for directed graph
    }

    fun addDirectedEdge(u: Int, v: Int) {
        adj[u]?.add(v)
    }

    fun neighbors(u: Int): List<Int> = adj[u] ?: emptyList()

    fun display() {
        adj.forEach { (node, neighbors) ->
            println("$node → $neighbors")
        }
    }
}


// Weighted graph using data class for edges
data class Edge(val to: Int, val weight: Int)

class WeightedGraph(private val v: Int) {
    val adj: MutableMap<Int, MutableList<Edge>> =
        (0 until v).associateWith { mutableListOf<Edge>() }.toMutableMap()

    fun addEdge(u: Int, v: Int, weight: Int) {
        adj[u]?.add(Edge(v, weight))
        adj[v]?.add(Edge(u, weight))  // Remove for directed
    }

    fun addDirectedEdge(u: Int, v: Int, weight: Int) {
        adj[u]?.add(Edge(v, weight))
    }

    fun neighbors(u: Int): List<Edge> = adj[u] ?: emptyList()
}


// Generic graph using defaultdict-style approach
fun buildGraph(edges: Array<IntArray>, directed: Boolean = false): Map<Int, MutableList<Int>> {
    val graph = mutableMapOf<Int, MutableList<Int>>()
    for ((u, v) in edges) {
        graph.getOrPut(u) { mutableListOf() }.add(v)
        if (!directed) graph.getOrPut(v) { mutableListOf() }.add(u)
    }
    return graph
}

// Usage
fun main() {
    val g = Graph(4)
    g.addEdge(0, 1)
    g.addEdge(0, 2)
    g.addEdge(1, 2)
    g.addEdge(2, 3)
    g.display()
}
```

| Operation | Time | Space |
|---|---|---|
| Add Edge | O(1) | O(V + E) |
| Check Edge | O(degree) | |
| Find Neighbors | O(degree) | |

---

### 4.3 Edge List

```kotlin
// Simple edge list
data class EdgeTriple(val u: Int, val v: Int, val weight: Int)

val edges = listOf(
    EdgeTriple(0, 1, 5),
    EdgeTriple(0, 2, 3),
    EdgeTriple(1, 2, 2),
    EdgeTriple(2, 3, 7)
)

// Sort by weight (for Kruskal's)
val sortedEdges = edges.sortedBy { it.weight }
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

**Idea:** Explore level by level using a **Queue (ArrayDeque in Kotlin)**

```
Graph:       BFS from node 0:
  0          Level 0: [0]
 /|\         Level 1: [1, 2, 3]
1 2 3        Level 2: [4, 5]
|   |
4   5
```

```kotlin
// Basic BFS
fun bfs(graph: Map<Int, List<Int>>, start: Int): List<Int> {
    val visited = mutableSetOf<Int>()
    val queue = ArrayDeque<Int>()
    val order = mutableListOf<Int>()

    queue.add(start)
    visited.add(start)

    while (queue.isNotEmpty()) {
        val node = queue.removeFirst()
        order.add(node)

        for (neighbor in graph[node] ?: emptyList()) {
            if (neighbor !in visited) {
                visited.add(neighbor)
                queue.add(neighbor)
            }
        }
    }
    return order
}


// BFS with level tracking
fun bfsLevels(graph: Map<Int, List<Int>>, start: Int): Map<Int, Int> {
    val visited = mutableSetOf(start)
    val queue = ArrayDeque<Pair<Int, Int>>()  // (node, level)
    val levels = mutableMapOf<Int, Int>()

    queue.add(Pair(start, 0))

    while (queue.isNotEmpty()) {
        val (node, level) = queue.removeFirst()
        levels[node] = level

        for (neighbor in graph[node] ?: emptyList()) {
            if (neighbor !in visited) {
                visited.add(neighbor)
                queue.add(Pair(neighbor, level + 1))
            }
        }
    }
    return levels
}


// BFS Shortest Path (unweighted)
fun bfsShortestPath(graph: Map<Int, List<Int>>, start: Int, end: Int): List<Int> {
    val visited = mutableSetOf(start)
    val queue = ArrayDeque<Pair<Int, List<Int>>>()
    queue.add(Pair(start, listOf(start)))

    while (queue.isNotEmpty()) {
        val (node, path) = queue.removeFirst()

        if (node == end) return path

        for (neighbor in graph[node] ?: emptyList()) {
            if (neighbor !in visited) {
                visited.add(neighbor)
                queue.add(Pair(neighbor, path + neighbor))
            }
        }
    }
    return emptyList()  // No path found
}
```

**Time:** O(V + E) | **Space:** O(V)

> 💡 **Kotlin tip:** Always use `ArrayDeque` (not `LinkedList`) for BFS queues — it's the idiomatic, faster option in Kotlin.

---

### 5.2 Depth-First Search (DFS)

```kotlin
// DFS - Recursive
fun dfsRecursive(
    graph: Map<Int, List<Int>>,
    node: Int,
    visited: MutableSet<Int> = mutableSetOf()
): List<Int> {
    val result = mutableListOf<Int>()
    visited.add(node)
    result.add(node)

    for (neighbor in graph[node] ?: emptyList()) {
        if (neighbor !in visited) {
            result.addAll(dfsRecursive(graph, neighbor, visited))
        }
    }
    return result
}


// DFS - Iterative (using explicit stack with ArrayDeque)
fun dfsIterative(graph: Map<Int, List<Int>>, start: Int): List<Int> {
    val visited = mutableSetOf<Int>()
    val stack = ArrayDeque<Int>()
    val order = mutableListOf<Int>()

    stack.addLast(start)

    while (stack.isNotEmpty()) {
        val node = stack.removeLast()
        if (node !in visited) {
            visited.add(node)
            order.add(node)
            // Add neighbors in reverse for consistent ordering
            for (neighbor in (graph[node] ?: emptyList()).reversed()) {
                if (neighbor !in visited) stack.addLast(neighbor)
            }
        }
    }
    return order
}


// DFS with pre/post timestamps
fun dfsTimestamps(graph: Map<Int, List<Int>>, start: Int): Pair<Map<Int, Int>, Map<Int, Int>> {
    val visited = mutableSetOf<Int>()
    val pre = mutableMapOf<Int, Int>()
    val post = mutableMapOf<Int, Int>()
    var timer = 0

    fun dfs(node: Int) {
        visited.add(node)
        pre[node] = ++timer

        for (neighbor in graph[node] ?: emptyList()) {
            if (neighbor !in visited) dfs(neighbor)
        }
        post[node] = ++timer
    }

    dfs(start)
    return Pair(pre, post)
}


// DFS across all components (handles disconnected graphs)
fun dfsAll(graph: Map<Int, List<Int>>, v: Int): List<Int> {
    val visited = mutableSetOf<Int>()
    val result = mutableListOf<Int>()

    fun dfs(node: Int) {
        visited.add(node)
        result.add(node)
        for (neighbor in graph[node] ?: emptyList()) {
            if (neighbor !in visited) dfs(neighbor)
        }
    }

    for (node in 0 until v) {
        if (node !in visited) dfs(node)
    }
    return result
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
│ Multi-source BFS   │ DFS tree properties    │
│ problems           │ (bridges, articulation)│
└────────────────────┴────────────────────────┘
```

---

## 6. Cycle Detection

### 6.1 Undirected Graph — DFS

```kotlin
fun hasCycleUndirected(graph: Map<Int, List<Int>>, v: Int): Boolean {
    val visited = mutableSetOf<Int>()

    fun dfs(node: Int, parent: Int): Boolean {
        visited.add(node)

        for (neighbor in graph[node] ?: emptyList()) {
            when {
                neighbor !in visited -> if (dfs(neighbor, node)) return true
                neighbor != parent   -> return true  // Back edge → cycle!
            }
        }
        return false
    }

    return (0 until v).any { it !in visited && dfs(it, -1) }
}
```

### 6.2 Undirected Graph — DSU

```kotlin
fun hasCycleDSU(edges: List<Triple<Int, Int, Int>>, v: Int): Boolean {
    val parent = IntArray(v) { it }
    val rank   = IntArray(v) { 0 }

    fun find(x: Int): Int {
        if (parent[x] != x) parent[x] = find(parent[x])  // Path compression
        return parent[x]
    }

    fun union(x: Int, y: Int): Boolean {
        val px = find(x); val py = find(y)
        if (px == py) return false  // Cycle!
        if (rank[px] < rank[py]) parent[px] = py
        else if (rank[px] > rank[py]) parent[py] = px
        else { parent[py] = px; rank[px]++ }
        return true
    }

    return edges.any { (u, v, _) -> !union(u, v) }
}
```

### 6.3 Directed Graph — 3-Color DFS

**Key insight:** Use 3 states:
- `0 = WHITE`: Not visited
- `1 = GRAY`: In current DFS stack (in-progress)
- `2 = BLACK`: Fully processed

```kotlin
fun hasCycleDirected(graph: Map<Int, List<Int>>, v: Int): Boolean {
    val color = IntArray(v) { 0 }  // 0=white, 1=gray, 2=black

    fun dfs(node: Int): Boolean {
        color[node] = 1  // Mark gray

        for (neighbor in graph[node] ?: emptyList()) {
            when (color[neighbor]) {
                1 -> return true   // Back edge → cycle!
                0 -> if (dfs(neighbor)) return true
            }
        }

        color[node] = 2  // Mark black (done)
        return false
    }

    return (0 until v).any { color[it] == 0 && dfs(it) }
}
```

---

## 7. Topological Sort

> **Only applicable to DAGs (Directed Acyclic Graphs)**  
> Ordering such that for every directed edge u→v, u appears before v

### 7.1 DFS-based Topological Sort

```kotlin
fun topoSortDFS(graph: Map<Int, List<Int>>, v: Int): List<Int> {
    val visited = mutableSetOf<Int>()
    val stack = ArrayDeque<Int>()

    fun dfs(node: Int) {
        visited.add(node)
        for (neighbor in graph[node] ?: emptyList()) {
            if (neighbor !in visited) dfs(neighbor)
        }
        stack.addLast(node)  // Push AFTER all neighbors are processed
    }

    for (node in 0 until v) {
        if (node !in visited) dfs(node)
    }

    return stack.reversed()  // Reverse for topological order
}
```

### 7.2 Kahn's BFS Algorithm ⭐ (Preferred for interviews)

**Key insight:** Repeatedly remove nodes with in-degree 0

```kotlin
fun topoSortKahn(graph: Map<Int, List<Int>>, v: Int): List<Int> {
    val inDegree = IntArray(v) { 0 }

    // Calculate in-degrees
    for (u in 0 until v) {
        for (nbr in graph[u] ?: emptyList()) inDegree[nbr]++
    }

    // Start with all 0-in-degree nodes
    val queue = ArrayDeque<Int>()
    for (node in 0 until v) {
        if (inDegree[node] == 0) queue.add(node)
    }

    val result = mutableListOf<Int>()

    while (queue.isNotEmpty()) {
        val node = queue.removeFirst()
        result.add(node)

        for (neighbor in graph[node] ?: emptyList()) {
            inDegree[neighbor]--
            if (inDegree[neighbor] == 0) queue.add(neighbor)
        }
    }

    // If result has fewer nodes, there's a cycle
    return if (result.size == v) result else emptyList()
}


// Real-world usage: Course Schedule
fun canFinish(numCourses: Int, prerequisites: Array<IntArray>): Boolean {
    val graph = mutableMapOf<Int, MutableList<Int>>()
    val inDegree = IntArray(numCourses)

    for ((course, prereq) in prerequisites) {
        graph.getOrPut(prereq) { mutableListOf() }.add(course)
        inDegree[course]++
    }

    val queue = ArrayDeque<Int>()
    for (c in 0 until numCourses) {
        if (inDegree[c] == 0) queue.add(c)
    }

    var completed = 0
    while (queue.isNotEmpty()) {
        val course = queue.removeFirst()
        completed++
        for (next in graph[course] ?: emptyList()) {
            if (--inDegree[next] == 0) queue.add(next)
        }
    }

    return completed == numCourses
}
```

### 7.3 Topo Sort — Step-by-Step Example

```
Tasks: A→C, B→C, C→D, C→E

In-degrees: A=0, B=0, C=2, D=1, E=1

Step 1: Queue=[A,B]
        Process A → queue=[B,C]  (C's in-degree: 2→1)
Step 2: Process B → queue=[C]   (C's in-degree: 1→0)
Step 3: Process C → queue=[D,E]
Step 4: Process D, E

Result: A → B → C → D → E  ✅ valid topological order
```

---

## 8. Shortest Path Algorithms

### 8.1 BFS — Unweighted Shortest Path

Already covered in Section 5.1. Time: O(V + E)

---

### 8.2 Dijkstra's Algorithm ⭐ (Non-negative weights)

**Idea:** Greedily pick the closest unvisited node using a **PriorityQueue**

```kotlin
import java.util.PriorityQueue

fun dijkstra(graph: Map<Int, List<Pair<Int, Int>>>, start: Int, v: Int): IntArray {
    /**
     * graph: Map<node, List<Pair<neighbor, weight>>>
     * Returns: dist[] = shortest distances from start
     */
    val dist = IntArray(v) { Int.MAX_VALUE }
    dist[start] = 0

    // Min-heap: (distance, node)
    val minHeap = PriorityQueue<Pair<Int, Int>>(compareBy { it.first })
    minHeap.add(Pair(0, start))

    while (minHeap.isNotEmpty()) {
        val (d, u) = minHeap.poll()

        if (d > dist[u]) continue  // Stale entry

        for ((v, weight) in graph[u] ?: emptyList()) {
            val newDist = dist[u] + weight
            if (newDist < dist[v]) {
                dist[v] = newDist
                minHeap.add(Pair(newDist, v))
            }
        }
    }
    return dist
}


// Dijkstra with path reconstruction
fun dijkstraWithPath(
    graph: Map<Int, List<Pair<Int, Int>>>,
    start: Int, end: Int, v: Int
): Pair<Int, List<Int>> {
    val dist = IntArray(v) { Int.MAX_VALUE }
    val prev = IntArray(v) { -1 }
    dist[start] = 0

    val minHeap = PriorityQueue<Pair<Int, Int>>(compareBy { it.first })
    minHeap.add(Pair(0, start))

    while (minHeap.isNotEmpty()) {
        val (d, u) = minHeap.poll()
        if (d > dist[u]) continue
        if (u == end) break

        for ((neighbor, weight) in graph[u] ?: emptyList()) {
            if (dist[u] + weight < dist[neighbor]) {
                dist[neighbor] = dist[u] + weight
                prev[neighbor] = u
                minHeap.add(Pair(dist[neighbor], neighbor))
            }
        }
    }

    // Reconstruct path
    val path = mutableListOf<Int>()
    var node = end
    while (node != -1) {
        path.add(node)
        node = prev[node]
    }
    return Pair(dist[end], path.reversed())
}


// Dijkstra on 2D grid (Path with Minimum Effort pattern)
fun dijkstraGrid(heights: Array<IntArray>): Int {
    val rows = heights.size
    val cols = heights[0].size
    val dist = Array(rows) { IntArray(cols) { Int.MAX_VALUE } }
    dist[0][0] = 0

    val minHeap = PriorityQueue<Triple<Int, Int, Int>>(compareBy { it.first })
    minHeap.add(Triple(0, 0, 0))  // (effort, row, col)

    val dirs = arrayOf(intArrayOf(0,1), intArrayOf(0,-1), intArrayOf(1,0), intArrayOf(-1,0))

    while (minHeap.isNotEmpty()) {
        val (effort, r, c) = minHeap.poll()
        if (r == rows - 1 && c == cols - 1) return effort
        if (effort > dist[r][c]) continue

        for ((dr, dc) in dirs) {
            val nr = r + dr; val nc = c + dc
            if (nr in 0 until rows && nc in 0 until cols) {
                val newEffort = maxOf(effort, Math.abs(heights[nr][nc] - heights[r][c]))
                if (newEffort < dist[nr][nc]) {
                    dist[nr][nc] = newEffort
                    minHeap.add(Triple(newEffort, nr, nc))
                }
            }
        }
    }
    return dist[rows-1][cols-1]
}
```

**Time:** O((V + E) log V) | **Space:** O(V)  
**Constraint:** ❌ Does NOT work with **negative weights**

---

### 8.3 Bellman-Ford Algorithm (Handles negative weights)

```kotlin
fun bellmanFord(edges: List<Triple<Int, Int, Int>>, v: Int, start: Int): IntArray? {
    /**
     * edges: List<Triple<u, v, weight>>
     * Returns: dist[] or null if negative cycle exists
     */
    val dist = IntArray(v) { Int.MAX_VALUE }
    dist[start] = 0

    // Relax all edges V-1 times
    repeat(v - 1) {
        for ((u, nbr, w) in edges) {
            if (dist[u] != Int.MAX_VALUE && dist[u] + w < dist[nbr]) {
                dist[nbr] = dist[u] + w
            }
        }
    }

    // V-th relaxation check: if any edge still relaxes, there's a negative cycle
    for ((u, nbr, w) in edges) {
        if (dist[u] != Int.MAX_VALUE && dist[u] + w < dist[nbr]) {
            return null  // Negative cycle detected
        }
    }
    return dist
}
```

**Time:** O(V × E) | **Space:** O(V)

---

### 8.4 Floyd-Warshall (All-pairs shortest path)

```kotlin
fun floydWarshall(v: Int, edges: List<Triple<Int, Int, Int>>): Array<IntArray>? {
    val INF = Int.MAX_VALUE / 2  // Avoid overflow on addition
    val dist = Array(v) { i -> IntArray(v) { j -> if (i == j) 0 else INF } }

    for ((u, nbr, w) in edges) {
        dist[u][nbr] = w
        dist[nbr][u] = w  // Remove for directed graph
    }

    // DP: try every node k as intermediate
    for (k in 0 until v) {
        for (i in 0 until v) {
            for (j in 0 until v) {
                if (dist[i][k] != INF && dist[k][j] != INF) {
                    dist[i][j] = minOf(dist[i][j], dist[i][k] + dist[k][j])
                }
            }
        }
    }

    // Check for negative cycles
    for (i in 0 until v) {
        if (dist[i][i] < 0) return null
    }
    return dist
}
```

**Time:** O(V³) | **Space:** O(V²)

---

### 8.5 Shortest Path Algorithm Selection Guide

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
│  DAG (directed, no cycles)?                         │
│    → Topological sort + relax O(V+E)                │
└─────────────────────────────────────────────────────┘
```

---

## 9. Minimum Spanning Tree (MST)

> An MST of a weighted undirected graph is a **spanning tree** (connects all V vertices) with **minimum total edge weight**. Always has exactly V-1 edges.

### 9.1 Kruskal's Algorithm

**Idea:** Sort edges by weight, add smallest edge that doesn't form a cycle (use DSU)

```kotlin
fun kruskal(v: Int, edges: List<Triple<Int, Int, Int>>): Pair<List<Triple<Int, Int, Int>>, Int> {
    val parent = IntArray(v) { it }
    val rank   = IntArray(v) { 0 }

    fun find(x: Int): Int {
        if (parent[x] != x) parent[x] = find(parent[x])
        return parent[x]
    }

    fun union(x: Int, y: Int): Boolean {
        val px = find(x); val py = find(y)
        if (px == py) return false
        when {
            rank[px] < rank[py] -> parent[px] = py
            rank[px] > rank[py] -> parent[py] = px
            else -> { parent[py] = px; rank[px]++ }
        }
        return true
    }

    val sortedEdges = edges.sortedBy { it.third }  // Sort by weight
    val mstEdges = mutableListOf<Triple<Int, Int, Int>>()
    var totalWeight = 0

    for (edge in sortedEdges) {
        val (u, v, w) = edge
        if (union(u, v)) {
            mstEdges.add(edge)
            totalWeight += w
            if (mstEdges.size == v - 1) break
        }
    }

    return Pair(mstEdges, totalWeight)
}
```

**Time:** O(E log E) | **Space:** O(V)

---

### 9.2 Prim's Algorithm

**Idea:** Grow MST from a start vertex — always pick cheapest edge to an unvisited vertex

```kotlin
import java.util.PriorityQueue

fun prim(graph: Map<Int, List<Pair<Int, Int>>>, v: Int, start: Int = 0): Pair<Int, List<Triple<Int, Int, Int>>> {
    /**
     * graph: Map<node, List<Pair<neighbor, weight>>>
     */
    val inMST = BooleanArray(v)
    // (weight, node, parent)
    val minHeap = PriorityQueue<Triple<Int, Int, Int>>(compareBy { it.first })
    minHeap.add(Triple(0, start, -1))

    var totalWeight = 0
    val mstEdges = mutableListOf<Triple<Int, Int, Int>>()

    while (minHeap.isNotEmpty()) {
        val (weight, u, parent) = minHeap.poll()

        if (inMST[u]) continue
        inMST[u] = true
        totalWeight += weight

        if (parent != -1) mstEdges.add(Triple(parent, u, weight))

        for ((neighbor, w) in graph[u] ?: emptyList()) {
            if (!inMST[neighbor]) {
                minHeap.add(Triple(w, neighbor, u))
            }
        }
    }

    return Pair(totalWeight, mstEdges)
}
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

> Also called **Union-Find**. Efficiently tracks and merges connected components.

```kotlin
class DSU(private val n: Int) {
    private val parent = IntArray(n) { it }
    private val rank   = IntArray(n) { 0 }
    private val size   = IntArray(n) { 1 }
    var components = n
        private set

    fun find(x: Int): Int {
        if (parent[x] != x) parent[x] = find(parent[x])  // Path compression
        return parent[x]
    }

    // Returns true if union was performed (they were in different sets)
    fun union(x: Int, y: Int): Boolean {
        val px = find(x); val py = find(y)
        if (px == py) return false

        // Union by rank
        when {
            rank[px] < rank[py] -> { parent[px] = py; size[py] += size[px] }
            rank[px] > rank[py] -> { parent[py] = px; size[px] += size[py] }
            else -> { parent[py] = px; size[px] += size[py]; rank[px]++ }
        }
        components--
        return true
    }

    fun connected(x: Int, y: Int): Boolean = find(x) == find(y)

    fun componentSize(x: Int): Int = size[find(x)]
}


// Usage example
fun main() {
    val dsu = DSU(5)
    dsu.union(0, 1)
    dsu.union(1, 2)
    dsu.union(3, 4)

    println("Components: ${dsu.components}")        // 2
    println("0 and 2 connected: ${dsu.connected(0, 2)}")  // true
    println("0 and 3 connected: ${dsu.connected(0, 3)}")  // false
    println("Component size of 0: ${dsu.componentSize(0)}")  // 3
}


// Practical: Number of Operations to Make Network Connected (LC 1319)
fun makeConnected(n: Int, connections: Array<IntArray>): Int {
    if (connections.size < n - 1) return -1  // Not enough cables

    val dsu = DSU(n)
    for ((u, v) in connections) dsu.union(u, v)

    return dsu.components - 1  // Need to move (components - 1) cables
}
```

**Operations:** O(α(n)) ≈ O(1) amortized (inverse Ackermann)

---

## 11. Advanced Graph Algorithms

### 11.1 Connected Components

```kotlin
fun countComponents(graph: Map<Int, List<Int>>, v: Int): Int {
    val visited = mutableSetOf<Int>()
    var count = 0

    fun dfs(node: Int) {
        visited.add(node)
        for (neighbor in graph[node] ?: emptyList()) {
            if (neighbor !in visited) dfs(neighbor)
        }
    }

    for (node in 0 until v) {
        if (node !in visited) {
            dfs(node)
            count++
        }
    }
    return count
}
```

---

### 11.2 Bipartite Check (2-Coloring)

```kotlin
fun isBipartite(graph: Map<Int, List<Int>>, v: Int): Boolean {
    val color = IntArray(v) { -1 }

    for (start in 0 until v) {
        if (color[start] != -1) continue

        val queue = ArrayDeque<Int>()
        queue.add(start)
        color[start] = 0

        while (queue.isNotEmpty()) {
            val node = queue.removeFirst()
            for (neighbor in graph[node] ?: emptyList()) {
                when (color[neighbor]) {
                    -1 -> {
                        color[neighbor] = 1 - color[node]
                        queue.add(neighbor)
                    }
                    color[node] -> return false  // Same color = not bipartite
                }
            }
        }
    }
    return true
}
```

---

### 11.3 Bridges (Tarjan's Algorithm)

> **Bridge:** An edge whose removal disconnects the graph

```kotlin
fun findBridges(graph: Map<Int, List<Int>>, v: Int): List<Pair<Int, Int>> {
    val visited = BooleanArray(v)
    val disc    = IntArray(v)
    val low     = IntArray(v)
    val bridges = mutableListOf<Pair<Int, Int>>()
    var timer   = 0

    fun dfs(u: Int, parent: Int) {
        visited[u] = true
        disc[u] = low[u]  also { timer++ ; low[u] = timer; disc[u] = timer }

        // Corrected version:
        visited[u] = true
        disc[u] = timer
        low[u] = timer
        timer++

        for (neighbor in graph[u] ?: emptyList()) {
            when {
                !visited[neighbor] -> {
                    dfs(neighbor, u)
                    low[u] = minOf(low[u], low[neighbor])
                    if (low[neighbor] > disc[u]) bridges.add(Pair(u, neighbor))
                }
                neighbor != parent -> low[u] = minOf(low[u], disc[neighbor])
            }
        }
    }

    for (node in 0 until v) {
        if (!visited[node]) dfs(node, -1)
    }
    return bridges
}


// Clean idiomatic Kotlin version
fun criticalConnections(n: Int, connections: List<List<Int>>): List<List<Int>> {
    val graph = Array(n) { mutableListOf<Int>() }
    for ((u, v) in connections) {
        graph[u].add(v); graph[v].add(u)
    }

    val disc = IntArray(n) { -1 }
    val low  = IntArray(n)
    val result = mutableListOf<List<Int>>()
    var timer = 0

    fun dfs(u: Int, parent: Int) {
        disc[u] = timer; low[u] = timer; timer++

        for (v in graph[u]) {
            if (v == parent) continue
            if (disc[v] == -1) {
                dfs(v, u)
                low[u] = minOf(low[u], low[v])
                if (low[v] > disc[u]) result.add(listOf(u, v))
            } else {
                low[u] = minOf(low[u], disc[v])
            }
        }
    }

    for (i in 0 until n) if (disc[i] == -1) dfs(i, -1)
    return result
}
```

---

### 11.4 Strongly Connected Components — Kosaraju's Algorithm

```kotlin
fun kosarajuSCC(graph: Map<Int, List<Int>>, v: Int): List<List<Int>> {
    val visited = BooleanArray(v)
    val stack = ArrayDeque<Int>()

    // Step 1: DFS on original graph, push to stack by finish time
    fun dfs1(node: Int) {
        visited[node] = true
        for (neighbor in graph[node] ?: emptyList()) {
            if (!visited[neighbor]) dfs1(neighbor)
        }
        stack.addLast(node)
    }

    for (node in 0 until v) {
        if (!visited[node]) dfs1(node)
    }

    // Step 2: Transpose the graph
    val transposed = mutableMapOf<Int, MutableList<Int>>()
    for (node in 0 until v) {
        for (neighbor in graph[node] ?: emptyList()) {
            transposed.getOrPut(neighbor) { mutableListOf() }.add(node)
        }
    }

    // Step 3: DFS on transposed graph in reverse finish order
    val visitedT = BooleanArray(v)
    val sccs = mutableListOf<List<Int>>()

    fun dfs2(node: Int, component: MutableList<Int>) {
        visitedT[node] = true
        component.add(node)
        for (neighbor in transposed[node] ?: emptyList()) {
            if (!visitedT[neighbor]) dfs2(neighbor, component)
        }
    }

    while (stack.isNotEmpty()) {
        val node = stack.removeLast()
        if (!visitedT[node]) {
            val component = mutableListOf<Int>()
            dfs2(node, component)
            sccs.add(component)
        }
    }

    return sccs
}
```

---

### 11.5 Flood Fill (Matrix as Graph)

```kotlin
// DFS Flood Fill
fun floodFill(image: Array<IntArray>, sr: Int, sc: Int, color: Int): Array<IntArray> {
    val oldColor = image[sr][sc]
    if (oldColor == color) return image

    val rows = image.size; val cols = image[0].size

    fun dfs(r: Int, c: Int) {
        if (r !in 0 until rows || c !in 0 until cols) return
        if (image[r][c] != oldColor) return

        image[r][c] = color
        dfs(r + 1, c); dfs(r - 1, c)
        dfs(r, c + 1); dfs(r, c - 1)
    }

    dfs(sr, sc)
    return image
}


// BFS Flood Fill (avoids stack overflow on large grids)
fun floodFillBFS(image: Array<IntArray>, sr: Int, sc: Int, color: Int): Array<IntArray> {
    val oldColor = image[sr][sc]
    if (oldColor == color) return image

    val rows = image.size; val cols = image[0].size
    val queue = ArrayDeque<Pair<Int, Int>>()
    queue.add(Pair(sr, sc))
    image[sr][sc] = color

    val dirs = arrayOf(intArrayOf(1,0), intArrayOf(-1,0), intArrayOf(0,1), intArrayOf(0,-1))

    while (queue.isNotEmpty()) {
        val (r, c) = queue.removeFirst()
        for ((dr, dc) in dirs) {
            val nr = r + dr; val nc = c + dc
            if (nr in 0 until rows && nc in 0 until cols && image[nr][nc] == oldColor) {
                image[nr][nc] = color
                queue.add(Pair(nr, nc))
            }
        }
    }
    return image
}
```

---

### 11.6 Multi-Source BFS

```kotlin
// Example: Rotting Oranges (LC 994) / 01 Matrix (LC 542)
fun multiSourceBFS(grid: Array<IntArray>): Int {
    val rows = grid.size; val cols = grid[0].size
    val dist = Array(rows) { IntArray(cols) { Int.MAX_VALUE } }
    val queue = ArrayDeque<Pair<Int, Int>>()

    // Enqueue ALL sources simultaneously
    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (grid[r][c] == 0) {  // Source condition
                dist[r][c] = 0
                queue.add(Pair(r, c))
            }
        }
    }

    val dirs = arrayOf(intArrayOf(1,0), intArrayOf(-1,0), intArrayOf(0,1), intArrayOf(0,-1))

    while (queue.isNotEmpty()) {
        val (r, c) = queue.removeFirst()
        for ((dr, dc) in dirs) {
            val nr = r + dr; val nc = c + dc
            if (nr in 0 until rows && nc in 0 until cols) {
                if (dist[r][c] + 1 < dist[nr][nc]) {
                    dist[nr][nc] = dist[r][c] + 1
                    queue.add(Pair(nr, nc))
                }
            }
        }
    }
    return dist
}
```

---

## 12. Graph Patterns for Interviews

### Pattern 1: Grid as Graph

```kotlin
// 4-directional movement
val dirs4 = arrayOf(intArrayOf(0,1), intArrayOf(0,-1), intArrayOf(1,0), intArrayOf(-1,0))

// 8-directional movement
val dirs8 = arrayOf(
    intArrayOf(0,1), intArrayOf(0,-1), intArrayOf(1,0), intArrayOf(-1,0),
    intArrayOf(1,1), intArrayOf(1,-1), intArrayOf(-1,1), intArrayOf(-1,-1)
)

// Standard bounds check
fun inBounds(r: Int, c: Int, rows: Int, cols: Int) = r in 0 until rows && c in 0 until cols

// Standard grid BFS template
fun gridBFS(grid: Array<IntArray>, sr: Int, sc: Int): Int {
    val rows = grid.size; val cols = grid[0].size
    val visited = Array(rows) { BooleanArray(cols) }
    val queue = ArrayDeque<Pair<Int, Int>>()

    queue.add(Pair(sr, sc))
    visited[sr][sc] = true
    var result = 0

    while (queue.isNotEmpty()) {
        val size = queue.size
        repeat(size) {
            val (r, c) = queue.removeFirst()
            // Process cell here
            for ((dr, dc) in dirs4) {
                val nr = r + dr; val nc = c + dc
                if (inBounds(nr, nc, rows, cols) && !visited[nr][nc] && grid[nr][nc] == 1) {
                    visited[nr][nc] = true
                    queue.add(Pair(nr, nc))
                }
            }
        }
        result++  // Increment level/step
    }
    return result
}
```

### Pattern 2: Multi-Source BFS Template

```kotlin
// Initialize all sources at dist=0, then standard BFS
fun multiSourceTemplate(grid: Array<IntArray>): Array<IntArray> {
    val rows = grid.size; val cols = grid[0].size
    val dist = Array(rows) { IntArray(cols) { Int.MAX_VALUE } }
    val queue = ArrayDeque<Pair<Int, Int>>()

    // Step 1: Enqueue ALL sources
    for (r in 0 until rows) for (c in 0 until cols) {
        if (isSource(grid[r][c])) {
            dist[r][c] = 0
            queue.add(Pair(r, c))
        }
    }

    // Step 2: Standard BFS from all sources simultaneously
    while (queue.isNotEmpty()) {
        val (r, c) = queue.removeFirst()
        for ((dr, dc) in dirs4) {
            val nr = r + dr; val nc = c + dc
            if (inBounds(nr, nc, rows, cols) && dist[r][c] + 1 < dist[nr][nc]) {
                dist[nr][nc] = dist[r][c] + 1
                queue.add(Pair(nr, nc))
            }
        }
    }
    return dist
}

fun isSource(cell: Int): Boolean = cell == 0  // Customize per problem
```

### Pattern 3: Dijkstra on Grid

```kotlin
// Use when cells have costs and you need minimum cost path
fun dijkstraGridTemplate(grid: Array<IntArray>): Int {
    val rows = grid.size; val cols = grid[0].size
    val dist = Array(rows) { IntArray(cols) { Int.MAX_VALUE } }
    dist[0][0] = 0

    // (cost, row, col)
    val heap = PriorityQueue<Triple<Int,Int,Int>>(compareBy { it.first })
    heap.add(Triple(0, 0, 0))

    while (heap.isNotEmpty()) {
        val (cost, r, c) = heap.poll()
        if (r == rows-1 && c == cols-1) return cost
        if (cost > dist[r][c]) continue

        for ((dr, dc) in dirs4) {
            val nr = r+dr; val nc = c+dc
            if (inBounds(nr, nc, rows, cols)) {
                val newCost = cost + grid[nr][nc]  // Or maxOf, abs, etc.
                if (newCost < dist[nr][nc]) {
                    dist[nr][nc] = newCost
                    heap.add(Triple(newCost, nr, nc))
                }
            }
        }
    }
    return dist[rows-1][cols-1]
}
```

### Pattern 4: Topo Sort for Dependencies

```kotlin
// Standard Kahn's template for dependency/prerequisite problems
fun topoTemplate(n: Int, deps: Array<IntArray>): List<Int> {
    val graph = Array(n) { mutableListOf<Int>() }
    val inDegree = IntArray(n)

    for ((task, prereq) in deps) {
        graph[prereq].add(task)
        inDegree[task]++
    }

    val queue = ArrayDeque<Int>()
    for (i in 0 until n) if (inDegree[i] == 0) queue.add(i)

    val order = mutableListOf<Int>()
    while (queue.isNotEmpty()) {
        val node = queue.removeFirst()
        order.add(node)
        for (next in graph[node]) {
            if (--inDegree[next] == 0) queue.add(next)
        }
    }

    return if (order.size == n) order else emptyList()  // empty = cycle
}
```

### Pattern 5: DSU for Dynamic Connectivity

```kotlin
// Use DSU when incrementally adding connections and querying connectivity
fun dynamicConnectivity(n: Int, queries: Array<IntArray>): List<Int> {
    val dsu = DSU(n)
    val answers = mutableListOf<Int>()

    for ((u, v) in queries) {
        dsu.union(u, v)
        answers.add(dsu.components)
    }
    return answers
}
```

---

## 13. Problem Bank with Solutions

### 🟢 Easy → 🟡 Medium → 🔴 Hard

| # | Problem | Pattern | Algorithm |
|---|---|---|---|
| 200 | Number of Islands | Grid DFS | DFS + visited |
| 733 | Flood Fill | Grid DFS | DFS |
| 1971 | Path Exists in Graph | BFS/DFS | BFS |
| 133 | Clone Graph | BFS + HashMap | BFS |
| 695 | Max Area of Island | Grid DFS | DFS |
| 207 | Course Schedule | Cycle in DAG | Topo / DFS |
| 210 | Course Schedule II | Topo Order | Kahn's BFS |
| 547 | Number of Provinces | Connected Components | DFS/DSU |
| 994 | Rotting Oranges | Multi-source BFS | BFS |
| 542 | 01 Matrix | Multi-source BFS | BFS |
| 417 | Pacific Atlantic | Reverse DFS | DFS |
| 130 | Surrounded Regions | Border DFS | DFS |
| 684 | Redundant Connection | Cycle detection | DSU |
| 261 | Graph Valid Tree | Connected + Acyclic | DSU |
| 1319 | Make Network Connected | Components | DSU |
| 743 | Network Delay Time | Single-source SP | Dijkstra |
| 787 | Cheapest Flights K Stops | Dijkstra variant | BFS+DP |
| 1631 | Path with Min Effort | Dijkstra grid | Dijkstra |
| 1584 | Min Cost Connect Points | MST | Kruskal/Prim |
| 721 | Accounts Merge | Component groups | DSU |
| 1192 | Critical Connections | Bridges | Tarjan's |
| 269 | Alien Dictionary | Topo sort | Kahn's |
| 127 | Word Ladder | BFS shortest path | BFS |
| 126 | Word Ladder II | BFS + backtrack | BFS + DFS |

---

### ✅ Full Solution: Number of Islands (LC 200)

```kotlin
fun numIslands(grid: Array<CharArray>): Int {
    val rows = grid.size; val cols = grid[0].size
    var count = 0

    fun dfs(r: Int, c: Int) {
        if (r !in 0 until rows || c !in 0 until cols || grid[r][c] != '1') return
        grid[r][c] = '#'  // Mark visited
        dfs(r+1, c); dfs(r-1, c); dfs(r, c+1); dfs(r, c-1)
    }

    for (r in 0 until rows) {
        for (c in 0 until cols) {
            if (grid[r][c] == '1') {
                dfs(r, c)
                count++
            }
        }
    }
    return count
}
```

---

### ✅ Full Solution: Course Schedule II (LC 210)

```kotlin
fun findOrder(numCourses: Int, prerequisites: Array<IntArray>): IntArray {
    val graph = Array(numCourses) { mutableListOf<Int>() }
    val inDegree = IntArray(numCourses)

    for ((course, prereq) in prerequisites) {
        graph[prereq].add(course)
        inDegree[course]++
    }

    val queue = ArrayDeque<Int>()
    for (i in 0 until numCourses) {
        if (inDegree[i] == 0) queue.add(i)
    }

    val order = mutableListOf<Int>()
    while (queue.isNotEmpty()) {
        val course = queue.removeFirst()
        order.add(course)
        for (next in graph[course]) {
            if (--inDegree[next] == 0) queue.add(next)
        }
    }

    return if (order.size == numCourses) order.toIntArray() else intArrayOf()
}
```

---

### ✅ Full Solution: Network Delay Time (LC 743)

```kotlin
import java.util.PriorityQueue

fun networkDelayTime(times: Array<IntArray>, n: Int, k: Int): Int {
    val graph = mutableMapOf<Int, MutableList<Pair<Int, Int>>>()
    for ((u, v, w) in times) {
        graph.getOrPut(u) { mutableListOf() }.add(Pair(v, w))
    }

    val dist = IntArray(n + 1) { Int.MAX_VALUE }
    dist[k] = 0

    val heap = PriorityQueue<Pair<Int, Int>>(compareBy { it.first })
    heap.add(Pair(0, k))

    while (heap.isNotEmpty()) {
        val (d, u) = heap.poll()
        if (d > dist[u]) continue

        for ((v, w) in graph[u] ?: emptyList()) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w
                heap.add(Pair(dist[v], v))
            }
        }
    }

    val maxDist = dist.drop(1).max() ?: Int.MAX_VALUE
    return if (maxDist == Int.MAX_VALUE) -1 else maxDist
}
```

---

### ✅ Full Solution: Rotting Oranges (LC 994)

```kotlin
fun orangesRotting(grid: Array<IntArray>): Int {
    val rows = grid.size; val cols = grid[0].size
    val queue = ArrayDeque<Pair<Int, Int>>()
    var fresh = 0

    for (r in 0 until rows) for (c in 0 until cols) {
        when (grid[r][c]) {
            2 -> queue.add(Pair(r, c))
            1 -> fresh++
        }
    }

    if (fresh == 0) return 0

    val dirs = arrayOf(intArrayOf(1,0), intArrayOf(-1,0), intArrayOf(0,1), intArrayOf(0,-1))
    var minutes = 0

    while (queue.isNotEmpty() && fresh > 0) {
        repeat(queue.size) {
            val (r, c) = queue.removeFirst()
            for ((dr, dc) in dirs) {
                val nr = r + dr; val nc = c + dc
                if (nr in 0 until rows && nc in 0 until cols && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2
                    fresh--
                    queue.add(Pair(nr, nc))
                }
            }
        }
        minutes++
    }

    return if (fresh == 0) minutes else -1
}
```

---

### ✅ Full Solution: Min Cost to Connect All Points (LC 1584) — Prim's

```kotlin
import java.util.PriorityQueue
import kotlin.math.abs

fun minCostConnectPoints(points: Array<IntArray>): Int {
    val n = points.size
    val inMST = BooleanArray(n)
    val minCost = IntArray(n) { Int.MAX_VALUE }
    minCost[0] = 0

    var totalCost = 0

    repeat(n) {
        // Find minimum cost vertex not in MST
        val u = (0 until n).filter { !inMST[it] }.minByOrNull { minCost[it] }!!
        inMST[u] = true
        totalCost += minCost[u]

        for (v in 0 until n) {
            if (!inMST[v]) {
                val dist = abs(points[u][0] - points[v][0]) + abs(points[u][1] - points[v][1])
                if (dist < minCost[v]) minCost[v] = dist
            }
        }
    }
    return totalCost
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

### Kotlin-Specific Performance Notes

| Kotlin Class | Use Case | Notes |
|---|---|---|
| `ArrayDeque<T>` | BFS queue, DFS stack | Faster than `LinkedList` in JVM |
| `PriorityQueue<T>` | Dijkstra min-heap | Use `compareBy { it.first }` |
| `mutableSetOf<Int>()` | Visited set | O(1) average lookup |
| `IntArray(n) { it }` | DSU parent init | Compact, cache-friendly |
| `Array(n) { mutableListOf<Int>() }` | Adjacency list | Idiomatic Kotlin |
| `mutableMapOf<Int, MutableList<Int>>()` | Dynamic graphs | Flexible for sparse |

---

## 15. Learning Roadmap

```
╔═══════════════════════════════════════════════════════════════════╗
║           GRAPH DSA LEARNING ROADMAP — KOTLIN                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  PHASE 1 — FOUNDATIONS (Week 1-2)                                ║
║  ─────────────────────────────────                               ║
║  □ Graph types and terminology                                   ║
║  □ Build adjacency list with Kotlin idioms                       ║
║  □ BFS with ArrayDeque — level traversal + shortest path         ║
║  □ DFS recursive + iterative with ArrayDeque as stack            ║
║  □ Handle disconnected graphs (iterate all unvisited nodes)      ║
║  □ Problems: Number of Islands, Flood Fill, Clone Graph,         ║
║             Find Path Exists (LC 1971)                           ║
║                                                                   ║
║  PHASE 2 — INTERMEDIATE (Week 3-4)                               ║
║  ──────────────────────────────────                              ║
║  □ Cycle detection: undirected DFS and DSU                       ║
║  □ Cycle detection: directed graph (3-color)                     ║
║  □ Topological sort: DFS-stack + Kahn's BFS                      ║
║  □ Connected components + bipartite check                        ║
║  □ Build and use DSU with path compression + union by rank       ║
║  □ Problems: Course Schedule I & II, Redundant Connection,       ║
║             Graph Valid Tree, Rotting Oranges, 01 Matrix,        ║
║             Number of Provinces, Make Network Connected          ║
║                                                                   ║
║  PHASE 3 — SHORTEST PATHS (Week 5)                               ║
║  ─────────────────────────────────                               ║
║  □ Dijkstra with PriorityQueue + path reconstruction             ║
║  □ Dijkstra on 2D grids                                          ║
║  □ Bellman-Ford for negative weights                             ║
║  □ Floyd-Warshall all-pairs                                      ║
║  □ Multi-source BFS pattern                                      ║
║  □ Problems: Network Delay Time, Cheapest Flights,               ║
║             Path with Min Effort, Pacific Atlantic,              ║
║             Swim in Rising Water                                 ║
║                                                                   ║
║  PHASE 4 — MST & DSU DEEP DIVE (Week 6)                         ║
║  ──────────────────────────────────────                          ║
║  □ Kruskal's with edge sorting and DSU                           ║
║  □ Prim's with PriorityQueue                                     ║
║  □ Advanced DSU: component size, dynamic connectivity            ║
║  □ Problems: Min Cost Connect All Points, Kruskal MST,           ║
║             Accounts Merge, Optimize Water Distribution          ║
║                                                                   ║
║  PHASE 5 — ADVANCED (Week 7-8)                                   ║
║  ─────────────────────────────                                   ║
║  □ Bridges and articulation points (Tarjan's)                    ║
║  □ Strongly Connected Components (Kosaraju's / Tarjan's)         ║
║  □ Euler Path/Circuit                                            ║
║  □ Bidirectional BFS                                             ║
║  □ Problems: Critical Connections, Word Ladder I & II,           ║
║             Alien Dictionary, Sequence Reconstruction,           ║
║             Bus Routes                                           ║
║                                                                   ║
║  PHASE 6 — MASTERY (Ongoing)                                     ║
║  ─────────────────────────                                       ║
║  □ Review all patterns with Kotlin-idiomatic code                ║
║  □ Solve 3-5 problems per pattern                                ║
║  □ Aim for 60+ graph problems on LeetCode in Kotlin              ║
║  □ Profile and optimize: ArrayDeque vs LinkedList benchmarks     ║
║  □ Review NeetCode 150 / Blind 75 graph section                  ║
║                                                                   ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  RECOMMENDED PROBLEM SEQUENCE ON LEETCODE (Kotlin)               ║
║  ─────────────────────────────────────────────────               ║
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

2. KOTLIN-SPECIFIC defaults:
   → BFS queue   → ArrayDeque (NOT LinkedList)
   → DFS stack   → ArrayDeque with addLast / removeLast
   → Min-heap    → PriorityQueue(compareBy { it.first })
   → Visited     → mutableSetOf<Int>()
   → Adj list    → mutableMapOf with getOrPut

3. VISITED SET is essential — always track visited nodes
   to avoid infinite loops in BFS/DFS.

4. GRID problems = graph problems:
   Each cell is a node; adjacent cells are edges.
   Use dirs = arrayOf(intArrayOf(0,1),...) for clean direction handling.

5. SHORTEST PATH selection:
   Unweighted → BFS
   Non-negative weights → Dijkstra
   Negative weights → Bellman-Ford

6. CYCLE detection:
   Undirected → DFS (check back edge != parent) or DSU
   Directed → DFS with 3-color (0/1/2 state array)

7. TOPOLOGICAL SORT only works on DAGs.
   Kahn's BFS naturally detects cycles (result.size != V).

8. DSU is ideal for:
   → Dynamic connectivity queries
   → Kruskal's MST
   → Redundant connections
   → Merging/grouping problems (Accounts Merge)

9. MULTI-SOURCE BFS: enqueue ALL sources at dist=0
   simultaneously for problems like Rotting Oranges, 01 Matrix.

10. DATA CLASS for edges: use data class Edge(val to: Int, val weight: Int)
    for clean weighted graph representations in Kotlin.
```

---

## 📖 Resources

| Resource | Type | Notes |
|---|---|---|
| [NeetCode Graph Playlist](https://neetcode.io) | Video | Best structured Kotlin-compatible walkthrough |
| [LeetCode Graph Tag](https://leetcode.com/tag/graph/) | Problems | Set language to Kotlin |
| [CP-Algorithms](https://cp-algorithms.com/graph/) | Theory | Deep algorithmic detail |
| [Kotlin Docs — Collections](https://kotlinlang.org/docs/collections-overview.html) | Docs | ArrayDeque, PriorityQueue usage |
| [Visualgo](https://visualgo.net/en/graphds) | Visual | Interactive graph animations |
| [William Fiset — Graph Theory (YouTube)](https://www.youtube.com/c/WilliamFiset-videos) | Video | Theory + code (easily portable to Kotlin) |
| [Kotlin Playground](https://play.kotlinlang.org) | Tool | Test all code snippets online |

---

*Last updated: May 2026 | Author: Akash | Language: Kotlin | Version: 1.0*

> ⭐ **Star this guide** if you found it helpful. Happy Coding in Kotlin! 🎯
