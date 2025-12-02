# Basic Operations: Advanced Patterns

## 1. Python Slicing Internals

Slicing `arr[start:end]` creates a **Shallow Copy**.

*   **Time**: $O(K)$ where $K = end - start$.
*   **Space**: $O(K)$.

**Implication**:
```python
# Bad: Recursive slicing
def sum_rec(arr):
    if not arr: return 0
    return arr[0] + sum_rec(arr[1:]) # Creates copy at every step!
```
**Total Time**: $N + (N-1) + ... + 1 = O(N^2)$.
**Fix**: Pass indices.

## 2. Sentinel Values

Avoid boundary checks by adding a "Sentinel" (Dummy) value at the start or end.

**Scenario**: Search in array.
Normal:
```python
for i in range(len(arr)):
    if arr[i] == target: return i
```
Sentinel (Micro-optimization, avoids `i < len` check in low-level langs):
```python
arr.append(target) # Sentinel
i = 0
while arr[i] != target:
    i += 1
if i == len(arr) - 1: return -1 # Found sentinel only
return i
```

**Scenario**: Linked List Operations (Dummy Head).
Simplifies inserting/deleting at the head.
```python
dummy = ListNode(0)
dummy.next = head
curr = dummy
# Now curr.next is always safe to access initially
```

## 3. The `bisect` Module

Python's built-in Binary Search.

*   `bisect_left(a, x)`: First index to insert `x` to maintain order. (First occurrence of `x`).
*   `bisect_right(a, x)`: Last index + 1. (Index after last occurrence of `x`).

**Count occurrences in Sorted Array**:
```python
from bisect import bisect_left, bisect_right

def count_occurrences(arr, x):
    return bisect_right(arr, x) - bisect_left(arr, x)
```

## 4. Custom Sorting (Lambda Power)

Sort complex objects using `key`.

**Scenario**: Sort strings by length, then alphabetically.
```python
strs = ["apple", "bat", "car", "banana"]
strs.sort(key=lambda s: (len(s), s))
# Result: ['bat', 'car', 'apple', 'banana']
```

**Scenario**: Sort intervals by start time.
```python
intervals = [[1,3], [2,6], [8,10], [15,18]]
intervals.sort(key=lambda x: x[0])
```
