# Cpp-Advanced

A comprehensive guide to advanced C++ concepts and best practices.

## Table of Contents

- [Smart Pointers](#smart-pointers)

---

## Smart Pointers

Smart pointers are objects that act like pointers but provide automatic memory management. They help prevent memory leaks and dangling pointers by automatically deallocating memory when it's no longer needed.

### Types of Smart Pointers:

---

### Unique Pointer

**File:** [Unique pointers.md](https://github.com/sushmitassathe25/Cpp-Advanced/blob/main/Unique%20pointers.md)

`std::unique_ptr` is a smart pointer that exclusively owns and manages memory. 

### Shared Pointer

**File:** [Shared pointer.md](https://github.com/sushmitassathe25/Cpp-Advanced/blob/main/Shared%20pointers.md)

`std::shared_ptr` is a smart pointer that allows multiple pointers to share ownership of the same memory. 

### Weak Pointer

**File:** [Weak pointer.md](https://github.com/sushmitassathe25/Cpp-Advanced/blob/main/Weak%20pointers.md)

`std::weak_ptr` is a smart pointer that holds a non-owning reference to an object managed by a `shared_ptr`. 

## Comparison Table

| Feature | unique_ptr | shared_ptr | weak_ptr |
|---------|-----------|-----------|----------|
| Ownership | Exclusive | Shared | None (non-owning) |
| Reference Counting | No | Yes | No |
| Copyable | No | Yes | Yes |
| Movable | Yes | Yes | Yes |
| Thread-safe | Pointer only | Reference count | No |
| Memory Overhead | Minimal | Moderate (ref count) | Minimal |
| Use for Ownership | ✓ | ✓ | ✗ |
| Use to Break Cycles | ✗ | ✗ | ✓ |

---

*Last Updated: June 2026*
