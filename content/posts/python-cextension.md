---
title: "Python: C extensions"
date: 2023-01-10T21:18:34+02:00
draft: false
---

# Python series, part 2: C Extensions

In the second part of the Python series, we will benchmark our Python code and see if we can speed it up
by writing a C extension. For this purpose, I wrote a function which, given a lower and upper bound,
returns the number of prime numbers in the given range. Also I wrote a small, simple function to measure
how much it takes to run the function ten times. The code is pretty straightforward, see below:

```python
import time

lower = 1_000_000
upper = 1_001_000

def time_measured(func):
    start = time.perf_counter()

    for i in range(1, 11):
        func(lower, upper)

    duration = time.perf_counter() - start

    return duration

def count_primes_py(lower, upper):
    count = 0

    for num in range(lower, upper+1):
        if num > 1:
            for i in range(2, num):
                if (num % i) == 0:
                    break
                elif (i+1) == num:
                    count += 1

    return count
```

As you can see, I chose rather big numbers for the number range, so that the program doesn't finish too fast.
Also, if you take a closer look inside the count_primes_py function, inside the second range, I wrote it as:

```python
for i in range(2, num):
    if (num % i) == 0:
        break
    elif (i+1) == num:
        count += 1
```

This is actually not optimal in Python since Python has a for-else construct that, on my machine for the given example,
basically runs twice as fast:

```python
for i in range(2, num):
    if (num % i) == 0:
        break
else:
    count += 1
```

I chose the former because I can map it easily to C for a fair comparison. Spoiler: The C extension would still run
much faster, even if I used the for-else construct! That being out the way, let's look at our C code. Before we
look at the function itself, let's first look at some boilerplate we have to setup. If you want to dig deeper
for yourself, here is the official [API reference](https://docs.python.org/3/c-api/index.html) and here is a more
gentle, also official [introduction](https://docs.python.org/3/extending/extending.html). We will cover only
a small subset of the API and won't touch topics like reference counting. Maybe in another post in the future!

```C
#define PY_SSIZE_T_CLEAN
#include <Python.h>

static PyMethodDef methods[] = {
    { "count_primes_c", count_primes_c, METH_VARARGS, "Counting primes!" },
    { NULL, NULL, 0, NULL}
};

static struct PyModuleDef primenumbers = {
    PyModuleDef_HEAD_INIT,
    "primenumbers",
    "Prime numbers within a given range",
    -1,
    methods
};

PyMODINIT_FUNC PyInit_primenumbers(){
    return PyModule_Create(&primenumbers);
};
```
As you can see, we first define PY_SSIZE_T_CLEAN and include the Python.h header. You should include it first,
before any other includes. From the official docs:

```
Since Python may define some pre-processor definitions which affect the standard headers on some systems,
you must include Python.h before any standard headers are included.
It is recommended to always define PY_SSIZE_T_CLEAN before including Python.h
```

As you can see, first we specify our methods. '{ NULL, NULL, 0, NULL}' just gets included, we can ignore it.
In our own definition you can see that we want our method to be called count_primes_c and we will use that exact
same name. Next we pass the METH_VARAGS flag, which in most cases will be correct, but there are other options,
feel free to check out the API reference! And last we pass a description as a string.
Next we have our module definition. As you can see it's a static struct with the following fields:
PyModuleDef_HEAD_INIT is a macro that we include, then follows the name of our module as a string and a description,
also as a string. Then you can see a -1. Here you can give it the, per documentation:

```
size of per-interpreter state of the module, or -1 if the module keeps state in global variables.
```
In our case -1 works just fine. In the end we pass our methods array and that's it for our module definition.
The last thing we have to do is to return our module with PyModule_Create(&primenumbers) with our PyMODINIT_FUNC.
Again, feel free to dig deeper in the API reference. With the boilerplate out of the way, let's actually
implement our count_primes_c function!

```C
PyObject *count_primes_c(PyObject *self, PyObject *args){
    int lower;
    int upper;
    int count = 0;

    PyArg_ParseTuple(args, "ii", &lower, &upper);

    for (int num = lower; num <= upper; num++) {
        if (num > 1) {
            for (int i = 2; i < num; i++) {
                if ((num % i) == 0) {
                    break;
                } else if ((i+1) == num) {
                    count++;
                }
            }
        }
    }

    return PyLong_FromLong((long)count);
};
```

A pretty straightforward implemantation of the equivalent Python function. You can see that the input parameters
and the return type are all PyObjects. We parse the args input with PyArg_ParseTuple(). There, we pass args and
a "magic" string, where i stands for integer. Every type will have one (or more!) chars that represent it. Then
we pass our variables as references. We return a PyObject with the PyLong_FromLong function, where we first have to
cast our int count as long. That's it. We compile the code with
'gcc primenumbers.c -shared -o primenumbers.so -I/usr/include/python3.10' and import it in our Python code with
'import primenumbers'. Now let's measure our pure Python and our C extension implementation with the addition of the
following code inside our Python file:

```Python
# Pure Python ---------------
print("Pure Python version:")
duration_py = time_measured(count_primes_py)
print(f"There are {count_primes_py(lower, upper)} prime numbers between {lower} and {upper}.")
print(f"Running 10 times, on average count_primes_py took {duration_py/10} seconds to run.")
print(f"Altogether that makes it about {int(duration_py)+1} seconds to complete.")
print()
# C  Extension --------------
print("C extension version:")
duration_c = time_measured(primenumbers.count_primes_c)
print(f"There are {primenumbers.count_primes_c(lower, upper)} prime numbers between {lower} and {upper}.")
print(f"Running 10 times, on average count_primes_c took {duration_c/10} seconds to run.")
print(f"Altogether that makes it about {int(duration_c)+1} seconds to complete.")
print()
# Comparison
print("Comparison:")
print(f"The function implemented as a C extension runs about {int((duration_py/10)/(duration_c/10))} times faster.")
```
We call 'python3 primenumbers.py' in our terminal and we get:

```
Pure Python version:
There are 75 prime numbers between 1000000 and 1001000.
Running 10 times, on average count_primes_py took 4.484800785299922 seconds to run.
Altogether that makes it about 45 seconds to complete.

C extension version:
There are 75 prime numbers between 1000000 and 1001000.
Running 10 times, on average count_primes_c took 0.24523699020010098 seconds to run.
Altogether that makes it about 3 seconds to complete.

Comparison:
The function implemented as a C extension runs about 18 times faster.

```
A massive difference! If you recall, in the beginning I told you how the for-else construct in Python leads to the
code being run in about half the time, which would mean the C extension function still runs almost 10 times as fast.
In any case, this shows how, if your code performs cpu heavy tasks, it can be beneficial to profile it and
implement specific functions in C. Although if you are fine with the performance of your Python code you obviously
should not introduce complexity by including a C extension. Next time, we will take a look at decorators in Python.
