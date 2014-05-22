# Java Quick Start Guide

This guide will walk you through deploying a Java application on Deis.

## Usage

```
$ deis create
Creating application... done, created gifted-turbojet
Git remote deis added
$ git push deis master
Counting objects: 81, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (60/60), done.
Writing objects: 100% (81/81), 19.19 KiB | 0 bytes/s, done.
Total 81 (delta 31), reused 0 (delta 0)
-----> Fetching custom buildpack
remote: Cloning into '/tmp/buildpacks/custom'...
-----> Java app detected
-----> Installing OpenJDK 1.6... done
-----> Installing Maven 3.0.3... done
-----> Installing settings.xml... done
-----> executing /tmp/cache/.maven/bin/mvn -B -Duser.home=/tmp/build -Dmaven.repo.local=/tmp/cache/.m2/repository -s /tmp/cache/.m2/settings.xml -DskipTests=true clean install
       [INFO] Scanning for projects...
       [INFO]
       [INFO] ------------------------------------------------------------------------
       [INFO] Building helloworld 1.0-SNAPSHOT
       [INFO] ------------------------------------------------------------------------
       [...]
       [INFO] ------------------------------------------------------------------------
       [INFO] BUILD SUCCESS
       [INFO] ------------------------------------------------------------------------
       [INFO] Total time: 28.315s
       [INFO] Finished at: Thu May 22 23:06:40 UTC 2014
       [INFO] Final Memory: 11M/93M
       [INFO] ------------------------------------------------------------------------
-----> Discovering process types
       Procfile declares types -> web
-----> Compiled slug size is 64M
remote: -----> Building Docker image
remote: Uploading context 66.55 MB
remote: Uploading context
remote: Step 0 : FROM deis/slugrunner
remote:  ---> 5567a808891d
remote: Step 1 : RUN mkdir -p /app
remote:  ---> Using cache
remote:  ---> 4096b5c0b838
remote: Step 2 : ADD slug.tgz /app
remote:  ---> 6f571d81ae1f
remote: Removing intermediate container 518b43b03a13
remote: Step 3 : ENTRYPOINT ["/runner/init"]
remote:  ---> Running in 2d5fda100465
remote:  ---> 4e87a9db9f68
remote: Removing intermediate container 2d5fda100465
remote: Successfully built 4e87a9db9f68
remote: -----> Pushing image to private registry
remote:
remote:        Launching... done, v2
remote:
remote: -----> gifted-turbojet deployed to Deis
remote:        http://gifted-turbojet.local.deisapp.com
remote:
remote:        To learn more, use `deis help` or visit http://deis.io
remote:
To ssh://git@local.deisapp.com:2222/gifted-turbojet.git
 * [new branch]      master -> master
$ curl http://gifted-turbojet.local.deisapp.com
Powered by Deis
```

## Additional Resources

* [Get Deis](http://deis.io/get-deis/)
* [GitHub Project](https://github.com/opdemand/deis)
* [Documentation](http://docs.deis.io/)
* [Blog](http://deis.io/blog/)
