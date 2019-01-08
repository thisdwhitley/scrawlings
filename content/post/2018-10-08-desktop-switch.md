+++
title = "Desktop switch"
date = "2018-10-08"
tags = ["technology","network"]
categories = ["tech"]
banner = "img/banners/banner-2.jpg"
showDisclosure: true
+++

## Background

When I started my current role, I had never worked from home before.  So I'm
pretty clueless as to the best way to do this.  I've got a *LOT* of work to do
in order to have a respectable home office.  But one thing I realized pretty
quickly was that I would like to dedicate my work laptop as a ***work***
computer, but still use my home computer for my personal tasks.  But I do not
want to have multiple monitors, keyboards, mice, etc.

So I am currently using my laptop (which is a BEAST) as a VNC server and connect
to it from my home computer.

## Issue

I have noticed that sometimes (and oddly *only* somtimes) my work laptop "feels"
laggy.  The thing is very powerful, so I do not suspect it is truely bogged down
but rather that VNC is not rendering it quickly enough.  This hypothosis is
strengthened by the occasional pixelated rendering.  So I have come to suspect
network speed.  My laptop is connected wirelessly and my home computer is wired
in.  Here is some `iperf` output for your viewing pleasure:

~~~
$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.20 port 47838
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-13.6 sec  21.5 MBytes  13.3 Mbits/sec
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.20 port 47848
[  4]  0.0-13.2 sec  4.50 MBytes  2.85 Mbits/sec
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.20 port 47850
[  4]  0.0-11.6 sec  10.2 MBytes  7.41 Mbits/sec
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.20 port 47852
[  4]  0.0-10.2 sec  22.1 MBytes  18.2 Mbits/sec
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.20 port 47854
[  4]  0.0-11.2 sec  9.88 MBytes  7.38 Mbits/sec
~~~

You can see from that output that those speeds are all over the place and
totally inconsistent.

## Fix

So I have repurposed an old 802.11g wireless router which isn't used anymore,
but that already has a gigabit switch.  I slapped [DD-WRT](https://dd-wrt.com)
on it (for no real reason...I'm pretty sure I won't need the additional
functionality) and disabled wireless on it.  Now check out the speeds:

~~~
$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.239 port 53416
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-10.0 sec   841 MBytes   703 Mbits/sec
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.239 port 53418
[  4]  0.0-10.0 sec   860 MBytes   720 Mbits/sec
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.239 port 53420
[  4]  0.0-10.0 sec   872 MBytes   730 Mbits/sec
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.239 port 53424
[  4]  0.0-10.0 sec   854 MBytes   715 Mbits/sec
[  4] local 10.10.10.9 port 5001 connected with 10.10.10.239 port 53430
[  4]  0.0-10.0 sec   869 MBytes   728 Mbits/sec
~~~

***BOOM*** that's a huge increase in speed.  Now if my laptop-through-VNC feels
sluggish, I will have to look somewhere else because the network speeds between
the VNC server and client are incredibly fast.  Time will tell...

## Side note

I realize I could have just purchased a [desktop switch](https://amzn.to/2pJAzWQ)
pretty inexpensively, but I already had this one sitting around and now I'll get
a little more life out of it.

## Next steps

I recently set up [pi-hole](https://pi-hole.net) on my home network and it is
serving as my DNS and DHCP server so I have to adjust my DHCP reservation for my
work laptop (you may have noticed the different IP addresses in the output).
