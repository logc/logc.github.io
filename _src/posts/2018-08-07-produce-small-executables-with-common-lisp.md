    Title: Produce small executables with Common Lisp
    Date: 2018-08-07T11:14:18
    Tags: lisp

You need [roswell](https://github.com/roswell/roswell),
[ECL](https://common-lisp.net/project/ecl/) and [UPX](https://upx.github.io/)
installed. 

    $ brew install --HEAD roswell
    $ brew install ecl upx
    
The reason to install the head roswell is that, as of this writing, some of
the Common Lisp installations did not work on the latest release on Homebrew.
    
The reason to install `ecl` on the system, and not using `ros install`, is that
I had problems installing the latest version of ECL that way. Also, the `ros
list versions` command did not find any available versions of ECL. So, I
resorted to using the system installed ECL.

    $ ros use ecl/system

<!-- more -->

Initialize a new roswell script

    $ ros init hello

Then add the following to `hello.ros`

```lisp
#!/bin/sh
#|-*- mode:lisp -*-|#
#|
exec ros -Q -- $0 "$@"
|#
(progn ;;init forms
  ;;(ros:ensure-asdf)
  ;;#+quicklisp(ql:quickload '() :silent t)
  )

(defpackage :ros.script.hello.3742624530
  (:use :cl))
(in-package :ros.script.hello.3742624530)


(defun main (&rest argv)
  (declare (ignorable argv))
  (format t "Hello world!~%"))
;;; vim: set ft=lisp lisp:
```

Compile into an executable using

    $ ros build hello.ros

The executable `hello` is now 1.6 MB on my system (Darwin 17.7.0)

You can bring this down to 457 KB using

    $ upx --best hello

The "hello" executable is now small, but not really fast

    $ time ./hello
    Hello world!
            0.41 real         0.40 user         0.02 sys
