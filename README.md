## Usage

To create the image `mdelapenya/tomcat-mysql`, execute the following command on the mdelapenya-docker-tomcat-mysql folder:

```shell
docker build -t mdelapenya/tomcat-mysql .
```

You can now push your new image to the registry:

```shell
docker push mdelapenya/tomcat-mysql
```

## Running your tomcat-mysql docker image

Start your image binding the external ports 8080 and 3306 in all interfaces to your container:

```shell
docker run -d -p 8080:8080 -p 3306:3306 mdelapenya/tomcat-mysql
```

Test your deployment:

```shell
curl http://localhost:8080/
```

Tomcat 7.0.77's home page should appear.

## Loading your custom Java application

Copy the WAR file representing your application to `/opt/apache-tomcat-7.0.77/webapps/` folder:

```shell
FROM mdelapenya/tomcat-mysql:7.0.77
COPY yourapp.war /opt/apache-tomcat-7.0.77/webapps/yourapp.war
```

After that, build the new `Dockerfile`:

```shell
docker build -t username/my-tomcat-mysql-app .
```

And test it:

```shell
docker run -d -p 8080:80 -p 3306:3306 username/my-tomcat-mysql-app
```

Test your deployment:

```shell
curl http://localhost:8080/yourapp
```

That's it!

## Connecting to the bundled MySQL server from within the container

The bundled MySQL server has a `root` user with no password for local connections.
Simply connect from your Java code with this user (using sql2o library):

```java
String jdbcUrl = "jdbc-url-connection";
String jdbcUser = "jdbc-user";
String jdbcPassword = "jdbc-password";

return new Sql2o(jdbcUrl, jdbcUser, jdbcPassword);
```

## Connecting to the bundled MySQL server from outside the container

The first time that you run the container, a new user `admin` with all privileges
will be created in MySQL with a random password. To get the password, check the logs
of the container by running:

```shell
docker logs $CONTAINER_ID
```

You will see an output like the following:

```shell
========================================================================
You can now connect to this MySQL Server using:

    mysql -uadmin -p47nnf4FweaKu -h<host> -P<port>

Please remember to change the above password as soon as possible!
MySQL user 'root' has no password but only allows local connections
========================================================================
```

In this case, `47nnf4FweaKu` is the password allocated to the `admin` user.

You can then connect to MySQL:

```shell
mysql -uadmin -p47nnf4FweaKu
```

Remember that the `root` user does not allow connections from outside the container -
you should use this `admin` user instead!

## Setting a specific password for the MySQL server admin account

If you want to use a preset password instead of a random generated one, you can
set the environment variable `MYSQL_PASS` to your specific password when running the container:

```shell
docker run -d -p 80:80 -p 3306:3306 -e MYSQL_PASS="mypass" mdelapenya/liferay-portal-mysql
```

You can now test your new admin password:

```shell
mysql -uadmin -p"mypass"
```