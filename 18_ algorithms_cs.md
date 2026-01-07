# Algorithms and computer science stuff

Geeky stuff, concepts and questions i met along the way. **90% of those** are not asked on a regulartech interview.

## Trees

### 🌳 Binary Tree

- Each node has ≤ 2 children
- No balancing rules

```plaintext
1
 \
  2
   \
    3
     \
      4
```

| Operation | Complexity        |
| --------- | ----------------- |
| Search    | `O(N)` worst-case |
| Insert    | `O(N)` worst-case |

### 🌳 Binary Search Tree (BST)

(! This is still not B-tree, cont be confused)

| Operation | Best       | Worst  |
| --------- | ---------- | ------ |
| Search    | `O(log N)` | `O(N)` |
| Insert    | `O(log N)` | `O(N)` |

### 🌳 AVL Tree (self-balancing BST)

- **Strict balancing** as core idea
- Height difference between **subtrees ≤ 1**
- **Fast reads**
- Slow writes - Rotations on insert/delete

| Operation | Complexity |
| --------- | ---------- |
| Search    | `O(log N)` |
| Insert    | `O(log N)` |
| Delete    | `O(log N)` |

> 💡 Named after soviet scientists

### 🌳 Red–Black Tree (self-balancing BST)

- Balance is **less strict than AVL**
- Guarantees max height ≤ `2 * log N`
- **Fast writes**
- Slightly **slower reads** thatn AVL

| Operation | Complexity |
| --------- | ---------- |
| Search    | `O(log N)` |
| Insert    | `O(log N)` |
| Delete    | `O(log N)` |

### 🌳 B-tree

Key differences

❗ Not binary

```
B-tree ≠ binary
Binary tree ≠ B-tree
```

- Each node has **many children**
- Optimized for **disk IO**

| Operation | Complexity |
| --------- | ---------- |
| Search    | `O(log N)` |
| Insert    | `O(log N)` |
