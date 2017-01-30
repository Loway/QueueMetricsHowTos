# How to Get a Working QueueMetrics System

QueueMetrics is able to operate in various server configurations in order to meet all your needs.

Depending on how big is your system you might want to have a single-server solution (Asterisk and QueueMetrics on the same server), a separated-server solution (Asterisk and QueueMetrics reside on two different servers) and a cluster solution (one QueueMetrics server monitoring many Asterisk servers).

# 1 Understanding How QueueMetrics Works

All the data which QueueMetrics works on are stored in Asterisk's queue_log file. In a single-server configuration, QueueMetrics is able to read directly from this file, though this is not an efficient approach.

The Qloader script provides an effective way to retrieve information from the _queue&#95;log_ file without overloading the Asterisk server(s) and it's essential in a cluster-server configuration.

It reads the new data from the queue_log file and sends it to the QueueMetrics database wherever it is.   
We always suggest to use the Qloader script whatever configuration you have.

# 2 Installing QueueMetrics with Espresso

If you are setting up a single-server solution on an Asterisk-ready Linux distribution (eg. Elastix, AsteriskNOW, FreePBX, etc) you can get a working QueueMetrics system in a few minutes.

TIP:	Check if the Espresso installer supports your distribution version in the _Supported PBXs_ section at http://queuemetrics.com/manuals/QM_Espresso-chunked .

Just add our repository and install the package:

    wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
    yum install queuemetrics-espresso
    
It will detect your operating system and your Asterisk versions and configure QueueMetrics to work with them.
It will also install and configure all the other software needed (Tomcat, MySQL, Qloader script) and creates a working AMI connection to Asterisk.

Now you can point your browser to http://hostname:8080/queuemetrics and start working.

TIP:	The default username is 'demoadmin' and the password is 'demo'.

# 3 Installing QueueMetrics manually

With this step we are going to install QueueMetrics on a server, though it still need to be configured in order to work.

### 3.1 On a RPM-based distribution

Just add our repository and install the package:

    wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
    yum install queuemetrics

Now, install the Qloader script in each of your Asterisk boxes.
    
	wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
	yum install qloaderd
    
### 3.2 On another system

Although we don't suggest it, [click here to learn how to install QueueMetrics manually](QueueMetricsManualInstall.md) on a non-RPM Linux distro or Windows and any other system where you can run Tomcat.

If you ask us, running QueueMetrics on CentOS (or whatever CentOS flavour, like Elastix or FreePBX) is always the best solution, even on a virtual machine.

When you finished with the installation come back here to know how move forward.

### 3.3 Does it work?

Now, to check that everything works you can open your browser and point it to _http://hostname:8080/queuemetrics_, the first time you run it you might get an error, it will disappear in a few seconds and QueueMetrics will guide you through the database configuration; probably you'll just need to confirm without make any change.
After that you will be able to log in with the default username _demoadmin_ and password _demo_.
Obviously, at the the moment, no data is shown.

# 4 Configuring QueueMetrics

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

If Qloader is on a different machine that QueueMetrics (and MySQL) you will need to create a new database user able to connect remotely.

Log in MySQL as *root*:

	# mysql -uroot -p
	
One you're in create a new remote user and give it all grants on the *queuemetrics* database:

	mysql> CREATE USER 'queuemetrics'@'%' IDENTIFIED BY 'javadude';
	mysql> GRANT ALL PRIVILEGES ON queuemetrics . * TO 'queuemetrics'@'%';
	mysql> FLUSH PRIVILEGES;

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

### 4.3 Include QueueMetrics dialplan in Asterisk

Now, to enable QueueMetrics to place calls and log the agents on and off the queues you need to include the QueueMetrics dialplan in Asterisk. If you installed QueueMetrics with *yum*, you can find it in **/usr/local/queuemetrics/qm-current/WEB-INF/mysql-utils/extensions-examples/extensions\_queuemetrics\_18.conf**.

Copy the file under **/etc/asterisk/** then add the following line at the end of your **extensions.conf**, or **extensions_custom.conf** if your PBX has this file:

	#include extensions_queuemetrics_18.conf

Then open the Asterisk CLI and reload the dialplan:

	Asterisk CLI> dialplan reload

### 4.4 Does it work?

QueueMetrics has some test tools to check if all is running the right way and for troubleshooting.
The System Diagnostic Tools page offers a number of tools that check database and AMI connections and more (see Troubleshooting).

# 5 Applying a License

If you have a QueueMetrics license key you can apply it from the license page  by clicking on Install new license key. You have to be logged as an administrator user.
Alternatively you can modify the LICENZA_ARCHITETTURA parameter in the _/usr/local/queuemetrics/tomcat/webapps/queuemetrics/WEB-INF/web.xml_ file.

# 6 Troubleshooting

### 6.1 QueueMetrics is not available

Often this is a firewall problem. First of all, check if QueueMetrics is running:

    lsof -i -P | grep 8080

If it returns something then QueueMetrics is running; if so, do try temporarily stopping the firewall:

    service iptables stop

Now retry pointing your browser to _http://hostname:8080/queuemetrics_.

If it works now, you should setup your firewall in order to accpet incoming connections to port 8080.

### 6.2 Editing the configuration.properties file

The configuration.properties file is located in the system path but you can edit it from QueueMetrics interface clicking on *Edit system parameters*.   
When you modify this file, always remember to log out and log again in QueueMetrics to reload the configuration.

### 6.3 The System Diagnostic Tools

QueueMetrics has some tools that help you to solve most of the possible issues. 
You can find them clicking on System Diagnostic Tools or pointing your browser to _http://hostname:8080/queuemetrics/dbtest/_.
* _View configuration_: this page lets you know what configuration properties are used by Queuemetrics and the JVM.
* _AMI tester_: it's useful to test to have a working AMI connection with the Asterisk server(s).
* _Live DB Inspector_: use it if you want to check QueueMetrics is receiving the correct data from the Asterisk server(s).
* _RAM caching_: it allows you to look at the QueueMetrics caching stats and to clean the cache when necessary.

### 6.4 QueueMetrics FAQs

The most common questions are answered in the QueueMetrics FAQs section in our website: _http://queuemetrics.com/faq.jsp_

# 7 Conclusion

QueueMetrics has a lot of configuration options to enable you personalizing it for your needs. We suggest you to read about them on the user manuals at http://queuemetrics.com/manual_list.jsp .
