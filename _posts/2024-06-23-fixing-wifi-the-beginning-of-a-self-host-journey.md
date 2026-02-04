---
layout: post
title: "Fixing WiFi - The Beginning of a Self-Host Journey"
author: Zhengli Wang
date: 2024-06-23
categories: 
  - "home-server"
tags: 
  - "home"
  - "linux"
  - "networking"
  - "selfhost"
  - "server"
  - "ubuntu"
  - "xiaomi"
coverImage: "selfhost_lenovo_laptop-e1719155532363.jpg"
---

If you're reading through this, amazing! Because this particular site has been self-hosted by me, at home :P. Ever since I got started learning Linux and stuff, I found out there are so many things you can do with a server running Linux! But there were so many hurdles to overcome before I could reach the level of expertise I have today, so I thought... Maybe it's interesting to share my story with you! This will be the first post of a series of posts about my self-host setup. The only problem is that I sometimes am really bad at remembering things, so let's see how in-depth the posts will be... Or actually, this could as well become my primary source of documenting things throughout my self-host journey!

I believe that the earliest thing I did that can be seen as self-hosting is running an own Bedrock Minecraft server on my PC in 2022, but eventually, I gave up. Here's the story...

It began this way: I set up everything on my PC, everything is accessible locally, but of course it should be somehow public. I had no idea of CG-NAT and little idea of networking in general. I only had public IPv6 addresses, but because of confusing naming in the router management UI, I never knew that I had to disable the "Firewall" toggle that said nothing about IPv6 to allow inbound IPv6 traffic... And the UI said it would re-activate the firewall anyways after 24 hours for "security purposes"... I really want to throw a brick at my ISP.

![I hate my router: The warning it shows when deactivating the firewall](/assets/images/image-4.png)

So I got my hands on a [Linode](https://www.linode.com/) Nanode server to set up [Fast Reverse Proxy](https://github.com/fatedier/frp) (cool project, by the way) to expose my server using the Nanode cloud server. But to be honest, 5$ per month just for exposing a server seemed kind of expensive, considering how nobody played on my Minecraft server anyway and I didn't care to maintain it :(. But I believe I did host a WordPress site on the Nanode instance promoting the server though, even if it never really was maintained...

But after finding super cheap servers from [Strato](https://www.strato.de/ "Strato"), I decided to try again. But this time, I didn't really care to set up a Minecraft server again, and I used the server for setting up a OpenVPN server ([this script](https://github.com/Nyr/openvpn-install "this script") saved my life). I hosted another WordPress site, but didn't have any ideas, so I just let Strato reinstall my 1GB, 1vCore instance and reinstalled OpenVPN on there, being the only running service on there this time.

That was about the failure of me trying to self-host cool stuff, until I began to make big leaps during winter in 2023...

I bought a pair of [AX3000T Xiaomi WiFi](https://www.mi.com/xiaomi-ax3000t "AX3000T Xiaomi WiFi") router/mesh access points (they even have WiFi 6!), since there was basically no WiFi in the bedroom at the end of the hall at home. For 199 CNY each (about 27.42$), this was an absolute steal. After I got back home, I deactivated WiFi on the ISP router, and set the AX3000Ts to access point mode. I also discovered the wall Ethernet ports I didn't know we had, so I bought a small switch for the storage room where all the ports can be hooked up.

![A picture of the Xiaomi AX3000T Router/Mesh/Whatever](/assets/images/1719169864003-scaled.jpg)

The AX300T in the living room is connected directly to the ISP router, which connects to the wall port. The AX3000T in the bedroom at the end of the hall is connected to the wall port as well. All wall ports have their other end in the storage room, which is where I put the switch. As a bonus, since I have an Ethernet port in my room, I can now hook my PC up with Ethernet to enjoy the full 1000 MBit/s download speed. :P

![My network topology in the current state](/assets/images/Screenshot-2024-06-16-at-17.06.47.png)

The WiFi problem that we had since we moved in is now fixed. And some months later, casually browsing through forum posts of the ISP, I noticed posts which hinted that our contract (1000 MBit/s Down, 50 MBit/s Up) includes a public IP, although not static by default. So I decided to try my luck and called the customer support. Luckily, after not quite twenty minutes, the ISP router rebooted and revealed some new options, for example, running in bridge mode and port forwarding - CG-NAT was thereby removed. :)

I still really wasn't satisfied with the options provided in the ISP router interface, so I put it in bridge mode and used the living room AX3000T for routing and WiFi simultaneously. It had everything I needed, and I wanted to self-host again, so I removed the lid and battery of my older Lenovo laptop and ran... who would have guessed, a Minecraft server on Ubuntu Server... again. :P But to be honest I have zero idea how often I've reinstalled that thing. Sometimes I put some Linux distro on there, sometimes I just use it with Windows to get work done... I don't remember. That thing actually handled about everything I threw at it. :P (And had detachable RAM, one M.2 and one SATA slot!)

![](/assets/images/selfhost_lenovo_laptop-e1719155532363.jpg)

But eventually, I switched back to the ISP router since the Mi Router wouldn't reconnect to the upstream after the upstream goes down. I assume it"s because it wouldn't re-initiate its DHCP client, but regardless of cause, I wanted to be able to reach my home network at all time, so I set it back to Layer 2 mode (access point mode :P), re-enabled the routing functionalities on my ISP router and called it a day.

The next exciting part is actually hosting the services on my network. For that, I purchased a small N5105... thing. It's intended to be used as a software router and looks just like a Mini PC but has 4 Ethernet ports, so I found it pretty useful for other things too. It has a server-grade BIOS with a lot of settings I haven't heard of before, but never really bothered to learn about.. yet. :P I have done a bunch of things since I installed the server, and I will certainly try to remember the things I did. But I think that my writing won't catch up to the speed of my urge to install another service... :P I haven't introduced my actual server yet, but we'll get to it, don't worry. I'll do my best, and see you next post!
