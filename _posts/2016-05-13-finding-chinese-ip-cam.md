---
layout: post
title: Setting up a chinese IP camera without manual
date: "2016-05-13"
tags: 
    - networking
    - wireshark
    - IP-camera
    - IP-address
---
This post is probably a no-brainer for quite a lot of people. And it should have been for me too. I don't know if my brain was just not working from the dry and hot el niño weather, but it took me quite a while to figure this out. Since chances are there are others with the same problem, I thought it'd be worth to write it up.

My boss bought a few IP-camera's on Ali-Express together with a recorder to monitor some areas with expensive equipment. I won't go into [the security of these types of camera's](https://www.youtube.com/watch?v=B8DjTcANBx0) here. And of course the cheap stuff from Ali-Express comes in an unmarked box without any manuals. There's also no brand to be found on the camera. Normally it is a matter of pride to not need manuals. But the problem with IP camera's is that you need to know their network settings somehow. That is where I ran into trouble.

The camera's have both a wired and wireless connection available. I was hoping the camera would be set to dynamic IP, so it would pickup an IP address from the DHCP server on my router, and I could see it in the router status page when I connected it with a wire. No such luck though. The next thing I tried was setup a DHCP server on my laptop. It's pretty straightforward to install dnsmasq, and the only settings you need to adjust are the interfaces directive (make sure it's not running on your wifi interface and mess up the wifi) and DHCP-range directive. And you need to set a static IP in the same sub-net for your wired network card. I'm not going into the details for this, but suffice it to say this didn't work either. So clearly the IP-camera wasn't set for a dynamic IP. I tried a few of the simple IP addresses like 192.168.0.2 192.168.1.2, etc but none worked. I was ready to give up what my brain finally kicked into gear: [wireshark](https://www.wireshark.org/).

I have used wireshark plenty of times before to analyse network traffic. It is an awesome tool. It allows you to see all network traffic passing through your network interfaces. I did see the light on the wired network interface of the camera blinking, so I knew it was trying to communicate through the network. So I connected the camera with a cable straight into the network card of my laptop. I ran wireshark on that network interface and looked at the traffic coming by. Very quickly I noticed [ARP-packets](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) where an IP-address 192.168.1.136 was [asking who has](https://ask.wireshark.org/questions/5412/what-does-arp-42-who-has-19216811-tell-192168133-mean) 192.168.1.1. This means that there is a device with IP-address 192.168.1.136 who is looking to contact IP-address 192.168.1.1. Chances are that this is the camera looking for a router to contact the internet. Indeed the camera's turned out to be configured with NTP enabled to sync it's hardware clock with a server on the internet, and it was trying to get on the internet. 
So now I knew the IP-address of the camera was 192.168.1.136. From that point onward it was straightforward again. Just set a static IP (192.168.1.2) on my laptop wired network interface, have a cable between the camera and the laptop, and I could access the web-interface of the camera on http://192.168.1.136.

Configuring could finally commence. It took me way to long to realize to get wireshark. It was pretty easy once I remembered.
