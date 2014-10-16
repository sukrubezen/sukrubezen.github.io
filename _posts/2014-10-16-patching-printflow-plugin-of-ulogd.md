---
layout: post
title: Patching Printflow Plugin of Ulogd
---

I am working at [Labris Networks] and I was working on a project in which I needed to be able to track every connection opened, ongoing or ended to a computer with the information of ip, port number, number of packets, size of data transferred and start-end times of the connection.

In the literature, this was called "Connection Tracking". 

>[Quoting Wikipedia]: “Connection tracking allows the kernel to keep track of all logical network connections or sessions, and thereby relate >all of the packets which may make up that connection.

I used [conntrack] as a connection tracking tool. When I executed ```conntrack -E``` I could see ongoing connections. Output was like:

```
[DESTROY] udp      17 src=192.168.0.22 dst=192.168.0.255 sport=138 dport=138 packets=1 bytes=241 
[UNREPLIED] src=192.168.0.255 dst=192.168.0.22 sport=138 dport=138 packets=0 bytes=0
```

The conntrack was providing me nearly all of the information I wanted to fetch except the time value but I will be talking about this after a while. 

If you look into the data we got, it had two main parts, source and destination, meaning we had the two sides of a connection successfully. On the other hand, we executed this from command line but we need a tool that provides that information as a stream so that we can store it in a database or file. For that purpose I used [ulogd] tool. Ulogd is a userspace logging daemon and now I will talk about how I managed to get connection tracking data from ulogd.

Ulogd has a configuration file in the ```/etc/ulogd.conf``` path. In this configuration file, there are various plugins that ulogd uses. Those plugins can be divided into three main categories.

- Input plugins
- Filter plugins
- Output plugins


Actually names do tell themselves but if you want to have some further information about a plugin, you can simply execute:

```
ulogd -v -i /usr/local/lib/ulogd/ulogd_inpflow_NFCT.so
```

The way ulogd's plugins work are like flows, think about water flowing through pipes. Firstly, logs are entering the flow process through input plugins and they can direct their output into either type of the plugins but exit of the flowing process must be output typed  plugins.

I used NFCT as an input plugin and PRINTFLOW as the output plugin. What PRINTFLOW gave as a output was:

```
Oct 16 14:15:54 lothlorien [DESTROY] ORIG: SRC=192.168.0.170 DST=255.255.255.255 PROTO=UDP SPT=17500 DPT=17500 PKTS=9 BYTES=1188 , 
REPLY: SRC=255.255.255.255 DST=192.168.0.170 PROTO=UDP SPT=17500 DPT=17500 PKTS=0 BYTES=0 
```

You may think that now I got the time information but you are WRONG. I need microsecond accurate time information and now I have only seconds besides I do not have year information.

So I wondered, what if PRINTFLOW did indeed take a microsecond accurate time variable as an input but filtered it along the way?

I just executed:

```
>> ulogd -i /usr/lib/ulogd/ulogd_filter_PRINTFLOW.so

>>Input keys:
        Key: orig.ip.saddr.str (IP addr)
        Key: orig.ip.daddr.str (IP addr)
        Key: orig.ip.protocol (unsigned int 8)
        Key: orig.l4.sport (unsigned int 16)
        Key: orig.l4.dport (unsigned int 16)
        Key: orig.raw.pktlen (unsigned int 32)
        Key: orig.raw.pktcount (unsigned int 32)
        Key: reply.ip.saddr.str (IP addr)
        Key: reply.ip.daddr.str (IP addr)
        Key: reply.ip.protocol (unsigned int 8)
        Key: reply.l4.sport (unsigned int 16)
        Key: reply.l4.dport (unsigned int 16)
        Key: reply.raw.pktlen (unsigned int 32)
        Key: reply.raw.pktcount (unsigned int 32)
        Key: icmp.code (unsigned int 8)
        Key: icmp.type (unsigned int 8)
        Key: ct.event (unsigned int 32)
Output keys:
        Key: print (string)
```


I saw that there was no trace of a missing time variable. So I wondered, what if NFCT plugin had this missing time variable in its output but PRINTFLOW was not taking it as an input?

I just executed:

```
>> ulogd -i /usr/lib/ulogd/ulogd_inpflow_NFCT.so

>>Input keys:
        Input plugin, No keys
Output keys:
        Key: orig.ip.saddr (IP addr)
        Key: orig.ip.daddr (IP addr)
        Key: orig.ip.protocol (unsigned int 8)
        Key: orig.l4.sport (unsigned int 16)
        Key: orig.l4.dport (unsigned int 16)
        Key: orig.raw.pktlen (unsigned int 32)
        Key: orig.raw.pktcount (unsigned int 32)
        Key: reply.ip.saddr (IP addr)
        Key: reply.ip.daddr (IP addr)
        Key: reply.ip.protocol (unsigned int 8)
        Key: reply.l4.sport (unsigned int 16)
        Key: reply.l4.dport (unsigned int 16)
        Key: reply.raw.pktlen (unsigned int 32)
        Key: reply.raw.pktcount (unsigned int 32)
        Key: icmp.code (unsigned int 8)
        Key: icmp.type (unsigned int 8)
        Key: ct.mark (unsigned int 32)
        Key: ct.id (unsigned int 32)
        Key: ct.event (unsigned int 32)
        Key: flow.start.sec (unsigned int 32)
        Key: flow.start.usec (unsigned int 32)
        Key: flow.end.sec (unsigned int 32)
        Key: flow.end.usec (unsigned int 32)
        Key: oob.family (unsigned int 8)
        Key: oob.protocol (unsigned int 8)
        Key: ct (raw data)
```

And VOILA !

flow.end.sec, flow.end.usec are what I was looking for. Now I have to dive into ulogd plugin PRINTFLOW's source code and add those fields to be able to use them.

Under the ulogd source tree, PRINTFLOW's path is ```util/printflow.c```

You can find the latest version of the file [here].
I just added PRINTFLOW_END_SEC and PRINTFLOW_END_USEC variables.

In addition, do not forget to change 15 to 17 in printflow.h

[Labris Networks]:http://labrisnetworks.com/
[Quoting Wikipedia]:http://en.wikipedia.org/wiki/Netfilter#Connection_tracking
[conntrack]:http://conntrack-tools.netfilter.org/
[ulogd]:http://www.netfilter.org/projects/ulogd/
[here]:https://gist.github.com/skr/dc69ae42b86ae5df9b8f


