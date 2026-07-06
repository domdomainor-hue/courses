# Sliding Window Cheat Sheet

## When to use

Use Sliding Window when the problem involves:

* Contiguous subarray / substring
* Max / Min length
* Max / Min sum
* Count of valid windows

---

## Core idea

```python
expand right
while invalid:
    shrink left
update answer
```

---

## 1. Fixed-size window

Use when size is exactly `k`

Example:

* Max sum subarray of size k
* Permutation / Anagram in string

Template:

```python
def fixed_window(arr, k):
    window_sum = 0
    best = float('-inf')

    for right in range(len(arr)):
        window_sum += arr[right]

        if right >= k:
            window_sum -= arr[right-k]

        if right >= k-1:
            best = max(best, window_sum)

    return best
```

---

## 2. Variable-size window (Longest)

Use for longest valid subarray / substring

Example:

* Longest subarray sum <= k
* Longest substring with at most K distinct chars
* Longest substring without repeating chars

Template:

```python
def longest_window(arr):
    left = 0

    for right in range(len(arr)):
        add(arr[right])

        while invalid:
            remove(arr[left])
            left += 1

        answer = max(answer, right-left+1)
```

---

## 3. Variable-size window (Smallest)

Use for smallest valid subarray

Example:

* Smallest subarray sum >= k

Template:

```python
def smallest_window(arr):
    left = 0

    for right in range(len(arr)):
        add(arr[right])

        while valid:
            answer = min(answer, right-left+1)
            remove(arr[left])
            left += 1
```

Remember:
Update answer BEFORE shrinking.

---

## Frequency map pattern

Used in string problems.

Template:

```python
freq = {}

for right in range(len(s)):
    freq[s[right]] = freq.get(s[right], 0) + 1

    while invalid:
        freq[s[left]] -= 1
        if freq[s[left]] == 0:
            del freq[s[left]]
        left += 1
```

---

## Common interview problems

### Max sum subarray size k

```python
arr = [2,1,5,1,3,2], k=3
# 9
```

### Longest subarray sum <= k

```python
arr = [2,1,5,1,3,2], k=7
# 3
```

### Smallest subarray sum >= k

```python
arr = [2,1,5,2,3,2], k=7
# 2
```

### Longest substring with at most K distinct chars

```python
s = "eceba", k=2
# 3
```

### Longest substring without repeating chars

```python
s = "abcabcbb"
# 3
```

### Permutation in string

```python
s1 = "ab"
s2 = "eidbaooo"
# True
```

---

## Complexity

Usually:

* Time: O(n)
* Space: O(k) or O(26)

Reason:
Each element enters and leaves the window at most once.

---

## How to recognize quickly

Ask:

1. Is it contiguous?
2. Fixed or variable size?
3. Need longest / smallest / count / exact match?
4. Need HashMap / Counter / Array[26]?

---

## Revision order

1. Fixed-size max sum
2. Longest subarray
3. Smallest subarray
4. At most K distinct
5. Without repeating
6. Anagram / permutation
