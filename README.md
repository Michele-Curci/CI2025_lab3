# CI2025_lab3

Repository for lab 3 of the course Computational Intelligence

## Project overview

This project investigates efficient solutions for the **All-Pairs Shortest Path (APSP)** problem in randomly generated, weighted, and directed graphs, focusing on scenarios involving  **negative edge weights** . The primary goal is to compare the optimality and performance of heuristic search methods (A*) against the established ground truth (Bellman-Ford), and to implement approximation strategies for computationally intractable, large-scale instances.

## Initial Approach: The Basic A* Limitation

The standard **A*** Search Algorithm is known for its efficiency in finding the single-source shortest path by using an informed heuristic, $h(n)$, to guide the search.

The evaluation function is defined as:

$$
f(n) = g(n) + h(n)
$$

where $g(n)$ is the actual cost from the start to node $n$.

### Why A* Fails

Standard A* (and its non-heuristic counterpart, Dijkstra's Algorithm) relies on the assumption that  **all edge weights are non-negative** . If a path relaxation encounters a negative weight, it can lead to a shorter path being found *after* a node has already been finalized and moved to the closed set, violating the core principle of A* and yielding sub-optimal or incorrect results.

This necessitated an advanced approach capable of handling negative weights while still exploiting the power of the A* heuristic.

## Advanced Approach: Reduced Cost A* with Potential Functions

To maintain the correctness of A* on graphs with negative weights (but  **no negative cycles** ), we employed the **Reduced Cost A*** Algorithm (a technique borrowed from Johnson's Algorithm).

The solution involves a two-phase process:

### Phase 1: Bellman-Ford Preprocessing

To eliminate negative weights without changing the true shortest path structure, I transform the graph by running the **Bellman-Ford Algorithm** once from a single, temporary dummy source ($\mathbf{s}$).

1. **Potential Function (** $p(v)$**):** The distance found by Bellman-Ford from $\mathbf{s}$ to every node $v$ is used as the node's **potential function** $p(v)$.
2. **Negative Cycle Detection:** If Bellman-Ford detects a negative cycle, the APSP problem is unbounded, and the experiment for that instance is immediately flagged and skipped, saving significant computation time.

### Phase 2: A* Search on Reduced Costs

The original edge cost $w(u, v)$ is replaced by a non-negative **reduced cost** $w'(u, v)$:

$$
w'(u, v) = w(u, v) + p(u) - p(v) \ge 0
$$

The A* search then proceeds using $w'(u, v)$ for the path cost ($g'$), maintaining optimality because the potential transformation only shifts the cost of cycles, not the relative cost of paths.

This approach ensures the A* results are **optimal** (matching Bellman-Ford) for all valid graphs, regardless of negative weights. The implementation was highly optimized using a **Python heapq priority queue** to manage the open set, reducing the complexity of the core search loop from $O(V)$ to $O(\log V)$.

## Approximation Strategy: Hub-Based Shortest Paths

Even the optimized Reduced Cost A* run $V(V-1)$ times for APSP has a high complexity  O(V · E + V² log V).

For large, dense graphs (e.g., $N=500$ and $D=0.5$), this complexity leads to unacceptably long execution times.

### Justification for Approximation

When the problem size ($N$) exceeds a set threshold (e.g., $N > 200$), the experiment switches to an approximation method to achieve a solution in minutes instead of hours.

### The Hub-Based Mechanism

I use a technique that sacrifices path optimality for speed by introducing a small set of "hub" nodes ($k$):

1. **SSSP Sampling (2k runs):** Instead of running A* $V(V-1)$ times, we run A* only $2k$ times: from every node to every hub, and from every hub to every node.
2. **Path Approximation:** For any source $s$ and destination $d$, the shortest path is approximated by the path that travels through the best intermediate hub ($h$):
       Distance(s, d) ≈ min h in Hubs (Distance(s, h) + Distance(h, d))

This method reduces the complexity from a factor of $V^2$ shortest path searches to a factor of $2k$ searches, yielding massive performance gains.

## Comparison and Validation (Ground Truth)

The results from the A* runs were validated against the **Bellman-Ford APSP** results (which serve as the ground truth).

A comparison can be found in the compare.ipynb.

## Use of LLM

LLMs were used for utility functions outside the scope of the course like plotting graphs, understanding better new libraries, generating useful comments in key parts of the code and to improve the README.

## Final note

Due to time constraints the algorithm was cut short and for the A* algorithm I couldn't finish the last 5 runs, while the Bellman Ford algorithm was cut at 160/224 runs.
