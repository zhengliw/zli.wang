---
layout: post
title: "PPPoE - Why, just why?!"
author: Zhengli Wang
date: 2025-11-04
tags: 
  - "mikrotik"
  - "networking"
  - "pppoe"
  - "rant"
coverImage: "IMG_20251104_223221-scaled.jpg"
---

Hi everyone,

I know... I promised to write more for this year, but as always, teardowns and re-setups happen all the time in the homelab, so maybe it's time for me to stop trying to catch every little detail and update about more specific things a tad more frequently. Maybe I will talk about the update schedule when I feel like it some time, but today, I'm here to share something I learned and wish was different - rant post, basically.

A bit of backstory beforehand: We moved into our apartment right when the building finished renovating, but fiber connection wasn't available at that time, only copper lines via coaxial cables. If you ask me how the underlying technology works, I will be honest and tell you that I have not the slightest idea. 😛 We had always gone with Vodafone, so we didn't overthink and went with Vodafone coaxial this time as well. For the longest period of time, it worked pretty well, but starting some time in the past year, I started to experience pretty noticeable network jitter in games like Counter Strike (Bingo Bango Bongo 🙃). Of course, I reported this to Vodafone via their ticketing system, but I, as a highly skilled technician (self-attributed), knew this is a fundamental bottleneck of the physical medium, especially when hundreds of other people are sharing the same lines with you.

Since two weeks, we switched to Telekom fiber, because I was convinced that Telekom _did a thing or two better_ than other ISPs for their popularity. Vodafone does offer fiber for our building as well, but I thought trying out something new might not be bad. And when the fiber box was finally installed in our apartment, I realized one thing - that was, how wrong I was... kind of.

Setting things up was the first headache - with Vodafone, I could operate their official router box in a special modem mode, so I could hook up my Mikrotik, which can pull address configuration over DHCP/v6 and communicate right away. Telekom offers two options: Use their Speedport router for a monthly or one-time fee, or bring your own device and only buy their modem. To use my Mikrotik, I decided for the latter, which requires the connecting router to use PPPoE - still very much so in 2025 (why?!). I wasn't too familiar with this protocol, but figured out what I needed to configure with [this guide](https://gist.github.com/madduci/8b8637b922e433d617261373220be44c). Afterwards, everything seemed to be up and running, so I called it a day and hopped on for some Counter Strike. No more lag spikes this time. Fiber for the win! 😉



![](/assets/images/IMG_20251104_221845-scaled.jpg)

_Not particularly fond of the cable management here..._



But as with every piece of technology, there are always some problems that you only notice after operating it for some time. For me, I first noticed issues when accessing certain Microsoft sites. They were extremely slow and when I checked the requests the pages were making, some of the CDN domains, from which the pages load JavaScript assets, were consistently timing out, such as wcpstatic.microsoft.com. I switched DNS servers on the Mikrotik to Cloudflare's DoH service, since that was what would've (sometimes) fixed things in the past. This time, however, nothing seemed to change.

I started furiously scraping the internet for what this might indicate and eventually ended up learning a few things about MTU, or the maximum transmission unit. This was also the core of the problem. Let's start from the beginning.

The **MTU** dictates how many bytes (or octets, if you happen to write RFCs) the IP payload in a frame can be. This is configured per interface and is not negotiated. This mainly has to do with link speeds and performance, but if the MTU of the receiver is lower than the size of the frame it should receive, the frame is dropped. The most common MTU is 1500, your machine is also probably using this setting.

Let's consider this example:

![](/assets/images/图片.png)

For some internal networks between servers, higher MTUs are used to reduce overhead. That creates a MTU "boundary" for routers between high-MTU networks and low-MTU networks. Our example router has MTU 1500 on interface A and MTU 9000 on interface B.

Since the MTU on network B is bigger than the MTU of network A, packets can be routed without any problems from A to B. Now, if a frame of size 9000 arrives at interface B and the IP packet inside needs to be routed to the network on interface A, the router has two options to handle this:

1. Split the packet and send multiple frames

3. Drop the packet on the floor

![](/assets/images/图片-5.png)

The router decides what to do based on the "**Don't fragment**" (or DF) bit in the IP packet. If it is not set, the router may perform option 1, formally called **fragmentation**. Fragmentation, however, was too costly for the little benefits it brought, so it is not a common practice anymore - many routers today wouldn't even accept or forward fragmented packets.

![](/assets/images/图片-6.png)

Nowadays, most packets (practically everything from HTTP to whatever) are sent with the DF bit set. If the packet happens to cross a lower MTU boundary, the router drops the package and informs the sender about which MTU it should use for this particular route path instead. This is done via **ICMP** (Internet Control Management Protocol), which is also, but not only used for diagnostics tools like ping. **PMTUD** (Path MTU discovery) is done with this principle: The host starts sending packets with its MTU, and if any router on the way complains, it reduces the packet size to the value that particular router suggests. It repeats this process until all routers in the chain are satisfied. The largest packet size that can be sent on a specific route is called the **PMTU** (path MTU). TCP has PMTUD built into it, and protocols using UDP have to implement this themselves.



![](/assets/images/Screenshot-2025-11-04-at-17.46.07.jpg)

_No senders were harmed in this illustration_



Just a side note: Option 1 is only relevant for IPv4 (although fragmentation itself close to being deprecated anyways), as IPv6 ditches fragmentation on the router completely (Don't fragment is implied in all packets). Hosts are expected to detect the path MTU themselves and adjust their packet size accordingly.

In TCP, the easiest (only?) way to adjust or reduce packet size is to change the MSS (maximum segment size), the maximum size of a TCP payload. It is calculated by the host, using the set MTU on the receiving interface:

- IPv4: MSS = MTU - 20 bytes (IP header) - 20 bytes (TCP header) = MTU - 40 bytes

- IPv6: MSS = MTU - 40 bytes (IP header) - 20 bytes (TCP header) = MTU - 60 bytes

If you're familiar with TCP, you might know that TCP packets sometimes have options and therefore longer headers, which this calculation does not account for. [RFC 6691](https://datatracker.ietf.org/doc/html/rfc6691) says that in this case, the sending host should subtract the option length from the MSS to determine the viable payload size.

In the SYN packets during the three-way handshake, the peers use the relevant TCP option to inform their other peer about its MSS. The other side should only send payloads that are not larger than the MSS.

![](/assets/images/Screenshot-2025-11-04-at-17.32.30.jpg)

Now you might see where this is going, so back to the problem in my network. My LAN network uses MTU 1500, the sane default for all network interfaces ever. But PPPoE, which Telekom requires, adds an **8-byte overhead** to every packet sent. That means that any communication between my LAN and the internet can only use an effective MTU of 1492 (1500, which is the MTU of the underlying Ethernet link, minus 8).

Another side note 🙃: Some ISPs compensate for that by using jumbo frames ([RFC 4638](https://www.rfc-editor.org/rfc/rfc4638.html)), that is, offering an MTU of 1508 on the underlying Ethernet interface so the resulting MTU of the PPPoE connection is still 1500. With the same MTU on the client devices and the upstream (and everywhere else on the internet), the following and many other problems can be avoided. But Telekom doesn't do jumbo frames. :(

For sending stuff from LAN to WAN, this is not a problem, as the devices on the LAN perform PMTUD and my Mikrotik router will tell them to reduce their packet size to 1492. The MSS that those devices send, however, are still calculated using the MTU 1500, which is set on the interface. Essentially, they're telling their other peer (e.g. a website, or the Microsoft CDNs that were timing out) to **send bigger packets than they're actually capable of receiving**.

![](/assets/images/Screenshot-2025-11-04-at-18.16.43.jpg)

In _healthy_ network setups, this is, surprisingly, not a problem: When the Microsoft CDN eventually communicates back to me, the too big packet will get dropped by Telekom's edge router. That router will now send back an ICMP message telling the Microsoft CDN to reduce its packet size to 1492, and the Microsoft CDN _should_ resend a packet with this MTU.

But see that firewall in-between? Exactly. Some administrator (probably on Microsoft's side, wild guess 🙃) thought it was a good idea to recklessly block all ICMP traffic. This causes a so-called **PMTU blackhole**.

![](/assets/images/Screenshot-2025-11-04-at-18.18.29.jpg)

As a workaround, people have come up with a technique called **MSS clamping**. You configure a router in the chain to "hijack" the TCP handshake and change the MSS field to match the lowest known MTU. For our particular environment, this means that when a device on my LAN sends a SYN packet containing MSS 1460, the router changes the MSS 1452 before forwarding the packet to the PPPoE interface, because it knows that the PPPoE interface has a MTU of 1492.

![](/assets/images/Screenshot-2025-11-04-at-18.30.39.jpg)

Since MSS is a TCP concept, it will not work for UDP streams. But since UDP is about low latency (e.g. for online gaming, VoIP...), we can expect that the packets aren't as big anyways and therefore below the MTU. Or the services using UDP implement PMTUD and don't blindly block ICMP. 🙃

In practice, you can do MSS clamping with something like this on Mikrotik devices:

```
/ip firewall mangle add out-interface=pppoe-out protocol=tcp tcp-flags=syn action=change-mss new-mss=clamp-to-pmtu chain=forward

/ipv6 firewall mangle add out-interface=pppoe-out protocol=tcp tcp-flags=syn action=change-mss new-mss=clamp-to-pmtu chain=forward
```

It's important to do this also for IPv6. I first only did MSS clamping for IPv4, because I expected at least ICMPv6 to be allowed through on most sites. After all, ICMPv6 is essential for IPv6 as a whole to function properly. But oh well... 😛

If you've read this far and are in charge of firewalls by chance, check your configuration regarding ICMP. You may save a random guy on the internet a few afternoons of pure headache. 🙈

A few good links for further reading:

[https://lostintransit.se/2024/09/05/mss-mss-clamping-pmtud-and-mtu/](https://lostintransit.se/2024/09/05/mss-mss-clamping-pmtud-and-mtu/)

[https://blog.benjojo.co.uk/post/why-is-ethernet-mtu-1500](https://blog.benjojo.co.uk/post/why-is-ethernet-mtu-1500) (Why the standard MTU is 1500)
