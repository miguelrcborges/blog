---
title: "Data Structures and Algorithms 1 - Arrays"
date: 2023-08-06T12:56:35+01:00
draft: false 
---

Data strucutres is an essential topic which mostly defines the quality
of a program. Replacing algorithms to more suited ones should also 
lead to speedups, although their influence will be magnitudes lower than
the speedup there would be replacing the data structures to the
adequate ones.


In this blog, I plan to start a series where I write about a specific
data structure. It should start from most simple data structures to more
complex ones. It should start friendly enough for people which are
just starting out.


# Arrays

Arrays correspond to a continuos block of memory of the same memory size.
This strucuture allows to assign multiple values with the same indentifier,
where you can specify and element with an index.

For a given type T, an implementation of an array should look like this:

```
struct TArray {
	T *arr;
	size_t len;
}
```

There are also native arrays on C, but they fallback to pointers when passing
to functions (reason why most function prototypes which receives arrays receive
separately the array pointer and its count).

```
T arr[] = { ... }
size_t len = sizeof(arr)/(sizeof(arr[0]))
```

C++ tries to solve this problem by implementing the
[array cointainer](https://en.cppreference.com/w/cpp/container/array).

Should also be noted that C supports
[declaring arrays in the function parameter list](https://en.cppreference.com/w/c/language/array).


## Data Structure Analysis

Being a continuous block of memory, it is the data structure which offers
the best memory locality. It is also the most efficient data structure in 
terms of memory consumption.

One of the problems associated with the arrays is that they have a constant
size. Such can be solved with resizable arrays.
C++ [std::vector](https://en.cppreference.com/w/cpp/container/vector) is an
example of a resizable array (and not really of a mathematical vector). In C,
you are able to resize an array with
[realloc](https://en.cppreference.com/w/c/memory/realloc) (this is, if the way
you allocated the array with a compatible function).

The only additional cost of resizable arrays over standard arrays is the on
initialization. Since arrays have a constant size, they can be stack allocated.

Arrays have constant time access. That makes access of any element pretty quick.
Their are however limited on inserting/deleting elements:
- Deleting an element implies moving all values from the removed element until the
end of the array a position backwards.
- Insertions may require to resize the array and will be needed to move elements
(either moving them by a position, or moving them to the new memory allocated
on the resize).


## Usecases

Arrays will be the data structure that most likely you will use more. They are also
common as a component of more specific data structures, like hash tables.

A lot of people believe that 
[your data strucutres should derive everytime of arrays](https://youtu.be/fHNmRkzxHWs?t=2457),
which you should take it with a grain of salt and benchmark to see which
structure fits the best on your scenario.

Arrays should probably be the best option when the amount of working memory is low,
since moving the memory on insertions/deletions will have a small cost.

In higher memory loads, should be considered the amount of needed insertions/deletions.
If it is needed to do frequently this operations, should be considered using a linked list
(the data structure which will be addressed in the next article).
Another possible solution would be a mix of the both structures, a linked list of
arrays.


# Extra resources:w

- [A paper which addresses the value of good memory locality. The paper is a bit complex so don't mind not getting it immediatally](https://www.akkadia.org/drepper/cpumemory.pdf)
- [Wikipedia's Array Page](https://en.wikipedia.org/wiki/Array_(data_structure))
- [C++'s std::array](https://en.cppreference.com/w/cpp/container/array)
- [C++'s std::vector (resizable array)](https://en.cppreference.com/w/cpp/container/vector)