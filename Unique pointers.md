# Unique Pointers


# Unique Pointers

As name suggests it is used for strict **Exclusive Ownership** i.e only one pointer can manage the resource. Copying is strictly forbidden by the compiler, though ownership can be transferred/moved.

std::unique_ptr is the simplest and most efficient smart pointer in modern C++. When the piece of heap memory is wrapped inside the unique_ptr, then that pointer becomes the sole owner of that memory and no other pointer can share or copy it.

A unique_ptr can only be moved with std::move. This means that the ownership of the memory resources is transferred to another unique_ptr and the original unique_ptr is no longer owns it.

unique_ptr ties the lifetime of heap allocation directly to the lifetime of stack variable using the RAII. If your functiom exists early because of return statement or thrown exception, C++ compiler guarantees that the unique_ptr destructor will run preventing any chances of a memory leak.   





## What is a Unique Pointer?

A **Unique Pointer** (`std::unique_ptr`) is a smart pointer that maintains exclusive ownership of a dynamically allocated object. It ensures that only one unique pointer can own a resource at any given time. When the unique pointer goes out of scope, it automatically deallocates the memory.

**Key Characteristics:**
- **Exclusive Ownership**: Only one unique_ptr can own the object at a time
- **Move Semantics**: Ownership can be transferred using move semantics
- **RAII Principle**: Resource is freed when the pointer goes out of scope
- **No Copying**: Copying is explicitly deleted to maintain uniqueness
- **Zero-Overhead**: No runtime performance cost compared to raw pointers

---

## Visual Representation: Unique Pointer Lifecycle

```mermaid
graph TD
    A["Stack Scope"] --> B["unique_ptr created"]
    B --> C["Object allocated on heap"]
    C --> D{"Still in scope?"}
    D -->|Yes| E["Access object via ptr"]
    E --> D
    D -->|No| F["Destructor called"]
    F --> G["Memory deallocated"]
    G --> H["Pointer destroyed"]
```

---

## Exclusive Ownership Concept

```mermaid
graph LR
    subgraph Valid["Valid State: Exclusive Ownership"]
        A["unique_ptr A"] -->|owns| B["Object on Heap"]
    end
    
    subgraph Invalid["Invalid State: Not Allowed"]
        C["unique_ptr A"] -->|owns| D["Object"]
        E["unique_ptr B"] -->|cannot own| D
    end
    
    style Valid fill:#90EE90
    style Invalid fill:#FFB6C6
```

---

## Move Semantics: Transferring Ownership

```mermaid
graph LR
    A["Step 1: Before Move<br/>unique_ptr A<br/>owns Object"] -->|std::move| B["Step 2: Move Operation<br/>Ownership transfers"]
    B --> C["Step 3: After Move<br/>unique_ptr B<br/>owns Object<br/>A = nullptr"]
    
    style A fill:#87CEEB
    style B fill:#FFD700
    style C fill:#90EE90
```

---

## Comparison: Raw Pointer vs Unique Pointer

```mermaid
graph TD
    subgraph Raw["Raw Pointer (Dangerous)"]
        A1["int* ptr = new int<br/>10;"]
        A2["delete ptr<br/>Memory leak if forgotten!"]
        A3["Can create multiple<br/>pointers to same object"]
        A4["Manual memory<br/>management"]
    end
    
    subgraph Unique["Unique Pointer (Safe)"]
        B1["auto ptr =<br/>make_unique<int>10;"]
        B2["Auto deletion<br/>when out of scope"]
        B3["Only one owner<br/>at a time"]
        B4["Automatic memory<br/>management"]
    end
    
    style Raw fill:#FFB6C6
    style Unique fill:#90EE90
```

---

## Code Example: Basic Unique Pointer Usage

```cpp
#include <memory>
#include <iostream>

class Person {
public:
    Person(std::string name) : name(name) {
        std::cout << "Person created: " << name << std::endl;
    }
    ~Person() {
        std::cout << "Person destroyed: " << name << std::endl;
    }
    void greet() {
        std::cout << "Hello, I'm " << name << std::endl;
    }
private:
    std::string name;
};

int main() {
    // Creating a unique pointer
    std::unique_ptr<Person> ptr = std::make_unique<Person>("Alice");
    
    // Using the unique pointer
    ptr->greet();
    
    // Transferring ownership (Move)
    std::unique_ptr<Person> ptr2 = std::move(ptr);
    
    // ptr is now nullptr
    if (!ptr) {
        std::cout << "ptr is now empty" << std::endl;
    }
    
    ptr2->greet();
    
    // Memory automatically freed when ptr2 goes out of scope
    return 0;
}

/* Output:
   Person created: Alice
   ptr is now empty
   Hello, I'm Alice
   Person destroyed: Alice
*/
```

---

## Key Operations with Unique Pointers

```mermaid
graph TD
    A["Unique Pointer Operations"] --> B["get()"]
    A --> C["move()"]
    A --> D["release()"]
    A --> E["reset()"]
    A --> F["operator*"]
    A --> G["operator->"]
    
    B --> B1["Returns raw pointer<br/>Does NOT transfer ownership"]
    C --> C1["Transfers ownership<br/>Source becomes nullptr"]
    D --> D1["Returns raw pointer<br/>Releases ownership"]
    E --> E1["Deletes current object<br/>and takes new one"]
    F --> F1["Dereferences pointer<br/>Access object value"]
    G --> G1["Access member<br/>via pointer"]
```

---

## Ownership Transfer Example

```mermaid
sequenceDiagram
    participant Main
    participant ptr1
    participant ptr2
    participant Heap
    
    Main->>ptr1: Create unique_ptr
    ptr1->>Heap: Allocate Object
    Main->>ptr2: std::move(ptr1)
    ptr1->>ptr2: Transfer ownership
    ptr1->>ptr1: Set to nullptr
    Main->>ptr2: Access object
    ptr2->>Heap: Use object
    Main->>ptr2: ptr2 goes out of scope
    ptr2->>Heap: Delete object
```

---

## When to Use Unique Pointers

```mermaid
graph TD
    A["Use Unique Pointer When:"] --> B["Single owner needed"]
    A --> C["Clear ownership hierarchy"]
    A --> D["Transferring ownership"]
    A --> E["Factory functions<br/>returning new objects"]
    A --> F["RAII pattern<br/>is important"]
    
    style B fill:#90EE90
    style C fill:#90EE90
    style D fill:#90EE90
    style E fill:#90EE90
    style F fill:#90EE90
```

---

## Memory Management: Before and After

```mermaid
graph LR
    subgraph Before["Without Unique Pointer"]
        A1["new Person"]
        A2["... code ..."]
        A3["delete Person<br/>❌ Easy to forget!"]
    end
    
    subgraph After["With Unique Pointer"]
        B1["make_unique<Person>"]
        B2["... code ..."]
        B3["Automatic deletion<br/>✅ Guaranteed!"]
    end
    
    style Before fill:#FFB6C6
    style After fill:#90EE90
```

---

## Best Practices

| Practice | Explanation |
|----------|-------------|
| **Use `std::make_unique`** | Safer than `new`, exception-safe |
| **Use `std::move`** | Transfer ownership explicitly |
| **Avoid manual `delete`** | Never manually delete unique_ptr managed memory |
| **Check for nullptr** | Before dereferencing after move operations |
| **Return by value** | When transferring ownership from functions |
| **Use `get()` for passing** | When you need to pass to functions not managing ownership |

---

## Unique Pointer in Collections

```cpp
std::vector<std::unique_ptr<Person>> people;

// Add objects
people.push_back(std::make_unique<Person>("Alice"));
people.push_back(std::make_unique<Person>("Bob"));

// Access
for (auto& person : people) {
    person->greet();
}

// Automatic cleanup when vector is destroyed
```

**Visual Representation:**

```mermaid
graph LR
    A["Vector"] --> B["unique_ptr 0"]
    A --> C["unique_ptr 1"]
    A --> D["unique_ptr 2"]
    
    B -->|owns| E["Object: Alice"]
    C -->|owns| F["Object: Bob"]
    D -->|owns| G["Object: Charlie"]
    
    style A fill:#87CEEB
    style E fill:#90EE90
    style F fill:#90EE90
    style G fill:#90EE90
```

---

## Summary

Unique pointers provide:
- ✅ **Automatic memory management**
- ✅ **Exception safety**
- ✅ **Clear ownership semantics**
- ✅ **Zero-cost abstraction**
- ✅ **Prevention of memory leaks**

Use them as your default choice for dynamic memory allocation in modern C++!
