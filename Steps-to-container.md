## Working with Red Hat UBI
https://developers.redhat.com/blog/2019/05/31/working-with-red-hat-enterprise-linux-universal-base-images-ubi/

Here are the steps to quickly create a docker application

1. Create a directory called "container-example"
1. Change to the new directory
1. Create two files: a Dockerfile and an index.php script.

Edit the index.php file to create an inspired “Hello, world” PHP-enhanced web page, for example:
```
<html>
  <body>
    <?php print "Hello, world!\n" ?>
  </body>
</html>
```

Edit the Dockerfile. This Dockerfile installs Apache HTTPd and PHP from the Software Collections Library, using the UBI package repositories:

```
FROM registry.redhat.io/ubi7/ubi

RUN yum -y install --disableplugin=subscription-manager \
  httpd24 rh-php72 rh-php72-php \
  && yum --disableplugin=subscription-manager clean all

ADD index.php /opt/rh/httpd24/root/var/www/html

RUN sed -i 's/Listen 80/Listen 8080/' \
  /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf \
  && chgrp -R 0 /var/log/httpd24 /opt/rh/httpd24/root/var/run/httpd \
  && chmod -R g=u /var/log/httpd24 /opt/rh/httpd24/root/var/run/httpd

EXPOSE 8080

USER 1001

CMD scl enable httpd24 rh-php72 -- httpd -D FOREGROUND
```

The –disableplugin=subscription-manager option avoids warning messages when building from a machine that has no active subscription, such as my personal Fedora machine.

> For a Red Hat OpenShift-friendly container image must work without root privileges and under a random userid. 
> Unlike several desktop container engines and some Kubernetes distributions, Red Hat OpenShift refuses to run containers that require elevated privileges by default.

Enter your work folder and use Podman to build the container image:
```
$ docker login
$ docker build -t php-ubi .
```

Next start the container and test your container using curl:
```
$ docker run --name hello -p 8080:8080 -d php-ubi

$ curl localhost:8080

<html>
<body>
Hello, world!
</body>
</html>

```
You can now publish your cont

## Building Locally

This project requires Java 8 to compile. It will not compile with Java 9 or later.

To build a runnable Spring Boot jar file, run the following command: 

~~~
$ ./gradlew clean assemble
~~~

## Running the application locally

One Spring bean profile should be activated to choose the database provider that the application should use. The profile is selected by setting the system property `spring.profiles.active` when starting the app.

The application can be started locally using the following command:

~~~
$ java -jar -Dspring.profiles.active=<profile> build/libs/spring-music.jar
~~~

where `<profile>` is one of the following values:

* `mysql`
* `postgres`
* `mongodb`
* `redis`

If no profile is provided, an in-memory relational database will be used. If any other profile is provided, the appropriate database server must be started separately. Spring Boot will auto-configure a connection to the database using it's auto-configuration defaults. The connection parameters can be configured by setting the appropriate [Spring Boot properties](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

If more than one of these profiles is provided, the application will throw an exception and fail to start.

















## Running the application on Cloud Foundry

When running on Cloud Foundry, the application will detect the type of database service bound to the application (if any). If a service of one of the supported types (MySQL, Postgres, Oracle, MongoDB, or Redis) is bound to the app, the appropriate Spring profile will be configured to use the database service. The connection strings and credentials needed to use the service will be extracted from the Cloud Foundry environment.

If no bound services are found containing any of these values in the name, an in-memory relational database will be used.

If more than one service containing any of these values is bound to the application, the application will throw an exception and fail to start.

After installing the 'cf' [command-line interface for Cloud Foundry](http://docs.cloudfoundry.org/cf-cli/), targeting a Cloud Foundry instance, and logging in, the application can be built and pushed using these commands:

~~~
$ cf push
~~~

The application will be pushed using settings in the provided `manifest.yml` file. The output from the command will show the URL that has been assigned to the application.

### Creating and binding services

Using the provided manifest, the application will be created without an external database (in the `in-memory` profile). You can create and bind database services to the application using the information below.

#### System-managed services

Depending on the Cloud Foundry service provider, persistence services might be offered and managed by the platform. These steps can be used to create and bind a service that is managed by the platform:

~~~
# view the services available
$ cf marketplace
# create a service instance
$ cf create-service <service> <service plan> <service name>
# bind the service instance to the application
$ cf bind-service <app name> <service name>
# restart the application so the new service is detected
$ cf restart
~~~

#### User-provided services

Cloud Foundry also allows service connection information and credentials to be provided by a user. In order for the application to detect and connect to a user-provided service, a single `uri` field should be given in the credentials using the form `<dbtype>://<username>:<password>@<hostname>:<port>/<databasename>`.

These steps use examples for username, password, host name, and database name that should be replaced with real values.

~~~
# create a user-provided Oracle database service instance
$ cf create-user-provided-service oracle-db -p '{"uri":"oracle://root:secret@dbserver.example.com:1521/mydatabase"}'
# create a user-provided MySQL database service instance
$ cf create-user-provided-service mysql-db -p '{"uri":"mysql://root:secret@dbserver.example.com:3306/mydatabase"}'
# bind a service instance to the application
$ cf bind-service <app name> <service name>
# restart the application so the new service is detected
$ cf restart
~~~

#### Changing bound services

To test the application with different services, you can simply stop the app, unbind a service, bind a different database service, and start the app:

~~~
$ cf unbind-service <app name> <service name>
$ cf bind-service <app name> <service name>
$ cf restart
~~~

#### Database drivers

Database drivers for MySQL, Postgres, Microsoft SQL Server, MongoDB, and Redis are included in the project.

To connect to an Oracle database, you will need to download the appropriate driver (e.g. from http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html). Then make a `libs` directory in the `spring-music` project, and move the driver, `ojdbc7.jar` or `ojdbc8.jar`, into the `libs` directory.
In `build.gradle`, uncomment the line `compile files('libs/ojdbc8.jar')` or `compile files('libs/ojdbc7.jar')` and run `./gradle assemble`
