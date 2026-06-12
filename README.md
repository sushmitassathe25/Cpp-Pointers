# Cpp-Advanced

A comprehensive guide to advanced C++ concepts and best practices.

## Table of Contents

- [Smart Pointers](#smart-pointers)
  - [Unique Pointer](#unique-pointer)
  - [Shared Pointer](#shared-pointer)
  - [Weak Pointer](#weak-pointer)

---

## Smart Pointers

Smart pointers are objects that act like pointers but provide automatic memory management. They help prevent memory leaks and dangling pointers by automatically deallocating memory when it's no longer needed.

### Types of Smart Pointers:

---

### Unique Pointer

**File:** [unique_pointer.md](https://github.com/sushmitassathe25/Cpp-Advanced/blob/main/Unique%20pointers.md)

`std::unique_ptr` is a smart pointer that exclusively owns and manages memory. Key features:

- **Exclusive Ownership**: Only one `unique_ptr` can own a resource at a time
- **Automatic Cleanup**: Memory is automatically deleted when the `unique_ptr` goes out of scope
- **Move Semantics**: Ownership can be transferred using move semantics
- **Zero Overhead**: No reference counting overhead
- **Non-copyable**: Cannot be copied, only moved

**Use Cases:**
- Owning heap-allocated objects with clear ownership semantics
- Resource management with RAII pattern
- When you need the fastest smart pointer with no shared ownership overhead

**Example:**
```cpp
std::unique_ptr<MyClass> ptr(new MyClass());
// or with make_unique (preferred)
auto ptr = std::make_unique<MyClass>();
```

---

### Shared Pointer

**File:** [shared_pointer.md](./shared_pointer.md)

`std::shared_ptr` is a smart pointer that allows multiple pointers to share ownership of the same memory. Key features:

- **Shared Ownership**: Multiple `shared_ptr` instances can own the same resource
- **Reference Counting**: Automatically tracks the number of owners
- **Automatic Cleanup**: Memory is deleted when the last `shared_ptr` is destroyed
- **Thread-Safe Reference Counting**: Reference count operations are atomic
- **Copy Semantics**: Can be freely copied

**Use Cases:**
- Sharing ownership of resources across multiple objects
- Complex data structures with multiple references
- Callback scenarios where lifetime is uncertain

**Example:**
```cpp
std::shared_ptr<MyClass> ptr1(new MyClass());
std::shared_ptr<MyClass> ptr2 = ptr1;  // Both own the same object
// Object is deleted when both ptr1 and ptr2 go out of scope
```

---

### Weak Pointer

**File:** [weak_pointer.md](./weak_pointer.md)

`std::weak_ptr` is a smart pointer that holds a non-owning reference to an object managed by a `shared_ptr`. Key features:

- **Non-owning Reference**: Does not affect the reference count of the managed object
- **Prevents Circular References**: Essential for breaking circular references
- **Must Convert to shared_ptr**: Must use `lock()` to safely access the resource
- **Automatic Invalidation**: Becomes invalid when the last `shared_ptr` is destroyed
- **Safety**: Accessing an expired `weak_ptr` returns null safely

**Use Cases:**
- Breaking circular references in data structures
- Caching with weak references to avoid memory leaks
- Observer patterns where objects shouldn't prevent cleanup
- Parent-child relationships where children shouldn't keep parents alive

**Example:**
```cpp
std::shared_ptr<MyClass> shared = std::make_shared<MyClass>();
std::weak_ptr<MyClass> weak = shared;  // Non-owning reference

// To access the object safely:
if (auto ptr = weak.lock()) {
    // Object still exists
    ptr->doSomething();
} else {
    // Object has been deleted
}
```

---

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

## Quick Links

- [Unique Pointer Documentation](./unique_pointer.md)
- [Shared Pointer Documentation](./shared_pointer.md)
- [Weak Pointer Documentation](./weak_pointer.md)

---

## Best Practices

1. **Prefer `make_unique` and `make_shared`** over `new` for exception safety
2. **Use `unique_ptr` by default** for exclusive ownership
3. **Use `shared_ptr` only when necessary** to avoid reference counting overhead
4. **Use `weak_ptr` to break circular references** in shared_ptr scenarios
5. **Avoid raw pointers** for ownership semantics
6. **Avoid circular references** which can lead to memory leaks even with shared_ptr

---

*Last Updated: June 2026*
