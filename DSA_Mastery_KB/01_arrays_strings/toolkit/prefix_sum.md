# Toolkit: Prefix Sum (Advanced)

## 1. 2D Prefix Sum (Matrix)

**Problem**: Calculate sum of rectangle defined by `(r1, c1)` and `(r2, c2)`.
**Preprocessing**: `P[i][j] = P[i-1][j] + P[i][j-1] - P[i-1][j-1] + matrix[i][j]`.

```python
class NumMatrix:
    def __init__(self, matrix):
        rows, cols = len(matrix), len(matrix[0])
        self.P = [[0] * (cols + 1) for _ in range(rows + 1)]
        
        for r in range(rows):
            for c in range(cols):
                self.P[r+1][c+1] = (self.P[r][c+1] + self.P[r+1][c] 
                                  - self.P[r][c] + matrix[r][c])

    def sumRegion(self, r1, c1, r2, c2):
        return (self.P[r2+1][c2+1] 
              - self.P[r1][c2+1] 
              - self.P[r2+1][c1] 
              + self.P[r1][c1])
```

## 2. Difference Array (Range Updates)

**Problem**: Increment range `[l, r]` by `val` efficiently.
**Technique**: `Diff[l] += val`, `Diff[r+1] -= val`.
**Reconstruction**: Prefix Sum of `Diff` array gives the final array.

```python
def range_updates(n, updates):
    diff = [0] * (n + 1)
    
    for l, r, val in updates:
        diff[l] += val
        diff[r + 1] -= val
        
    # Reconstruct
    res = [0] * n
    curr = 0
    for i in range(n):
        curr += diff[i]
        res[i] = curr
    return res
```

## 3. Modulo Arithmetic

**Problem**: Subarray Sum divisible by `k`.
**Logic**: `(P[j] - P[i]) % k == 0` implies `P[j] % k == P[i] % k`.
**Pattern**: Store `prefix_sum % k` in Hash Map.

```python
def subarrays_div_by_k(nums, k):
    count = {0: 1}
    prefix = 0
    res = 0
    
    for num in nums:
        prefix = (prefix + num) % k
        # Handle negative modulo in Python
        if prefix < 0: prefix += k
        
        res += count.get(prefix, 0)
        count[prefix] = count.get(prefix, 0) + 1
        
    return res
```
