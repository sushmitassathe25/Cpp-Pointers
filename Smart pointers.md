#Smart Pointers

In C++, **Pointers** are same as the variables, which stores the address of the another variable. This is the standard defintion of Pointers in C++, but think of the issues with this standard pointers are, they hold the memory address, but they don't know who owns the memory, how many people are using it, or when it should be cleaned up. And if you forgot to delete it, you get a **memory leak**. And if you delete it early, you will get a **dangling pointer** etc.

So, In modern C++ i.e. C++11 and onwards Smart pointers are introduced. 

**Smart Pointers** are wrappers around the raw pointer that adds up the layer of intelligence, primarily by managing the lifetime of object they pooint to. And They automatically deallocates the memory when it is no longer needed i.e automatica garbage collection behavior.

1. Unique pointer
2. Shared pointer
3. Weak pointer
