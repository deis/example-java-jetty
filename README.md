# Java Quick Start Guide

This guide will walk you through deploying a Java application on [Deis Workflow][].

## Usage

```console
$ git clone https://github.com/deis/example-java-jetty.git
$ cd example-java-jetty
$ deis create
Creating Application... done, created vulcan-keypunch
Git remote deis added
remote available at ssh://git@deis-builder.deis.rocks:2222/vulcan-keypunch.git
$ git push deis master
Counting objects: 89, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (68/68), done.
Writing objects: 100% (89/89), 21.01 KiB | 0 bytes/s, done.
Total 89 (delta 37), reused 0 (delta 0)
Starting build... but first, coffee!
-----> Java app detected
-----> Installing OpenJDK 1.8... done
-----> Installing Maven 3.3.9... done
-----> Executing: mvn -B -DskipTests clean dependency:list install
       [INFO] Scanning for projects...
       [INFO] Downloading: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-dependency-plugin/2.4/maven-dependency-plugin-2.4.pom
...
       [INFO] Downloaded: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-digest/1.0/plexus-digest-1.0.jar (12 KB at 246.4 KB/sec)
       [INFO] Downloaded: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/3.0.5/plexus-utils-3.0.5.jar (226 KB at 3814.3 KB/sec)
       [INFO] Installing /tmp/build/target/helloworld-1.0-SNAPSHOT.jar to /tmp/cache/.m2/repository/com/example/helloworld/1.0-SNAPSHOT/helloworld-1.0-SNAPSHOT.jar
       [INFO] Installing /tmp/build/pom.xml to /tmp/cache/.m2/repository/com/example/helloworld/1.0-SNAPSHOT/helloworld-1.0-SNAPSHOT.pom
       [INFO] ------------------------------------------------------------------------
       [INFO] BUILD SUCCESS
       [INFO] ------------------------------------------------------------------------
       [INFO] Total time: 15.769 s
       [INFO] Finished at: 2016-06-09T18:49:40+00:00
       [INFO] Final Memory: 20M/244M
       [INFO] ------------------------------------------------------------------------
-----> Discovering process types
       Procfile declares types -> web
-----> Compiled slug size is 49M
Build complete.
Launching App...
Done, vulcan-keypunch:v2 deployed to Deis

Use 'deis open' to view this application in your browser

To learn more, use 'deis help' or visit https://deis.com/

To ssh://git@deis-builder.deis.rocks:2222/vulcan-keypunch.git
 * [new branch]      master -> master
$ curl http://vulcan-keypunch.deis.rocks
Powered by Deis
```

## Additional Resources

* [GitHub Project](https://github.com/deis/workflow)
* [Documentation](https://deis.com/docs/workflow/)
* [Blog](https://deis.com/blog/)

[Deis Workflow]: https://github.com/deis/workflow#readme
