    Title: Reduce in standard C++
    Date: 2014-01-20T13:47:25Z+0100
    Tags: C++

## Accumulate instead of loop

You can traverse a C++ vector in many ways. Looping over a counter variable
that goes from 0 to the vector size or taking an iterator from the vector's
`begin` to its `end` pointer positions are well-known ways to access its
elements. When you are collapsing the elements into a single accumulator value
(an operation usually called `reduce` in functional programming), you can use
the `std::accumulate` function in the `numeric` header.

<!-- more -->


```c++
#include <iostream>
#include <vector>
#include <numeric>

struct cell {
    double probability;
};

int main()
{
    double result;
    std::vector<cell> grid;
    for (size_t i = 0; i < 3; i++) {
        cell c = {.probability = 0.1 * i};
        grid.push_back(c);
    }

    result = std::accumulate(
        grid.begin(), grid.end(), 0.0,
        [] (double sum, const cell& c) {return sum + c.probability;});
    std::cout << result;
    return 0;
}
```

## Compile and run

In this example, the compilation needs a `-std=c++11` flag in order to set the
standard to C++11, a fairly recent standard that is not supported by all
compilers and systems. However, the part that is written in this new standard
is the *accumulating function*, which is expressed as a lambda in this example.
If you move its code to a named function outside the `main`, this code should
compile with many more compilers.

```console
$ c++ main.cpp -o example -std=c++11 && ./example
0.3
```
