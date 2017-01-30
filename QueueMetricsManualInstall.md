QueueMetrics is a Java servlet application that runs over a Tomcat server. Potentially you can run QueueMetrics on every Linux distribution and Operating System, Windows as well.
Although the procedure may change amongst the various different systems, conceptually it's ever the same: install all the dependencies and put QueueMetrics on the Tomcat server.
Given that we can't cover every possible Operating System in this tutorial, we'll use a Debian machine to show you the steps to get QueueMetrics working. Your part will be to adapt the steps to your system and situation and remember that filenames and directories may be different.

# 1 Installing Java
    
* Download Java JRE: http://www.oracle.com/technetwork/java/javase/downloads/index.html
* Unpack it and move it where you retain suitable.
    
        tar xfvz jre-7u40-linux-i586.tar.gz 
        mv jre1.7.0_40 /opt/jre1.7.0_40

* Complete the set-up
    
        update-alternatives --install /usr/bin/java java /opt/jre1.7.0_40/bin/java

# 2 Installing MySQL

In Debian you can install it with a simple command and follow the procedure.

        apt-get install mysql-server mysql-client

# 3 Installing Tomcat

* Download Tomcat: http://tomcat.apache.org
* Unpack it and move it where you retain suitable.
    
        tar xfvz apache-tomcat-7.0.42.tar.gz
        mv apache-tomcat-7.0.42 /opt/tomcat-7.0.42

* Add to the /etc/rc.local the following line, to make Tomcat start with the system:

        /opt/tomcat-7.0.42/bin/startup.sh &
        
* Restart and point the browser to the server at port 8080 (e.g. 10.10.1.1:8080) to verify that Tomcat is working correctly.

# 4 Installing MySQL connector for Java

* Download the MySQL-Java connector: http://dev.mysql.com/downloads/connector/j/
* Unpack it and move it where you retain suitable.

        tar xzfv mysql-connector-java-5.1.26.tar.gz 
        mv mysql-connector-java-5.1.26 /opt/mysql-connector-java-5.1.26

# 5 Installing QueueMetrics

* Download QueueMetrics: http://queuemetrics.com/download.jsp
* Unpack it and move it to the Tomcat's webapp directory
        
        tar xzfv QueueMetrics-13.04.2-trial.tar.gz
        cp queuemetrics-13.04.2 /opt/tomcat-7.0.42/webapps/queuemetrics

* Copy the MySQL-Java connector in the QueueMetrics' WEB-INF/lib directory

        cp /opt/mysql-connector-java-5.1.26/mysql-connector-java-5.1.26-bin.jar /opt/tomcat-7.0.42/webapps/queuemetrics/WEB-INF/lib/mysql-connector-java-5.1.26-bin.jar

* Restart Tomcat

		/opt/tomcat-7.0.42/bin/shutdown.sh
		/opt/tomcat-7.0.42/bin/startup.sh
