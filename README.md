Memory Pool
==========

This is a template class implementation of a memory pool allocator that is very simple to use and extremely fast with minimal overhead for each allocation/deallocation. The provided class is mostly compliant with the C++ Standard Library with a few exceptions (see C++ Compliance for details).

This library uses variadic templates for perfect argument forwarding and some other optimizations, and thus requires C++11 features. I may be willing to implement a C++98 version if somebody really needs it ([send me an email](mailto:coskuacay@gmail.com) if you are interested). 

What is a Memory Pool
-----------------------------------------
You would normally use `malloc` or `new` for dynamic memory management in C/C++. These functions are rather slow and have some memory overhead attached to them. This is fine if you make a few calls and ask for large chunks of memory, but if you need to store many small objects, the time and memory overhead may be unacceptable for high performance programs. This is where a memory pool comes in. 
A memory pool allocates memory in big chunks and splits the memory into smaller pieces. Every time you request memory, one of these small chunks is returned instead making a call to the OS or the heap allocator. You can only use a memory pool if you know the size of the objects beforehand, but if you do, a memory pool has quite a few advantages:
* It is substantially faster than `malloc` or `new`
* There is almost no memory overhead since the size of each object is known beforehand (i.e. no need to store allocation metadata)
* There is little to no memory fragmentation
* You do not need to free object one by one. The allocator will free all the memory it allocated once its destructor is called. Note that this only works if the objects have a default destructor.

A memory pool has just a few disadvantages:
* Objects have a fixed size which must be known beforehand. This is usually not a problem and mostly the case if you need to allocate them in a bunch
* You may need to fine tune them for your specific application. This is made very easy with the use of template classes.

C++ Compliance
-------------------------
MemoryPool is mostly compliant with the C++ Standard Library allocators. This means you can use it with `allocator_traits` ([see here] (http://www.cplusplus.com/reference/memory/allocator_traits/)) or just like you would use the `std::allocator` ([see here] (http://www.cplusplus.com/reference/memory/allocator/)). There are some differences though:
* MemorPool **cannot** allocate multiple objects with a single call to `allocate` and will simply ignore the count value you pass to the allocate/deallocate function. Fixing this is not too hard, but it would deteriorate performance and create memory fragmentation.
* This is **NOT** thread safe. You should create a different instance for each thread (suggested) or find some way of scheduling queries to the allocator.

Usage
-------------------------
Put `MemoryPool.h` and `MemoryPool.tpp` into your project folder and include `MemoryPool.h` into your project. These files define a single template class in the common namespace:
   Template <typename T, size_t BlockSize = 4096>
Here, `T` is the type of the objects you want to allocate and `BlockSize` is the size of the chunks in bytes the allocator will ask from the system. `T` can be any object, while `BlockSize` needs to be at least twice the size of `T`. After that, you create an instance of `MemoryPool` class and use it just like a standard allocator object. Here is an example:
   

How to Pick BlockSize
-------------------------
`BlockSize` is the size of the chunks in bytes the allocator will ask from the system. Picking the correct `BlockSize` (the second, optional, template parameter) is essential for good performance. I suggest you pick a power of two, which may decrease memory fragmentation depending on your system. Also, make sure that `BlockSize` is at least several hundred times larger than the size of `T` for maximum performance. The idea is, the greater the `BlockSize`, the less calls to `malloc` the library will make. However, picking a size too big might increase memory usage unnecessarily and actually decrease the performance because `malloc` may need to make many system calls. For objects that contain several pointers, the default size of 4096 bytes should be good. If you need bigger object, you may need to time your code with larger sizes for `BlockSize`.

About the Code
-------------------------
Here are a few important points that you should know about the code:
* This code handles alignment for the objects automatically, which is necessary for high performance memory access and may even be required on certain processors. However, it assumes the memory returned by `operator new` is "aligned enough" to store a pointer. This assumption was not necessary, but since it is reasonable enough, I decided to remove two lines of code. I am not sure if this is required by the standard, but all C code using `malloc` would crash (or run extremely slowly) if this was not the case.
* The allocator does not free any memory until it is destructed (though it does re-use the memory that you deallocate).  This is fine if you only allocate objects, or if you deallocate and re-allocate objects equally. This is only a problem if you have peaks of high memory usage and you expect the memory to be freed during low memory periods. This was a design choice to substantially improve the performance, and under most cases you do not need to worry about this. It is good to keep this mind for projects that require special handling of the memory though.

License
-------------------------
This code is distributed under the MIT License, which is reproduced below and at the top of the project files. This pretty much means you can do whatever you want with the code, but I will not be liable for ANY kind of damage that this code might cause. Here is the full license which you should read before using the code:
Copyright (c) 2013 Cosku Acay, http://www.coskuacay.com
Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

Contact Me
-------------------------
I am open to any type of suggestions, bug reports, or requests for the code. You can either email me at coskuacay@gmail.com or visit my [website] (http://www.coskuacay.com) to contact me.

