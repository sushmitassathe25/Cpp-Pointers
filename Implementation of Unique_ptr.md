# My Own version of unique_ptr implementation

Here’s an example implementation of a simple unique pointer class in C++:

```cpp
#include <iostream>

using namespace std;

template <typename T>
class UniquePtr {
    private:
        T* ptr_; // pointer member to hold the resource

    public:
        //constructor
        explicit UniquePtr(T* ptr = nullptr) : ptr_(ptr) {}

        // destructor
        ~UniquePtr() {
            cout<<"Destructor called, releasing resource..."<<endl;
            delete ptr_; // release the resource
        }

        // move constructor
        UniquePtr(UniquePtr&& other) noexcept : ptr_(other.ptr_) {
            cout<<"Move constructor called, transferring ownership..."<<endl;
            other.ptr_ = nullptr; // transfer ownership
        }

        // move assignment operator
        UniquePtr& operator=(UniquePtr&& other) noexcept {  
            cout<<"Move assignment operator called, transferring ownership..."<<endl;
            if (this != &other) { // check for self-assignment
                delete ptr_; // release current resource
                ptr_ = other.ptr_; // transfer ownership
                other.ptr_ = nullptr; // transfer ownership
            }
            return *this;
        }

        // delete copy constructor and copy assignment operator to prevent copying
        UniquePtr(const UniquePtr&) = delete;   
        UniquePtr& operator=(const UniquePtr&) = delete; 

        //operator overloads
        // overload dereference operator
        T& operator*() const {
            return *ptr_;
        }

        // overload arrow operator
        T* operator->() const {
            return ptr_;
        }
        
        //Member fucntions
        //get the raw pointer
        T* get() const {
            return ptr_;
        }

        //release ownership of the resource and return the raw pointer
        T* release() {  
            cout<<"Releasing ownership of the resource..."<<endl;      
            T* temp = ptr_;
            ptr_ = nullptr; // release ownership
            return temp; // return the raw pointer
        }

        //reset the UniquePtr to manage a new resource 
        void reset(T* ptr = nullptr) {
            cout<<"Resetting UniquePtr to manage a new resource..."<<endl;
            delete ptr_; // release current resource
            ptr_ = ptr; // manage new resource
        }

        // check if the UniquePtr is managing a resource
        bool isNull() const {
            return ptr_ == nullptr;
        }

        //move semantics for function parameters
        void transferOwnership(UniquePtr&& other) { 
            cout<<"Transferring ownership from another UniquePtr..."<<endl;
            if (this != &other) { // check for self-assignment
                delete ptr_; // release current resource
                ptr_ = other.ptr_; // transfer ownership
                other.ptr_ = nullptr; // transfer ownership
            }
        }
};

int main() {
    // create a UniquePtr managing an integer
    UniquePtr<int> ptr1(new int(42)); 
    cout<<"Value in ptr1: "<<*ptr1<<endl; 

    // transfer ownership to ptr2 using move semantics
    UniquePtr<int> ptr2 = std::move(ptr1); 
    cout<<"Value in ptr2 after move: "<<*ptr2<<endl; 

    if (ptr1.isNull()) {
        cout<<"ptr1 is now null after move."<<endl;
    }

    // reset ptr2 to manage a new integer
    ptr2.reset(new int(100)); 
    cout<<"Value in ptr2 after reset: "<<*ptr2<<endl; // dereference to get the new value

    // create an empty UniquePtr and transfer ownership from ptr2
    UniquePtr<int> ptr3; 

    ptr3.transferOwnership(std::move(ptr2)); // transfer ownership from ptr2 to ptr3
    cout<<"Value in ptr3 after transferring ownership: "<<*ptr3<<endl; 

    if (ptr2.isNull()) {
        cout<<"ptr2 is now null after transferring ownership."<<endl;
    }
    return 0;
}
```

---

Above class is a simplified version of the std::unique_ptr class in the C++ standard library. It implements the basic functionality of a unique pointer, including memory management, move semantics, and pointer access operations. The class has a templated type parameter T that specifies the type of the object being managed by the pointer.

The copy constructor and copy assignment operator are explicitly deleted to prevent the class from being copied, as unique pointers are designed to be unique and non-sharable. The move constructor and move assignment operator are implemented to allow the transfer of ownership of the pointer to another unique pointer. And overloaded dereference and arrow operators. Along with the set of member functions for accessing and manipulating the pointer.

`explicit :` As the constructor uses the **explicit** keyword, which prevents dangerous, accidental implicit conversion.

`= delete :` (to copy constructor and copy assignment operator) explicitly instruct the compiler to block copying at compile time.

`noexcept :` (on move operations i.e move constructor and move assignment operator) This optimization flag informs the standard library containers (like vector) that it is completely safe to move your smart pointer around during container reallocations without throwing runtime exceptions.





