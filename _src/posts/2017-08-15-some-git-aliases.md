    Title: Some Git aliases
    Date: 2017-08-15T18:18:11
    Tags: git

Here are some of the Git alias configurations that I use. My philosophy in
writing them was to keep them descriptive, and *not to keep them easy to type*,
following the approach in [this blog
post](http://blog.wittchen.biz.pl/my-approach-to-git-aliases/). In order to keep
my typing short, I use the [fish shell](http://fishshell.com).

```
[alias]
	amend = commit --amend
	discard = checkout --
	docommit = commit --verbose --all
	hist = log --color --pretty=format:\"%C(yellow)%h%C(reset) %s%C(bold red)%d%C(reset) %C(green)%ad%C(reset) %C(blue)[%an]%C(reset)\" --relative-date --decorate
	lastedit = log --pretty=format: --name-only -n 1
	logshort = log --oneline -n 10
    lsmodified = diff --name-only head
	mkbranch = checkout -b
	rmbranch = branch -d
```

The aliases `amend` and `discard` are pretty self-explanatory. `docommit`
reflects my own preferences when finishing a commit: just commit everything that
is currently modified and add a diff under the commit message, so that I can
review what I actually modified everywhere. `hist` is something I don't use very
much; I copied it from somewhere and I keep it around to avoid going to a GUI
when I need to review history in-depth.

`lastedit` lists which files were edited in the last commit. I find it useful to
just open them again in an editor:

```
$ vim (git lastedit)
```

`logshort` is probably the alias I use the most. It just gives me an idea of
what was going on in a branch, without defaulting to show me all history since
the beginning of time. `lsmodified` is a recent addition. It lists all files
modified but not commited. It is intended to be used together with other
aliases:

```
$ git discard (git lsmodified)
```

`mkbranch` and `rmbranch` are also quite useful to me. I tend to see Git as a
file system: sometimes I start to type `ls -l` when I actually need `git
status`. In that mindset, I tend to identify branches with directories, and
therefore creating and deleting them as if they were directories makes sense.
