# Toolkit: Sliding Window (Advanced)

## 1. "At Most K" Pattern

**Problem**: Longest substring with **at most** `k` distinct characters.
**Logic**: Standard dynamic sliding window. Shrink when `count > k`.

```python
def at_most_k(s, k):
    count = {}
    left = 0
    res = 0
    
    for right in range(len(s)):
        count[s[right]] = count.get(s[right], 0) + 1
        
        while len(count) > k:
            count[s[left]] -= 1
            if count[s[left]] == 0:
                del count[s[left]]
            left += 1
            
        res = max(res, right - left + 1)
    return res
```

## 2. "Exactly K" Pattern

**Problem**: Number of subarrays with **exactly** `k` distinct integers.
**Logic**: It's hard to maintain "exactly k" directly.
**Trick**: `Exactly(K) = AtMost(K) - AtMost(K-1)`.

```python
def subarrays_with_k_distinct(nums, k):
    return at_most_k(nums, k) - at_most_k(nums, k - 1)

def at_most_k(nums, k):
    count = {}
    left = 0
    res = 0
    for right in range(len(nums)):
        count[nums[right]] = count.get(nums[right], 0) + 1
        while len(count) > k:
            count[nums[left]] -= 1
            if count[nums[left]] == 0:
                del count[nums[left]]
            left += 1
        res += (right - left + 1) # Count subarrays ending at right
    return res
```

## 3. Handling Negative Numbers (Why Sliding Window Fails)

**Scenario**: Subarray Sum equals `k`.
*   **If Positive Only**: Sliding Window works (Sum increases monotonically).
*   **If Negatives Allowed**: Sum fluctuates. Window logic breaks.
*   **Solution**: Use **Prefix Sum + Hash Map**.

## 4. Optimization: Index Mapping

Instead of shrinking one by one (`left += 1`), jump `left` to the next valid position.
**Scenario**: Longest Substring Without Repeating Characters.

```python
def lengthOfLongestSubstring(s):
    seen = {} # char -> index
    left = 0
    res = 0
    
    for right, char in enumerate(s):
        if char in seen and seen[char] >= left:
            left = seen[char] + 1 # Jump!
            
        seen[char] = right
        res = max(res, right - left + 1)
    return res
```
