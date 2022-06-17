---
layout: post
title:  "SSH to VirtualBox guest with NAT network adapter"
date:   2022-06-17 19:19:00 +0300
tags: virtualbox nat ssh scp
categories: issue solution
---
# Issue

I was forced to switch my VBox based VM from "bridged networking" to the NAT networking.
In VBox that means that from guest system I can access internet but it is by default impossible
to access any network service running inside VM from outside world. Even from the host machine
you cannot connect to the guest system.

Then I had to copy some file from the guest to the host. While there are such options
as VBox'es shared folders or emailing files from the guest I wanted to use simple ssh or scp
command line.

# Investigation

At first I did a quick lookup for the port numbers used by scp and ssh. Result is port 22.

Some resources from the "External Links" below suggested to configure additional network adapter (in Host-Only networking mode) for VM then do some 
configuration steps in the guest. But I eventually found simpler way using port forwarding rules with NAT networking.

# Solution

To configure port forwarding do the following steps:

- open VM's network settings
- check that network adadpter is attached to `NAT` - all this solution is checked with NAT only
- expand `Advanced` section to see `Port forwarding` button
- click `Port forwarding` button then click `Plus` icon to add new port forwarding rule
- for our ssh/scp issue we need to add rule with
  - protocol: TCP
  - host ip: 127.0.0.1
  - host port: 2222
  - guest ip: leave it blank
  - guest port: 22
- host ip value "127.0.0.1" only allows access from the host machine; if this property is blank it means connections from any external machine is allowed too

With above configuration I can run `pscp.exe` from Windows host machine this way:

    pscp.exe -P 2222 igor@127.0.0.1:/home/igor/arc.tar.gz C:\Users\igor\Downloads\

`-P 2222` parameter is essential here - it forces pscp to use non-standard port.

# External Links

[6.4. Network Address Translation Service](https://www.virtualbox.org/manual/ch06.html#network_nat_service)

[6.7. Host-Only Networking](https://www.virtualbox.org/manual/ch06.html#network_hostonly)

[VirtualBox host-only network - ssh to remote machine](https://code-maven.com/virtualbox-host-only-network-ssh-to-remote-machine)

[How to Forward Ports to a Virtual Machine and Use It as a Server](https://www.howtogeek.com/122641/how-to-forward-ports-to-a-virtual-machine-and-use-it-as-a-server/)

[VirtualBox lab: Port Forwarding](https://nsrc.org/workshops/2014/btnog/raw-attachment/wiki/Track2Agenda/ex-virtualbox-portforward-ssh.htm) (solution is based on this)
