---
layout: "post"
title: "Tracing SIP messages using Tcpdump"
date: "2019-06-27 00:00:00"
---

Tcpdump is a command line tool to capture contents of packets passing through a network interface.

SIP is a signalling protocol used for initiating, maintaining, and terminating real-time media sessions.

Since SIP is a plain text based protocol it's really easy to trace directly using tcpdump without having to write packets to a file and then analyse them later using Wireshark. 

To do this you can use the following command:

`tcpdump -A -s 0 -n -nn -i any port 5060`

The arguments do the following:

* `-A` : Print each packet in ASCII
* `-s 0` : The number of byes to capture from each packet ... setting to 0 sets it to 262144 bytes
* `-n` : Don't convert host addresses to names
* `-nn` : Don't convert protocol and port numbers to names
* `-i any` : Listen on interface ... an argument of 'any' can be used to capture packets from all interfaces
* `port 5060` : Print all packets to and from port 5060

After running this command you'll see a real time display of SIP packets passing through the network interface:

```
14:41:44.404801 IP 192.168.47.176.5060 > 192.168.47.122.5060: SIP: INVITE sip:12345@192.168.47.122 SIP/2.0
Eh....@.@.F.../.../z......d.INVITE sip:12345@192.168.47.122 SIP/2.0
Via: SIP/2.0/UDP 192.168.47.176:5060;rport;branch=z9hG4bK1064390103
From: <sip:peter@192.168.47.176>;tag=291367939
To: <sip:12345@192.168.47.122>
Call-ID: 2105648811
CSeq: 20 INVITE
Contact: <sip:peter@192.168.47.176>
Content-Type: application/sdp
Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, NOTIFY, MESSAGE, SUBSCRIBE, INFO
Max-Forwards: 70
User-Agent: Linphone/3.6.1 (eXosip2/3.6.0)
Subject: Phone call
Content-Length:   211

v=0
o=peter 193 3850 IN IP4 192.168.47.176
s=Talk
c=IN IP4 192.168.47.176
t=0 0
m=audio 7078 RTP/AVP 0 8 101
a=rtpmap:0 PCMU/8000
a=rtpmap:8 PCMA/8000
a=rtpmap:101 telephone-event/8000
a=fmtp:101 0-11

................

14:41:44.413779 IP 192.168.47.122.5060 > 192.168.47.176.5060: SIP: SIP/2.0 100 Trying
E..49...@._.../z../...... ..SIP/2.0 100 Trying
Via: SIP/2.0/UDP 192.168.47.176:5060;rport=5060;branch=z9hG4bK1064390103
From: <sip:peter@192.168.47.176>;tag=291367939
To: <sip:12345@192.168.47.122>
Call-ID: 2105648811
CSeq: 20 INVITE
User-Agent: FreeSWITCH-mod_sofia/1.6.19~64bit
Content-Length: 0

................

14:41:44.460851 IP 192.168.47.122.5060 > 192.168.47.176.5060: SIP: SIP/2.0 200 OK
E...9...@.] ../z../........FSIP/2.0 200 OK
Via: SIP/2.0/UDP 192.168.47.176:5060;rport=5060;branch=z9hG4bK1064390103
From: <sip:peter@192.168.47.176>;tag=291367939
To: <sip:12345@192.168.47.122>;tag=1Uyy4pFFS9N7p
Call-ID: 2105648811
CSeq: 20 INVITE
Contact: <sip:12345@192.168.47.122:5060;transport=udp>
User-Agent: FreeSWITCH-mod_sofia/1.6.19~64bit
Accept: application/sdp
Allow: INVITE, ACK, BYE, CANCEL, OPTIONS, MESSAGE, INFO, UPDATE, REFER, NOTIFY
Supported: timer, path, replaces
Allow-Events: talk, hold, conference, refer
Session-Expires: 300;refresher=uas
Content-Type: application/sdp
Content-Disposition: session
Content-Length: 224
Remote-Party-ID: "12345" <sip:12345@192.168.47.122>;party=calling;privacy=off;screen=no

v=0
o=FreeSWITCH 1561585300 1561585301 IN IP4 192.168.47.122
s=FreeSWITCH
c=IN IP4 192.168.47.122
t=0 0
m=audio 61204 RTP/AVP 0 101
a=rtpmap:0 PCMU/8000
a=rtpmap:101 telephone-event/8000
a=fmtp:101 0-16
a=ptime:20

................

14:41:44.466941 IP 192.168.47.176.5060 > 192.168.47.122.5060: SIP: ACK sip:12345@192.168.47.122:5060;transport=udp SIP/2.0
Eh....@.@.G.../.../z........ACK sip:12345@192.168.47.122:5060;transport=udp SIP/2.0
Via: SIP/2.0/UDP 192.168.47.176:5060;rport;branch=z9hG4bK671645580
From: <sip:peter@192.168.47.176>;tag=291367939
To: <sip:12345@192.168.47.122>;tag=1Uyy4pFFS9N7p
Call-ID: 2105648811
CSeq: 20 ACK
Contact: <sip:peter@192.168.47.176>
Max-Forwards: 70
User-Agent: Linphone/3.6.1 (eXosip2/3.6.0)
Content-Length: 0
```

Once you're done you can stop tcpdump using `Ctrl^C`.
