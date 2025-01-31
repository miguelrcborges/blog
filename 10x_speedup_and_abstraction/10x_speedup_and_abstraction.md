# How I made a program I wrote 10 times faster

and how abstraction is enemy of performance.



## How it started

I recently developed an interest in functional programming and
decided to explore OCaml by using it to solve the latest Advent
of Code challenges.

In order to keep the habit of programming OCaml, I decided to keep solving
problems on [Project Euler](https://projecteuler.net/) routinely.

I was solving the [Problem no. 10](https://projecteuler.net/problem=10) and,
once I solved it with a few optimization work (handrolled my own Growable array
for faster insertions at the end), I was skeptical of its speed: "Is it normal
that it takes 40 seconds to solve this?".

Here is the code I had ended up:

```ocaml
module DynamicArray =
   struct 
      type 'a t = { 
         mutable arr : 'a array;
         mutable length : int;
         (* capacity is stored in arr *)
      }

      let create initial_capacity value = {
         arr = Array.make initial_capacity value;
         length = 0;
      }

      let resize da =
         let new_capacity = max 1 (2 * Array.length da.arr) in
         let new_arr = Array.make new_capacity da.arr.(0) in
         Array.blit da.arr 0 new_arr 0 da.length;
         da.arr <- new_arr

      let add value da =
         if da.length = Array.length da.arr then resize da;
         da.arr.(da.length) <- value;
         da.length <- da.length + 1

      let for_all predicate da =
         let rec aux i =
            if i >= da.length then true
            else if not (predicate da.arr.(i)) then false
            else aux (i + 1)
         in
         aux 0

      let exists predicate da =
         let rec aux i =
            if i >= da.length then false
            else if predicate da.arr.(i) then true
            else aux (i + 1)
         in
         aux 0

      let iter f da = 
         let rec aux i = 
            if i >= da.length then ()
            else
               begin
                  f da.arr.(i) ;
                  aux (i + 1)
               end
         in
         aux 0

      let fold f init da =
         let rec aux i acc = 
            if i >= da.length then acc
            else aux (i + 1) (f acc da.arr.(i))
         in
         aux 0 init
   end
;;

let get_primes_array limit =
   let arr = DynamicArray.create 4 0 in
   List.iter (fun x -> DynamicArray.add x arr) [2; 3; 5; 7] ;
   for i = 11 to limit do
      if DynamicArray.for_all (fun e -> i mod e <> 0) arr then 
         DynamicArray.add i arr
   done;
   arr
;;

let primes = get_primes_array 2000000 ;;
let result = DynamicArray.fold (+) 0 primes ;;
Printf.printf "%d\n" result;
```

**Note**: The DynamicArray module has some more methods that it needed because
some of them were used while testing (iter) and others because it feels goofy
not having them (`exists` when `for_all` is defined)

**Note 2**: Yes I know this isn't the best algorithm to find primes, but I
wanted one that would be easily implemented and at the same time deterministic.
(And yes, I could have stopped the iteration and sqrt(N) instead of the last
prime).



## Rewriting in C

Once I found out weird the runtime performance of the OCaml implementation,
I translated it to equivalent C code:

```c
#include <stdlib.h>
#include <stdio.h>
#include <stddef.h>

const static size_t limit = 2000000;

typedef struct {
    size_t *values;
    size_t length;
    size_t capacity;
} DynArray;

void DynArray_Add(DynArray *da, size_t v) {
    if (da->length == da->capacity) {
        size_t new_capacity = da->capacity == 0 ? 1 : da->capacity * 2;
        da->capacity = new_capacity;
        da->values = realloc(da->values, da->capacity * sizeof(da->values[0]));
    }
    da->values[da->length] = v;
    da->length += 1;
}

void TestPrime(DynArray *primes, size_t val) {
    for (size_t i = 0; i < primes->length; i += 1) {
        if (val % primes->values[i] == 0) {
            return;
        }
    }
    DynArray_Add(primes, val);
}

int main(void) {
    DynArray primes = {0};
    DynArray_Add(&primes, 2);
    DynArray_Add(&primes, 3);
    DynArray_Add(&primes, 5);
    DynArray_Add(&primes, 7);

    for (size_t i = 11; i < limit; i += 1) {
        TestPrime(&primes, i);
    }

    size_t sum = 0;
    for (size_t i = 0; i < primes.length; i += 1) {
        sum += primes.values[i];
    }
    printf("%zu\n", sum);
    
    return 0;
}
```

**Note 3**: Yes, I know that in the C version the dynamic array doesn't start
with any preallocated slots for the dynamic array. Those microseconds difference
won't matter in the final outcode and I want to do [ZII](https://www.youtube.com/watch?v=xt1KNDmOYqA).

Granted, after compiling the code with optimizations (both OCaml and C were being compiled with `-O3`),
found out that the C and OCaml version were at the same speed.



## But can we do better?

After facing this results, I wondered if I could make it better.

When looking at the code, I assumed that the code could be memory bottlenecked,
so I went down to compress the memory bandwidth required during the inner loop.

**Note 4**: Assumptions are **BAD**! I should have been this confident on this
take. It wasn't harmful to me to investigate such option because it was still a
possible case, but not take a possibility for granted!

```c
#include <stdlib.h>
#include <stdio.h>
#include <stddef.h>
#include <stdint.h>

const static size_t limit = 2000000;

typedef struct {
    uint32_t *values;
    size_t length;
    size_t capacity;
} DynArray;

void DynArray_Add(DynArray *da, uint32_t v) {
    if (da->length == da->capacity) {
        size_t new_capacity = da->capacity == 0 ? 1 : da->capacity * 2;
        da->capacity = new_capacity;
        da->values = realloc(da->values, da->capacity * sizeof(da->values[0]));
    }
    da->values[da->length] = v;
    da->length += 1;
}

void TestPrime(DynArray *primes, uint32_t val) {
    for (size_t i = 0; i < primes->length; i += 1) {
        if (val % primes->values[i] == 0) {
            return;
        }
    }
    DynArray_Add(primes, val);
}

int main(void) {
    DynArray primes = {0};
    DynArray_Add(&primes, 2);
    DynArray_Add(&primes, 3);
    DynArray_Add(&primes, 5);
    DynArray_Add(&primes, 7);

    for (uint32_t i = 11; i < limit; i += 2) {
        TestPrime(&primes, i);
    }

    size_t sum = 0;
    for (size_t i = 0; i < primes.length; i += 1) {
        sum += primes.values[i];
    }
    printf("%zu\n", sum);
    
    return 0;
}
```

Since the maximum prime could fit into a 32 bits unsigned integer (would also
fit in a signed, but who cares), I could reduce the memory that would be
fetched during the loop by half.

When I ran this version, I was surprised to see no speed up (which in some way
*humbled* me).

Seeing these results, I was able to see it wasn't a memory bottleneck but a
compute one (however, the reduction of memory bandwidth will still be useful
when we get to improve the compute).


## Vectorizing the code

Since processors aren't able to increase nowadays their clock speed, their trick
to improve performance is to increase the core count and the number of
operations the processor can do per clock (a combination of IPC (instructions
per clock) and vector instructions).

Our current C code has two limitations that restricts it of making the inner loop
(the TestPrime one) vectorizable:
- It early returns when it finds a condition that doesn't match. 
- It does an integer division (integer division and modulo are the same instruction) which doesn't have a vector equivalent instruction.

Although integer division has no direct vector equivalent, floating-point division does. 

To check if two numbers are divisible using floats, I can do the following:

```
double d_dividend = (double)dividend;
double d_divisor = (double)divisor;
double q = d_dividend / d_divisor; 
double q_int_part = trunc(q) // This is the equivalent to the integer division, but would be a floating number
bool are_divisible = d_dividend = q_int_part * d_divisor // There are equal if there would be no remainder
```

In order to hint the compiler to vectorize these operations, I can just create a nested loop.

This is the final result:

```c
#include <stdlib.h>
#include <stdio.h>
#include <stddef.h>
#include <stdint.h>
#include <math.h>
#include <stdbool.h>


const static size_t limit = 2000000;

typedef struct {
    uint32_t *values;
    size_t length;
    size_t capacity;
} DynArray;

void DynArray_Add(DynArray *da, uint32_t v) {
    if (da->length == da->capacity) {
        size_t new_capacity = da->capacity == 0 ? 1 : da->capacity * 2;
        da->capacity = new_capacity;
        da->values = realloc(da->values, da->capacity * sizeof(da->values[0]));
    }
    da->values[da->length] = v;
    da->length += 1;
}

void TestPrime(DynArray *primes, uint32_t val) {
    size_t i = 0;
    while (i + 7 < primes->length) {
        double tmp[8];
        double dval = val;
        for (size_t ii = 0; ii < 8; ii += 1) {
            tmp[ii] = primes->values[i + ii];
        }
        bool should_return = false;
        for (size_t ii = 0; ii < 8; ii += 1) {
            double q = trunc(dval / tmp[ii]);
            double r = dval - q * tmp[ii];
            should_return = dval == q * tmp[ii] ? true : should_return; 
        }
        if (should_return) {
            return;
        }
        i += 8;
    }
    while (i < primes->length) {
        if (val % primes->values[i] == 0) {
            return;
        }
        i += 1;
    }
    DynArray_Add(primes, val);
}

int main(void) {
    DynArray primes = {0};
    DynArray_Add(&primes, 2);
    DynArray_Add(&primes, 3);
    DynArray_Add(&primes, 5);
    DynArray_Add(&primes, 7);

    for (uint32_t i = 11; i < limit; i += 2) {
        TestPrime(&primes, i);
    }

    size_t sum = 0;
    for (size_t i = 0; i < primes.length; i += 1) {
        sum += primes.values[i];
    }
    printf("%zu\n", sum);
    
    return 0;
}
```


## Results

Well, if you have read the title, you should know what to expect.

Executables with "avx2" in the name were compiled with the "-mavx2" flag. Every executable was compiled with "-O3".

```
Benchmark 1: c1.exe 
    Time (mean ± σ):        40.546 s ±  0.015 s    [User: 40.530 s, System: 0.007 s] 
    Range (min … max):      40.518 s … 40.570 s    10 runs 

Benchmark 2: c1avx2.exe 
    Time (mean ± σ):        40.553 s ±  0.007 s    [User: 40.546 s, System: 0.007 s] 
    Range (min … max):      40.541 s … 40.564 s    10 runs

Benchmark 3: c2.exe 
    Time (mean ± σ):        40.528 s ±  0.008 s    [User: 40.511 s, System: 0.003 s] 
    Range (min … max):      40.510 s … 40.540 s    10 runs 

Benchmark 4: c2avx2.exe 
    Time (mean ± σ):        40.515 s ±  0.011 s    [User: 40.511 s, System: 0.004 s] 
    Range (min … max):      40.497 s … 40.531 s    10 runs

Benchmark 5: c3avx2.exe 
    Time (mean ± σ):         3.783 s ±  0.003 s    [User: 3.774 s, System: 0.010 s] 
    Range (min … max):       3.778 s …  3.786 s    10 runs

Benchmark 6: ocaml1.exe 
    Time (mean ± σ):        40.630 s ±  0.008 s    [User: 40.621 s, System: 0.016 s] 
    Range (min … max):      40.620 s … 40.646 s    10 runs
```


I tried making an OCaml version equal to the new C version (even though it is
pointless since OCaml doesn't do vectorization).

Here how it looks like (note: it runs worse than the original)

```ocaml
module DynamicArray =
   struct 
      type 'a t = { 
         mutable arr : 'a array;
         mutable length : int;
         (* capacity is stored in arr *)
      }

      let create initial_capacity value = {
         arr = Array.make initial_capacity value;
         length = 0;
      }

      let resize da =
         let new_capacity = max 1 (2 * Array.length da.arr) in
         let new_arr = Array.make new_capacity da.arr.(0) in
         Array.blit da.arr 0 new_arr 0 da.length;
         da.arr <- new_arr

      let add value da =
         if da.length = Array.length da.arr then resize da;
         da.arr.(da.length) <- value;
         da.length <- da.length + 1

      let for_all predicate da =
         let rec aux i =
            if i >= da.length then true
            else if i + 7 < da.length then
               begin
                  let b1 = predicate da.arr.(i) in
                  let b2 = predicate da.arr.(i+1) in
                  let b3 = predicate da.arr.(i+2) in
                  let b4 = predicate da.arr.(i+3) in
                  let b5 = predicate da.arr.(i+4) in
                  let b6 = predicate da.arr.(i+5) in
                  let b7 = predicate da.arr.(i+6) in
                  let b8 = predicate da.arr.(i+7) in
                  if b1 && b2 && b3 && b4 && b5 && b6 && b7 && b8 then aux (i + 8)
                  else false
               end
            else if predicate da.arr.(i) then aux (i + 1)
            else false
         in
         aux 0

      let exists predicate da =
         let rec aux i =
            if i >= da.length then false
            else if predicate da.arr.(i) then true
            else aux (i + 1)
         in
         aux 0

      let iter f da = 
         let rec aux i = 
            if i >= da.length then ()
            else
               begin
                  f da.arr.(i) ;
                  aux (i + 1)
               end
         in
         aux 0

      let fold f init da =
         let rec aux i acc = 
            if i >= da.length then acc
            else aux (i + 1) (f acc da.arr.(i))
         in
         aux 0 init
   end
;;

let get_primes_array limit =
   let arr = DynamicArray.create 4 0l in
   List.iter (fun x -> DynamicArray.add x arr) [2l; 3l; 5l; 7l] ;
   let rec loop i =
      let i_float = float_of_int i in
      let predicate divisor =
         let divisor_float = Int32.to_float divisor in
         let q = i_float /. divisor_float in
         let q_t = Float.floor q in
         i_float <> q_t *. divisor_float
      in
      if DynamicArray.for_all predicate arr then 
         DynamicArray.add (Int32.of_int i) arr
      else () ;
      if i >= limit then ()
      else loop (i + 2)
   in
   loop 11 ;
   arr
;;

let primes = get_primes_array 2000000 ;;
let result = DynamicArray.fold (fun acc el -> acc + (Int32.to_int el)) 0 primes ;;
Printf.printf "%d\n" result;
```

I could have written a version that doesn’t rely on passing
predicates, but that would clash with the functional
programming ethos. At that point, I might as well not be using
OCaml at all.


## Comments

1. Even though FP makes it easier to trivial parallize some code (e.g. maps over a container or folding if your accumulation function is associative), it doesn't make it obvious for vectorization. 
2. The same trivialness can be achieved in procedural languages (you can just pass a function pointer). You don't need a whole language relying on the idea of functions.
3. SIMD is crazy. 
4. OCaml isn't as fast as I expected (is faster).
    - Still can't touch optimized code.
5. C compilers aren't magic so you have to give them a hand.
6. If you want your compiler to vectorize:
    - Make sure your data is continuous.
    - You are using instructions that can be used on vectors (no integer division!)
    - Make all conditional code be reliant on ternaries instead of ifs.
    - No early returns, at least not in the vectorizing section: create a nested loop for the vector part.
7. Use ISPC if you want to make sure your compiler will vectorize (it forces you to modelate your program for vectorization).
8. I wish the main Go compiler had vectorization.
    - Even in Python you can import numpy and have a SPMD programming approach.
9. Even though I used FP in this example, OOP paradigms wouldn't be better.
