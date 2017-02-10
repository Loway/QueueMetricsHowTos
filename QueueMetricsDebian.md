**IMPORTANT: you should install QueueMetrics manually only if you are an expert user. Unfortunately this is a non-standard installation in a non-standard environment and Loway won't be able to support you with system administraton problems (tomcat configuration, database permissions and so on), although the support on the QueueMetrics software itself will remain available as always.**


# Installing QueueMetrics on a Debian or Ubuntu server

This guide will help you install a working QueueMetrics system on a Debian or Ubuntu box. This guide is meant to be run
as a root user.

## Install dependencies

The following commands install the Java environment and a MariaDB server. Please note that QueueMetrics will
not currently work with Java 8.

     apt-get update -y
     apt-get install -y openjdk-7-jdk tomcat7 libmysql-java mariadb-client mariadb-server

When the database is being set up, it will ask for a root password. Enter it and write it down.

## Download and unpack QueueMetrics

You can find the latest QueueMetrics version at https://www.queuemetrics.com/download.jsp

cd /var/lib/tomcat7/webapps/
wget http://downloads.loway.ch/beta/QueueMetrics-16.10.8.tar.gz
mv queuemetrics-16.10.8/ QM
cp /usr/share/java/mysql-connector-java.jar QM/WEB-INF/lib/
chown tomcat7.tomcat7 QM/WEB-INF/*.properties

## Configure Tomcat

Edit the file '/etc/default/tomcat7' and change the line with JAVA_OPTS to:

For a small server (just testing, up to 20 users):

    JAVA_OPTS="-Djava.awt.headless=true -server -Xms256m -Xmx256m -XX:+UseConcMarkSweepGC"

For a larger server (20+ users) use the following:

    JAVA_OPTS="-Djava.awt.headless=true -server -Xms3072m -Xmx3072m -XX:+UseConcMarkSweepGC"


## Start services and make them restart on boot

Make sure services start on reboot:

    update-rc.d tomcat7 enable
    update-rc.d mysql enable

Start the servers:

    service mysql restart
    service tomcat7 restart


## Finish the installation

Connect to QueueMetrics at http://server.address/QM and:

* You will see a database error (because the database does not exist); in a few seconds will redirect you to the Create Database wizard
* On the "Create Database Now" page, enter the MySQL root password
* The initial database will be uploaded; click on "Start" to start QM
* Accept license terms
* Log-in as "demoadmin" password "demo"

## Install a license key

If you have a license key, you can install it in the file 

    cd /var/lib/tomcat7/webapps/
    vi QM/WEB-INF/tpf.properties


