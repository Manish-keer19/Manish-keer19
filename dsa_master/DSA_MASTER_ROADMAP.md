# DSA MASTER ROADMAP

> **Goal:** Master Data Structures, Algorithms, Problem-Solving Patterns, and become interview-ready for strong software engineering / product-company roles.

> **How to use this file:** Work through each phase **in order**. Check off each item as you complete it. Do **not** skip phases — every phase builds on previous ones.

---

## ══════════════════════════════════════════════════
## PHASE 0 — PROGRAMMING + JAVA BEFORE DSA
## ══════════════════════════════════════════════════

> Complete this entire phase before touching any DSA topic.

### 0.1 — Java Fundamentals

- [ ] Java installation / setup (JDK, IDE)
- [ ] Program structure
- [ ] `main` method
- [ ] Variables
- [ ] Data types
- [ ] Primitive types (`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`)
- [ ] Type casting (widening, narrowing)
- [ ] Operators (arithmetic, relational, logical, bitwise, assignment, ternary)
- [ ] Input / Output (`System.out.println`, `System.out.print`, `System.out.printf`)
- [ ] `if` / `else`
- [ ] `switch`
- [ ] `for` loop
- [ ] `while` loop
- [ ] `do-while` loop
- [ ] Nested loops
- [ ] `break` and `continue`

### 0.2 — Methods

- [ ] Method declaration
- [ ] Parameters and arguments
- [ ] Return values
- [ ] Method overloading
- [ ] Scope (local, instance, class)
- [ ] Pass by value (how Java passes primitives and references)

### 0.3 — Java OOP Required for DSA

- [ ] Class
- [ ] Object
- [ ] Constructor (default, parameterized)
- [ ] `this` keyword
- [ ] `static` keyword
- [ ] Encapsulation (access modifiers, getters/setters)
- [ ] Inheritance (`extends`)
- [ ] Polymorphism (compile-time, runtime)
- [ ] Abstraction
- [ ] Interface (`implements`)
- [ ] Abstract class
- [ ] `final` keyword
- [ ] `Object` class (`equals`, `hashCode`, `toString`)

### 0.4 — Java DSA Essentials

- [ ] Arrays (declaration, initialization, traversal)
- [ ] 2D Arrays (declaration, initialization, traversal)
- [ ] `String` (immutability, common methods)
- [ ] `StringBuilder`
- [ ] `StringBuffer`
- [ ] `char` and `Character` utility methods
- [ ] `Integer` wrapper class
- [ ] `Math` class
- [ ] `Arrays` utility class (`sort`, `binarySearch`, `fill`, `copyOf`)
- [ ] Collections Framework overview
- [ ] `ArrayList`
- [ ] `LinkedList` (as Java class)
- [ ] `HashMap`
- [ ] `HashSet`
- [ ] `TreeMap`
- [ ] `TreeSet`
- [ ] `LinkedHashMap`
- [ ] `LinkedHashSet`
- [ ] Stack concept in Java
- [ ] `ArrayDeque` (as Stack and Queue)
- [ ] `Queue` interface
- [ ] `Deque` interface
- [ ] `PriorityQueue`
- [ ] `Comparator` interface
- [ ] `Comparable` interface
- [ ] Custom sorting with `Comparator`
- [ ] `Collections.sort()`
- [ ] `Arrays.sort()`

### 0.5 — Java Concepts for Recursive / Data-Structure Code

- [ ] References vs primitives
- [ ] Object references
- [ ] `null`
- [ ] Recursion in Java
- [ ] Call stack
- [ ] `static` methods
- [ ] Custom `Node` classes (for linked list)
- [ ] Custom `TreeNode` classes (for trees)
- [ ] Custom `Graph` representation classes
- [ ] Generics awareness (`<T>`, `<K, V>`)

### 0.6 — Java Performance / Interview Essentials

- [ ] `BigInteger` awareness
- [ ] Integer overflow handling
- [ ] `int` vs `long` selection
- [ ] Character / ASCII handling
- [ ] String performance (`+` vs `StringBuilder`)
- [ ] Array vs `ArrayList` trade-offs
- [ ] `HashMap` internal behavior (hashing, collisions, load factor)
- [ ] `HashSet` internal behavior
- [ ] `Stack` vs `ArrayDeque` (why prefer `ArrayDeque`)
- [ ] `Queue` implementations comparison
- [ ] `PriorityQueue` behavior (min-heap by default)
- [ ] Time complexity of all collection operations

### 0.7 — Java DSA Input / Output

- [ ] `Scanner`
- [ ] `BufferedReader`
- [ ] `StringTokenizer`
- [ ] Fast input techniques
- [ ] String parsing (`split`, `trim`, `substring`, `charAt`, `toCharArray`)
- [ ] `PrintWriter` for fast output

---

## ══════════════════════════════════════════════════
## PHASE 1 — DSA FOUNDATIONS
## ══════════════════════════════════════════════════

> Learn these BEFORE serious problem solving.

### 1.1 — Core Concepts

- [ ] What is a data structure?
- [ ] What is an algorithm?
- [ ] Why DSA matters
- [ ] Problem → Data Structure → Algorithm thinking

### 1.2 — Complexity Analysis

- [ ] Time complexity
- [ ] Space complexity
- [ ] Big-O notation
- [ ] Big-Ω (Omega) notation
- [ ] Big-Θ (Theta) notation
- [ ] Best case / Average case / Worst case
- [ ] Amortized complexity

### 1.3 — Complexity Orders (recognize and compare)

- [ ] O(1) — Constant
- [ ] O(log n) — Logarithmic
- [ ] O(√n) — Square root
- [ ] O(n) — Linear
- [ ] O(n log n) — Linearithmic
- [ ] O(n²) — Quadratic
- [ ] O(n³) — Cubic
- [ ] O(2ⁿ) — Exponential
- [ ] O(n!) — Factorial
- [ ] Auxiliary space vs total space

### 1.4 — Foundational Thinking

- [ ] Recursion basics
- [ ] Iteration vs recursion
- [ ] Constraints analysis (input size → expected complexity)
- [ ] Brute force thinking
- [ ] Optimization thinking
- [ ] Basic mathematical reasoning (modular arithmetic, logarithms, exponents)

---

## ══════════════════════════════════════════════════
## PHASE 2 — ARRAYS
## ══════════════════════════════════════════════════

> Learn Arrays completely FIRST. This is the most important data structure.

### 2.1 — Array Fundamentals

- [ ] 1D Arrays
- [ ] 2D Arrays
- [ ] Array traversal (forward, backward)
- [ ] Array insertion / deletion
- [ ] Array searching (linear)
- [ ] Array manipulation
- [ ] In-place operations
- [ ] Matrix basics

### 2.2 — Array Patterns (learn in this order)

- [ ] Basic traversal patterns
- [ ] Frequency counting (using array)
- [ ] Hashing with arrays (index-based)
- [ ] Two pointers (same direction)
- [ ] Two pointers (opposite direction)
- [ ] Three pointers
- [ ] Sliding window — fixed-size
- [ ] Sliding window — variable-size
- [ ] Sliding window — shrinking condition
- [ ] Prefix sum
- [ ] Suffix sum
- [ ] Prefix XOR
- [ ] Difference array
- [ ] Kadane's algorithm (maximum subarray sum)
- [ ] Kadane's variations
- [ ] Sorting + scanning
- [ ] Sorting + two pointers
- [ ] Binary search on sorted array
- [ ] Binary search variations
- [ ] Binary search on answer
- [ ] Binary search on rotated sorted array
- [ ] Cyclic sort
- [ ] Dutch National Flag (3-way partition)
- [ ] Subarray techniques
- [ ] Subsequence techniques
- [ ] In-place array manipulation (swap, reverse, rotate)
- [ ] Array partitioning

### 2.3 — Matrix Patterns

- [ ] Matrix traversal (row-wise, column-wise)
- [ ] Spiral matrix traversal
- [ ] Matrix rotation (90°, 180°, 270°)
- [ ] Matrix transpose
- [ ] Diagonal traversal
- [ ] Anti-diagonal traversal
- [ ] Grid traversal (DFS / BFS preview)
- [ ] Search in sorted matrix
- [ ] Search in row-wise & column-wise sorted matrix

---

## ══════════════════════════════════════════════════
## PHASE 3 — STRINGS
## ══════════════════════════════════════════════════

> After Arrays, learn Strings.

### 3.1 — String Fundamentals

- [ ] String basics (immutability, creation)
- [ ] Character manipulation
- [ ] ASCII values
- [ ] Character frequency counting
- [ ] `StringBuilder` usage
- [ ] String comparison (`equals`, `compareTo`)
- [ ] String manipulation methods

### 3.2 — String Patterns

- [ ] Frequency counting (array-based, map-based)
- [ ] Hashing on strings
- [ ] Anagram detection
- [ ] Anagram grouping
- [ ] Two pointers on strings
- [ ] Sliding window on strings — fixed-size
- [ ] Sliding window on strings — variable-size
- [ ] Prefix techniques
- [ ] Palindrome check
- [ ] Palindrome techniques (expand around center)
- [ ] Longest palindromic substring
- [ ] Substring techniques
- [ ] Subsequence techniques
- [ ] String matching (brute force)

### 3.3 — Advanced String Algorithms

- [ ] String hashing (polynomial hashing)
- [ ] KMP algorithm
- [ ] Prefix function (failure function)
- [ ] Z algorithm
- [ ] Rabin-Karp algorithm
- [ ] Trie-based string problems (learn after Phase 18)
- [ ] Manacher's algorithm (longest palindromic substring in O(n))
- [ ] Suffix Array (awareness)
- [ ] Suffix Automaton (awareness)

---

## ══════════════════════════════════════════════════
## PHASE 4 — HASHING
## ══════════════════════════════════════════════════

> Now master hashing deeply.

### 4.1 — Hashing Fundamentals

- [ ] Hash function concept
- [ ] Collision handling concept
- [ ] `HashMap` usage
- [ ] `HashSet` usage
- [ ] `LinkedHashMap` usage
- [ ] `LinkedHashSet` usage

### 4.2 — Hashing Patterns

- [ ] Frequency map
- [ ] Frequency array (for characters / bounded values)
- [ ] Complement lookup (e.g., Two Sum)
- [ ] Duplicate detection
- [ ] First non-repeating element
- [ ] Grouping by key (anagram grouping, etc.)
- [ ] Prefix sum + HashMap
- [ ] Prefix XOR + HashMap
- [ ] Subarray sum equals K
- [ ] Longest subarray with sum K
- [ ] Longest consecutive sequence
- [ ] Hashing + sliding window
- [ ] Hashing + two pointers
- [ ] Count distinct elements in window
- [ ] Hashing for coordinate mapping

---

## ══════════════════════════════════════════════════
## PHASE 5 — SORTING
## ══════════════════════════════════════════════════

> Learn sorting algorithms in this order.

### 5.1 — Comparison-Based Sorts

- [ ] Bubble Sort
- [ ] Selection Sort
- [ ] Insertion Sort
- [ ] Merge Sort
- [ ] Quick Sort (Lomuto, Hoare partition)
- [ ] Heap Sort (learn implementation after Phase 14)

### 5.2 — Non-Comparison Sorts

- [ ] Counting Sort
- [ ] Radix Sort
- [ ] Bucket Sort

### 5.3 — Sorting Concepts

- [ ] Stability in sorting
- [ ] In-place vs out-of-place
- [ ] Custom `Comparator`
- [ ] `Comparable` interface
- [ ] Sorting objects by custom criteria
- [ ] Multi-key sorting

### 5.4 — Sorting Combination Patterns

- [ ] Sorting + greedy
- [ ] Sorting + two pointers
- [ ] Sorting + binary search
- [ ] Sorting + intervals
- [ ] Sorting + frequency
- [ ] Sorting + merge technique

---

## ══════════════════════════════════════════════════
## PHASE 6 — LINKED LIST
## ══════════════════════════════════════════════════

### 6.1 — Linked List Fundamentals

- [ ] Singly Linked List
- [ ] Doubly Linked List
- [ ] Circular Linked List
- [ ] Circular Doubly Linked List
- [ ] Linked List implementation from scratch
- [ ] Node manipulation
- [ ] Insertion (head, tail, middle)
- [ ] Deletion (head, tail, middle, by value)
- [ ] Searching
- [ ] Traversal
- [ ] Length calculation
- [ ] Reversal (iterative)
- [ ] Reversal (recursive)

### 6.2 — Linked List Patterns

- [ ] Dummy node technique
- [ ] Fast and slow pointers (tortoise & hare)
- [ ] Find middle element
- [ ] Detect cycle
- [ ] Find cycle entry point
- [ ] Find cycle length
- [ ] Remove Nth node from end
- [ ] Reverse entire linked list
- [ ] Reverse a sublist
- [ ] Reverse in K groups
- [ ] Merge two sorted lists
- [ ] Merge K sorted lists (learn after Phase 14)
- [ ] Find intersection point of two lists
- [ ] Check palindrome linked list
- [ ] Reorder linked list (L0→Ln→L1→Ln-1→…)
- [ ] Sort linked list (merge sort)
- [ ] Add two numbers (linked list representation)
- [ ] Flatten linked list
- [ ] Copy list with random pointer
- [ ] LRU Cache (linked list + HashMap)
- [ ] Linked list + recursion patterns

---

## ══════════════════════════════════════════════════
## PHASE 7 — STACK
## ══════════════════════════════════════════════════

### 7.1 — Stack Fundamentals

- [ ] Stack concept (LIFO)
- [ ] Stack implementation — array-based
- [ ] Stack implementation — linked-list-based
- [ ] Java `ArrayDeque` as stack
- [ ] Stack operations (push, pop, peek, isEmpty)

### 7.2 — Stack Patterns

- [ ] Basic stack usage
- [ ] Parentheses matching / validation
- [ ] Multiple types of parentheses
- [ ] Expression evaluation — postfix
- [ ] Expression evaluation — prefix
- [ ] Infix to postfix conversion
- [ ] Min Stack (O(1) getMin)
- [ ] Max Stack
- [ ] Monotonic Stack (increasing)
- [ ] Monotonic Stack (decreasing)
- [ ] Next Greater Element (right)
- [ ] Next Smaller Element (right)
- [ ] Previous Greater Element (left)
- [ ] Previous Smaller Element (left)
- [ ] Largest Rectangle in Histogram
- [ ] Maximal Rectangle
- [ ] Trapping Rain Water (stack approach)
- [ ] Stock Span
- [ ] Remove K digits
- [ ] Stack + greedy
- [ ] Stack + string manipulation
- [ ] Decode String
- [ ] Basic Calculator
- [ ] Asteroid Collision

---

## ══════════════════════════════════════════════════
## PHASE 8 — QUEUE / DEQUE
## ══════════════════════════════════════════════════

### 8.1 — Queue Fundamentals

- [ ] Queue concept (FIFO)
- [ ] Queue implementation — array-based
- [ ] Queue implementation — linked-list-based
- [ ] Circular Queue
- [ ] `Deque` concept
- [ ] `ArrayDeque` usage

### 8.2 — Queue Patterns

- [ ] Queue using two stacks
- [ ] Stack using two queues
- [ ] Monotonic Queue (deque-based)
- [ ] Sliding Window Maximum (monotonic deque)
- [ ] Sliding Window Minimum (monotonic deque)
- [ ] BFS queue concept (preview for graphs)
- [ ] First non-repeating character in stream
- [ ] Circular tour / Gas Station (queue thinking)
- [ ] Task scheduling with queue

---

## ══════════════════════════════════════════════════
## PHASE 9 — RECURSION
## ══════════════════════════════════════════════════

> Master recursion BEFORE backtracking, trees, and DP.

### 9.1 — Recursion Fundamentals

- [ ] Recursion concept
- [ ] Base case
- [ ] Recursive case
- [ ] Call stack visualization
- [ ] Recursion tree visualization
- [ ] Stack overflow awareness

### 9.2 — Recursion Practice Areas

- [ ] Recursion on numbers (factorial, power, GCD)
- [ ] Recursion on arrays (sum, min, max, search)
- [ ] Recursion on strings (reverse, palindrome, subsequences)
- [ ] Multiple recursive calls (branching)
- [ ] Tail recursion awareness
- [ ] Recursion time complexity analysis (recurrence relations)
- [ ] Recursion space complexity analysis
- [ ] Convert recursion to iteration
- [ ] Convert iteration to recursion

### 9.3 — Divide and Conquer

- [ ] Divide and Conquer concept
- [ ] Merge Sort as D&C
- [ ] Quick Sort as D&C
- [ ] Binary Search as D&C
- [ ] Count inversions
- [ ] Maximum subarray (D&C approach)
- [ ] Closest pair of points (awareness)
- [ ] Master theorem (awareness)

---

## ══════════════════════════════════════════════════
## PHASE 10 — BACKTRACKING
## ══════════════════════════════════════════════════

> Requires strong recursion skills.

### 10.1 — Backtracking Fundamentals

- [ ] Decision tree concept
- [ ] Choice → Explore → Unchoose pattern
- [ ] Pruning

### 10.2 — Backtracking Problems (in order)

- [ ] Subsets (power set)
- [ ] Subsets with duplicates
- [ ] Subsequences
- [ ] Permutations
- [ ] Permutations with duplicates
- [ ] Combinations
- [ ] Combination Sum (unlimited use)
- [ ] Combination Sum II (each element once)
- [ ] Combination Sum III
- [ ] Duplicate handling techniques
- [ ] Constraint-based search
- [ ] Generate Parentheses
- [ ] Palindrome Partitioning
- [ ] Letter Combinations of Phone Number
- [ ] N-Queens
- [ ] N-Queens II (count solutions)
- [ ] Sudoku Solver
- [ ] Rat in a Maze
- [ ] Word Search (2D grid)
- [ ] Word Search II (Backtracking + Trie — learn after Phase 18)
- [ ] Expression Add Operators
- [ ] Partition to K Equal Sum Subsets

---

## ══════════════════════════════════════════════════
## PHASE 11 — BINARY SEARCH (DEEP MASTERY)
## ══════════════════════════════════════════════════

> Master binary search as a standalone topic.

### 11.1 — Binary Search Fundamentals

- [ ] Basic binary search (iterative)
- [ ] Basic binary search (recursive)
- [ ] Search space concept
- [ ] Loop invariant

### 11.2 — Binary Search Variations

- [ ] Lower bound (first element ≥ target)
- [ ] Upper bound (first element > target)
- [ ] First occurrence of target
- [ ] Last occurrence of target
- [ ] First and last position of target
- [ ] Search insert position
- [ ] Count occurrences
- [ ] Floor and ceil

### 11.3 — Binary Search Applications

- [ ] Binary search on sorted arrays
- [ ] Binary search on rotated sorted arrays (with / without duplicates)
- [ ] Find minimum in rotated sorted array
- [ ] Search in rotated sorted array
- [ ] Peak element (1D)
- [ ] Peak element (2D)
- [ ] Search in 2D sorted matrix
- [ ] Search in row-wise sorted matrix
- [ ] Median of two sorted arrays
- [ ] Kth element of two sorted arrays

### 11.4 — Binary Search on Answer

- [ ] Binary search on answer concept
- [ ] Minimum feasible answer (minimize the maximum)
- [ ] Maximum feasible answer (maximize the minimum)
- [ ] Aggressive cows / Book allocation pattern
- [ ] Painter's partition pattern
- [ ] Split array largest sum
- [ ] Capacity to ship packages
- [ ] Koko eating bananas
- [ ] Search on monotonic function

---

## ══════════════════════════════════════════════════
## PHASE 12 — TREES
## ══════════════════════════════════════════════════

### 12.1 — Tree Fundamentals

- [ ] Tree terminology (root, node, edge, leaf, parent, child, sibling, ancestor, descendant)
- [ ] Binary Tree
- [ ] `TreeNode` class
- [ ] Full, Complete, Perfect, Balanced, Degenerate trees
- [ ] Tree height vs depth

### 12.2 — Tree Traversals

- [ ] Preorder (recursive)
- [ ] Inorder (recursive)
- [ ] Postorder (recursive)
- [ ] Preorder (iterative — using stack)
- [ ] Inorder (iterative — using stack)
- [ ] Postorder (iterative — using stack, two-stack, one-stack)
- [ ] Level Order (BFS — using queue)
- [ ] Reverse Level Order
- [ ] Morris Traversal — Inorder
- [ ] Morris Traversal — Preorder

### 12.3 — Tree Patterns

- [ ] DFS on trees
- [ ] BFS on trees
- [ ] Height of binary tree
- [ ] Depth of a node
- [ ] Diameter of binary tree
- [ ] Check balanced tree
- [ ] Check symmetric tree
- [ ] Check identical trees
- [ ] Invert / Mirror binary tree
- [ ] Path Sum (root to leaf)
- [ ] Path Sum II (all root-to-leaf paths)
- [ ] Maximum Path Sum (any node to any node)
- [ ] Root-to-leaf paths (all)
- [ ] Root-to-node path
- [ ] Lowest Common Ancestor (LCA)

### 12.4 — Tree View Problems

- [ ] Left View
- [ ] Right View
- [ ] Top View
- [ ] Bottom View
- [ ] Vertical Order Traversal
- [ ] Boundary Traversal
- [ ] Zigzag / Spiral Level Order Traversal
- [ ] Diagonal Traversal

### 12.5 — Tree Construction & Serialization

- [ ] Construct tree from Preorder + Inorder
- [ ] Construct tree from Postorder + Inorder
- [ ] Construct tree from Preorder + Postorder (awareness)
- [ ] Serialize binary tree
- [ ] Deserialize binary tree

### 12.6 — Advanced Tree Patterns

- [ ] Count nodes in complete binary tree
- [ ] Tree + HashMap patterns
- [ ] Tree with state (passing info up / down)
- [ ] Tree DP (preview — deep dive in Phase 20)
- [ ] Flatten binary tree to linked list
- [ ] Check subtree
- [ ] Maximum width of binary tree
- [ ] All nodes at distance K

---

## ══════════════════════════════════════════════════
## PHASE 13 — BINARY SEARCH TREE
## ══════════════════════════════════════════════════

### 13.1 — BST Fundamentals

- [ ] BST properties (left < root < right)
- [ ] Search in BST
- [ ] Insert into BST
- [ ] Delete from BST (3 cases)
- [ ] Inorder traversal gives sorted order

### 13.2 — BST Patterns

- [ ] Validate BST
- [ ] Kth smallest element
- [ ] Kth largest element
- [ ] LCA in BST
- [ ] Inorder predecessor
- [ ] Inorder successor
- [ ] Floor in BST
- [ ] Ceil in BST
- [ ] Recover BST (two swapped nodes)
- [ ] Construct BST from preorder
- [ ] Sorted array to balanced BST
- [ ] Sorted linked list to balanced BST
- [ ] Merge two BSTs
- [ ] Two Sum in BST
- [ ] BST iterator

---

## ══════════════════════════════════════════════════
## PHASE 14 — HEAP / PRIORITY QUEUE
## ══════════════════════════════════════════════════

### 14.1 — Heap Fundamentals

- [ ] Heap concept (complete binary tree + heap property)
- [ ] Min Heap
- [ ] Max Heap
- [ ] Heap representation using array
- [ ] Heapify (sift up, sift down)
- [ ] Insert into heap
- [ ] Delete from heap (extract min/max)
- [ ] Build heap from array — O(n)
- [ ] Heap Sort
- [ ] Java `PriorityQueue` (min-heap default)
- [ ] Java `PriorityQueue` with custom `Comparator`

### 14.2 — Heap Patterns

- [ ] Top K elements (min-heap of size K)
- [ ] Kth largest element
- [ ] Kth smallest element
- [ ] K closest points / elements
- [ ] K most frequent elements
- [ ] Sort nearly sorted array
- [ ] Merge K sorted lists
- [ ] Merge K sorted arrays
- [ ] Two Heaps pattern (min-heap + max-heap)
- [ ] Find Median from Data Stream
- [ ] Sliding window median
- [ ] Heap + Greedy patterns
- [ ] Task Scheduler
- [ ] Reorganize String
- [ ] Priority Queue scheduling problems
- [ ] Minimum cost to connect ropes / sticks

---

## ══════════════════════════════════════════════════
## PHASE 15 — GREEDY
## ══════════════════════════════════════════════════

> Learn greedy after sorting, heap, arrays, and intervals concepts.

### 15.1 — Greedy Fundamentals

- [ ] Greedy choice property
- [ ] Greedy vs DP — when greedy works
- [ ] Proving greedy correctness (exchange argument awareness)

### 15.2 — Greedy Problem Patterns

- [ ] Sorting + Greedy
- [ ] Activity Selection
- [ ] Fractional Knapsack
- [ ] Job Sequencing with Deadlines
- [ ] Jump Game I
- [ ] Jump Game II
- [ ] Gas Station
- [ ] Candy Distribution
- [ ] Minimum Platforms
- [ ] Meeting Rooms I
- [ ] Meeting Rooms II
- [ ] Interval Scheduling Maximization
- [ ] Merge Intervals
- [ ] Non-overlapping Intervals
- [ ] Assign Cookies
- [ ] Lemonade Change
- [ ] Huffman Coding
- [ ] Minimum number of coins
- [ ] Heap + Greedy patterns
- [ ] Task Scheduler (greedy approach)

---

## ══════════════════════════════════════════════════
## PHASE 16 — INTERVALS
## ══════════════════════════════════════════════════

### 16.1 — Interval Patterns

- [ ] Merge Intervals
- [ ] Insert Interval
- [ ] Overlapping Intervals detection
- [ ] Non-overlapping Intervals (minimum removals)
- [ ] Meeting Rooms I (can attend all?)
- [ ] Meeting Rooms II (minimum rooms)
- [ ] Interval Scheduling (maximum non-overlapping)
- [ ] Interval Intersection
- [ ] Remove Covered Intervals
- [ ] Interval Greedy approaches

### 16.2 — Advanced Interval Techniques

- [ ] Sweep Line algorithm
- [ ] Event-based processing (start/end events)
- [ ] Difference Array for range updates
- [ ] Coordinate Compression
- [ ] Interval + Priority Queue patterns

---

## ══════════════════════════════════════════════════
## PHASE 17 — GRAPHS
## ══════════════════════════════════════════════════

### 17.1 — Graph Fundamentals

- [ ] Graph terminology (vertex, edge, path, cycle, connected, component)
- [ ] Directed graph
- [ ] Undirected graph
- [ ] Weighted graph
- [ ] Unweighted graph
- [ ] Dense vs sparse graph
- [ ] Adjacency Matrix representation
- [ ] Adjacency List representation
- [ ] Edge List representation
- [ ] Degree of a vertex
- [ ] In-degree / Out-degree (directed)

### 17.2 — Graph Traversal

- [ ] BFS (Breadth-First Search) — iterative (queue)
- [ ] DFS (Depth-First Search) — recursive
- [ ] DFS — iterative (stack)
- [ ] Visited array / set
- [ ] Connected components (undirected)
- [ ] Grid DFS
- [ ] Grid BFS
- [ ] Number of Islands
- [ ] Flood Fill
- [ ] Surrounded Regions
- [ ] Rotting Oranges (multi-source BFS)
- [ ] Multi-source BFS pattern
- [ ] 0/1 Matrix (nearest 0)
- [ ] Word Ladder (BFS)

### 17.3 — Cycle Detection

- [ ] Cycle detection — undirected graph (BFS)
- [ ] Cycle detection — undirected graph (DFS)
- [ ] Cycle detection — directed graph (DFS + coloring)
- [ ] Cycle detection — directed graph (Kahn's / BFS)

### 17.4 — Topological Sort

- [ ] Topological Sort concept (DAG only)
- [ ] Kahn's Algorithm (BFS — indegree-based)
- [ ] DFS-based Topological Sort
- [ ] Course Schedule I
- [ ] Course Schedule II
- [ ] Alien Dictionary
- [ ] DAG (Directed Acyclic Graph)

### 17.5 — Shortest Path Algorithms

- [ ] BFS shortest path (unweighted)
- [ ] Dijkstra's Algorithm (non-negative weights)
- [ ] Dijkstra's with Priority Queue
- [ ] Bellman-Ford Algorithm (negative weights, cycle detection)
- [ ] Floyd-Warshall Algorithm (all-pairs shortest paths)
- [ ] 0-1 BFS (deque-based, 0/1 weights)
- [ ] Shortest path in DAG (topological sort + relaxation)
- [ ] Cheapest Flights Within K Stops

### 17.6 — Minimum Spanning Tree

- [ ] MST concept
- [ ] Prim's Algorithm
- [ ] Kruskal's Algorithm
- [ ] MST applications

### 17.7 — Disjoint Set Union (Union-Find)

- [ ] Disjoint Set Union (DSU) concept
- [ ] `find` with Path Compression
- [ ] `union` by Rank
- [ ] `union` by Size
- [ ] Connected components using DSU
- [ ] Cycle detection using DSU (undirected)
- [ ] Kruskal's using DSU
- [ ] Number of provinces / connected components
- [ ] Accounts Merge
- [ ] Making a Large Island

### 17.8 — Advanced Graph Algorithms

- [ ] Bridges in graph (Tarjan's)
- [ ] Articulation Points
- [ ] Strongly Connected Components (SCC)
- [ ] Kosaraju's Algorithm
- [ ] Tarjan's SCC Algorithm
- [ ] Condensation Graph
- [ ] Euler Path / Circuit (awareness)
- [ ] Hamiltonian Path (awareness)
- [ ] Network Flow (awareness)
- [ ] Bipartite Graph check (BFS / DFS)
- [ ] Graph Coloring (awareness)

### 17.9 — DAG DP

- [ ] DP on DAG concept
- [ ] Longest path in DAG
- [ ] Shortest path in DAG
- [ ] Number of paths in DAG

---

## ══════════════════════════════════════════════════
## PHASE 18 — TRIE
## ══════════════════════════════════════════════════

### 18.1 — Trie Fundamentals

- [ ] Trie structure (prefix tree)
- [ ] Trie Node design
- [ ] Insert word
- [ ] Search word
- [ ] Prefix search (startsWith)
- [ ] Delete word
- [ ] Count words with prefix
- [ ] Count distinct substrings

### 18.2 — Trie Patterns

- [ ] Word Dictionary (with wildcard `.`)
- [ ] Autocomplete / Suggestions
- [ ] Word Search II (backtracking + Trie)
- [ ] Longest common prefix using Trie
- [ ] Maximum XOR using Trie (bitwise Trie)
- [ ] Maximum XOR of two numbers
- [ ] Trie + backtracking combinations

---

## ══════════════════════════════════════════════════
## PHASE 19 — BIT MANIPULATION
## ══════════════════════════════════════════════════

### 19.1 — Bit Fundamentals

- [ ] Binary number representation
- [ ] AND (`&`)
- [ ] OR (`|`)
- [ ] XOR (`^`)
- [ ] NOT (`~`)
- [ ] Left shift (`<<`)
- [ ] Right shift (`>>`, `>>>`)

### 19.2 — Bit Operations

- [ ] Set a bit at position i
- [ ] Clear a bit at position i
- [ ] Toggle a bit at position i
- [ ] Check if bit is set at position i
- [ ] Count set bits (Brian Kernighan's)
- [ ] Check power of two
- [ ] Get lowest set bit
- [ ] Turn off lowest set bit

### 19.3 — Bit Manipulation Patterns

- [ ] XOR tricks (a ^ a = 0, a ^ 0 = a)
- [ ] Missing Number (XOR approach)
- [ ] Single Number (one unique, rest appear twice)
- [ ] Two unique numbers (XOR + partition)
- [ ] Single Number II (one unique, rest appear three times)
- [ ] Swap without temp variable
- [ ] Reverse bits
- [ ] Bitmasking concept
- [ ] Generate all subsets using bitmask
- [ ] Bitmask + DP (preview — deep dive in Phase 20)
- [ ] Bitwise AND of range
- [ ] Counting bits (0 to n)

---

## ══════════════════════════════════════════════════
## PHASE 20 — DYNAMIC PROGRAMMING
## ══════════════════════════════════════════════════

> **Prerequisites:** Recursion (Phase 9), Backtracking (Phase 10), Arrays, Strings, Trees, Graphs fundamentals, Greedy (Phase 15).
>
> Do NOT start DP until the above are comfortable.

### 20.1 — DP Fundamentals

- [ ] DP concept — what makes a problem DP?
- [ ] Overlapping subproblems
- [ ] Optimal substructure
- [ ] State definition
- [ ] State transition (recurrence relation)
- [ ] Base case
- [ ] Top-down (Memoization)
- [ ] Bottom-up (Tabulation)
- [ ] Space optimization (rolling array, two variables)
- [ ] Recursion → Memoization → Tabulation → Space-optimized pipeline

### 20.2 — 1D DP

- [ ] Fibonacci
- [ ] Climbing Stairs
- [ ] Min Cost Climbing Stairs
- [ ] House Robber
- [ ] House Robber II (circular)
- [ ] Decode Ways
- [ ] Maximum sum non-adjacent
- [ ] Tribonacci
- [ ] Perfect Squares
- [ ] Word Break

### 20.3 — Grid / 2D DP

- [ ] Grid paths (count)
- [ ] Grid paths with obstacles
- [ ] Minimum path sum
- [ ] Triangle minimum path sum
- [ ] Maximum falling path sum
- [ ] Cherry Pickup
- [ ] Dungeon Game

### 20.4 — Subsequence / Subset DP

- [ ] Subset Sum
- [ ] Partition Equal Subset Sum
- [ ] Target Sum
- [ ] Count subsets with given sum
- [ ] Minimum subset sum difference
- [ ] 0/1 Knapsack
- [ ] Unbounded Knapsack
- [ ] Coin Change (minimum coins)
- [ ] Coin Change II (count ways)
- [ ] Rod Cutting

### 20.5 — String DP

- [ ] Longest Common Subsequence (LCS)
- [ ] Longest Common Substring
- [ ] Shortest Common Supersequence
- [ ] Edit Distance (Levenshtein)
- [ ] Distinct Subsequences
- [ ] Minimum deletions to make palindrome
- [ ] Interleaving Strings
- [ ] Wildcard Matching
- [ ] Regular Expression Matching

### 20.6 — LIS Pattern

- [ ] Longest Increasing Subsequence (O(n²))
- [ ] Longest Increasing Subsequence (O(n log n) — binary search)
- [ ] Longest Bitonic Subsequence
- [ ] Number of LIS
- [ ] Largest Divisible Subset
- [ ] Russian Doll Envelopes
- [ ] Maximum Length of Pair Chain

### 20.7 — Palindromic DP

- [ ] Longest Palindromic Subsequence
- [ ] Longest Palindromic Substring (DP approach)
- [ ] Minimum insertions to make palindrome
- [ ] Palindrome Partitioning II (minimum cuts)
- [ ] Count palindromic substrings

### 20.8 — Interval / Partition DP

- [ ] Matrix Chain Multiplication
- [ ] Burst Balloons
- [ ] Minimum cost to merge stones
- [ ] Palindrome Partitioning II
- [ ] Evaluate Boolean Expression
- [ ] Scramble String
- [ ] Partition DP pattern (try all partitions)

### 20.9 — State Machine DP

- [ ] Best Time to Buy and Sell Stock (all variations)
- [ ] Stock I (one transaction)
- [ ] Stock II (unlimited transactions)
- [ ] Stock III (at most 2 transactions)
- [ ] Stock IV (at most K transactions)
- [ ] Stock with cooldown
- [ ] Stock with transaction fee

### 20.10 — Tree DP

- [ ] DP on trees concept
- [ ] Maximum path sum in tree
- [ ] Diameter of tree (DP approach)
- [ ] House Robber III (tree)
- [ ] Binary Tree Camera
- [ ] Longest path in tree

### 20.11 — DAG DP (revisit from Phase 17.9)

- [ ] DP on DAG
- [ ] Longest path in DAG
- [ ] Shortest path in DAG
- [ ] Number of paths in DAG

### 20.12 — Bitmask DP

- [ ] Bitmask DP concept
- [ ] Travelling Salesman Problem (TSP)
- [ ] Minimum cost to visit all nodes
- [ ] Assign tasks to workers
- [ ] Shortest superstring
- [ ] Partition into K equal sum subsets (bitmask)

### 20.13 — Digit DP

- [ ] Digit DP concept
- [ ] Count numbers with certain digit property in range [L, R]
- [ ] Count numbers without consecutive 1s
- [ ] Count numbers with digit sum = S

### 20.14 — Advanced DP (Optional/Competitive)

- [ ] DP with profiles (broken profile DP)
- [ ] DP on subsets of subsets
- [ ] Convex Hull Trick (awareness)
- [ ] Knuth's optimization (awareness)
- [ ] Divide and Conquer optimization (awareness)

---

## ══════════════════════════════════════════════════
## PHASE 21 — ADVANCED DATA STRUCTURES
## ══════════════════════════════════════════════════

> Only after core DSA (Phases 1–20) is strong.

### 21.1 — Segment Tree

- [ ] Segment Tree concept
- [ ] Build Segment Tree
- [ ] Point update
- [ ] Range query (sum, min, max)
- [ ] Range update with Lazy Propagation
- [ ] Merge Sort Tree (awareness)

### 21.2 — Fenwick Tree (Binary Indexed Tree)

- [ ] Fenwick Tree / BIT concept
- [ ] Point update + prefix query
- [ ] Range sum query
- [ ] 2D Fenwick Tree (awareness)

### 21.3 — Other Advanced Structures

- [ ] Sparse Table (static RMQ, O(1) query)
- [ ] Range Minimum Query (RMQ)
- [ ] Range Sum Query
- [ ] Coordinate Compression
- [ ] Advanced DSU (rollback, weighted DSU)
- [ ] Sqrt Decomposition (awareness)

### 21.4 — Advanced Query Techniques

- [ ] Sweep Line algorithm
- [ ] Offline Queries concept
- [ ] Mo's Algorithm (awareness)
- [ ] Persistent data structures (awareness)
- [ ] Balanced BST variants: AVL, Red-Black (awareness)

---

## ══════════════════════════════════════════════════
## PHASE 22 — ADVANCED ALGORITHMIC TECHNIQUES
## ══════════════════════════════════════════════════

### 22.1 — Advanced Techniques

- [ ] Divide and Conquer (advanced applications)
- [ ] Meet in the Middle
- [ ] Randomized Algorithms (awareness)
- [ ] Two Pointers on sorted/merged structures

### 22.2 — Mathematics for DSA

- [ ] Fast Exponentiation (binary exponentiation)
- [ ] Matrix Exponentiation
- [ ] Modular Arithmetic
- [ ] GCD (Euclidean Algorithm)
- [ ] Extended Euclidean Algorithm
- [ ] LCM
- [ ] Prime Checking
- [ ] Sieve of Eratosthenes
- [ ] Prime Factorization
- [ ] Modular Exponentiation
- [ ] Modular Inverse
- [ ] Combinatorics (nCr, nPr)
- [ ] Pascal's Triangle
- [ ] Catalan Numbers (awareness)
- [ ] Euler's Totient Function (awareness)
- [ ] Number Theory basics for competitive programming

---

## ══════════════════════════════════════════════════
## PHASE 23 — PROBLEM-SOLVING PATTERN MASTER LIST
## ══════════════════════════════════════════════════

> Final checklist of every major pattern you must recognize on sight.

### Fundamental Patterns

- [ ] Brute Force
- [ ] Hashing / Frequency Counting
- [ ] Two Pointers (same direction)
- [ ] Two Pointers (opposite direction)
- [ ] Fast / Slow Pointers
- [ ] Sliding Window (fixed size)
- [ ] Sliding Window (variable size)
- [ ] Prefix Sum
- [ ] Prefix XOR
- [ ] Suffix Sum
- [ ] Difference Array
- [ ] Kadane's Algorithm

### Sorting-Based Patterns

- [ ] Sorting + Scanning
- [ ] Sorting + Two Pointers
- [ ] Sorting + Binary Search
- [ ] Cyclic Sort
- [ ] Dutch National Flag (3-way partition)

### Search Patterns

- [ ] Binary Search (on array)
- [ ] Binary Search on Answer
- [ ] Ternary Search (awareness)

### Stack / Queue Patterns

- [ ] Monotonic Stack
- [ ] Monotonic Queue
- [ ] Stack for expression parsing

### Linked List Patterns

- [ ] Fast / Slow Pointers (linked list)
- [ ] Dummy Node technique
- [ ] In-place reversal

### Recursion / Backtracking Patterns

- [ ] Recursion
- [ ] Backtracking (choice/explore/unchoose)
- [ ] Divide and Conquer
- [ ] Generate all subsets / permutations / combinations

### Tree Patterns

- [ ] DFS on tree
- [ ] BFS on tree
- [ ] Tree DP

### Graph Patterns

- [ ] BFS
- [ ] DFS
- [ ] Multi-source BFS
- [ ] Topological Sort (Kahn's / DFS)
- [ ] Shortest Path (Dijkstra / Bellman-Ford / Floyd-Warshall)
- [ ] Union Find (DSU)
- [ ] MST (Prim's / Kruskal's)
- [ ] Bipartite Check

### Greedy Patterns

- [ ] Greedy choice after sorting
- [ ] Heap + Greedy
- [ ] Interval Greedy
- [ ] Sweep Line

### Trie Patterns

- [ ] Prefix-based search
- [ ] Bitwise Trie (XOR)

### Bit Manipulation Patterns

- [ ] XOR tricks
- [ ] Bitmask enumeration
- [ ] Bit counting

### DP Patterns

- [ ] 1D DP
- [ ] Grid / 2D DP
- [ ] Subsequence / Subset DP (Knapsack family)
- [ ] String DP (LCS, Edit Distance)
- [ ] LIS pattern
- [ ] Palindromic DP
- [ ] Interval / Partition DP (MCM family)
- [ ] State Machine DP
- [ ] Tree DP
- [ ] DAG DP
- [ ] Bitmask DP
- [ ] Digit DP

### Advanced Patterns

- [ ] Segment Tree / Fenwick Tree queries
- [ ] Coordinate Compression
- [ ] Meet in the Middle
- [ ] Advanced String Algorithms (KMP / Z / Rabin-Karp)
- [ ] Advanced Graph Algorithms (SCC / Bridges / Articulation Points)

---

## ══════════════════════════════════════════════════
## PHASE 24 — DSA PROBLEM PRACTICE PROGRESSION
## ══════════════════════════════════════════════════

> For **every** major data structure and pattern from Phases 2–22, follow this progression:

### Per-Topic Practice Stages

- [ ] **Stage 1:** Learn the concept / data structure / pattern
- [ ] **Stage 2:** Solve basic implementation problems
- [ ] **Stage 3:** Solve Easy problems (direct application)
- [ ] **Stage 4:** Solve Easy-Medium problems (small twist)
- [ ] **Stage 5:** Solve Medium problems (pattern recognition)
- [ ] **Stage 6:** Solve Medium-Hard problems (combine patterns)
- [ ] **Stage 7:** Solve Hard problems (optimization, edge cases)
- [ ] **Stage 8:** Solve unseen problems (no topic hint)
- [ ] **Stage 9:** Solve mixed-pattern problems
- [ ] **Stage 10:** Solve timed interview-style problems

### Practice Tracking Per Topic

| Topic | Stage 1 | Stage 2 | Stage 3 | Stage 4 | Stage 5 | Stage 6 | Stage 7 | Stage 8 | Stage 9 | Stage 10 |
|-------|---------|---------|---------|---------|---------|---------|---------|---------|---------|----------|
| Arrays | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Strings | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Hashing | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Sorting | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Linked List | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Stack | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Queue / Deque | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Recursion | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Backtracking | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Binary Search | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Trees | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| BST | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Heap | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Greedy | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Intervals | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Graphs | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Trie | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Bit Manipulation | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (1D) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (2D/Grid) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (Subsequence/Knapsack) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (String) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (Interval/Partition) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (State Machine) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (Tree/DAG) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (Bitmask) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP (Digit) | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Segment Tree | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Fenwick Tree | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Advanced Graphs | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Advanced Strings | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Math / Number Theory | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |

---

## ══════════════════════════════════════════════════
## PHASE 25 — BLIND PATTERN RECOGNITION
## ══════════════════════════════════════════════════

> After learning all patterns, practice identifying them without topic hints.

### Blind Solving Process

- [ ] Solve problems **without** knowing the topic/category
- [ ] Read the problem
- [ ] Identify constraints (input size)
- [ ] Estimate expected time complexity from constraints
- [ ] Think brute force first
- [ ] Identify the bottleneck in brute force
- [ ] Recognize the likely pattern
- [ ] Select the optimized pattern
- [ ] Implement the solution
- [ ] Test with edge cases
- [ ] Analyze time complexity
- [ ] Analyze space complexity
- [ ] Explain solution verbally (as in an interview)

### Blind Practice Milestones

- [ ] Solve 10 random Easy problems blind
- [ ] Solve 20 random Medium problems blind
- [ ] Solve 10 random Hard problems blind
- [ ] Achieve ≥70% success rate on blind Medium problems
- [ ] Achieve ≥40% success rate on blind Hard problems
- [ ] Identify correct pattern within 5 minutes for Medium problems
- [ ] Solve Medium problems within 25 minutes consistently
- [ ] Solve Hard problems within 45 minutes consistently

---

## ══════════════════════════════════════════════════
## PHASE 26 — INTERVIEW DSA PREPARATION
## ══════════════════════════════════════════════════

### 26.1 — Preparation Sequence

- [ ] Solve 100+ Easy problems across all topics
- [ ] Solve 200+ Medium problems across all topics
- [ ] Solve 50+ Hard problems across key topics
- [ ] Solve mixed-pattern problems (no topic hint)
- [ ] Solve timed problems (set timer before starting)
- [ ] Study previous interview questions (company-tagged)
- [ ] Study company-tagged problems (target companies)
- [ ] Do mock interviews (peer / platform)
- [ ] Whiteboard coding practice
- [ ] Explain solutions verbally while coding
- [ ] Explain time and space complexity after solving
- [ ] Handle edge cases systematically
- [ ] Debug under time pressure

### 26.2 — Interview Communication Skills

- [ ] Clarify the problem before coding
- [ ] Discuss approach before writing code
- [ ] Walk through examples
- [ ] Identify edge cases upfront
- [ ] Write clean, readable code
- [ ] Test solution with examples
- [ ] Optimize and discuss trade-offs
- [ ] Handle follow-up questions

### 26.3 — Common Interview Problem Categories

- [ ] Array manipulation
- [ ] String processing
- [ ] HashMap / HashSet lookups
- [ ] Linked List pointer tricks
- [ ] Stack-based parsing
- [ ] Binary Search (on array and on answer)
- [ ] Tree traversal and properties
- [ ] Graph BFS / DFS
- [ ] Greedy decisions
- [ ] Dynamic Programming
- [ ] Design Data Structures (LRU Cache, Min Stack, etc.)

---

## ══════════════════════════════════════════════════
## PHASE 27 — PRODUCT COMPANY READINESS
## ══════════════════════════════════════════════════

### 27.1 — Company Tiers

- [ ] **Startup interviews** — Arrays, Strings, Hashing, Basic DP, System Design basics
- [ ] **Service-based company interviews** — Core DSA (Arrays through Trees), basic Graph, basic DP
- [ ] **Product-based company interviews** — Full DSA, Graphs, DP, Design problems
- [ ] **Strong product companies** — All of above + Hard DP, Advanced Graphs, Trie, Segment Tree
- [ ] **FAANG-style interviews** — All of above + Pattern mastery, Communication, Optimization, Follow-ups

### 27.2 — Topic Readiness Checklist for Product Companies

| Topic | Concept Clear | Easy Done | Medium Done | Hard Done | Blind Ready |
|-------|:---:|:---:|:---:|:---:|:---:|
| Arrays | ☐ | ☐ | ☐ | ☐ | ☐ |
| Strings | ☐ | ☐ | ☐ | ☐ | ☐ |
| Hashing | ☐ | ☐ | ☐ | ☐ | ☐ |
| Linked Lists | ☐ | ☐ | ☐ | ☐ | ☐ |
| Stack | ☐ | ☐ | ☐ | ☐ | ☐ |
| Queue / Deque | ☐ | ☐ | ☐ | ☐ | ☐ |
| Binary Search | ☐ | ☐ | ☐ | ☐ | ☐ |
| Trees | ☐ | ☐ | ☐ | ☐ | ☐ |
| BST | ☐ | ☐ | ☐ | ☐ | ☐ |
| Heap / PQ | ☐ | ☐ | ☐ | ☐ | ☐ |
| Greedy | ☐ | ☐ | ☐ | ☐ | ☐ |
| Intervals | ☐ | ☐ | ☐ | ☐ | ☐ |
| Graphs | ☐ | ☐ | ☐ | ☐ | ☐ |
| Trie | ☐ | ☐ | ☐ | ☐ | ☐ |
| Backtracking | ☐ | ☐ | ☐ | ☐ | ☐ |
| DP | ☐ | ☐ | ☐ | ☐ | ☐ |
| Bit Manipulation | ☐ | ☐ | ☐ | ☐ | ☐ |
| Advanced DSA | ☐ | ☐ | ☐ | ☐ | ☐ |

### 27.3 — FAANG-Level Preparation Checklist

- [ ] 300+ problems solved (balanced across topics)
- [ ] All major patterns recognized on sight
- [ ] Can solve Medium in < 25 min
- [ ] Can solve Hard in < 45 min
- [ ] Can explain approach before coding
- [ ] Can handle follow-up questions (optimize, change constraints)
- [ ] Can identify edge cases without prompting
- [ ] Can write clean production-quality code under pressure
- [ ] Mock interviews completed (5+ rounds)
- [ ] System Design basics (awareness for senior roles)

---

## ══════════════════════════════════════════════════
## PHASE 28 — REVISION
## ══════════════════════════════════════════════════

### 28.1 — Spaced Repetition Schedule

| Revision Round | When | Focus |
|---|---|---|
| ☐ First revision | Same day | Redo the problem from scratch |
| ☐ 1-day revision | Next day | Recall approach, re-solve if stuck |
| ☐ 3-day revision | After 3 days | Pattern recall, quick re-solve |
| ☐ 7-day revision | After 1 week | Blind solve, no hints |
| ☐ 14-day revision | After 2 weeks | Blind solve, timed |
| ☐ 30-day revision | After 1 month | Blind solve, explain out loud |
| ☐ 60-day revision | After 2 months | Final check — fully retained? |

### 28.2 — Revision Quality Tracker

For each revised problem, verify:

- [ ] Concept remembered without looking up
- [ ] Template / approach remembered
- [ ] Pattern recognized immediately
- [ ] Problem solved completely without help
- [ ] Unseen variation solved using same pattern
- [ ] Timed solve completed within target time

### 28.3 — Weak Area Tracking

- [ ] Maintain a list of topics where you struggle
- [ ] Revisit weak topics every week
- [ ] Solve 5 extra problems per weak topic
- [ ] Re-attempt previously failed problems
- [ ] Track improvement over time

---

## ══════════════════════════════════════════════════
## PHASE 29 — FINAL DSA MASTERY CHECKLIST
## ══════════════════════════════════════════════════

> One final master checklist. Check off each level when ALL items within it are complete.

---

### ☐ LEVEL 1 — Java Ready

- [ ] Java fundamentals (variables, types, operators, control flow, loops)
- [ ] Methods (declaration, overloading, scope)
- [ ] OOP (class, object, constructor, inheritance, polymorphism, abstraction, interface)
- [ ] Arrays, Strings, StringBuilder
- [ ] Collections Framework (ArrayList, LinkedList, HashMap, HashSet, TreeMap, TreeSet)
- [ ] Stack, Queue, Deque, PriorityQueue in Java
- [ ] Comparator, Comparable, custom sorting
- [ ] References, null, custom Node/TreeNode classes
- [ ] Java performance essentials (overflow, String performance, collection internals)
- [ ] Fast I/O (Scanner, BufferedReader, StringTokenizer)

---

### ☐ LEVEL 2 — DSA Foundation

- [ ] Data structures and algorithms concept
- [ ] Time and space complexity (Big-O, Big-Ω, Big-Θ)
- [ ] All complexity orders (O(1) through O(n!))
- [ ] Amortized complexity
- [ ] Constraints → expected complexity mapping
- [ ] Brute force and optimization thinking
- [ ] Basic math reasoning

---

### ☐ LEVEL 3 — Core Data Structures

- [ ] Arrays (1D, 2D, matrix operations)
- [ ] Strings (manipulation, comparison, StringBuilder)
- [ ] Hashing (HashMap, HashSet, frequency maps)
- [ ] Sorting (all major algorithms, custom sorting)
- [ ] Linked Lists (singly, doubly, circular, all operations)
- [ ] Stacks (implementation, parentheses, expression evaluation)
- [ ] Queues / Deques (implementation, circular queue)
- [ ] Recursion (fundamentals, arrays, strings, multiple calls)
- [ ] Binary Search (all variations, search on answer)
- [ ] Binary Trees (traversals, properties, construction)
- [ ] BST (search, insert, delete, validate, Kth element)
- [ ] Heaps / Priority Queue (min/max heap, heapify, Top K)
- [ ] Trie (insert, search, prefix search)

---

### ☐ LEVEL 4 — Core Patterns

- [ ] Two Pointers (same direction, opposite direction)
- [ ] Sliding Window (fixed, variable)
- [ ] Prefix Sum / Prefix XOR / Difference Array
- [ ] Kadane's Algorithm
- [ ] Fast / Slow Pointers
- [ ] Dummy Node
- [ ] Monotonic Stack
- [ ] Monotonic Queue
- [ ] Backtracking (subsets, permutations, combinations, constraint search)
- [ ] Divide and Conquer
- [ ] BFS / DFS (on trees and graphs)
- [ ] Topological Sort
- [ ] Shortest Path (Dijkstra, Bellman-Ford)
- [ ] Union Find (DSU)
- [ ] MST (Prim's, Kruskal's)
- [ ] Greedy (sorting + greedy, interval greedy, heap + greedy)
- [ ] Intervals (merge, insert, scheduling, sweep line)
- [ ] Bit Manipulation (XOR tricks, bitmask, subsets)

---

### ☐ LEVEL 5 — Dynamic Programming

- [ ] DP fundamentals (memoization, tabulation, space optimization)
- [ ] 1D DP (Fibonacci, Climbing Stairs, House Robber)
- [ ] Grid / 2D DP (paths, minimum sum)
- [ ] Subsequence / Subset DP (Knapsack, Coin Change, Subset Sum)
- [ ] String DP (LCS, Edit Distance, Distinct Subsequences)
- [ ] LIS pattern
- [ ] Palindromic DP
- [ ] Interval / Partition DP (MCM, Burst Balloons)
- [ ] State Machine DP (Stock problems)
- [ ] Tree DP
- [ ] DAG DP
- [ ] Bitmask DP
- [ ] Digit DP

---

### ☐ LEVEL 6 — Advanced DSA

- [ ] Segment Tree (build, query, update, lazy propagation)
- [ ] Fenwick Tree / BIT
- [ ] Sparse Table
- [ ] Advanced Graph (Bridges, Articulation Points, SCC, Tarjan, Kosaraju)
- [ ] Advanced String (KMP, Z-algorithm, Rabin-Karp, Manacher)
- [ ] Advanced Techniques (Meet in the Middle, Matrix Exponentiation)
- [ ] Mathematics (GCD, Sieve, Modular Arithmetic, Combinatorics)
- [ ] Coordinate Compression
- [ ] Sweep Line
- [ ] Mo's Algorithm (awareness)

---

### ☐ LEVEL 7 — Interview Ready

- [ ] All major patterns recognized on sight
- [ ] Can solve without topic hints (blind solving)
- [ ] 100+ Easy, 200+ Medium, 50+ Hard solved
- [ ] Timed solving ability (Medium < 25 min, Hard < 45 min)
- [ ] Mock interviews completed
- [ ] Can explain approach and complexity verbally
- [ ] Can handle edge cases and follow-up questions
- [ ] Clean, readable code under pressure
- [ ] Spaced repetition revision active

---

### ☐ LEVEL 8 — Product Company Ready

- [ ] All Level 7 items complete
- [ ] 300+ problems solved across all topics
- [ ] Company-tagged problems studied for target companies
- [ ] All DP sub-patterns practiced (knapsack, LCS, MCM, state machine, bitmask)
- [ ] Advanced graph problems comfortable
- [ ] Can identify and switch patterns mid-problem
- [ ] Can handle "optimize further" follow-ups
- [ ] FAANG-level mock interviews passed
- [ ] Weak areas identified, tracked, and improved
- [ ] Revision system actively maintained

---

## ══════════════════════════════════════════════════
## DEPENDENCY MAP
## ══════════════════════════════════════════════════

```
Phase 0  (Java)
  │
  ▼
Phase 1  (DSA Foundations)
  │
  ├──▶ Phase 2  (Arrays)
  │      │
  │      ▼
  │    Phase 3  (Strings)
  │      │
  │      ▼
  │    Phase 4  (Hashing)
  │      │
  │      ▼
  │    Phase 5  (Sorting)
  │
  ├──▶ Phase 6  (Linked List)
  │
  ├──▶ Phase 7  (Stack)
  │
  ├──▶ Phase 8  (Queue / Deque)
  │
  ├──▶ Phase 9  (Recursion) ──────▶ Phase 10 (Backtracking)
  │
  ├──▶ Phase 11 (Binary Search)
  │
  ├──▶ Phase 12 (Trees) ──────────▶ Phase 13 (BST)
  │
  ├──▶ Phase 14 (Heap / PQ)
  │
  ├──▶ Phase 15 (Greedy)  ◀── requires Sorting, Heap, Arrays
  │
  ├──▶ Phase 16 (Intervals) ◀── requires Sorting, Greedy
  │
  ├──▶ Phase 17 (Graphs)  ◀── requires BFS/DFS, Queue, Recursion
  │
  ├──▶ Phase 18 (Trie)
  │
  ├──▶ Phase 19 (Bit Manipulation)
  │
  └──▶ Phase 20 (DP)  ◀── requires Recursion, Arrays, Strings,
         │                    Trees, Graphs, Greedy fundamentals
         ▼
       Phase 21 (Advanced DS) ◀── requires strong core DSA
         │
         ▼
       Phase 22 (Advanced Algorithms)
         │
         ▼
       Phase 23–29 (Patterns, Practice, Interview Prep)
```

---

> **START AT PHASE 0. DO NOT SKIP. FOLLOW THE ORDER. CHECK ITEMS AS YOU COMPLETE THEM.**
>
> **This roadmap is your long-term companion. Come back to it daily.**

---

*Last updated: October 2026*
