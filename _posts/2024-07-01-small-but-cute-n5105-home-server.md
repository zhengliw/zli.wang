---
layout: post
title: "Small, but cute - N5105 Home Server"
author: Zhengli Wang
date: 2024-07-01
categories: 
  - "home-server"
coverImage: "1719329742411-scaled.jpg"
---

Previously, I showed you my how I [fixed the WiFi problem](https://crunchystudio.cc/2024/06/23/fixing-wifi-the-beginning-of-a-self-host-journey/ "Fixing WiFi – The Beginning of a Self-Host Journey") in our apartment and finally obtained an public IPv4 address from my ISP. With the fundamental issues out of the way, I can finally set up my own little self-host environment. My laptop was great for that, but eventually, I had to put Windows back on it since it was needed for some work.

A while back, I didn't tinker around with technology that much and just stuck with whatever worked at the moment. Self-hosting was cool, but not quite necessary to be honest. Until January this year, when I discovered an interesting device type referred to as a software router.

You've probably heard of [OpenWrt](https://openwrt.org/): Instead of using your ISP router (yuck :P), you run your own router that handles everything from basic routing, NAT, DHCP to VPNs, Firewall and WiFi if you don't have it separately. The Linux-based OpenWrt is the software that makes it easy to do so. You either directly get an image that is tailored for a supported router, or you put it onto a USB drive and run the system on a generic PC system. The second option allows for more extensibility and running other services on top of OpenWrt, but OpenWrt itself was appealing enough that I looked into devices that could run it. So I bought [the Beikong N5105](https://3.cn/2-1nbVkT) off jd.com, and one of the showcase images looked like this in case you're wondering...

\[Update 6th July, 2024\] Reading through this post again today, I noticed I never mentioned on what platform this server runs... It's a Celeron N5105, so it's an x86\_64 device. 😛

![A front panel and back panel showcase of the Beikong N5105 software router](/assets/images/4daf8070f801faf1748e5113073b982a.jpg)

I bought the variant without RAM and storage since buying them separately was a bit cheaper and I had a M.2 SSD lying around. I was pretty impressed when I first got it, and I knew it was a perfect opportunity to see how an "actual" server feels like. It has a lot of ports, including an HDMI/DP and a serial port in case I screw up my network configuration, couple of USB ports, a M.2 NVMe and a SATA connection on its inside. It's probably more upgradable than any lightweight laptop out there today, with its two SO-DIMM RAM slots instead of soldered RAM... Also, I heard some manufacturers started soldering storage as well? ¯\\\_(ツ)\_/¯

As for the SIM card and WiFi slots in the picture, I don't think I found any modems or WiFi chips inside the device. I suppose it's there in case you want to upgrade it with some antennas or a mobile broadband modem, but it's not important for me since this server will run at home. :P

After throwing in the SSD and the 4GB RAM stick I bought, it was time to choose an OS to install, and that's when I changed my mind. With this thing, you could do so much more than just running OpenWrt, and it makes no sense to go through the hassle of setting things up and then having nothing to expose to the internet with... The ISP router sucks, but it works at least. It had basic port forwarding options, which is all I needed. With that I mind, I scratched the idea and threw [Ubuntu Server 22.04 LTS](https://ubuntu.com/ "Ubuntu Server 22.04 LTS") on that thing. It was January, and 24.04 wasn't out yet...

The installation went quick and I could get the system running pretty easily. I configured netplan to let the system use DHCP to obtain its addresses, and marked three of the ports as optional since the wait-online service would otherwise delay the boot until all ports have connection when only one port is actually connected... And also, I secured SSH so only hosts with a valid key file can log in and disabled password authentication. Come hack me now! 😎 (please don't)

Running for about five months already, here's how it looks right now since I couldn't find an earlier photo on my phone...

![](/assets/images/1719329742394-scaled.jpg)

![](/assets/images/1719329742368-scaled.jpg)

![](/assets/images/1719329742411-scaled.jpg)

Doesn't it look amazing! :P The lights also feel weirdly calming to watch, maybe that's something to do when I get bored...

With this in place now, I installed Docker onto this thing. I discovered Docker a bit earlier but never had an environment to experiment with it, so the home server is the ultimate opportunity for it. Looking back from now, all my services are running in Docker. And using bind mounts, all software configuration and stuff are all well organized and in one place... with the exception for Nextcloud. The developers behind the all-in-one variant must have gone really, really crazy... I will write a post just to tell all about it some time.

On the network side of things, the ISP router supported [noip.com](http://noip.com "noip.com") so I just went with that for DDNS and a free hostname. I later moved to Cloudflare, but that's a topic for another post. :P

So yeah, that was the initial setup of my server. I still am not sure how detailed I want to go with this series since sometimes, I just deploy something and forget how I did it or even when I did it... But I will definitely go through all the services I have running right now. So, stay tuned, see you next post!

Also, forgot to publish this post since big things happened since I started writing this. It's now the first of July, and I completely redesigned my network... I won't spoil everything, you will get to know the setup in the following posts. :P Make sure you're subscribed!
