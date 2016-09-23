    Title: Open files from last commit in Vim
    Date: 2015-07-15T11:05:47Z+0200
    Tags: vim,git

It is always a nuisance to remember which files were you working on when you
last exited your editing session. I have tried different approaches, including:

- leaving a test that does not succeed in order to get an errortrace that
  points to the next task I should tackle
- opening the editor inside a `tmux` or `screen` session, and therefore never
  ending the editing session

The last idea I had was to create a unfinished Git commit (called "Unfinished:
The commit title"), and use Git to retrieve the name of the files that were
touched in that commit. You can do it like this:

    $ git log --pretty=format: --name-only -n 1

If you alias that command in your `.gitconfig`, e.g. to `lastedited`, you can
then open the files in your editor by command subtitution:

    $ vim $(git lastedited)
