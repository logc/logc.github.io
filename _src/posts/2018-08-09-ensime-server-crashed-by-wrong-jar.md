    Title: Ensime server crashed by wrong JAR
    Date: 2018-08-09T14:10:00
    Tags: scala,ensime,spacemacs

If you have a dodgy source jar file, it will probably blow up the ensime server
with java.util.zip.ZipException: error in opening zip file. The solution offered
in [the project's Github
issue](https://github.com/ensime/ensime-server/issues/825) is to delete your
local copy of the bad file and add it to the list of excluded classifiers in ivy
(i.e. `~/.ivy2/exclude_classifiers`), and report the file to the nexus hosts to
have it deleted.

Mine is a bit longer ...

<!-- more -->

I struggled with this error for weeks! In my case, the offending JAR was a
transitive dependency of a project, and the solution offered here was not clear
enough for me.

This problem is tricky if you previously loaded the project into an IDE: they
download by default all transitive dependencies with sources and javadoc, and
the ivy cache is shared by all projects in the same machine.

Here is my solution, in case it helps others:

- determine which dependency is pulling the problem JAR with the [sbt dependency
  graph plugin](https://github.com/jrudolph/sbt-dependency-graph) (note: the
  command `whatDependsOn` only works in interactive mode)
- move away the current ivy cache folder
```console
$ mv ~/.ivy2/ ~/.ivy2_old
```
- (optional) create a new clone to have a known state
```console
$ git clone $repo ~/dev/ensime-$repo
```
- edit `build.sbt` to add an exclusion rule for the transitive dependency at the line of the explicit dependency that needs it, e.g.
```sbt
libraryDependencies ++= Seq(
  "com.company.notifications" % "notifications-schema" exclude("org.json", "json")
)
```
- compile, go for a coffee while ivy downloads the internet :smile:, then run ensime commands
```console
$ sbt compile
$ sbt ensimeConfig ensimeConfigProject
```

