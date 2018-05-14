    Title: Debug Scala with Spacemacs
    Date: 2018-05-14T16:08:11
    Tags: scala,spacemacs

The [documentation about debugging in
ENSIME](http://ensime.github.io/editors/emacs/userguide/#debugging) was a bit
confusing to me. Here are my notes on how to start debugging an (sbt-based)
Scala project from Emacs (Spacemacs):

1. Launch the debugger from the sbt console first. Use the sbt-mode shell. I
   don't really know if this works launching SBT on a terminal not running under
   Emacs. Use `ensimeTestOnlyDebug` if you are running a test, otherwise use
   `ensimeRunDebug` for an implementation class.

    ```
    > ensimeTestOnlyDebug com.company.division.squad.project.package.TestClass
    ...
    Listening for transport dt_socket at address: 5005
    ```

2. Set a breakpoint in a source code line by moving your cursor to that line and
   using `, d b`

3. Attach to the debugger by using `, d A`. The first time, you will need to
   specify Host (`localhost`) and Port (the number in the `Listening for
   transport ...` line above)
   
4. When (if) the debugger hits the line where your breakpoint is set, the Emacs
   cursor will jump to that position, and you can start using other debugging
   commands (`, d n`, `, d i`, etc ...)
