#Introduction
The purpose of this document is to show the basics involved in configuring an Amazon EC2 VM as a Virtual Private Network (VPN) and then how to connect to this service using a Linux machine. There are further instruction and resources explaining how to connect to such a VPN using an iOS or Android device in the Bibliography.

##NOTE
This guide assumes the Amazon EC2 VM is an Ubuntu Server install, although it should be easy enough to follow for almost any *NIX distribution. This guide also assumes you have the Amazon EC2 VM installed, setup and have access to a terminal over SSH.

##Installation
This guide will cover the configuration and setup of the PPTP daemon. Firstly install the software from the repositories. On Ubuntu systems this can be done with the following command.

`sudo apt-get install pptpd`


Configuration of the PPTP daemon is relatively painless, using your preferred text editor with root priviledges open the following
file: `/etc/pptpd.conf`

At the very bottom of the file, uncomment and edit the following lines
```
localip 192.168.240.1 		# <1>
remoteip 192.168.240.2-9	# <2>
```
<1> The *localip* is the IP address of the VPN itself on its private network.
<2> The *remoteip* is the range of IP's used by clients who connect to the VPN.

This setting ensures that the VPN gets the IP address `192.168.240.1`, while the clients will recieve an IP in the range of `192.168.240.2 - 192.168.240.9` which means there are a maximum of 8 clients allowed to connect. If this number is too low, simply increase the IP address range in the settings.

It is a good idea to ensure that this network has access to DNS settings, so we will next configure the default DNS servers. Again with your preferred root privilege enabled text editor open the following file: `/etc/ppp/pptpd-options`

Add the following lines to this file.
```
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```
The next step is to add credentials to the VPN system to allow clients to connect. You may add as many different users as you see fit using this method.

Change _USERNAME_ and _PASSWORD_ in the following to the credentials used by each user of the VPN system. Repeat this
step to add each user to the system.

`echo "USERNAME pptpd PASSWORD *" | sudo tee -a /etc/ppp/chap-secrets`

The PPTP daemon can now be restarted in order to take the new settings into account. You can restart the service using the following command.

`sudo /etc/init.d/pptpd restart`

At this point it will be possible to open a connection to the VPN server, although it will not be possible to gain access to the internet though the VPN until we enable packet forwarding. Packet forwarding is enabled on an Ubuntu system in the following way. Using a root text editor open the following file: `/etc/sysctl.conf`

Uncomment the following line:

`net.ipv4.ip_forward=1`

We next need to ensure this system configuration file is reloaded, we can do this using the following command:

`sudo sysctl -p`

The final step is to add a network address translation or NAT, which forwards on packets to the main internet connected interface which is most likely _eth0_ on an Ubuntu system.

Open the following file for editing `/etc/rc.local` and place this line of code just above the _exit 0_

`sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`

This ensures that should the server be rebooted the NAT will be reconfigured at boot time once more.


#Testing the VPN
The VPN is now configured for access from clients. If you have not configured the Amazon EC2 to have a static IP, you will need to carry out a number of further steps at this stage to ensure the VM is reachable using a DDNS system, further instructions are availible in the following website in this document we are working on the assumption the Amazon EC2 VM has been given a fixed IP address. For instructions on configuration for iOS or Android devices see the following website

##Configuring a Linux Client to connect to the VPN
In order to connect to the VPN from a Linux machine, the PPTPClient must be installed. It can be installed using the following

command:
Ubuntu:
`sudo apt-get install pptpclient`

##Bibliography:

[bibliography]
- [[[dikant1]]] 'How to setup a vpn server on amazon ec2', Peter Dikant. 2010. http://www.dikant.de/2010/10/08/setting-up-a-vpn-server-on-amazon-ec2/
- [[[dikant2]]] 'Configuring a PPTP VPN on iOS and Android', Peter Dikant. 2011. http://www.dikant.de/2011/10/03/configuring-a-pptp-vpn-on-ios-and-android/
- [[[dlabal]]] 'PPTP connection tutorial', Jan Dlabal. 2012. http://houbysoft.com/v/en/papers/pptp/
