# Comprehensive Guide to Sorting Algorithms

Sorting is a fundamental operation in computer science used to arrange data in a
specific order. This summary covers the key algorithms we discussed, their logic,
and their Python implementations.

## 1. Simple Sorting Algorithms - $O(n^2)$

These algorithms are intuitive, but they become inefficient as data size
increases.

### Bubble Sort

**Concept:** Repeatedly steps through the list, compares neighboring elements,
and swaps them if they are in the wrong order.

**Visual:** The largest elements "bubble up" to the end of the list.

### Insertion Sort

**Concept:** Builds a sorted section one item at a time by inserting each new
element into its correct position.

**Best for:** Small datasets or lists that are already nearly sorted.

## 2. Divide and Conquer Algorithms - $O(n \log n)$

These algorithms are significantly faster and are used in real-world
applications.

### Merge Sort

**Concept:** Recursively splits the list in half until each sub-list has one
element, then merges the sub-lists back together in order.

**Key feature:** Predictable performance, but requires extra memory for the
sub-lists.

### Quick Sort

**Concept:** Picks a pivot and partitions the array into two sub-arrays:
elements less than the pivot and elements greater than the pivot.

**Key feature:** Highly efficient and can sort in place to save memory.

## 3. Comparison Summary

| Algorithm | Best Case | Worst Case | Memory Usage |
| --- | --- | --- | --- |
| Bubble Sort | $O(n)$ | $O(n^2)$ | Low, in place |
| Insertion Sort | $O(n)$ | $O(n^2)$ | Low, in place |
| Merge Sort | $O(n \log n)$ | $O(n \log n)$ | High, extra space |
| Quick Sort | $O(n \log n)$ | $O(n^2)$ | Low, in place |

## 4. Python Implementations

### Bubble Sort

```python
def bubble_sort(arr):
    n = len(arr)

    for i in range(n):
        swapped = False

        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True

        if not swapped:
            break

    return arr
```

### Insertion Sort

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1

        while j >= 0 and key < arr[j]:
            arr[j + 1] = arr[j]
            j -= 1

        arr[j + 1] = key

    return arr
```

### Merge Sort

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)


def merge(left, right):
    result = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] < right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])
    result.extend(right[j:])

    return result
```

### Quick Sort

```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr

    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]

    return quick_sort(left) + middle + quick_sort(right)
```
