# Graph DSA Algorithms — Complete Reference Guide


## Choosing the right algorithm — quick reference


The right choice depends on three questions: what do you need (path, tree, flow, components), what kind of edges (unweighted, non-negative, negative), and what scale (single pair, single source, all pairs).

<img width="1366" height="1160" alt="image" src="https://github.com/user-attachments/assets/dca45121-d736-4d41-b056-f7ead236c321" />

---
ipynb - https://colab.research.google.com/drive/1XWEyH4IKdn8jG7heJniwpZMd9kW6I0ZV#scrollTo=js_Cabnrdtxn

---
A language-agnostic, comprehensive reference covering every major graph algorithm: what it is, when to use it, why it works, and where it fits in the real world.

---

## Table of Contents

1. [Graph Representation](#1-graph-representation)
2. [Traversal Algorithms](#2-traversal-algorithms)
   - [BFS — Breadth-First Search](#bfs--breadth-first-search)
   - [DFS — Depth-First Search](#dfs--depth-first-search)
3. [Shortest Path Algorithms](#3-shortest-path-algorithms)
   - [Dijkstra's Algorithm](#dijkstras-algorithm)
   - [Bellman-Ford Algorithm](#bellman-ford-algorithm)
   - [Floyd-Warshall Algorithm](#floyd-warshall-algorithm)
   - [A* Algorithm](#a-algorithm)
4. [Minimum Spanning Tree](#4-minimum-spanning-tree-mst)
   - [Kruskal's Algorithm](#kruskals-algorithm)
   - [Prim's Algorithm](#prims-algorithm)
5. [Cycle Detection](#5-cycle-detection)
6. [Topological Sort](#6-topological-sort)
7. [Connected Components](#7-connected-components)
   - [BFS/DFS Component Labelling](#bfsdfs-component-labelling)
   - [Union-Find (DSU)](#union-find-disjoint-set-union--dsu)
   - [Strongly Connected Components (SCCs)](#strongly-connected-components-sccs)
8. [Network Flow](#8-network-flow)
9. [Bipartite Checking](#9-bipartite-checking)
10. [Eulerian and Hamiltonian Paths](#10-eulerian-and-hamiltonian-paths)
11. [Bridge and Articulation Point Detection](#11-bridge-and-articulation-point-detection)
12. [Shortest Path in Special Graph Types](#12-shortest-path-in-special-graph-types)
13. [Algorithm Comparison Tables](#13-algorithm-comparison-tables)
14. [Additional Points Worth Knowing](#14-additional-points-worth-knowing)

---

## 1. Graph Representation

Before any algorithm, you must choose how to store the graph. Two primary structures exist.

### Adjacency List

Each node stores a list of its neighbours (and edge weights if applicable).

```
Node 0 → [1, 2]
Node 1 → [0, 3]
Node 2 → [0, 3, 4]
Node 3 → [1, 2]
Node 4 → [2]
```

| Property     | Value                     |
|--------------|---------------------------|
| Space        | O(V + E)                  |
| Edge lookup  | O(degree of node)         |
| Best for     | Sparse graphs             |
| Used by      | BFS, DFS, Dijkstra, Prim  |

### Adjacency Matrix

A V×V matrix where `matrix[i][j]` = weight of edge from i to j (or 0/∞ if no edge).

```
     0    1    2    3    4
0  [ 0    1    1    0    0 ]
1  [ 1    0    0    1    0 ]
2  [ 1    0    0    1    1 ]
3  [ 0    1    1    0    0 ]
4  [ 0    0    1    0    0 ]
```

| Property     | Value                        |
|--------------|------------------------------|
| Space        | O(V²)                        |
| Edge lookup  | O(1)                         |
| Best for     | Dense graphs                 |
| Used by      | Floyd-Warshall, dense Prim   |

> **Rule of thumb:** Default to adjacency list. Switch to adjacency matrix when E ≈ V² or when O(1) edge lookups are critical.

---

## 2. Traversal Algorithms

### BFS — Breadth-First Search

Explores all neighbours at depth *d* before moving to depth *d+1*, using a **queue** (FIFO).

```
Traversal order on a tree:
         1          ← Level 0
       /   \
      2     3       ← Level 1
     / \   / \
    4   5 6   7     ← Level 2

BFS visit order: 1 → 2, 3 → 4, 5, 6, 7
```

**When to use:**
- Shortest path in an **unweighted** graph
- Level-order traversal
- Finding all nodes within a given distance (k-hop neighbourhood)
- Detecting if a graph is bipartite
- Multi-source shortest path

**Why it works:** FIFO ordering guarantees you always reach a node via the fewest possible edges first — so the first time BFS visits a node, that visit *is* the shortest path.

**Complexity:**

| Time   | Space |
|--------|-------|
| O(V+E) | O(V)  |

**Real-world uses:** Social network friend suggestions ("people you may know" at 2 hops), GPS routing on unweighted grids, web crawlers, peer-to-peer network discovery, level-by-level tree serialisation.

---

### DFS — Depth-First Search

Explores as far as possible down one path before backtracking, using a **stack** (or recursion).

```
Traversal order on the same tree:
         1
       /   \
      2     3
     / \   / \
    4   5 6   7

DFS visit order: 1 → 2 → 4 (backtrack) → 5 (backtrack) → 3 → 6 (backtrack) → 7
```

**When to use:**
- Detecting cycles
- Topological sorting
- Finding connected/strongly connected components
- Solving mazes and puzzles (exploring all paths)
- Checking if a path exists between two nodes
- Generating all permutations / subsets (backtracking)

**Why it works:** By going deep before backtracking, DFS naturally tracks the recursion call stack — which is exactly what you need for problems that care about *structure* (cycles, ordering, components) rather than *distance*.

**Complexity:**

| Time   | Space |
|--------|-------|
| O(V+E) | O(V)  |

**Real-world uses:** Dependency resolution (build systems like Make, Bazel), game tree search (minimax), detecting deadlocks in OS scheduling, compiler symbol-table construction, generating mazes.

---

## 3. Shortest Path Algorithms

### Dijkstra's Algorithm

Greedily picks the unvisited node with the smallest known distance, then **relaxes** its neighbours. Uses a min-heap (priority queue).

```
Relaxation: if dist[u] + weight(u,v) < dist[v], then update dist[v]

Graph:
    A --1-- B --2-- D
    |       |
    4       1
    |       |
    C --1-- E

Starting from A:
  dist = {A:0, B:1, C:4, D:3, E:2}
```

**When to use:**
- Shortest path with **non-negative edge weights**
- Single-source to all nodes
- Single-source to one specific target (terminate early)
- Foundation for A* (add a heuristic)

**Why it works:** Because all edge weights are ≥ 0, once a node is extracted from the min-heap as "finalised," its distance can never improve — making the greedy extraction safe.

**Complexity:**

| Time                  | Space |
|-----------------------|-------|
| O((V + E) log V)      | O(V)  |

> ⚠️ **Constraint:** Fails with negative edge weights. Use Bellman-Ford instead.

**Real-world uses:** Google Maps / GPS navigation, network routing protocols (OSPF), game pathfinding (A* is Dijkstra + heuristic), network latency optimisation.

---

### Bellman-Ford Algorithm

Relaxes **all edges V-1 times** in a loop. On the Vth pass, any further relaxation means a negative cycle exists.

```
For each of V-1 iterations:
    For each edge (u, v, w):
        if dist[u] + w < dist[v]:
            dist[v] = dist[u] + w

After V-1 iterations, run one more pass:
    If any dist still updates → negative cycle detected!
```

**When to use:**
- Graphs with **negative edge weights**
- **Detecting negative weight cycles**
- When Dijkstra fails
- Distributed routing where each node only knows its neighbours (SPFA variant)

**Why it works:** The longest possible shortest path in a V-node graph has at most V-1 edges. Relaxing all edges V-1 times is guaranteed to propagate the shortest distance across any path of that length.

**Complexity:**

| Time      | Space |
|-----------|-------|
| O(V × E)  | O(V)  |

**Real-world uses:** Financial arbitrage detection (negative cycle = profit loop), routing protocols where costs can be negative (BGP with certain metrics), network toll/penalty modelling, currency exchange rate optimisation.

---

### Floyd-Warshall Algorithm

Uses **dynamic programming**. For every pair (i, j), tries every intermediate node k:

```
dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])

For k from 0 to V-1:
    For i from 0 to V-1:
        For j from 0 to V-1:
            dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
```

**When to use:**
- **All-pairs** shortest paths — you need the shortest distance between every pair
- Works with negative weights (but not negative cycles)
- Small to medium dense graphs (V ≤ ~400)
- Transitive closure of a directed graph

**Why it works:** By iterating over every possible intermediate node, it exhaustively considers every possible path. The DP formulation guarantees optimal substructure — the shortest path from i to j through k is the shortest path from i to k plus the shortest from k to j.

**Complexity:**

| Time  | Space |
|-------|-------|
| O(V³) | O(V²) |

**Real-world uses:** Network routing tables where all-pairs latency is needed, transitive closure, traffic matrices in telecommunications, finding diameter of a graph.

---

### A* Algorithm

Dijkstra extended with a **heuristic function** `h(n)` that estimates the remaining cost from node n to the target.

```
Priority = g(n) + h(n)
  g(n) = actual cost from start to n
  h(n) = estimated cost from n to goal (must be admissible: never overestimates)

Common heuristics:
  Grid (4-directional): Manhattan distance |dx| + |dy|
  Grid (8-directional): Chebyshev distance max(|dx|, |dy|)
  Euclidean space:      sqrt(dx² + dy²)
```

**When to use:**
- Single-source to **single-target** with a spatial or domain heuristic
- When you want faster-than-Dijkstra performance with optimality guarantees
- Game pathfinding, robotics motion planning

**Why it works:** A good **admissible** heuristic guides the search toward the target, drastically reducing explored nodes while maintaining optimality guarantees. With `h(n) = 0`, A* degenerates to Dijkstra.

**Complexity:**

| Time                      | Space |
|---------------------------|-------|
| O(E) best, O(V log V) avg | O(V)  |

**Real-world uses:** Game AI pathfinding (every major game engine), robotics motion planning, GPS route planning with estimated distances, puzzle solvers (15-puzzle, Sokoban).

---

## 4. Minimum Spanning Tree (MST)

Both algorithms find the minimum-weight set of edges that connects all V nodes with exactly V-1 edges and no cycles.

```
Graph:            MST result:
A -4- B            A -4- B
|  \  |            |
2   3  5     →     2
|    \ |            |
C -1- D            C -1- D   Total weight: 7
```

### Kruskal's Algorithm

Sort all edges by weight. Use **Union-Find** (DSU). Add an edge if it connects two different components (doesn't form a cycle).

```
Steps:
1. Sort all edges by weight ascending
2. For each edge (u, v, w):
   a. If find(u) ≠ find(v):   ← not in the same component
      add edge to MST
      union(u, v)
3. Stop when MST has V-1 edges
```

**When to use:**
- Sparse graphs (E << V²)
- When you already have a sorted edge list
- When you need a spanning forest (handles disconnected graphs naturally)

**Complexity:**

| Time         | Space |
|--------------|-------|
| O(E log E)   | O(V)  |

---

### Prim's Algorithm

Greedily grow a tree. At each step, add the minimum-weight edge connecting a non-tree node to the current tree. Uses a min-heap.

```
Steps:
1. Start from any node, add to tree
2. Add all its edges to the min-heap
3. Extract min edge (u, v, w):
   - If v is already in tree: skip
   - Else: add v to tree, add v's edges to heap
4. Repeat until V-1 edges added
```

**When to use:**
- Dense graphs (E ≈ V²)
- When starting from a specific node
- When graph is given as adjacency matrix

**Complexity:**

| Time              | Space |
|-------------------|-------|
| O((V+E) log V)    | O(V)  |

**Real-world uses (both MST algorithms):** Network cable laying (minimise wire length), circuit board design, cluster analysis in ML (single-linkage clustering), designing road/pipeline/power grid infrastructure.

---

## 5. Cycle Detection

### Undirected Graphs

Use DFS. If you reach a visited node that is **not the parent** of the current node, a cycle exists.

```
DFS from A:
A → B → C → A  (C's neighbour A is visited and not C's parent B)
                → CYCLE DETECTED
```

### Directed Graphs

Use DFS with a **recursion stack** (in-progress set). If you reach a node already in the current active path, a cycle exists.

```
States per node:
  WHITE = unvisited
  GRAY  = currently in recursion stack (in progress)
  BLACK = fully processed

If DFS reaches a GRAY node → CYCLE DETECTED
```

> ⚠️ Union-Find is **not sufficient** for directed graphs. The direction of edges matters.

**Complexity:**

| Time   | Space |
|--------|-------|
| O(V+E) | O(V)  |

**Real-world uses:** Deadlock detection in operating systems, detecting circular imports/dependencies in build systems and package managers (pip, npm, Maven), validating DAGs before topological sort, detecting circular references in garbage collection.

---

## 6. Topological Sort

Only valid on **Directed Acyclic Graphs (DAGs)**. Produces a linear ordering of nodes such that for every directed edge (u → v), u appears before v.

```
DAG:
  Maths 101 → Linear Algebra ──┐
                                ├──→ ML Foundations → Deep Learning
  CS 101    → Data Structures ─┘

Valid topological order:
  Maths 101 → CS 101 → Linear Algebra → Data Structures → ML Foundations → Deep Learning
```

### Kahn's Algorithm (BFS-based)

```
1. Compute in-degree for every node
2. Enqueue all nodes with in-degree = 0
3. While queue is not empty:
   a. Dequeue node u, add to result
   b. For each neighbour v of u:
      decrement in-degree[v]
      if in-degree[v] == 0: enqueue v
4. If result.size < V → graph has a cycle
```

### DFS-based

```
1. Run DFS on every unvisited node
2. After all neighbours of u are processed: push u to stack
3. Reverse of stack = topological order
```

**Complexity:**

| Time   | Space |
|--------|-------|
| O(V+E) | O(V)  |

**Real-world uses:** `make` and build systems (Bazel, Gradle, CMake), package managers resolving install order (npm, pip), Kubernetes pod startup ordering, compiler instruction scheduling, spreadsheet formula evaluation.

---

## 7. Connected Components

### BFS/DFS Component Labelling

Run BFS or DFS from every unvisited node, labelling all reachable nodes with the same component ID.

```
Graph (undirected, disconnected):
  Component 1: A — B — C
  Component 2: D — E
  Component 3: F

Result: {A,B,C} = comp 0, {D,E} = comp 1, {F} = comp 2
```

**When to use:** Counting isolated subgraphs, checking if a graph is fully connected, island-counting problems on grids (`number of islands`), finding all reachable nodes from a source.

---

### Union-Find (Disjoint Set Union — DSU)

A data structure with two operations:
- `find(a)` — returns the component representative of a
- `union(a, b)` — merges the components of a and b

```
Optimisations (both required):
  Path compression:  find(a) flattens the tree during traversal
  Union by rank:     always attach smaller tree under larger tree root

find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])   ← path compression
    return parent[x]

union(a, b):
    ra, rb = find(a), find(b)
    if ra == rb: return   ← already same component
    if rank[ra] < rank[rb]: swap(ra, rb)
    parent[rb] = ra
    if rank[ra] == rank[rb]: rank[ra]++
```

**When to use:** Dynamic connectivity (edges added one at a time, real-time queries), Kruskal's MST, image segmentation, dynamic friend/group membership systems.

**Complexity:**

| Operation  | Time (amortised) |
|------------|-----------------|
| find       | O(α(V)) ≈ O(1)  |
| union      | O(α(V)) ≈ O(1)  |
| Space      | O(V)            |

> α is the inverse Ackermann function — effectively constant for all practical inputs.

---

### Strongly Connected Components (SCCs)

An SCC is a maximal set of nodes in a **directed graph** where every node is reachable from every other.

```
Directed graph:
  A → B → C → A   ← SCC: {A, B, C}
  C → D → E       ← SCCs: {D}, {E}

Condensation DAG:
  {A,B,C} → {D} → {E}
```

#### Kosaraju's Algorithm (2 DFS passes)

```
Pass 1: DFS on original graph, record finish order (stack)
Pass 2: DFS on transposed graph in reverse finish order
        → each DFS tree in Pass 2 = one SCC
```

#### Tarjan's Algorithm (1 DFS pass)

```
Maintain: discovery time disc[], low-link low[], stack, onStack[]
For node u:
    low[u] = disc[u] = timer++
    push u to stack
    For each neighbour v:
        if unvisited: DFS(v), low[u] = min(low[u], low[v])
        elif onStack[v]: low[u] = min(low[u], disc[v])
    If low[u] == disc[u]:
        pop stack until u → one SCC found
```

> Tarjan's is preferred in practice — single pass, better cache performance.

**Complexity:**

| Algorithm  | Time   | Space |
|------------|--------|-------|
| Kosaraju   | O(V+E) | O(V)  |
| Tarjan     | O(V+E) | O(V)  |

**Real-world uses:** Compiler optimisation (identifying loops in control-flow graphs), web page ranking (finding link clusters), social network analysis (tight-knit groups in follow graphs), circuit analysis, package dependency cycle grouping.

---

## 8. Network Flow

### Ford-Fulkerson / Edmonds-Karp

Finds the **maximum flow** from a source node to a sink node in a directed, capacity-constrained graph.

```
Graph with capacities:
  S --10--> A --5--> T
  S --8---> B --7--> T
             A --3--> B

Max flow: 13 (S→A→T: 5, S→A→B→T: 3, S→B→T: 5)
```

**Core idea:**
```
While an augmenting path (source → sink with remaining capacity) exists:
    Find the path (BFS in Edmonds-Karp for shortest path)
    Find bottleneck capacity = min capacity along the path
    Push flow along path
    Update residual capacities (forward: subtract, backward: add)
```

**Max-Flow Min-Cut Theorem:** The maximum flow equals the minimum capacity of any cut separating source from sink.

**When to use:**
- Maximum flow / minimum cut problems
- Bipartite matching (job assignment, stable matching)
- Scheduling with capacity limits
- Image segmentation (graph cut)
- Circulation problems

**Complexity (Edmonds-Karp):**

| Time        | Space  |
|-------------|--------|
| O(V × E²)  | O(V+E) |

**Real-world uses:** Traffic flow optimisation, hospital–patient matching, airline crew scheduling, image segmentation, internet bandwidth routing, project selection problems.

---

## 9. Bipartite Checking

Use BFS/DFS with **2-colouring**. Assign alternating colours (0/1) to nodes. If two adjacent nodes share a colour, the graph is not bipartite.

```
Bipartite (2-colourable):         Not bipartite (odd cycle):
  A(0) — B(1)                       A(0) — B(1)
  |       |                          |       |
  C(1) — D(0)                       C(0) — (A has same colour as C)
                                          ↑ CONFLICT
```

**When to use:**
- Verifying if a matching problem is solvable
- Problems where nodes fall into two disjoint sets (students/classes, workers/jobs)
- Detecting odd-length cycles

**Complexity:**

| Time   | Space |
|--------|-------|
| O(V+E) | O(V)  |

**Real-world uses:** Stable matching (Gale-Shapley), job assignment problems, conflict-free colouring/scheduling, movie/actor recommendation systems.

---

## 10. Eulerian and Hamiltonian Paths

### Eulerian Path / Circuit

A path that visits every **edge** exactly once.

```
Conditions (undirected):
  Eulerian circuit: ALL nodes have even degree
  Eulerian path:    EXACTLY 2 nodes have odd degree (start and end)

Example:
  A -─- B -─- C         A──B──C──D──A──C  ← Eulerian circuit
   \   / \   /                              (all degrees = 2)
    \ /   \ /
     D ─── E
```

**Algorithm:** Hierholzer's — DFS with a stack.

**Complexity:**

| Time   | Space |
|--------|-------|
| O(V+E) | O(V)  |

**Real-world uses:** Route planning where every road must be covered exactly once (postal delivery, garbage collection, snow ploughing), circuit board trace routing, DNA sequencing (De Bruijn graphs).

---

### Hamiltonian Path / Circuit

A path that visits every **node** exactly once.

> ⚠️ This is **NP-complete** in general — no known polynomial-time algorithm.

**Approaches:**

| Approach                          | Time           | Suitable for      |
|-----------------------------------|----------------|-------------------|
| Backtracking DFS with pruning     | Exponential    | Small V (≤ 20)    |
| Bitmask DP (Held-Karp)            | O(2^V × V²)    | V ≤ 20            |
| Approximation (e.g. Christofides) | Polynomial     | Large V, approx   |

**Real-world uses:** Travelling Salesman Problem (TSP), DNA fragment assembly, logistics route optimisation, knight's tour puzzle, circuit board drilling optimisation.

---

## 11. Bridge and Articulation Point Detection

A **bridge** is an edge whose removal disconnects the graph.
An **articulation point** (cut vertex) is a node whose removal disconnects the graph.

```
Graph:
  A — B — C — D
       \
        E

Bridge: C—D (removing it disconnects D)
Articulation point: B (removing it disconnects {A} from {C,D,E})
```

**Algorithm (Tarjan's, single DFS pass):**

```
Maintain: disc[] (discovery time), low[] (lowest disc reachable)

For edge (u → v):
  Bridge condition:    low[v] > disc[u]
  Articulation point: low[v] >= disc[u]  (with special root check)
```

**Complexity:**

| Time   | Space |
|--------|-------|
| O(V+E) | O(V)  |

**Real-world uses:** Network reliability (identifying critical routers/links), power grid vulnerability analysis, supply chain single-point-of-failure identification, road network resilience planning.

---

## 12. Shortest Path in Special Graph Types

### DAG Shortest / Longest Path

In a DAG, use topological order + edge relaxation. No cycles → no revisiting needed.

```
Steps:
1. Topological sort the DAG
2. Process nodes in topological order
3. Relax outgoing edges of each node
```

**Complexity:** O(V + E) — faster than Dijkstra.

**When to use:** Critical Path Method (CPM) in project scheduling, longest path in a DAG (NP-hard in general, polynomial in DAGs), evaluating DAG-based computation graphs.

---

### 0-1 BFS

When edges have weights of **only 0 or 1**, use a **deque** instead of a priority queue.

```
Rule:
  Weight-0 edge: push destination to FRONT of deque
  Weight-1 edge: push destination to BACK of deque
```

**Complexity:** O(V + E) vs O((V+E) log V) for Dijkstra — significant speedup.

**When to use:** Grid problems where moving costs 0 or 1 (e.g. minimum flips, minimum door-openings), any problem reducible to 0/1 edge weights.

---

### Multi-Source BFS

Initialise BFS with **all source nodes** in the queue simultaneously.

```
Use case: "Find the nearest exit from any cell in a maze"
Instead of: run BFS per source → O(S × (V+E))
Do:         add all sources to queue at dist=0 → O(V+E)
```

**When to use:** Nearest fire/water/facility from any point, distance-to-nearest-X for every cell.

---

## 13. Algorithm Comparison Tables

### Shortest Path — Which Algorithm When?

| Algorithm      | Negative weights | All-pairs | Heuristic | Time complexity    |
|----------------|-----------------|-----------|-----------|-------------------|
| BFS            | ✗ (unweighted)  | ✗         | ✗         | O(V + E)          |
| Dijkstra       | ✗               | ✗         | ✗         | O((V+E) log V)    |
| Bellman-Ford   | ✓               | ✗         | ✗         | O(V × E)          |
| Floyd-Warshall | ✓               | ✓         | ✗         | O(V³)             |
| A*             | ✗               | ✗         | ✓         | O(E) best case    |
| 0-1 BFS        | ✗ (0/1 only)    | ✗         | ✗         | O(V + E)          |
| DAG relaxation | ✓ (DAG only)    | ✗         | ✗         | O(V + E)          |

---

### Problem Type → Algorithm Mapping

| Problem                              | Algorithm                    |
|--------------------------------------|------------------------------|
| Shortest path, unweighted graph      | BFS                          |
| Shortest path, non-negative weights  | Dijkstra                     |
| Shortest path, negative weights      | Bellman-Ford                 |
| Shortest path, all pairs             | Floyd-Warshall               |
| Shortest path, spatial heuristic     | A*                           |
| Minimum spanning tree, sparse graph  | Kruskal                      |
| Minimum spanning tree, dense graph   | Prim                         |
| Cycle detection (undirected)         | DFS (parent tracking)        |
| Cycle detection (directed)           | DFS (recursion stack)        |
| Task ordering with dependencies      | Topological sort (Kahn/DFS)  |
| Connected components, undirected     | BFS/DFS labelling or DSU     |
| Connected components, directed       | Tarjan's / Kosaraju's SCC    |
| Dynamic connectivity queries         | Union-Find (DSU)             |
| Maximum flow / minimum cut           | Edmonds-Karp                 |
| Bipartite verification               | BFS 2-colouring              |
| Cover every edge once                | Eulerian path (Hierholzer's) |
| Cover every node once                | Hamiltonian / TSP (NP-hard)  |
| Critical infrastructure / bridges    | Tarjan's bridge detection    |
| Grid problems with 0/1 costs         | 0-1 BFS                      |
| Nearest source for all nodes         | Multi-source BFS             |

---

## 14. Additional Points Worth Knowing

### Implicit Graphs

Many problems don't give you an explicit graph. The key skill is recognising that these *are* graphs and constructing edges on the fly during traversal.

| Problem type             | Nodes are         | Edges are             |
|--------------------------|-------------------|-----------------------|
| Grid pathfinding         | Cells             | Adjacent cells        |
| Word ladder              | Words             | 1-letter edits        |
| State-space search       | System states     | Valid transitions     |
| Sliding puzzle           | Board configs     | Valid moves           |

### Bidirectional BFS

Run BFS simultaneously from both source and target. Meets in the middle.

- Nodes explored: O(b^d) normal BFS vs O(b^(d/2)) bidirectional
- Significant on large sparse graphs (social networks, web graphs)
- Use when: single pair shortest path on a large undirected unweighted graph

### DSU Without Both Optimisations Is Dangerous

Without path compression and union by rank, chains degrade to O(V) per operation. In a loop of 10⁶ queries, that's the difference between microseconds and seconds.

### Negative Cycle Workaround

Bellman-Ford's Vth iteration check is exact but slow. A common competitive programming trick: run Bellman-Ford with extra iterations and check if distances keep decreasing. If yes → reachable from a negative cycle.

### Pruning in Backtracking (Hamiltonian / TSP)

Always add early termination: if the current partial path cost already exceeds the best known solution, abandon that branch immediately. This makes NP-hard problems tractable for small inputs (V ≤ 20).

### Graph Condensation

After finding SCCs with Tarjan's / Kosaraju's, replace each SCC with a single super-node. The resulting condensation is always a DAG — and many hard problems on general directed graphs become easy on a DAG.

```
Original (cyclic directed graph):
  A ↔ B → C ↔ D → E

After SCC condensation (DAG):
  {A,B} → {C,D} → {E}

Now topological sort, longest path, etc. all apply.
```

### Choosing BFS vs DFS

```
Need shortest path?          → BFS
Need to explore all paths?   → DFS
Need structural info?        → DFS (cycles, SCCs, topo sort)
Need level / distance info?  → BFS
Memory constrained?          → DFS (stack depth vs BFS queue width)
Very deep graph?             → BFS (DFS may stack overflow)
```

---

## Quick Reference — Complexity Summary

| Algorithm              | Time              | Space | Notes                              |
|------------------------|-------------------|-------|------------------------------------|
| BFS                    | O(V + E)          | O(V)  | Unweighted shortest path           |
| DFS                    | O(V + E)          | O(V)  | Structure, cycles, components      |
| Dijkstra               | O((V+E) log V)    | O(V)  | Non-negative weights only          |
| Bellman-Ford           | O(V × E)          | O(V)  | Negative weights, cycle detection  |
| Floyd-Warshall         | O(V³)             | O(V²) | All-pairs, dense small graphs      |
| A*                     | O(E) to O(V logV) | O(V)  | Single target + heuristic          |
| Kruskal                | O(E log E)        | O(V)  | MST, sparse graphs                 |
| Prim                   | O((V+E) log V)    | O(V)  | MST, dense graphs                  |
| Topological Sort       | O(V + E)          | O(V)  | DAG only                           |
| Union-Find (DSU)       | O(α(V)) per op    | O(V)  | Dynamic connectivity               |
| Tarjan's SCC           | O(V + E)          | O(V)  | Directed SCCs, bridges, art. pts   |
| Edmonds-Karp           | O(V × E²)         | O(V+E)| Max flow                           |
| Bipartite check        | O(V + E)          | O(V)  | 2-colouring via BFS/DFS            |
| Hierholzer (Eulerian)  | O(V + E)          | O(V)  | All-edges-once traversal           |
| Held-Karp (Hamiltonian)| O(2^V × V²)       | O(2^V)| NP-complete, exact for small V     |

---

*Language-agnostic. Pseudocode-level. Applicable to competitive programming, system design, and production engineering.*
