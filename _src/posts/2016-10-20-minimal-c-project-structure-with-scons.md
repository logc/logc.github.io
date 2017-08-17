    Title: Minimal C project structure with SCons
    Date: 2016-10-20T20:17:59
    Tags: C,scons,build

This is a possible project structure in order to have a C project using SCons
as the build system. It enables you to:

- compile most files as a library, and link that to a `main` file with application code.
- separate `src` and `include` directories
- separate unit tests using libcheck
- the unit tests link to the library
- Macports-installed libraries

Layout your code like this:

```
.
├── include
│   └── core
│       └── mylib.h
├── sconstruct
├── src
│   ├── core
│   │   └── mylib.c
│   └── main.c
└── tests
    ├── core
    │   ├── test_mylib.c
    │   ├── test_mylib.h
    │   └── tests.h
    └── main.c
```

where `mylib` should have a more descriptive name for your project.

`sconstruct` is:

```python
env = Environment(CPPPATH=['include', '/opt/local/include'])
env.Library(target='mylib', source=Glob('src/core/*.c'))
env.Program(target='mylib', source=Glob('src/*.c'),
            LIBS=['mylib'], LIBPATH=['.'])
env.Program(target='test_mylib', source=Glob('tests/*.c') + Glob('tests/**/*.c'),
            LIBS=['mylib', 'check'], LIBPATH=['.', '/opt/local/lib/'])
```

<!-- more -->

## Implementation

`src/core/mylib.c` (and all other files in subdirectories of `src/`) holds the
core logic without user interactions (pure functions, if you wish :) ). It can
look like this:

```c
int whatever() {
    return 0;
}
```

`src/main.c` uses all other files in order to do something useful for a user.
For instance:

```c
#include "core/mylib.h"

int main() {
    return whatever();
}
```

`main.c` and `core/mylib.c` see each other via the header files in the include
directory, where `core/mylib.h` holds:

```c
int whatever();
```

## Tests

Tests have the same subdirectory structure as the source subtree (a `core`
directory to hold tests for `core` implementation files), but headers are
included in the same subtree, because tests are not expected to need to publish
their API to end-users, only their results.

A typical test file, e.g. `tests/core/test_mylib.c`, should define tests, and
then define a function that puts them together into a test suite. This is the
function that we will want to expose via a header file:

```c
#include <check.h>

START_TEST(test_fails)
{
    ck_assert_int_eq(1, 2);
}
END_TEST

Suite * core_suite(void)
{
    Suite *s;
    TCase *tc_core;

    s = suite_create("Core/mylib");

    /* Core test case */
    tc_core = tcase_create("Core");

    tcase_add_test(tc_core, test_fails);
    suite_add_tcase(s, tc_core);

    return s;
}
```

In this example, we create a header file for each test file in a directory, and
then an aggregated header file called `tests.h` which includes all other header
files in the same directory:

```c
// test_mylib.h
Suite * core_suite(void);

// tests.h
#include "test_mylib.h"
```

Finally, the test runner executable is defined in the tests/main.c file. It
includes all tests headers, uses the exported test suite creating functions to
create all test suites, and runs them.

```c
// tests/main.c
#include <stdlib.h>

#include <check.h>

#include "core/tests.h"

int main(void)
{
    int number_failed;
    Suite *s;
    SRunner *sr;

    s = core_suite();
    sr = srunner_create(s);

    srunner_run_all(sr, CK_NORMAL);
    number_failed = srunner_ntests_failed(sr);
    srunner_free(sr);
    return (number_failed == 0) ? EXIT_SUCCESS : EXIT_FAILURE;
}
```

## Usage

Build your project with:

```
$ scons
```

You will find three products:

- a static library called `libmylib`
- an executable called `mylib`, which links to that library
- a test executable called `test_mylib`, also linked to that library

You can run all tests with `./test_mylib`, and execute the program with
`./mylib`

You can clean all object files and the products with

```
$ scons --clean
```
