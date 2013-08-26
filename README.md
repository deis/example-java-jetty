# Java Quick Start Guide

This guide will walk you through deploying a Java application to Amazon EC2 using [Deis](http://github.com/opdemand/deis).

## Prerequisites

* You need to have set up Deis according to the instructions in [Deis: Getting Started](https://github.com/opdemand/deis#getting-started)
* You will need [Git](http://git-scm.com), [RubyGems](http://rubygems.org/pages/download), [Pip](http://www.pip-installer.org/en/latest/installing.html), the [Amazon EC2 API Tools](http://aws.amazon.com/developertools/351), [EC2 Credentials](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/SettingUp_CommandLine.html#set_aws_credentials_linux) and a Chef Server with a working [Knife](http://docs.opscode.com/knife.html) client.


## Setup your workstation

* Install [RubyGems](http://rubygems.org/pages/download) to get the `gem` command on your workstation
* Install [Foreman](http://ddollar.github.com/foreman/) with `gem install foreman`
* Install a [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) if you don't have one already

## Clone your Application

If you want to use an existing application, no problem.  You can also fork Deis' sample application located at <https://github.com/bengrunfeld/example-java-jetty>.  After forking the project, clone it to your local workstation using the SSH-style URL:

	$ git clone git@github.com:mygithubuser/example-java-jetty.git
    $ cd example-java-jetty

## Prepare your Application

To use a Java application with Deis, you will need to conform to 3 basic requirements:

 1. Use [Maven](http://maven.apache.org/guides/getting-started/index.html) to compile code and manage dependencies
 2. Use [Foreman](http://ddollar.github.com/foreman/) to manage processes
 3. Use [Environment Variables](https://help.ubuntu.com/community/EnvironmentVariables) to manage configuration inside your application

If you're deploying the example application, it already conforms to these requirements.

### 1. Use Maven to compile code and manage dependencies

Every time you deploy, Deis will run a `mvn package` on all application instances to ensure dependencies are up to date, and code is compiled and packaged.  Maven requires that you explicitly declare your dependencies using a [pom.xml](http://www.pip-installer.org/en/latest/requirements.html) file.  Here is a very [basic example](https://github.com/bengrunfeld/example-java-jetty/blob/master/pom.xml).
    
You can then use `mvn package` to install dependencies, compile and package your application on your local workstation:

    $ mvn package
    [INFO] Scanning for projects...
    [INFO]                                                                         
    [INFO] ------------------------------------------------------------------------
    [INFO] Building helloworld 1.0-SNAPSHOT
    [INFO] ------------------------------------------------------------------------
    ...
    [INFO] Copying jetty-util-7.6.0.v20120127.jar to /Users/ben/workspace/example-java-jetty/target/dependency/jetty-util-7.6.0.v20120127.jar
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 51.956s
    [INFO] Finished at: Sat Apr 06 15:21:20 MDT 2013
    [INFO] Final Memory: 11M/81M
    [INFO] ------------------------------------------------------------------------

If your dependencies require any system packages, you can install those later by specifying a list of custom packages in the Instance configuration or by customizing the deploy script to install your own packages.

### 2. Use Foreman to manage processes

Deis uses [Foreman](http://ddollar.github.com/foreman/) to manage the processes that serve up your application.  Foreman relies on a `Procfile` that lives in the root of your repository.  This is where you define the command(s) used to run your application.  Here is an example `Procfile`:

    web: java -cp target/classes:target/dependency/* HelloWorld

This tells Deis to run web application workers using the `java HelloWorld` command with a classpath of `target/classes:target/dependency/*`.  You can test this locally by running `foreman start`.

    $ foreman start
    15:25:37 web.1  | started with pid 90321
    15:25:37 web.1  | 2013-04-06 15:25:37.377:INFO:oejs.Server:jetty-7.6.0.v20120127
    15:25:37 web.1  | 2013-04-06 15:25:37.440:INFO:oejsh.ContextHandler:started o.e.j.s.ServletContextHandler{/,null}
    15:25:37 web.1  | 2013-04-06 15:25:37.468:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:5000

You should now be able to access your application locally at <http://localhost:5000>.

### 3. Working with Environment Variables

Deis uses [environment variables](https://help.ubuntu.com/community/EnvironmentVariables) to manage your application's configuration. 


#### Setting an Environment Variable

Environment variables can be set straight from the command line:

	deis config:set VAR_NAME=value

#### Retrieving an Environment Variable

For example, if your application listener uses the value of the `PORT` environment variable, the following code will retrieve it, so that it can be used.

    import org.eclipse.jetty.server.Server;
    ...
	Server server = new Server(Integer.valueOf(System.getenv("PORT")));

The same is true for external services like databases, caches and queues.  Here is an example in that shows how to make a JDBC connection to a PostgreSQL database using environment variables:

    private static Connection getConnection() throws SQLException {

      String hostname = System.getenv("POSTGRES_HOST");
      String username = System.getenv("POSTGRES_USERNAME");
      String password = System.getenv("POSTGRES_PASSWORD");
      String dbname = System.getenv("POSTGRES_DBNAME");
      String dbUrl = "jdbc:postgresql://" + hostname + ':' + port + "/" + dbname;

      return DriverManager.getConnection(dbUrl, username, password);
    }

    
# The Deis Workflow

## 1. Start Your Engine

1. Log in to the **EC2 Management Console**
2. Navigate to **Instances**
3. Right-click on your controller and hit **Start**
4. Attach an elastic IP *(optional)*
5. Login to Deis: `deis login <controller-ip-address>`

## 2. Create a Formation

Once your controller is [set up](https://github.com/opdemand/deis#3-provision-a-deis-controller), you can use `deis create` to choose which region of EC2 you'd like to deploy to. This sets up the layers that will be deployed with the `layers` command.

	deis create --flavor=ec2-us-west-2

## 3. Scale the Formation

You can choose how many proxies and runtime layers to deploy to your formation in a single line of code with the `deis layers` command.

	deis layers:scale proxy=1 runtime=1

*(optional)* You can specify how many LXC containers you want to scale to within the runtime layer (the default number created with `deis layers` is 1). 

	deis containers:scale web=4 worker=2

## 4. Push Your Application to the Formation

Before you can access your application, you need to push it to your formation.

	git push deis master

## 5. To Open Your Application in a Browser

Open your web app in a browser, simply by using:

	deis open

## 6. Destroy the Formation or Individual Nodes

At some point, you may want to destroy your formation, either because it has finished its lifecycle, or
because you're in development and simply want to save on resource costs while you power down overnight.

	deis destroy
	
The `deis destroy` command will terminate your proxy and runtime layers. It *does not* turn off your EC2 Instance, which is the Controller. To do that you need to go into your **EC2 Management Console**, navigate to **Instances**, find your **Controller**, right-click on it, and hit **Stop**.

To destroy individual nodes, use:

	deis node:destroy <ID>

## Update your Application

Using the regular **Git** workflow, update your application code locally, then when you have performed a commit, simply push your code using:

	git push deis master

## Additional Resources

* [Deis Documentation](http://docs.deis.io)
* [Deis README.md](http://github.com/opdemand/deis/blob/master/README.md)
* [Deis Website](http://deis.io)

