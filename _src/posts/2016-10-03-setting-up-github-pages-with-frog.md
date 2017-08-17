    Title: Setting up Github Pages with Frog
    Date: 2016-10-03T20:24:03
    Tags: racket,frog,blog

To set up a static blog in Github Pages using the Racket package Frog, you need
to do the following:

- download and install Racket (duh!)
- install Frog with raco:

```
$ raco install frog
```


- create a folder and initialize a Frog project there:

```
$ mkdir new-blog
$ cd new-blog
$ raco frog --init
```

- customize the .frogrc file. Set at least scheme/host to https://your-username.github.io, and title & author to whatever you want.

- add some content of your own:

```
$ raco frog -n "New post"
# ... output says something like:
~/new-blog/_src/posts/2016-10-03-new-post.md
# ... edit that file with your preferred editor ...
when you are finished editing, preview your glorious site with

$ raco frog -bp
```

- initialize an empty Git repository on the project folder, and tell git to
  ignore the Frog machinery and your source directory

```
$ git init .
$ cat << EOF > .gitignore
/.frog/
/.frogrc
/_src/
EOF
```

- commit everything, except for what we just ignored

```
$ git add .
$ git commit
```

- on github.com, create a new repository with the name your-username.github.io.
  Once created, add it as a remote to your local repo, and push your local master
  to it.

```
$ git remote add origin <your new repo>
$ git push -u origin master
```

- after some seconds, your new blog should be available as http://your-username.github.io

Now, each time that you want to add a new post, it’s just a matter of

```
$ raco frog -n "Another post"
# ... edit that post ...
$ raco frog -b
$ git add .
$ git commit -a
$ git push
```
