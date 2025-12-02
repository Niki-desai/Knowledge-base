# Complexity Analysis: Advanced

## 1. Formal Definitions

*   **Big O ($O$)**: Upper Bound. $f(n) \in O(g(n))$ if $f(n) \le c \cdot g(n)$ for large $n$. (Worst Case).
*   **Big Omega ($\Omega$)**: Lower Bound. $f(n) \ge c \cdot g(n)$. (Best Case).
*   **Big Theta ($\Theta$)**: Tight Bound. $c_1 \cdot g(n) \le f(n) \le c_2 \cdot g(n)$.

## 2. Amortized Analysis (Dynamic Array)

Why is `append()` O(1) if we sometimes resize (O(N))?

**Aggregate Method Proof**:
Let's double capacity when full.
*   Insert 1: Cost 1
*   Insert 2: Cost 1
*   Insert 3: Cost 1 (Resize 2->4, Copy 2) + 1 = 3
*   Insert 4: Cost 1
*   Insert 5: Cost 1 (Resize 4->8, Copy 4) + 1 = 5
*   ...
*   Insert $2^k + 1$: Cost $2^k$ (Copy) + 1.

Total Cost for $N$ insertions includes copying $1 + 2 + 4 + ... + N/2$ elements.
Sum of geometric series $\sum_{i=0}^{\log N} 2^i = 2N - 1$.
Total work $\approx 3N$.
Amortized Cost = $\frac{Total Work}{N} = \frac{3N}{N} = O(1)$.

## 3. Recursion & Space Complexity

Space complexity of recursion is determined by the **Maximum Depth of the Call Stack**.

### Example 1: Binary Search (Recursive)
```python
def binary_search(arr, l, r, target):
    if l > r: return -1
    mid = (l + r) // 2
    if arr[mid] == target: return mid
    if arr[mid] < target: return binary_search(arr, mid+1, r, target)
    return binary_search(arr, l, mid-1, target)
```
*   **Depth**: $\log N$
*   **Space**: $O(\log N)$ (Stack frames)

### Example 2: Sum of Array (Recursive)
```python
def sum_arr(arr, idx):
    if idx == len(arr): return 0
    return arr[idx] + sum_arr(arr, idx+1)
```
*   **Depth**: $N$
*   **Space**: $O(N)$ (Often causes Stack Overflow for large N)

## 4. Auxiliary vs Total Space

*   **Auxiliary Space**: Extra space used by the algorithm (excluding input).
*   **Total Space**: Auxiliary Space + Input Space.

**Example**: Merge Sort
*   Input: O(N)
*   Auxiliary: O(N) (Temp arrays)
*   Total: O(N)

**Example**: In-place Heap Sort
*   Input: O(N)
*   Auxiliary: O(1)
*   Total: O(N)
