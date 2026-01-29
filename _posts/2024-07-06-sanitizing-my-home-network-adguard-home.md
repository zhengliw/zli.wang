---
layout: post
title: "Sanitizing My Home Network - AdGuard Home"
author: Zhengli Wang
date: 2024-07-06
categories: 
  - "home-server"
tags: 
  - "adguard"
  - "cloudflare"
  - "dns"
  - "home"
  - "networking"
  - "openwrt"
  - "selfhost"
  - "server"
coverImage: "1720293938700-1-scaled.jpg"
---

Hey there! Hope you could follow along the [last post](https://crunchystudio.cc/2024/07/05/automating-the-dream-this-is-the-way-to-smart-home/) quite well. I admit that it's gotten quite long, but hope you could enjoy it nonetheless. 😛 And here I'm back, to show off other services I have running on my home server - yes, I have quite a lot of services running, and just writing about those will be enough to keep my blog updated for about a month... or two weeks if I hurry things up. Let's see. 🤪

You probably have internet access - you are reading through this page, after all... But only a handful of folks have a picture of what really happens when you visit a website. It's quite simple, right? You open your browser, your browser connects to the website server and the server responds with everything you need. Images, style sheets, HTML files... everything!

![Simple flowchart with directional arrows from web server to user directly and vice versa](https://crunchystudio.cc/wp-content/uploads/2024/07/image-7.png)

Well, that's at least how things worked in the past... I think, I'm quite young and haven't seen the ages of the internet when most sites are simple and hosted by individuals. 😕 But at least I know about how it works today...

Online ads and trackers. They're everywhere... Every shopping platform, every social media site, and basically everything else you can imagine of. If used responsibly, they are a pretty solid way to generate revenue as traffic starts to grow, especially for smaller sites...

If used _responsibly_... 😵 You get it.

![An advertisement example on a website](https://crunchystudio.cc/wp-content/uploads/2024/07/Screenshot-2024-07-06-190956-1024x729.png)

<figure>

![An advertisement example on a website](https://crunchystudio.cc/wp-content/uploads/2024/07/Screenshot-2024-07-06-191827-1024x735.png)

<figcaption>

Sites with "free software downloads" are the best ones to demonstrate online ads...

</figcaption>

</figure>

Now you've seen how cruel ads can be. I used to have only uBlock Origin to block ads, and that only worked for websites accessed inside the browser. It is already very decent and did its job really well, but when I discovered AdGuard Home, I thought... Hey, why not? 😛

<figure>

![Screenshot of AdGuard Home dashboard](https://crunchystudio.cc/wp-content/uploads/2024/07/image-9-1024x661.png)

<figcaption>

AdGuard Home WebUI... More on that in a moment

</figcaption>

</figure>

What sets AdGuard Home aside from ordinary ad blockers is the fact that it's not an plugin for browsers or an application for your OS of choice. Well, AdGuard, the company/brand behind AdGuard Home also offer apps and browser addons to block ads on a per-device basis, but AdGuard Home goes down another path - It replaces the existing DNS server on the network with its own one, so all devices get basic protection from ads, for all network traffic.

I will have to leave out a lot of details here, but if you're not familiar with DNS, or Domain Name System - it's a standard for querying IP addresses and other info associated with a hostname or more commonly known as domain names. Since devices on networks only talk to each other with IP addresses, we need a DNS server which holds a list of DNS records - those associate hostnames with IP addresses and other information. There are a handful of so-called authoritative DNS servers on the internet that get notified whenever a domain is registered and DNS records are created, and other "lower-level" DNS servers query those to get their information. Often, a local, "low-level" DNS server runs right on your home router, alongside of a DHCP server which informs the client about the most important network configuration when they join the network, including the DNS server location. It's quite an interesting topic, and [this article from Cloudflare](https://www.cloudflare.com/learning/dns/what-is-a-dns-server/) probably explains it much better than I do... But that doesn't stop me from making a graph nevertheless. 😛

<figure>

![Graph explaining DNS in a typical home network setup, demonstrating what it takes to access google.com starting from DHCP](https://crunchystudio.cc/wp-content/uploads/2024/07/image-8.png)

<figcaption>

Just a simplified example! Google has a ton of addresses...

</figcaption>

</figure>

Now, this is what a normal home network looks like. Once AdGuard Home becomes your default DNS server, things will change a bit. You see, AdGuard won't just go and forward everything to its upstream DNS servers. Instead, it first takes a look at the incoming queries and sees if any of the requested hostnames are found inside one of the so-called blocklists. And if it matches a blocklist rule, it will answer to the client as if the hostname does not exist ([NXDOMAIN](https://help.dnsfilter.com/hc/en-us/articles/4408415850003-DNS-return-codes)).

<figure>

![A diagram showing how normal websites and ad websites are handled differently by AdGuard home](https://crunchystudio.cc/wp-content/uploads/2024/07/image-10.png)

<figcaption>

Probably not a real conversation...

</figcaption>

</figure>

Now onto what AdGuard Home really has to offer. First, configuration is super simple with a sleek web-based first time setup and dashboard, so you don't have to touch any config files when configuring your DNS settings. I'm always paranoid of messing something up without a GUI, so that's amazing! In the WebUI, you can configure basically everything - upstream DNS servers, parental control to block unwanted content (for which an AdGuard API is actually leveraged instead of a static list), and of course - filtering configuration. AdGuard comes with one blocklist pre-installed, but you can always add more to block more unwanted hostnames. To block other endpoints that aren't included in the default list, you can install additional lists from the AdGuard selection or supply your own with an URL.

![A list of blocklists available to install with one-click on AGH WebUI](https://crunchystudio.cc/wp-content/uploads/2024/07/image-11-1024x443.png)

![A prompt where a custom blocklist can be supplied with an URL](https://crunchystudio.cc/wp-content/uploads/2024/07/image-12-1024x308.png)

You can even add custom rewrite rules! This is especially useful if you host a server at home like me, and don't want to access the WAN IP addresses the whole time... I rewrote crunchystudio.cc to use 192.168.1.1 instead. 😛

![AGH WebUI for configuring DNS rewrites](https://crunchystudio.cc/wp-content/uploads/2024/07/image-13-1024x168.png)

And one feature that surprised me was that this AdGuard Home thing - hold your breath - has a DHCP server built-in! I actually had to use this feature when I was [still running Ubuntu Server](https://crunchystudio.cc/2024/07/01/small-but-cute-n5105-home-server/) along with my ISP router. I installed AdGuard Home into a Docker container and put it onto the `macvlan` network Home Assistant was also using. The problem was that the IP address of AdGuard Home was 192.168.0.3. It normally meant configuring the DHCP server on my ISP router so it announces 192.168.0.3 as the DNS server, but the router only showed a toggle for de/activating DHCP altogether and no individual options. So I deactivated the built-in DHCP server on my ISP router and enabled the one in AdGuard Home. Thanks to the devs for thinking of this edge case!

![Settings for the custom DHCP server in AGH](https://crunchystudio.cc/wp-content/uploads/2024/07/image-14-1024x663.png)

But soon I noticed another problem - IPv6 traffic still used the ISP DNS. And that was because even though I could deactivate DHCPv4 on the ISP router, DHCPv6/SLAAC (auto-configuration mechanisms used in IPv6) is just always active no matter what I do. And no DHCPv6 options are exposed on the router WebUI, so I couldn't configure my clients to use AdGuard Home instead of what the router told them to use... I didn't enable DHCP IPv6 on AdGuard Home for some good reasons: We have a changing IPv6 prefix, and you can't really deactivate the IPv6 auto-configuration mechanisms on the ISP router which would conflict with AGH otherwise. Since most of my devices use IPv6 by default (which is good 🙃), that eventually became a reason for me to switch to OpenWrt... So, thanks ISP, I guess...? 😆

As of now, I'm happily running AdGuard Home directly on OpenWrt. I can even use AdGuard Home for IPv6 now since the ULA of the router never changes and OpenWrt allows me to configure which DNS servers to announce in both DHCPv4 and DHCPv6/SLAAC. I deactivated the default Quad9 upstream resolver and set AdGuard Home to use [Cloudflare DNS](https://one.one.one.one/). They claim to have the best resolving speeds of all, and I wanted to see if that's true. 😛

![Upstream DNS settings in AGH, with four entries of Cloudflare DNS](https://crunchystudio.cc/wp-content/uploads/2024/07/image-15-1024x430.png)

To avoid conflicts with the default `dnsmasq` DNS server in OpenWRT: I moved it to listen on port 5353 instead of 53, disabled rebind protection and set AdGuard Home to use 127.0.0.1:5353 for PTR queries, inspired by [this guide](https://openwrt.org/docs/guide-user/services/dns/adguard-home#setup). Have a look if you want to setup AGH on OpenWRT as well! Running AGH on the same device that routes your traffic is the cleanest way of running AGH at all, in my opinion.

But although we now have AdGuard Home, I still keep my beloved uBlock Origin activated for the time being. The reason for this is that AdGuard Home will only handle DNS, and anything the clients do with the queried information is none of its business anymore. I'm not sure if sites really implement my guess, but if a website (example.com) serves its advertisements from another HTTP path on the same domain (example.com/ads), AdGuard Home wouldn't be able to help. That's because DNS won't handle anything but hostname resolving, and "HTTP paths" are a concept of HTTP which operates on a whole other network level. Think of it as a meetup of the client and the web server - the only task the DNS server has to fulfill is to inform the client about the web server's "address". The only thing the DNS server can do is not to say the address of a web server (NXDOMAIN), but once the client knows where the web server "lives", the DNS server can't do anything to stop them talking with each other. 🙃

![Diagram showing how ads from the same hostname can't be blocked](https://crunchystudio.cc/wp-content/uploads/2024/07/image-16.png)

The good news is that this is just my guess, and almost all sites will use some sort of third-party ad service. Those can be easily blocked on the DNS level since third-party will basically always mean a different hostname than the accessed website itself. And that goes for _all_ your devices on the network. Crazy, huh? 😝

Alright, that was the introduction of the second service I have running on my home network. I initially thought this wasn't going to be too in-depth or technical, but I hope you could follow along... Actually, tell me in the comments! I still have a couple of things on my network that I want to show, so make sure you're subscribed. See you next post!
