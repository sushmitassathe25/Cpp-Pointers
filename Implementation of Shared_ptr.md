# My Own version of unique_ptr implementation

Here’s an example implementation of a simple Shared pointer class in C++:

```cpp
#include<iostream>

using namespace std;

template<typename T>
class SharedPtr {
private:
    // Pointer to the managed object and reference count to manage shared ownership
    T* ptr;
    int* refCount;

public :
    //constructor to initialize the shared pointer with a raw pointer
    SharedPtr(T* ptr = nullptr) : ptr(ptr), refCount(new int(1)) {}

    //copy constructor to create a new shared pointer that shares ownership of the same object
    SharedPtr(const SharedPtr& other) : ptr(other.ptr), refCount(other.refCount) {
        (*refCount)++; // Increment the reference count
    }   

    //assignment operator to assign one shared pointer to another, managing reference counts appropriately
    SharedPtr& operator=(const SharedPtr& other) {
        if (this != &other) { // Check for self-assignment
            // Decrement the current reference count and delete if it reaches zero
            if (--(*refCount) == 0) {
                delete ptr;
                delete refCount;
            }
            // Copy the pointer and reference count from the other shared pointer
            ptr = other.ptr;
            refCount = other.refCount;
            (*refCount)++; // Increment the reference count for the new ownership
        }
        return *this; // Return the current object to allow for chained assignments
    }

    //destructor to clean up the managed object and reference count when the shared pointer goes out of scope
    ~SharedPtr() {      
        if (--(*refCount) == 0) { // Decrement the reference count and check if it reaches zero
            delete ptr; // Delete the managed object
            delete refCount; // Delete the reference count
        }
    }

    //overload the dereference operator to allow access to the managed object
    T& operator*() {
        return *ptr; // Return a reference to the managed object
    }

    //overload the arrow operator to allow access to the members of the managed object
    T* operator->() {   
        return ptr; // Return the pointer to the managed object
    }

    //function to get the current reference count for debugging or informational purposes
    int use_count() const { 
        return *refCount; // Return the current reference count
    }

    //function to check if the shared pointer is managing an object (i.e., not null)
    bool unique() const {
        return *refCount == 1; // Return true if the reference count is 1, indicating unique ownership
    }           

    //function to reset the shared pointer, releasing ownership of the current object and optionally taking ownership of a new object
    void reset(T* newPtr = nullptr) {
        // Decrement the current reference count and delete if it reaches zero
        if (--(*refCount) == 0) {
            delete ptr; // Delete the managed object
            delete refCount; // Delete the reference count
        }
        // Assign the new pointer and reset the reference count
        ptr = newPtr; // Set the new pointer
        refCount = new int(1); // Reset the reference count to 1 for the new object
    }

};

int main()
{
    SharedPtr<int> sp1(new int(10)); // Create a shared pointer managing an integer with value 10
    cout << "Value: " << *sp1 << ", Reference Count: " << sp1.use_count() << endl; // Output the value and reference count of sp1  
    SharedPtr<int> sp2 = sp1; // Create another shared pointer that shares ownership of the same integer
    cout << "Value: " << *sp2 << ", Reference Count: " << sp2.use_count() << endl; // Output the value and reference count of sp2 (should be the same as sp1)
    sp1.reset(); // Reset sp1, releasing ownership of the integer   
    cout << "After resetting sp1:" << endl; // Output a message indicating that sp1 has been reset
    cout << "sp1 is " << (sp1.unique() ? "unique" : "shared") << ", Reference Count: " << sp1.use_count() << endl; // Output the ownership status and reference count of sp1 (should indicate it's not managing an object)
    cout << "sp2 is " << (sp2.unique() ? "unique" : "shared") << ", Reference Count: " << sp2.use_count() << endl; // Output the ownership status and reference count of sp2 (should indicate it's still managing the integer)
    
    return 0;
}
```

---

Above class is a simplified version of the std::shared_ptr class in the C++ standard library. Here the SharedPtr class template takes a type parameter T representing the type of the managed object. The class has two private member variables, ptr and refCount, which respectively point to the managed object and a reference count to manage shared ownership.

The constructor of the class takes a raw pointer to the managed object and initializes the reference count to 1. The copy constructor and copy assignment operator increment the reference count and copy the pointer to the managed object when a new shared pointer is created from an existing one. The destructor decrements the reference count and deletes the managed object when the last shared pointer goes out of scope.

Shared pointers are compatible with the standard C++ library algorithms and containers. But Shared pointers are not suitable for managing certain types of resources, such as file handles or network connections, which require specialized management. Also Shared pointers are not thread-safe by default. If multiple threads access the same shared pointer simultaneously, there is a risk of a data race or other concurrency-related issues.
