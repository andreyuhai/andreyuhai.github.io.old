---
layout: post
title: "How to configure VirtualBox VM with 2 network adapters"
excerpt: "This post describes how to create a VM which has a connection to the internet and also to other VMs in the virtual network."
modified: 2013-10-24 20:43:34 +0300
categories: articles
tags: [virtualbox, virtualization, vm, network, ubuntu]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

This post describes how to create a VM which has a connection to the internet and also to other VMs in the virtual network. The easiest way to achieve it would be to set the VMs to use a bridged network configuration. In some cases, it is not possible, because of the host's network setup. I guess this has something to do with the dhcp configuration. A bit of background. I have an Ubuntu installation as the host and 2 Ubuntu Server virtual machines.

* Install a new virtual machine
* In networking settings, set the 1st network adapter to NAT. Set the 2nd adapter to host-only. In addition, a new virtual host-only network has to be created in Virtualbox preferences.
* Start the VM
* Running `ifconfig`,  it can be seen that only 1 network interface is availabe, (two, if the loopback interface is counted)
* Open `/etc/network/interfaces`, find the description for `eth0`, copy these lines and create the same description for `eth1`
* Reboot VM
* Now the VM has 2 network interfaces
* Create a new VM and repeat the previous steps
* Now both VMs have access to the Internet and can communicate between each other

Host-only mode allows VMs inside the host to communicate between one another but they can not be accessed from the outside world.
