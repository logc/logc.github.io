    Title: Maven does not find JAVA_HOME in Fish shell
    Date: 2017-08-25T14:37:39
    Tags: fish, maven, java

After reinstalling `java` from `brew cask`, it could happen that Maven does not
find anymore the JAVA_HOME. The error looks like this:

```console
$ mvn clean install
The JAVA_HOME environment variable is not defined correctly
This environment variable is needed to run this program
NB: JAVA_HOME should point to a JDK not a JR
```

In Fish shell, on Mac OS X, the solution is to set the variable in universal
mode:

```console
$ set -U JAVA_HOME (/usr/libexec/java_home)
```
