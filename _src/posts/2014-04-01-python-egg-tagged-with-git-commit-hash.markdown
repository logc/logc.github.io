    Title: Python egg tagged with Git commit hash
    Date: 2014-04-01T15:22:00Z+0200
    Tags: python,git

## SVN

Python's setuptools provide a simple mechanism to tag your built distributions
with the SVN revision they belong to, by creating a `setup.cfg` file along the
normal `setup.py`, and there write

```
[egg_info]
# Add svn revision to the file name
tag_svn_revision = 1
```

## Git

How to do the same for Git? Or in general, with other information? (Date
tagging is also supported by default).

Answer: modify directly the `options` dictionary in the call to `setup`, in
`setup.py`.  There, you can put anything you can compute with Python or system
calls. E.g. for git:

```python
import shlex
from subprocess import check_output

GIT_HEAD_REV = check_output(shlex.split('git rev-parse --short HEAD')).strip()


setup(
    # ... other keys like project name, version, etc ...
    options = dict(egg_info = dict(tag_build = "dev_" + GIT_HEAD_REV)),
)
```

With that setup, distributions (`sdist` or `bdist`) would be tagged with the
string "dev" and the git hash of the latest commit:

```console
$ bin/python setup.py sdist bdist_egg
$ ls dist
pyhello-0.1dev-92ffa06.tar.gz    pyhello-0.1dev_92ffa06-py2.7.egg
```

Whenever you want to build a stable release (without any "dev" tags), just set
the key to empty in your `setup.cfg`, like this:

```
[egg_info]
tag_build =
```
