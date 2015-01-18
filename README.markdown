# Java init script for Ubuntu

The code in this repository is in the public domain. See [CC0 1.0 Universal (CC0 1.0) Public Domain Dedication](http://creativecommons.org/publicdomain/zero/1.0/) for more information.

## Introduction

This repository contains an init script to run Java applications as a daemon.

The init script is based on the Tomcat 7 init script for Ubuntu, simplified
where possible.

## Instructions

To use the init script, the `init.d/javadaemon` file must be copied
into the `/etc/init.d` and renamed to your application. The file itself
requires a number of modifications to make it for your specific
application:

* All references to `javadaemon` must be replaced with the name of
  your application.
* All references `Java Daemon` must be replaced with a descriptive name
  for your application.

The init script has a number of defaults which may need to be adjusted
for your specific application. These are:

* By default JDK versions 8, 7 and 6 are searched in that order. If
  your application is not compatible with any of these versions,
  the search algorithm in `find_openjdks` must be adjusted and
  the incompatible JDK versions must be removed from the `JDK_DIRS`
  variable.
* The scripts expects the base directory to be `/var/local/$NAME`.
  If this is incorrect, change the initialization of the `RUN_BASE`
  variable.
* The default Java options are `-Xmx256M`. This can be modified by
  setting the `JAVA_OPTS` variable. The options set here can be
  overriden in the `/etc/defaults/$NAME` script (see below). Mandatory
  Java options should be set at a later point in the script.
* The test whether the application is installed is done by checking
  whether the `/var/local/$NAME` directory exists. You can expand this
  by changing the test under `# Validate the installation directory`.
* The scripts defines three timeouts which may need to be adjusted:
  * `STARTUP_TIMEOUT` specifies how many seconds the scripts waits
    for the initial startup to finish.
  * `TERM_TIMEOUT` specifies how many seconds `start-stop-daemon`
    waits for the application to cleanly shutdown after receiving
    the `TERM` signal. See below for how to incorporate the `TERM`
    signal in your application.
  * `KILL_TIMEOUT` specifies how many seconds `start-stop-daemon`
    waits for the application to respond to the `KILL` signal.

The initial value of the `JAVA_OPTS` variable specifies the default
Java options which can be overridden in the `/etc/defaults/$NAME`
script. A default version for this script can be found at
`defaults/javadaemon` in the repository. To use this script, rename
it to your application and place it in `/etc/defaults`.

Mandatory Java options are placed in the `run_daemon` function in
the script. The script in this repository specifies the following
default Java options:

* At the start of `run_daemon`, the Java temporary directory is set
  which is initialized from the script. You should not modify this.
* After that, the classpath value and options for your application are
  set. As an example, the script adds the `-Dfile.encoding=UTF-8`
  option. The classpath is stored in a separate variable to prevent
  glob expansion from corrupting wildcards in the classpath; Java
  requires these wildcards to be passed verbatim to the JVM. If you
  do not need a classpath, remove the variable and
  `-classpath "$CLASSPATH"` from the `start-stop-daemon` command.
* Last the entry point of the application is set. As an example,
  the class `org.example.App` is specified which must be changed to the
  correct class for your application. Alternatively a Jar may be
  specified. However, classpath's and Jars cannot be used together,
  so choose one of those.

## Adding a shutdown hook to Java

To make your Java application respond correctly to a TERM signal,
a shutdown hook must be added. [`Runtime.addShutdownHook()`](http://docs.oracle.com/javase/7/docs/api/java/lang/Runtime.html#addShutdownHook%28java.lang.Thread%29)
for more information on this. One thing to take care of when using a
shutdown hook to cleanly shut down your application is that the
the JVM will shutdown when the shutdown hook finishes. This means
that either all cleanup code must be put into the shutdown hook,
or that the shutdown hook must wait until the main thread finishes
shutting down, e.g. by having the shutdown hook wait for the main
thread to signal a clean shutdown.

## Bugs

Bugs should be reported through github at
[http://github.com/pvginkel/UbuntuJavaInit/issues](http://github.com/pvginkel/UbuntuJavaInit/issues).
