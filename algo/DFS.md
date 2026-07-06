# Create DFS cheat sheet markdown file

## What is DFS?
DFS = explore as deep as possible before backtracking

- Uses recursion (call stack) or stack
- Opposite of BFS

---

## When to use DFS?
- Path existence
- Connected components
- Tree traversal
- Backtracking problems

---

## Core idea
Visit node → go to neighbor → go deeper → repeat

---

## Pattern (recursive)

def dfs(node):
    mark visited
    for neighbor:
        if not visited:
            dfs(neighbor)

---

## Pattern (search with early stop)

def dfs(node):
    if node == target:
        return True

    mark visited

    for neighbor:
        if not visited:
            if dfs(neighbor):
                return True

    return False

---

## Key rules
1. Always track visited
2. Explore ALL neighbors
3. Return recursive results properly

---

## DFS vs BFS

DFS:
- Stack / recursion
- Goes deep

BFS:
- Queue
- Level by level

---

## Common mistakes
- Forgetting visited
- Returning too early
- Not returning recursive result
- Mixing BFS with DFS

---

## Mental model
DFS = go deep, if stuck → backtrack

---

## Final Python Implementation

```python
n = 5
target = 4
edges = [
    [0,1],
    [0,2],
    [1,3],
    [3,4]
]

graph = [[] for _ in range(n)]
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)

visited = [False] * n

def has_path(node):
    if node == target:
        return True

    visited[node] = True

    for nei in graph[node]:
        if not visited[nei]:
            if has_path(nei):
                return True

    return False

print(has_path(0))
