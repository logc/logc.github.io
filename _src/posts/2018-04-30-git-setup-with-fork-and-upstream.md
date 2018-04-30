    Title: Git setup with fork and upstream
    Date: 2018-04-30T11:46:41
    Tags: git

Here is how to setup a fresh clone of a git repository so that it pulls changes
from an upstream, but only pushes by default to another repository (usually,
your own fork of the project).

I use the fish shell. If you use Bash or Zsh it should be straightforward to
define these variables in your environment.

```fish
set gituser logc
set gitproject chat
set gitorg NacionLumpen
```

And this is how to setup the clone:

```fish
git clone git@github.com:$gituser/$gitproject.git
cd $gitproject
git remote add upstream git@github.com:$gitorg/$gitproject.git
git remote rename origin fork
git fetch upstream
git branch --set-upstream-to=upstream/master
git config remote.pushDefault fork
```

As always with git, it is trivial, if your name is 'Linus' :)
