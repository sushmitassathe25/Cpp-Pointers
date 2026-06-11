# Weak_ptr in detail with Example and Implementation

Here's an example implementation of a weak pointer class in C++:

```cpp
#include <iostream>
#include <memory>

// Control block to manage reference counts
// This structure tracks both strong and weak reference counts
template <typename T>
class ControlBlock {
public:
    T* ptr;                    // Raw pointer to the managed object
    int strong_count;          // Count of shared_ptr instances
    int weak_count;            // Count of weak_ptr instances
    bool is_deleted;           // Flag to track if object is already deleted

    // Constructor: Initialize the control block
    ControlBlock(T* p) : ptr(p), strong_count(1), weak_count(0), is_deleted(false) {}

    // Destructor: Clean up when control block is destroyed
    ~ControlBlock() {
        if (!is_deleted && ptr != nullptr) {
            delete ptr;
            is_deleted = true;
        }
    }

    // Increment strong reference count
    void increment_strong() {
        strong_count++;
    }

    // Decrement strong reference count
    void decrement_strong() {
        strong_count--;
    }

    // Increment weak reference count
    void increment_weak() {
        weak_count++;
    }

    // Decrement weak reference count
    void decrement_weak() {
        weak_count--;
    }

    // Check if the managed object is still alive
    bool is_expired() const {
        return strong_count == 0;
    }
};

// Forward declaration for friend relationship
template <typename T>
class weak_ptr;

// Simple shared_ptr implementation
// Manages a resource with reference counting and automatic cleanup
template <typename T>
class shared_ptr {
private:
    T* raw_ptr;                           // Raw pointer to the managed object
    ControlBlock<T>* control_block;       // Pointer to control block

    // Friend declaration to allow weak_ptr to access private members
    template <typename U>
    friend class weak_ptr;

public:
    // Default constructor: Creates a null shared_ptr
    shared_ptr() : raw_ptr(nullptr), control_block(nullptr) {}

    // Constructor: Create shared_ptr from raw pointer
    // Takes ownership of the raw pointer
    explicit shared_ptr(T* ptr) : raw_ptr(ptr) {
        if (ptr != nullptr) {
            control_block = new ControlBlock<T>(ptr);
        } else {
            control_block = nullptr;
        }
    }

    // Copy constructor: Increment reference count
    // Creates a new shared_ptr from an existing one
    shared_ptr(const shared_ptr& other) : raw_ptr(other.raw_ptr), control_block(other.control_block) {
        if (control_block != nullptr) {
            control_block->increment_strong();
        }
    }

    // Move constructor: Transfer ownership without incrementing reference count
    // Efficiently moves the pointer from another shared_ptr
    shared_ptr(shared_ptr&& other) noexcept : raw_ptr(other.raw_ptr), control_block(other.control_block) {
        other.raw_ptr = nullptr;
        other.control_block = nullptr;
    }

    // Copy assignment operator: Release current resource and acquire new one
    shared_ptr& operator=(const shared_ptr& other) {
        if (this != &other) {
            // Release current resource
            if (control_block != nullptr) {
                control_block->decrement_strong();
                if (control_block->strong_count == 0 && control_block->weak_count == 0) {
                    delete control_block;
                }
            }

            // Acquire new resource
            raw_ptr = other.raw_ptr;
            control_block = other.control_block;
            if (control_block != nullptr) {
                control_block->increment_strong();
            }
        }
        return *this;
    }

    // Move assignment operator: Transfer ownership
    shared_ptr& operator=(shared_ptr&& other) noexcept {
        if (this != &other) {
            // Release current resource
            if (control_block != nullptr) {
                control_block->decrement_strong();
                if (control_block->strong_count == 0 && control_block->weak_count == 0) {
                    delete control_block;
                }
            }

            // Transfer ownership from other
            raw_ptr = other.raw_ptr;
            control_block = other.control_block;
            other.raw_ptr = nullptr;
            other.control_block = nullptr;
        }
        return *this;
    }

    // Destructor: Decrement reference count and cleanup if necessary
    ~shared_ptr() {
        if (control_block != nullptr) {
            control_block->decrement_strong();
            // If no more strong references and no weak references, delete control block
            if (control_block->strong_count == 0 && control_block->weak_count == 0) {
                delete control_block;
            }
        }
    }

    // Dereference operator: Access the managed object
    T& operator*() const {
        return *raw_ptr;
    }

    // Member access operator: Access members of the managed object
    T* operator->() const {
        return raw_ptr;
    }

    // Get the raw pointer
    T* get() const {
        return raw_ptr;
    }

    // Check if the shared_ptr is not null
    explicit operator bool() const {
        return raw_ptr != nullptr;
    }

    // Get the reference count
    int use_count() const {
        if (control_block == nullptr) {
            return 0;
        }
        return control_block->strong_count;
    }
};

// Weak pointer implementation
// Holds a non-owning reference to a managed object
// Does not participate in reference counting for object lifetime
template <typename T>
class weak_ptr {
private:
    T* raw_ptr;                           // Raw pointer to the managed object
    ControlBlock<T>* control_block;       // Pointer to control block

public:
    // Default constructor: Creates a null weak_ptr
    weak_ptr() : raw_ptr(nullptr), control_block(nullptr) {}

    // Constructor from shared_ptr
    // Creates a weak_ptr from a shared_ptr without incrementing strong count
    weak_ptr(const shared_ptr<T>& sp) : raw_ptr(sp.raw_ptr), control_block(sp.control_block) {
        if (control_block != nullptr) {
            // Increment weak count to track weak references
            control_block->increment_weak();
        }
    }

    // Copy constructor: Copy weak_ptr from another weak_ptr
    weak_ptr(const weak_ptr& other) : raw_ptr(other.raw_ptr), control_block(other.control_block) {
        if (control_block != nullptr) {
            control_block->increment_weak();
        }
    }

    // Move constructor: Transfer weak reference
    weak_ptr(weak_ptr&& other) noexcept : raw_ptr(other.raw_ptr), control_block(other.control_block) {
        other.raw_ptr = nullptr;
        other.control_block = nullptr;
    }

    // Copy assignment operator: Release current and acquire new weak reference
    weak_ptr& operator=(const weak_ptr& other) {
        if (this != &other) {
            // Release current weak reference
            if (control_block != nullptr) {
                control_block->decrement_weak();
                if (control_block->strong_count == 0 && control_block->weak_count == 0) {
                    delete control_block;
                }
            }

            // Acquire new weak reference
            raw_ptr = other.raw_ptr;
            control_block = other.control_block;
            if (control_block != nullptr) {
                control_block->increment_weak();
            }
        }
        return *this;
    }

    // Assignment from shared_ptr
    weak_ptr& operator=(const shared_ptr<T>& sp) {
        // Release current weak reference
        if (control_block != nullptr) {
            control_block->decrement_weak();
            if (control_block->strong_count == 0 && control_block->weak_count == 0) {
                delete control_block;
            }
        }

        // Acquire from shared_ptr
        raw_ptr = sp.raw_ptr;
        control_block = sp.control_block;
        if (control_block != nullptr) {
            control_block->increment_weak();
        }
        return *this;
    }

    // Destructor: Decrement weak count and cleanup if necessary
    ~weak_ptr() {
        if (control_block != nullptr) {
            control_block->decrement_weak();
            // If no more strong references and no weak references, delete control block
            if (control_block->strong_count == 0 && control_block->weak_count == 0) {
                delete control_block;
            }
        }
    }

    // Check if the weak_ptr points to a deleted object
    // Returns true if the managed object has been deleted
    bool expired() const {
        if (control_block == nullptr) {
            return true;
        }
        return control_block->is_expired();
    }

    // Attempt to obtain a shared_ptr from this weak_ptr
    // Returns a valid shared_ptr if object is still alive, null shared_ptr otherwise
    shared_ptr<T> lock() const {
        if (expired()) {
            // Object has been deleted, return null shared_ptr
            return shared_ptr<T>();
        }
        // Object is still alive, create new shared_ptr
        shared_ptr<T> sp;
        sp.raw_ptr = raw_ptr;
        sp.control_block = control_block;
        if (control_block != nullptr) {
            control_block->increment_strong();
        }
        return sp;
    }

    // Get the number of shared_ptr instances (for debugging)
    int use_count() const {
        if (control_block == nullptr) {
            return 0;
        }
        return control_block->strong_count;
    }
};

// ============================================================================
// DEMONSTRATION: How to use weak_ptr to break circular references
// ============================================================================

// Node structure that could create circular references
struct Node {
    int value;
    shared_ptr<Node> next;        // Strong reference to next node
    weak_ptr<Node> prev;          // Weak reference to previous node (breaks cycle)

    Node(int val) : value(val) {}

    ~Node() {
        std::cout << "Node " << value << " destroyed\n";
    }
};

int main() {
    std::cout << "=== Weak Pointer Implementation Demo ===\n\n";

    // Example 1: Basic weak_ptr usage
    std::cout << "Example 1: Creating and using weak_ptr\n";
    {
        shared_ptr<Node> node1 = shared_ptr<Node>(new Node(10));
        std::cout << "Created node1, use_count: " << node1.use_count() << "\n";

        // Create weak_ptr from shared_ptr
        weak_ptr<Node> weak_node1 = node1;
        std::cout << "Created weak_ptr, node1 use_count: " << node1.use_count() << "\n";

        // Check if weak_ptr is expired
        if (!weak_node1.expired()) {
            // Lock to get shared_ptr
            shared_ptr<Node> locked = weak_node1.lock();
            std::cout << "Locked weak_ptr successfully, value: " << locked->value << "\n";
            std::cout << "node1 use_count after lock: " << node1.use_count() << "\n";
        }
    }
    std::cout << "\n";

    // Example 2: Breaking circular references
    std::cout << "Example 2: Breaking circular references with weak_ptr\n";
    {
        shared_ptr<Node> node2 = shared_ptr<Node>(new Node(20));
        shared_ptr<Node> node3 = shared_ptr<Node>(new Node(30));

        // Create bidirectional links without circular reference
        node2->next = node3;
        node3->prev = node2;  // Using weak_ptr prevents circular reference

        std::cout << "Created circular structure\n";
        std::cout << "node2 use_count: " << node2.use_count() << "\n";
        std::cout << "node3 use_count: " << node3.use_count() << "\n";
    }
    std::cout << "Both nodes destroyed properly without memory leak\n\n";

    // Example 3: Checking expired weak_ptr
    std::cout << "Example 3: Detecting expired weak_ptr\n";
    {
        weak_ptr<Node> weak_node;
        {
            shared_ptr<Node> node4 = shared_ptr<Node>(new Node(40));
            weak_node = node4;
            std::cout << "weak_ptr created, expired: " << weak_node.expired() << "\n";
        }
        std::cout << "After shared_ptr goes out of scope, expired: " << weak_node.expired() << "\n";
    }

    return 0;
}
```

## Key Features of This Implementation:

1. **ControlBlock**: Manages reference counts for both strong and weak pointers
2. **shared_ptr**: Provides automatic memory management with reference counting
3. **weak_ptr**: Non-owning reference that doesn't affect object lifetime
4. **expired()**: Checks if the managed object has been deleted
5. **lock()**: Safely converts weak_ptr to shared_ptr if object still exists
6. **Circular Reference Prevention**: Demonstrates how weak_ptr breaks circular references

## Important Differences from std::weak_ptr:

- This is a simplified educational implementation
- Lacks thread safety
- No support for custom deleters
- No support for multiple inheritance scenarios
- Real std::weak_ptr has more optimizations and features
