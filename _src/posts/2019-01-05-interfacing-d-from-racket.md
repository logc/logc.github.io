    Title: Interfacing D functions from Racket
    Date: 2019-01-05T16:44:58
    Tags: D, racket

This is how you can use a function written in D from a Racket script.

In a file called `logc.d`:

```d
import std.stdio;

extern(C)
void hello()
{
  writeln("hi from D");
}
```

Compile to a shared library with:

```console
$ ldc2 -shared -m64 logc.d
```

or with

```console
$ dmd -shared -m64 -fPIC -defaultlib=libphobos2.so logc.d
```

Use the generated `liblogc.so` (or `liblogc.dylib`) from a Racket script:

```racket
(require ffi/unsafe)

(define logc-lib (ffi-lib "liblogc"))

(define logc-hello
  (get-ffi-obj "hello" logc-lib
               (_fun -> _void)))
```

Try the results:

```racket
> (logc-hello)
hi from D
```

This post is based on
[Call D from Ruby using FFI](https://wiki.dlang.org/Call_D_from_Ruby_using_FFI)
and
[tutorial using racket's ffi](http://prl.ccs.neu.edu/blog/2016/06/27/tutorial-using-racket-s-ffi/)
