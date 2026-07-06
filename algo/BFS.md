# BFS Cheat Sheet (Concise & Practical)

---

## One-line Memory Summary

### BFS graph shortest path
```
Queue + dist = shortest steps
```

### BFS grid shortest path
```
Same BFS, node becomes (row, col)
```

### Number of Islands
```
Every new unvisited land starts one BFS
```

### Rotting Oranges
```
All sources in queue, each level = 1 minute
```

---

## Super Short Revision Block

```
BFS = queue

Shortest path:
    dist[start] = 0
    dist[next] = dist[cur] + 1

Grid:
    use (r,c) and 4 directions

Islands:
    scan all cells
    new unvisited land => count++, BFS flood fill

Rotting oranges:
    queue all rotten first
    process by levels
    each level = 1 minute
```

---

## 1. BFS on Graph (Shortest Distance)

**Use when**
- Graph is unweighted
- Need minimum steps

**Pattern**
```
dist = [-1]
dist[start] = 0
queue = [start]

while queue:
    u = pop
    for v in neighbors:
        if dist[v] == -1:
            dist[v] = dist[u] + 1
            push v
```

---

## 2. BFS on Grid (Shortest Path)

**Use when**
- Matrix/grid
- Moves have equal cost

**Directions**
```
(1,0), (-1,0), (0,1), (0,-1)
```

**Pattern**
```
dist[][] = -1
dist[start] = 0
queue = [(r,c)]

while queue:
    r,c = pop
    for each direction:
        if valid and not visited:
            dist[nr][nc] = dist[r][c] + 1
            push
```

---

## 3. Number of Islands (Flood Fill)

**Use when**
- Count connected components in grid

**Pattern**
```
for each cell:
    if land and not visited:
        count += 1
        BFS/DFS flood fill
```

---

## 4. Rotting Oranges (Multi-source BFS)

**Use when**
- Multiple starting points
- Spread over time

**Pattern**
```
queue = all rotten
fresh = count fresh

while queue and fresh > 0:
    process one level
    minutes += 1

if fresh > 0: return -1
```

---

## 5. Core BFS Questions

- Shortest path? → use dist[]
- Connected components? → use visited[]
- Grid? → use (r,c) + directions
- Spread over time? → multi-source BFS

---

## 6. Universal BFS Template

```
queue = [start]
mark start

while queue:
    u = pop
    for v in neighbors:
        if not visited:
            mark visited
            push v
```

---

## 7. Grid BFS Template

```
dirs = [(1,0), (-1,0), (0,1), (0,-1)]

while queue:
    r,c = pop
    for dr,dc:
        if valid:
            process
```

---

## 8. Common Mistakes

- Forgetting visited/dist
- Wrong bounds in grid
- Not pushing neighbor to queue
- Counting time per node instead of per level

---

# Python Implementations

## BFS Graph (Shortest Path)

```python
from collections import deque

def bfs(n, start, graph):
    dist = [-1] * n
    dist[start] = 0
    q = deque([start])

    while q:
        u = q.popleft()
        for v in graph[u]:
            if dist[v] != -1:
                continue
            dist[v] = dist[u] + 1
            q.append(v)

    return dist
```

---

## BFS Grid (Shortest Path)

```python
from collections import deque

def shortest_path(grid):
    n, m = len(grid), len(grid[0])
    dist = [[-1] * m for _ in range(n)]
    dirs = [(1,0), (-1,0), (0,1), (0,-1)]

    if grid[0][0] == 1:
        return -1

    q = deque([(0, 0)])
    dist[0][0] = 0

    while q:
        r, c = q.popleft()
        for dr, dc in dirs:
            nr, nc = r + dr, c + dc
            if nr < 0 or nc < 0 or nr >= n or nc >= m:
                continue
            if grid[nr][nc] == 1 or dist[nr][nc] != -1:
                continue
            dist[nr][nc] = dist[r][c] + 1
            q.append((nr, nc))

    return dist[n-1][m-1]
```

---

## Number of Islands

```python
from collections import deque

def num_islands(grid):
    n, m = len(grid), len(grid[0])
    visited = [[False]*m for _ in range(n)]
    dirs = [(1,0), (-1,0), (0,1), (0,-1)]

    def bfs(sr, sc):
        q = deque([(sr, sc)])
        visited[sr][sc] = True

        while q:
            r, c = q.popleft()
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if nr < 0 or nc < 0 or nr >= n or nc >= m:
                    continue
                if grid[nr][nc] == 0 or visited[nr][nc]:
                    continue
                visited[nr][nc] = True
                q.append((nr, nc))

    count = 0
    for i in range(n):
        for j in range(m):
            if grid[i][j] == 1 and not visited[i][j]:
                count += 1
                bfs(i, j)

    return count
```

---

## Rotting Oranges

```python
from collections import deque

def oranges_rotting(grid):
    n, m = len(grid), len(grid[0])
    q = deque()
    fresh = 0
    dirs = [(1,0), (-1,0), (0,1), (0,-1)]

    for i in range(n):
        for j in range(m):
            if grid[i][j] == 2:
                q.append((i, j))
            elif grid[i][j] == 1:
                fresh += 1

    minutes = 0

    while q and fresh > 0:
        for _ in range(len(q)):
            r, c = q.popleft()
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if nr < 0 or nc < 0 or nr >= n or nc >= m:
                    continue
                if grid[nr][nc] != 1:
                    continue
                grid[nr][nc] = 2
                fresh -= 1
                q.append((nr, nc))
        minutes += 1

    return minutes if fresh == 0 else -1
```
