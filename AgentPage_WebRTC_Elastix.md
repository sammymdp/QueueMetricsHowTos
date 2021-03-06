# QueueMetrics + WebRTC + Elastix how to
_Author: Marco Signorini_   
This file is a step by step guide to integrate Icon, the new QueueMetrics agent realtime page with embedded WebRTC softphone, with Elastix.


## Step 1: Install Elastix
* Download Elastix 2.4.0 Stable (we tested the 32 bit version) and proceed installing the ISO as usual.

## Step 2: Upgrade to Asterisk 11.12.0
* Access to the Elastix configuration panel and click on System->Updates menu. Switch to the Package view by clicking on the left side link.
* Click on the Show Filter menu then type asterisk inside the "Name" edit box. Select "All" in the status dropdown then click on "Search".
* In the resulting table click "Install" on asterisk 11.12.0 row, then wait until the process terminates.

(Tested with asterisk 11.7.0 and asterisk 11.12.0)


## Step 3: Installing QueueMetrics with Espresso
* Open an SSH session pointing your SSH client to the Elastix machine.
* Type wget -P /etc/yum.repos.d http://yum.loway.ch/loway.repo
* Install QueueMetrics by typing yum install queuemetrics-espresso

The yum command will download QueueMetrics and all of its dependencies and install them on your system. This may take a while, depending on your internet connection speed. When asked to confirm the installation, type "y" to proceed.
When the installation is complete, you will have to point your browser to the address http://myserver:8080/queuemetrics and you should see the QueueMetrics home page (you will be asked to accept the EULA first).
Log in as demoadmin with password demo; if you don't see any errors, then your system is correctly configured.

## Step 4: Configure an inbound queue in Elastix
* Access to the Elastix configuration panel and click on PBX menu.
* On the left side menu, select Queues.
* Type 300 on the Queue Number and Queue Name fields then click on Submit Changes. 
* Apply configuration changes on Asterisk PBX.

## Step 5: Configure a set of SIP account for WebRTC clients
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

## Step 6: Enable WebRTC connections in Asterisk
* Through an SSH connection, enter in the /etc/asterisk folder and edit the http.conf file.
* Set the key enabled=yes and bindaddr=0.0.0.0 configuration variables.
* Save the file and exit.
* Edit the file sip_general_custom.conf and add the following keys to enable the ws transport:


```
allowguest=no
transport=udp,ws,wss
```	

## Step 7: Define two sample callers extensions in Elastix
Two samples extensions are used in this tutorial to simulate calls flowing to the inbound queue. To do this, proceed with the following steps.
* Access to the Elastix configuration panel and click on PBX menu.
* On the left side menu, select Extensions.
* Select Generic SIP Device from the dropdown, then click "Submit".
* Fill the User Extension, Display Name and secret fields with 200, then press "Submit".
* Repeat the same for the extension 201.
* Apply configuration changes to the Asterisk PBX.

## Step 8: Setup queues and agents in QueueMetrics
* Access to the QueueMetrics administration page by pointing the browser to  http://myserver:8080/queuemetrics and using demoadmin and demo as username and password.
* Click on Setup wizard link located on Edit QueueMetrics settings submenu.
* Select "File" as data source. Follow the wizard until completes. The definition of the queue 300 is now imported in QueueMetrics.
* Click on Edit queues under Edit QueueMetrics settings submenu.
* Click on the icon "Edit agents" for the queue 300 and assign the agents agent/101 and agent/102 as a Main.

We need to associated the SIP credentials defined at step 5 to the agents in QueueMetrics. To do that:
* In QueueMetrics, click on the Cfg Agents tab in the upper menu, then click on the pencil icon on the agent 101 row.
* Fill the Current terminal, WebPhone Username and WebPhone Password with the value 101 then click on Save.
* Repeat the same for the agent 102 (but with credentials defined for the extension 102).

## Step 9: Configure the QueueMetrics WebRTC softphone
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

## Step 10: Use the WebRTC softphone integrated in Icon
* Access to the QueueMetrics Agent page by pointing a Chrome browser to  http://myserver:8080/queuemetrics and using Agent/101 and 999 as username and password.
* Click on Icon by clicking on Show inbound calls for agent Agent/101 under Inbound calls menu. Icon will load. 
* Click on the top left menu and show the Agent Logon panel. 
* Click on the "queue 300" item you can find under Available Queues, then click on the ">" button to start a log-in procedure. In a few seconds the bullet icon located on the
right side top menu should mark green. You're now logged into the queue 300.
* By mean of an external phone you previously registered with SIP credential defined in the step 7, place a call to the internal extension 300.

The agent softphone pops up and starts ringing. You can answer by clicking on the red flashing icon on this panel.

NOTE: Chrome asks for a confirmation on sharing microphone and speakers each time a new call flows in. This could be avoided by setting up QueueMetrics to be served through HTTPS
instead of HTTP. Please refer to the QueueMetrics Advanced Configuration Manual located at http://manuals.loway.ch/QM_AdvancedConfig-chunked
