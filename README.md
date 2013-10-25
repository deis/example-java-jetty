# Java Quick Start Guide

This guide will walk you through deploying a Java application on Deis.

## Prerequisites

* A [User Account](http://docs.deis.io/en/latest/client/register/) on a [Deis Controller](http://docs.deis.io/en/latest/terms/controller/).
* A [Deis Formation](http://docs.deis.io/en/latest/gettingstarted/concepts/#formations) that is ready to host applications

If you do not yet have a controller or a Deis formation, please review the [Deis installation](http://docs.deis.io/en/latest/gettingstarted/installation/) instructions.


## Setup your workstation

* Install [RubyGems](http://rubygems.org/pages/download) to get the `gem` command on your workstation
* Install [Foreman](http://ddollar.github.com/foreman/) with `gem install foreman`
* Install a [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) if you don't have one already

## Clone your Application

If you want to use an existing application, no problem.  You can also use the Deis sample application located at <https://github.com/bengrunfeld/example-java-jetty>.  Clone the example application to your local workstation:

	$ git clone https://github.com/bengrunfeld/example-java-jetty.git
	$ cd example-java-jetty

## Prepare your Application

To use a Java application with Deis, you will need to conform to 3 basic requirements:

 1. Use [Maven](http://maven.apache.org/guides/getting-started/index.html) to compile code and manage dependencies
 2. Use [Foreman](http://ddollar.github.com/foreman/) to manage processes
 3. Use [Environment Variables](https://help.ubuntu.com/community/EnvironmentVariables) to manage configuration inside your application

If you're deploying the example application, it already conforms to these requirements.

### 1. Use Maven to compile code and manage dependencies

Maven requires that you explicitly declare your dependencies using a [pom.xml](http://www.pip-installer.org/en/latest/requirements.html) file.  Here is a very [basic example](https://github.com/bengrunfeld/example-java-jetty/blob/master/pom.xml). You can then use `mvn package` to install dependencies, compile and package your application on your local workstation:

	$ mvn package
	[INFO] Scanning for projects...
	[INFO]                                                                         
	[INFO] ------------------------------------------------------------------------
	[INFO] Building helloworld 1.0-SNAPSHOT
	[INFO] ------------------------------------------------------------------------
	[INFO] 
	[INFO] --- maven-resources-plugin:2.4.3:resources (default-resources) @ helloworld ---

### 2. Use Foreman to manage processes

Deis relies on a [Foreman](http://ddollar.github.com/foreman/) `Procfile` that lives in the root of your repository.  This is where you define the command(s) used to run your application.  Here is an example `Procfile`:

    web: java -cp target/classes:target/dependency/* HelloWorld


This tells Deis to run `web` workers using the command `java HelloWorld`. You can test this locally by running `foreman start`.

	$ foreman start
	13:06:10 web.1  | started with pid 36840
	13:06:11 web.1  | 2013-10-25 13:06:11.182:INFO:oejs.Server:jetty-7.6.0.v20120127
	13:06:11 web.1  | 2013-10-25 13:06:11.268:INFO:oejsh.ContextHandler:started o.e.j.s.ServletContextHandler{/,null}
	13:06:11 web.1  | 2013-10-25 13:06:11.319:WARN:oejuc.AbstractLifeCycle:FAILED SelectChannelConnector@0.0.0.0:5000: java.net.BindException: Address already in use


You should now be able to access your application locally at <http://localhost:5000>.

### 3. Working with Environment Variables

Deis uses environment variables to manage your application's configuration. For example, your application listener must use the value of the `PORT` environment variable. The following code snippet demonstrates how this can work inside your application:

    import org.eclipse.jetty.server.Server;
    ...
	Server server = new Server(Integer.valueOf(System.getenv("PORT")));


## Create a new Application

Per the prerequisites, we assume you have access to an existing Deis formation. If not, please review the Deis [installation instuctions](http://docs.deis.io/en/latest/gettingstarted/installation/).

Use the following command to create an application on an existing Deis formation.

	$ deis create --formation=<formationName> --id=<appName>
	Creating application... done, created yearly-pendulum
	Git remote deis added
	
If an ID is not provided, one will be auto-generated for you.

## Deploy your Application

Use `git push` to deploy your application.

	$ git push deis master
	Counting objects: 48, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (30/30), done.
	Writing objects: 100% (48/48), 13.05 KiB, done.
	Total 48 (delta 14), reused 18 (delta 3)
	       Java app detected
	-----> Installing OpenJDK 1.6... done
	-----> Installing Maven 3.0.3... done
	-----> Installing settings.xml... done


Once your application has been deployed, use `deis open` to view it in a browser. To find out more info about your application, use `deis info`.

## Scale your Application

To scale your application's [Docker](http://docker.io) containers, use `deis scale`.

	$ deis scale web=8
	Scaling containers... but first, coffee!
	done in 17s
	
	=== yearly-pendulum Containers
	
	--- web: `java -cp target/classes:target/dependency/* HelloWorld`
	web.1 up 2013-10-25T19:24:24.054Z (jettyFormation-runtime-1)
	web.2 up 2013-10-25T19:25:46.251Z (jettyFormation-runtime-1)
	web.3 up 2013-10-25T19:25:46.266Z (jettyFormation-runtime-1)
	web.4 up 2013-10-25T19:25:46.281Z (jettyFormation-runtime-1)
	web.5 up 2013-10-25T19:25:46.297Z (jettyFormation-runtime-1)
	web.6 up 2013-10-25T19:25:46.315Z (jettyFormation-runtime-1)
	web.7 up 2013-10-25T19:25:46.333Z (jettyFormation-runtime-1)
	web.8 up 2013-10-25T19:25:46.352Z (jettyFormation-runtime-1)
	


## Configure your Application

Deis applications are configured using environment variables. The example application includes a special `POWERED_BY` variable to help demonstrate how you would provide application-level configuration. 

	$ curl -s http://yourapp.com
	Powered by null
	$ deis config:set POWERED_BY=Jetty
	== yearly-pendulum
	JAVA_OPTS: -Xmx384m -Xss512k -XX:+UseCompressedOops
	PATH: /app/.jdk/bin:/usr/local/bin:/usr/bin:/bin
	POWERED_BY: Jetty
	MAVEN_OPTS: -Xmx384m -Xss512k -XX:+UseCompressedOops
	$ curl -s http://yourapp.com
	Powered by Jetty

This method is also how you connect your application to backing services like databases, queues and caches.

To experiment in your application environment, use `deis run` to execute one-off commands against your application.

	$ deis run ls -la
	drwxr-xr-x  8 root root 4096 Oct 25 19:24 .
	drwxr-xr-x 57 root root 4096 Oct 25 19:30 ..
	-rw-r--r--  1 root root   54 Oct 25 19:23 .gitignore
	drwxr-xr-x  6 root root 4096 Oct 25 19:23 .jdk
	drwxr-xr-x  3 root root 4096 Oct 25 19:24 .m2
	drwxr-xr-x  6 root root 4096 Oct 25 19:24 .maven
	drwxr-xr-x  2 root root 4096 Oct 25 19:24 .profile.d
	-rw-r--r--  1 root root  210 Oct 25 19:24 .release
	-rw-r--r--  1 root root  553 Oct 25 19:23 LICENSE
	-rw-r--r--  1 root root   60 Oct 25 19:23 Procfile
	-rw-r--r--  1 root root 8264 Oct 25 19:23 README.md
	-rw-r--r--  1 root root 1622 Oct 25 19:23 pom.xml
	drwxr-xr-x  3 root root 4096 Oct 25 19:23 src
	-rw-r--r--  1 root root   25 Oct 25 19:23 system.properties
	drwxr-xr-x  6 root root 4096 Oct 25 19:24 target

## Troubleshoot your Application

To view your application's log output, including any errors or stack traces, use `deis logs`.

	$ deis logs
	<show output>

## Additional Resources

* [Get Deis](http://deis.io/get-deis/)
* [GitHub Project](https://github.com/opdemand/deis)
* [Documentation](http://docs.deis.io/)
* [Blog](http://deis.io/blog/)
