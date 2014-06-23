# QueueMetrics + WebRTC + Elastix how to

This file is a step by step guide to integrate Icon, the new QueueMetrics agent realtime page iwith embedded WebRTC softphone, with Elastix 


## Step 1: Install Elastix
Download Elastix 2.4.0 Stable (we tested the 32 bit version) and proceed installing the ISO as usual.

## Step 2: Upgrade to Asterisk 11.7.0
* Access to the Elastix configuration panel and click on System->Updates menu. Then switch to the Package view by clicking on the left side link.
* Click on the Show Filter menu then type asterisk inside the "Name" edit box. Select "All" in the status dropdown then click on "Search"
* In the resulting table, click "Install" on asterisk 11.7.0 row then wait until the process terminates

## Step 3: Installing QueueMetrics with Espresso
* Open an SSH session pointing your SSH client to the Elastix machine
* Type wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
* Install QueueMetrics by typing yum install queuemetrics-espresso

The yum command will download QueueMetrics and all of its dependencies and install them on your system. This may take a while, depending on your internet connection speed. When asked to confirm the installation, type "y" to proceed.
When the installation is complete, you will have to point your browser to the address http://myserver:8080/queuemetrics and you should see the QueueMetrics home page (you will be asked to accept the EULA first).
Log in as demoadmin with password demo; if you don’t see any errors, then your system is correctly configured.

## Step 4: Configure an inbound queue in Elastix
* Access to the Elastix configuration panel and click on PBX menu
* On the left side menu, select Queues
* Type 300 on the Queue Number and Queue Name fields then click on Submit Changes. 
* Apply configuration changes on Asterisk PBX

