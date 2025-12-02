# Toolkit: Two Pointers (Advanced)

## 1. 3-Way Partitioning (Dutch National Flag)

**Problem**: Sort array of 0s, 1s, and 2s in one pass.
**Logic**: Three pointers. `low` (0 boundary), `mid` (scanner), `high` (2 boundary).

```python
def sort_colors(nums):
    low, mid, high = 0, 0, len(nums) - 1
    
    while mid <= high:
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low += 1
            mid += 1
        elif nums[mid] == 1:
            mid += 1
        else: # nums[mid] == 2
            nums[high], nums[mid] = nums[mid], nums[high]
            high -= 1
            # Do NOT increment mid, we need to check swapped value
```

## 2. Floyd's Cycle Detection (Tortoise and Hare)

**Problem**: Detect cycle in Linked List or Array (Duplicate Number).
**Logic**: Fast pointer moves 2x speed. If cycle exists, they will meet.

```python
def has_cycle(head):
    slow, fast = head, head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```

**Find Start of Cycle**:
1.  Detect meeting point.
2.  Reset `slow` to head. Keep `fast` at meeting point.
3.  Move both 1 step at a time. They meet at cycle start.

## 3. Trapping Rain Water

**Problem**: Compute how much water can be trapped.
**Logic**: Water at index `i` = `min(max_left, max_right) - height[i]`.
**Optimization**: Two Pointers. Maintain `left_max` and `right_max`.

```python
def trap(height):
    l, r = 0, len(height) - 1
    left_max, right_max = 0, 0
    res = 0
    
    while l < r:
        if height[l] < height[r]:
            if height[l] >= left_max: left_max = height[l]
            else: res += left_max - height[l]
            l += 1
        else:
            if height[r] >= right_max: right_max = height[r]
            else: res += right_max - height[r]
            r -= 1
    return res
```
