#Smart Pointers

In C++, **Pointers** are same as the variables, which stores the memory address of the another variable. This is the standard defintion of Pointers. So this raw pointers isn't fail because they are inherently bad, but it's because they lack Semantic meaning. The raw pointer holds the memory address, but they don't know who owns the memory, how many people are using it, when it should be cleaned up or who is responsible for cleaning it up. So, if you forgot to delete it, you will get a **memory leak**. And if you delete it early, you will get a **dangling pointer**.

So, In modern C++ starting from C++11, it relies heavily on **RAII (Resource Acquisition is Initialization)**, This means an object's lifetime is tightly bound to its stack scope. When a stcak object dies, its destructor automatically cleans up its heap reasources. The main principle of RAII is to give ownership of any heap-allocated resource to stack-allocated object whose destructor contains the code to delete or free the resource and also any associated cleanup code. Such objects include dynamically allocated memory or system object handles.

**Smart Pointers** are wrappers around the raw pointer that adds up the layer of intelligence, primarily by managing the lifetime of object they pooint to. And They automatically deallocates the memory when it is no longer needed i.e automatic garbage collection behavior.

They are defined in the **std** namespace in the <memory> header file. They are designed to be as efficient as possible both in terms of memory and performance.

By wrapping the raw pointers inside the smart pointer classes, we turn memory management from an active developer chore into a passive compiler gurantee(so Smart Pointers is perfect example of **Encapsulation**).



1. Unique pointer
2. Shared pointer
3. Weak pointer
