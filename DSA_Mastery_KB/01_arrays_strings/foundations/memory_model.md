# Memory Model: Arrays & Strings

Understanding the low-level memory layout is what separates a junior developer from a senior engineer. It impacts performance (Cache Locality) and behavior (Immutability).

## 1. Arrays & Cache Locality

An **Array** is a contiguous block of memory. This contiguity is the key to its performance, not just for O(1) access, but for **CPU Caching**.

### The CPU Cache Hierarchy
When the CPU needs data, it fetches a "Cache Line" (usually 64 bytes) from RAM.
*   **Array**: Fetching `arr[0]` likely fetches `arr[1]`, `arr[2]`, etc., into the L1 Cache. Subsequent accesses are instant.
*   **Linked List**: Nodes are scattered in heap memory. Fetching `node` does NOT pre-fetch `node.next`. This causes "Cache Misses".

**Impact**: Iterating an Array is significantly faster than a Linked List, even if both are O(N), due to spatial locality.

### Python Lists (Dynamic Arrays)
Python lists are arrays of *pointers* (references) to objects.
*   **Layout**: `[Ptr1, Ptr2, Ptr3]` is contiguous.
*   **Indirection**: The actual integers/objects are scattered in the Heap.
*   **Performance**: Slower than C arrays (double indirection), but still faster than Linked Lists.

## 2. Strings & Interning

Strings are immutable sequences of characters.

### String Interning (Python)
Python automatically "interns" (caches) some strings to save memory and speed up comparison.
*   **Identifiers**: Variable names, function names, and short strings looking like identifiers are interned.
*   **Mechanism**: A global table stores unique interned strings.
*   **Comparison**: `a is b` (pointer comparison) works for interned strings and is O(1). `a == b` checks content and is O(N).

```python
import sys

a = "hello"
b = "hello"
print(a is b)  # True (Implicitly interned)

c = "hel" + "lo"
print(a is c)  # True (Compiler optimization)

d = "".join(["h", "e", "l", "l", "o"])
print(a is d)  # False (Created at runtime)

# Explicit Interning
e = sys.intern(d)
print(a is e)  # True
```

## 3. Memory Overhead

How much memory does an integer take?
*   **C/C++**: `int32` = 4 bytes.
*   **Python**: `int` is an object (`PyObject_HEAD` + value).
    *   Small Ints: ~28 bytes.
    *   List of 1M ints: ~8MB (pointers) + ~28MB (objects) = ~36MB.
    *   C Array of 1M ints: ~4MB.

## 4. Stack vs Heap

| Feature | Stack | Heap |
| :--- | :--- | :--- |
| **Allocation** | Automatic (Function call) | Manual / GC (Object creation) |
| **Access Speed** | Very Fast (L1 Cache) | Slower (Pointer chasing) |
| **Size** | Small (MBs) | Large (GBs) |
| **Lifetime** | Scope of function | Until GC collects it |

**Recursion Limit**: Deep recursion overflows the Stack (`RecursionError`).
**Memory Leak**: Unreferenced objects on Heap not collected (rare in Python due to Ref Counting + Cycle Detector).
