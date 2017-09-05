    Title: Ensime server crash because of transitive dependency
    Date: 2017-09-05T15:47:32
    Tags: Scala, Emacs

The Scala layer for Spacemacs uses Ensime by default. There is an error that
made me struggle for weeks: [Bad source jar kills the server: ZipException:
error in opening zip file](https://github.com/ensime/ensime-server/issues/825)

The offending JAR was a transitive dependency of a project, and the solution
offered on the issue was not clear for me.

This problem is tricky if you previously loaded the project into an IDE: they
download by default all transitive dependencies with sources and javadoc, and
the ivy cache is shared by all projects in the same machine.

Here is my solution, as found also on the issue, in case it helps others:

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
- edit `build.sbt` to add an exclusion rule for the transitive dependency at the
  line of the explicit dependency that needs it, e.g.
```sbt
libraryDependencies ++= Seq(
  "com.company.notifications" % "notifications-schema" exclude("org.json", "json")
)
```
- compile, go for a coffee while ivy downloads the internet :), then run ensime commands
```console
$ sbt compile
$ sbt ensimeConfig ensimeConfigProject
```

