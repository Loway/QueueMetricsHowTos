# How to Get a Working QueueMetrics System

QueueMetrics is able to operate in various server configurations in order to meet all your needs.
Depending on how big is your system you might want to have a single-server solution (Asterisk and QueueMetrics on the same server), a separated-server solution (Asterisk and QueueMetrics reside on two different servers) and a cluster solution (one QueueMetrics server monitoring many Asterisk servers).

## 1 Understanding How QueueMetrics Works

All the data which QueueMetrics works on are stored in Asterisk's queue_log file. In a single-server configuration, QueueMetrics is able to read directly from this file, though this is not an efficient approach.
The Qloader script provides an effective way to retrieve information from the _queue&#95;log_ file without overloading the Asterisk server(s) and it's essential in a cluster-server configuration.
It reads the new data from the queue_log file and sends it to the QueueMetrics database wherever it is.
We always suggest to use the Qloader script whatever configuration you have.

## 2 Installing QueueMetrics with Espresso

If you are setting up a single-server solution on an Asterisk-ready Linux distribution (eg. Elastix, AsteriskNOW, FreePBX, etc) you can get a working QueueMetrics system in a few minutes.

TIP:	Check if the Espresso installer supports your distribution version in the _Supported PBXs_ section at http://queuemetrics.com/manuals/QM_Espresso-chunked .

Just add our repository and install the package:

    wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
    yum install queuemetrics-espresso
    
It will detect your operating system and your Asterisk versions and configure QueueMetrics to work with them.
It will also install and configure all the other software needed (Tomcat, MySQL, Qloader script) and creates a working AMI connection to Asterisk.

Now you can point your browser to http://hostname:8080/queuemetrics and start working.

TIP:	The default username is 'demoadmin' and the password is 'demo'.

## 3 Installing QueueMetrics manually

With this step we are going to install QueueMetrics on a server, though it still need to be configured in order to work.

### 3.1 On a RPM-based distribution

Just add our repository and install the package:

    wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
    yum install queuemetrics

Now, install the Qloader script in each of your Asterisk boxes.
    
	wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
	yum install qloaderd
    
### 3.2 On another system (yes, Windows too)

QueueMetrics is a Java servlet application that runs over a Tomcat server. Potentially you can run QueueMetrics on every Linux distribution and Operating System, Windows as well.
Although the procedure may change amongst the various different systems, conceptually it's ever the same: install all the dependencies and put QueueMetrics on the Tomcat server.
Given that we can't cover every possible Operating System in this tutorial, we'll use a Debian machine to show you the steps to get QueueMetrics working. Your part will be to adapt the steps to your system and situation and remember that filenames and directories may be different.

#### 3.2.1 Installing Java
    
* Download Java JRE: http://www.oracle.com/technetwork/java/javase/downloads/index.html
* Unpack it and move it where you retain suitable.
    
        tar xfvz jre-7u40-linux-i586.tar.gz 
        mv jre1.7.0_40 /opt/jre1.7.0_40

* Complete the set-up
    
        update-alternatives --install /usr/bin/java java /opt/jre1.7.0_40/bin/java

#### 3.2.2 Installing MySQL

In Debian you can install it with a simple command and follow the procedure.

        apt-get install mysql-server mysql-client

#### 3.2.3 Installing Tomcat

* Download Tomcat: http://tomcat.apache.org
* Unpack it and move it where you retain suitable.
    
        tar xfvz apache-tomcat-7.0.42.tar.gz
        mv apache-tomcat-7.0.42 /opt/tomcat-7.0.42

* Add to the /etc/rc.local the following line, to make Tomcat start with the system:

        /opt/tomcat-7.0.42/bin/startup.sh &
        
* Restart and point the browser to the server at port 8080 (e.g. 10.10.1.1:8080) to verify that Tomcat is working correctly.

#### 3.2.4 Installing MySQL connector for Java

* Download the MySQL-Java connector: http://dev.mysql.com/downloads/connector/j/
* Unpack it and move it where you retain suitable.

        tar xzfv mysql-connector-java-5.1.26.tar.gz 
        mv mysql-connector-java-5.1.26 /opt/mysql-connector-java-5.1.26

#### 3.2.5 Installing QueueMetrics

* Download QueueMetrics: http://queuemetrics.com/download.jsp
* Unpack it and move it to the Tomcat's webapp directory
        
        tar xzfv QueueMetrics-13.04.2-trial.tar.gz
        cp queuemetrics-13.04.2 /opt/tomcat-7.0.42/webapps/queuemetrics

* Copy the MySQL-Java connector in the QueueMetrics' WEB-INF/lib directory

        cp /opt/mysql-connector-java-5.1.26/mysql-connector-java-5.1.26-bin.jar /opt/tomcat-7.0.42/webapps/queuemetrics/WEB-INF/lib/mysql-connector-java-5.1.26-bin.jar

* Restart Tomcat

		/opt/tomcat-7.0.42/bin/shutdown.sh
		/opt/tomcat-7.0.42/bin/startup.sh

### 3.3 Does it work?

Now, to check that everything works you can open your browser and point it to _http://hostname:8080/queuemetrics_, the first time you run it you might get an error, it will disappear in a few seconds and QueueMetrics will guide you through the database configuration; probably you'll just need to confirm without make any change.
After that you will be able to log in with the default username _demoadmin_ and password _demo_.
Obviously, at the the moment, no data is shown.

## 4 Configuring QueueMetrics

Now that you have got QueueMetrics running you need to configure it in order to retrieve data from Asterisk.

### 4.1 Qloader configuration

On the Asterisk systems edit the _/etc/sysconfig/qloaderd_ file; you need to set the following values in order to tell Qloaderd to work with the QueueMetrics database:
  
    MYSQLHOST=
    MYSQLUSER=
    MYSQLPASS=
    
If you're running an Asterisk server cluster you also have to modify the PARTITION value, giving to each of them a different name (usually P001, P002, P003, etc). This is fundamental to avoid any possible concurrent writing issue.

CAUTION:	Each Asterisk server's Qloader script have to use a different partition.

Now restart qloaderd:

    service qloaderd restart


### 4.2 QueueMetrics configuration

If you have a single Asterisk server and have not modified the PARTITION value in the Qloader configuration add (or, if exists, modify), in the configuration.properties file, the following values:

    default.queue_log_file=sql:P001
    callfile.dir=tcp:admin:password@ip-address
    		# user:pass are the credentials for the Asterisk's AMI interface.
    		# You might have to configure a user in each of the 
    		# Asterisk servers' /etc/manager.conf file.
    		# serverhost is the IP address of that server.

QueueMetrics is now already able to retrieve information from Asterisk.
Instead, if you have a server cluster add or modify these values as follows:

    default.queue_log_file=cluster:*
    
    # Put here the server names (A, B, C, etc could be a choice) separated
    # by a pipe.
    cluster.servers=A|B|C
    
    # For each of them put these values, replacing 'servername' with 
    # the name chosen above
  	cluster.servername.manager=tcp:user:pass@serverhost 
    		# user:pass are the credentials for the Asterisk's AMI interface.
    		# You might have to configure a user in each of the 
    		# Asterisk servers' /etc/manager.conf file.
    		# serverhost is the IP address of that server.
  	cluster.servername.queuelog=sql:partition-name
    		# partition-name is the value of PARTITION in the Qloader's
    		# configuration file in that server.

Now, to enable QueueMetrics to place calls and log the agents on and off the queues you need to include the QueueMetrics dialplan in Asterisk. You can find it in _/usr/local/queuemetrics/qm-current/WEB-INF/mysql-utils/extensions-examples_.

The last thing to do is to give QueueMetrics' configuration files the right permissions in order to edit them directly from the QueueMetrics web interface:

    cd queuemetrics-path/WEB-INF
    chmod a+w configuration.properties
    chmod a+w tpf.properties

### 4.3 Does it work?

QueueMetrics has some test tools to check if all is running the right way and for troubleshooting.
The System Diagnostic Tools page offers a number of tools that check database and AMI connections and more (see Troubleshooting).

## 5 The Hotdesking Mode

When you install it, QueueMetrics is working in Extension mode. It is meant for scenarios where one phone is always used by the same person and one person always uses the same phone.
If you want to let your agents to work from any phone the Hotdesking mode comes to help you. 
In Hotdesking mode, a person (or better, an agent) can sit at whatever phone and logs in QueueMetrics to tell it who he or she is, what phone is using and what queues wants to work on.
We always suggest using the Hotdesking mode.

TIP:	If you've used the Espresso installer, QueueMetrics is already running in Hotdesking mode.

To enable the hotdesking mode you need to add or change the following values in the configuration.properties file.

    # With 0 hotdesking is disabled, any other value > 0 enables 
    # the hotdesking and is the amount of time (in seconds) how much 
    # time QM looks back for the hotdesking. (86400 is one day).
    default.hotdesking=86400
    
    # Enable the Hotdesking buttons in the Agent's page.
    callfile.agentpause_ht.enabled=true
    callfile.agentpause_ht.channel=Local/32@queuemetrics/n
    callfile.agentpause_ht.extension=10
    callfile.agentpause_ht.context=queuemetrics
    
    callfile.agentunpause_ht.enabled=true
    callfile.agentunpause_ht.channel=Local/33@queuemetrics/n
    callfile.agentunpause_ht.extension=10
    callfile.agentunpause_ht.context=queuemetrics
    
    callfile.agentaddmember_ht.enabled=true
    callfile.agentaddmember_ht.channel=Local/35@queuemetrics/n
    callfile.agentaddmember_ht.extension=10
    callfile.agentaddmember_ht.context=queuemetrics
    
    callfile.agentremovemember_ht.enabled=true
    callfile.agentremovemember_ht.channel=Local/37@queuemetrics/n
    callfile.agentremovemember_ht.extension=10
    callfile.agentremovemember_ht.context=queuemetrics
    # Disable the Extension mode buttons
    callfile.agentlogin.enabled=false
    callfile.agentlogoff.enabled=false


## 6 Applying a License

If you have a QueueMetrics license key you can apply it from the license page  by clicking on Install new license key. You have to be logged as an administrator user.
Alternatively you can modify the LICENZA_ARCHITETTURA parameter in the _/usr/local/queuemetrics/tomcat/webapps/queuemetrics/WEB-INF/web.xml_ file.

## 7 Troubleshooting

### 7.1 QueueMetrics is not available

Often this is a firewall problem. First of all, check if QueueMetrics is running:

    lsof -i -P | grep 8080

If it returns something then QueueMetrics is running; if so, do try temporarily stopping the firewall:

    service iptables stop

Now retry pointing your browser to _http://hostname:8080/queuemetrics_.

### 7.2 Editing the configuration.properties file

The configuration.properties file is located in the system path but you can edit it from QueueMetrics' interface clicking on Edit system parameters.

### 7.3 Finding the system path

If you need to know where QueueMetrics is installed you can find the system path in the license page.

### 7.4 The System Diagnostic Tools

QueueMetrics has some tools that help you to solve most of the possible issues. 
You can find them clicking on System Diagnostic Tools or pointing your browser to _http://hostname:8080/queuemetrics/dbtest/_.
* _View configuration_: this page lets you know what configuration properties are used by Queuemetrics and the JVM.
* _AMI tester_: it's useful to test to have a working AMI connection with the Asterisk server(s).
* _Live DB Inspector_: use it if you want to check QueueMetrics is receiving the correct data from the Asterisk server(s).
* _RAM caching_: it permits to look at the QueueMetrics' caching stats and to clean the cache when necessary.

### 7.5 QueueMetrics FAQs

The most common questions are answered in the QueueMetrics FAQs section in our website: _http://queuemetrics.com/faq.jsp_

### 7.6 The Loway ticketing system

If you experience any problem with QueueMetrics ask us on _http://support.loway.ch_.

## 8 Conclusion

QueueMetrics has a lot of configuration options to enable you personalizing it for your needs. We suggest you to read about them on the user manuals at http://queuemetrics.com/manual_list.jsp .
