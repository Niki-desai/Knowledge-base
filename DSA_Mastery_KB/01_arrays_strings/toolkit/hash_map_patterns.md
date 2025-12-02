# Toolkit: Hash Map Patterns (Advanced)

## 1. Rolling Hash (Rabin-Karp)

**Problem**: Find substring pattern in text.
**Logic**: Compute hash of window in O(1) using sliding window.
`H_new = (H_old - val_out * base^L) * base + val_in`.

```python
def search(text, pattern):
    d = 256 # Alphabet size
    q = 101 # Prime number
    M = len(pattern)
    N = len(text)
    p = 0 # Hash pattern
    t = 0 # Hash text
    h = 1 # d^(M-1) % q
    
    for i in range(M-1):
        h = (h * d) % q
        
    # Calculate initial hashes
    for i in range(M):
        p = (d * p + ord(pattern[i])) % q
        t = (d * t + ord(text[i])) % q
        
    for i in range(N - M + 1):
        if p == t:
            if text[i:i+M] == pattern:
                print(f"Match at {i}")
                
        if i < N - M:
            t = (d*(t - ord(text[i])*h) + ord(text[i+M])) % q
            if t < 0: t += q
```

## 2. Sparse Vectors

**Problem**: Dot product of two huge vectors with mostly 0s.
**Storage**: Hash Map `{index: value}`.

```python
class SparseVector:
    def __init__(self, nums):
        self.data = {i: n for i, n in enumerate(nums) if n != 0}

    def dotProduct(self, vec):
        res = 0
        # Iterate over the smaller map for efficiency
        if len(self.data) < len(vec.data):
            for i, val in self.data.items():
                if i in vec.data:
                    res += val * vec.data[i]
        else:
            for i, val in vec.data.items():
                if i in self.data:
                    res += val * self.data[i]
        return res
```

## 3. Isomorphic Strings

**Problem**: Check if `s` can be replaced to get `t`.
**Logic**: Two Maps `s->t` and `t->s`.

```python
def isIsomorphic(s, t):
    map_s_t = {}
    map_t_s = {}
    
    for c1, c2 in zip(s, t):
        if (c1 in map_s_t and map_s_t[c1] != c2) or \
           (c2 in map_t_s and map_t_s[c2] != c1):
            return False
        map_s_t[c1] = c2
        map_t_s[c2] = c1
        
    return True
```
