    Title: Setting JAVA_HOME for SBT inside Emacs GUI
    Date: 2018-08-10T12:06:41
    Tags: scala,spacemacs

Is your terminal SBT and your Emacs (or Spacemacs) SBT giving you different
results? More confusingly, is SBT doing the correct thing from inside Emacs if
Emacs is started from the terminal and not as a GUI application?

This can be caused by Emacs producing its own environment if started as a
GUI application! (And you thought all environments were created equal, out of
`.bashrc` or whatever ... Nope.)

This new env that Emacs creates for itself is based on ... magic, I guess.

In any case, it does *not* necessarily respect the `JAVA_HOME` that you
carefully set in your normal (terminal) environment.

In order to correct this, add to your `init.el` (or `dotspacemacs/user-init` in
`.spacemacs`):

```elisp
(setenv "JAVA_HOME"
        "/Library/Java/JavaVirtualMachines/jdk1.8.0_172.jdk/Contents/Home/")
```

or whatever it is that `JAVA_HOME` should point to.

As an alternative, you can set this variable per project by adding a file
`.sbtopts` in the project root with the following:

```
-java-home /Library/Java/JavaVirtualMachines/jdk1.8.0_172.jdk/Contents/Home/
```
