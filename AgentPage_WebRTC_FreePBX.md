# QueueMetrics + WebRTC + FreePBX how to

This file is a step by step guide to integrate Icon, the new QueueMetrics agent realtime page with embedded WebRTC softphone, with FreePBX.


## Step 1: Install FreePBX
* Download FreePBX BETA-6.12.65-13 (we tested the 32 bit version) and proceed installing the ISO as usual. Select FreePBX 6.12.65 with Asterisk 12, Full Install. 
Asterisk 12.3.2 will be installed.

## Step 2: Installing QueueMetrics with Espresso
* Open an SSH session pointing your SSH client to the FreePBX machine.
* Type wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
* Install QueueMetrics by typing yum install queuemetrics-espresso

The yum command will download QueueMetrics and all of its dependencies and install them on your system. This may take a while, depending on your internet connection speed. When asked to confirm the installation, type "y" to proceed.
When the installation is complete, you will have to point your browser to the address http://myserver:8080/queuemetrics and you should see the QueueMetrics home page (you will be asked to accept the EULA first).
Log in as demoadmin with password demo; if you don't see any errors, then your system is correctly configured.

## Step 3: Configure an inbound queue in FreePBX
* Access to the FreePBX configuration panel and click on Applications->Queues menu.
* Type 300 on the Queue Number and Queue Name fields then click on Submit Changes. 
* Apply configuration changes on Asterisk PBX.

## Step 4: Configure a set of SIP account for WebRTC clients
* Through an SSH connection, enter in the /etc/asterisk folder and edit the sip_custom.conf file.
* Add the following sketch. This defines a template for agents using the WebRTC softphone integrated in Icon, the QueueMetrics realtime page.   

 
```
[WebRTC](!)
type=peer
host=dynamic
nat=force_rport,comedia
context=from-internal
callcounter=yes
busylevel=1
call-limit=1
encryption = yes
qualify=yes
avpf = yes
allow=all
icesupport = yes
srtpcapable=yes
videosupport=no

[101](WebRTC)
username=101
secret=101

[102](WebRTC)
username=102
secret=102
```

The sketch defines also two SIP accounts (101 and 102) used by two sample agents defined in QueueMetrics.


## Step 5: Enable WebRTC connections in Asterisk
* Through an SSH connection, enter in the /etc/asterisk folder and edit the sip_general_custom.conf file.
* Add the following keys to enable the ws transport:


```
allowguest=no
transport=udp,ws,wss
```

* Edit rtp_additional.conf file located on the same folder and modify the key icesupport to true:


 
```
icesupport=true
```
	
* Save the file and restart asterisk by issuing the command:


```
/etc/init.d/asterisk restart
```	
	

## Step 6: Define two sample callers extensions in FreePBX
Two samples extensions are used in this tutorial to simulate calls flowing to the inbound queue. To do this, proceed with the following steps.
* Access to the FreePBX configuration panel and click on Applications->Extensions menu.
* Select Generic CHAN SIP Device from the dropdown, then click "Submit".
* Fill the User Extension, Display Name and secret fields with 200, then press "Submit".
* Repeat the same for the extension 201.
* Apply configuration changes to the Asterisk PBX.

## Step 7: Setup queues and agents in QueueMetrics
* Access to the QueueMetrics administration page by pointing the browser to  http://myserver:8080/queuemetrics and using demoadmin and demo as username and password.
* Click on Setup wizard link located on Administrative tools settings submenu.
* Select "File" as data source. Follow the wizard until completes. The definition of the queue 300 is now imported in QueueMetrics.
* Click on Edit queues under Edit QueueMetrics settings submenu.
* Click on the icon "Edit agents" for the queue 300 and assign the agents agent/101 and agent/102 as a Main.

We need to associated the SIP credentials defined at step 5 to the agents in QueueMetrics. To do that:
* In QueueMetrics, click on the Cfg Agents tab in the upper menu, then click on the pencil icon on the agent 101 row.
* Fill the Current terminal, WebPhone Username and WebPhone Password with the value 101 then click on Save.
* Repeat the same for the agent 102 (but with credentials defined for the extension 102).

## Step 8: Configure the QueueMetrics WebRTC softphone
QueueMetrics needs to know the IP where the softphone will register. To do that:
* From the QueueMetrics home page, click on the Edit system parameters link under the Administrative tools section.
* Search the default.sipaddress, default.websocketurl and default.rtcWebBreaker keys and edit as following:
 

```
default.sipaddress=XXX.XXX.XXX.XXX
default.websocketurl=ws://XXX.XXX.XXX.XXX:8088/ws
default.rtcWebBreaker=true
```

where XXX.XXX.XXX.XXX is the IP address valid for your install: this is the IP address associated to your PBX.

* Save and logout from the QueueMetrics administation panel.

## Step 9: Use the WebRTC softphone integrated in Icon
* Access to the QueueMetrics Agent page by pointing a Chrome browser to  http://myserver:8080/queuemetrics and using Agent/101 and 999 as username and password.
* Enter on Icon by clicking on Show inbound calls for agent Agent/101 under Inbound calls menu. Icon will load. 
* Click on the top left menu and show the Agent Logon panel. 
* Click on the "queue 300" item you can find under Available Queues, then click on the ">" button to start a log-in procedure. In a few seconds the bullet icon located on the
right side top menu should mark green. You're now logged into the queue 300.
* By mean of an external phone you previously registered with SIP credential defined in the step 7 (please note chan_sip is running on port 5061), place a call to the internal extension 300.

The agent softphone pops up and starts ringing. You can answer by clicking on the red flashing icon on this panel.

NOTE: Chrome asks for a confirmation on sharing microphone and speakers each time a new call flows in. This could be avoided by setting up QueueMetrics to be served through HTTPS
instead of HTTP. Please refer to the QueueMetrics Advanced Configuration Manual located at http://manuals.loway.ch/QM_AdvancedConfig-chunked

