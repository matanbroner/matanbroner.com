---
layout: post
title:  "Learning about WebTransport with Go" 
date:   2022-01-16 11:20:00 +0700
categories: go webtransport quic
---
I am always interested in new technologies that arise for the web, especially those relating to real-time client-server communication.
In the past, I have used WebSockets on projects such as <a href="https://github.com/matanbroner/Tryout">Tryout</a>, and have remained curious about persistent connections over the web. Furthermore, this past semester I studied Wireless Networking, which exposed me to the evolution of TCP/UDP into technologies such as MPTCP and QUIC. 

In this writeup, I aim to explore the "in the works" WebTransport protocol, a potential replacement to WebSockets which relies on QUIC's UDP streams for real-time communication.

# What is QUIC?
Taken from Wikipedia:
> QUIC is a general-purpose transport layer network protocol initially designed by Jim Roskind at Google, implemented, and deployed in 2012, announced publicly in 2013 as experimentation broadened, and described at an IETF meeting.

It aims to provide the guarantees of TCP, such as in order delivery, without the performance faults of TCP such as head-of-line blocking. I had a chance to learn about QUIC and play with it a bit this semester, and I recommend that if you haven't already heard of it, give the <a href="https://www.chromium.org/quic">Chromium Project's writeup</a> a read.

![HTTP/2 vs. QUIC](https://devopedia.org/images/article/309/4402.1611487615.jpg)
*HTTP/2 vs. QUIC, observe the multiple streams in QUIC. Credit: devopedia.org*

# What is WebTransport?
WebTransport is first and foremost not the same thing as WebSockets. Both are protocols intended for persistent bi-directional communication over the web, but WebSockets use a TCP connection, whereas WebTransport is intended for use over HTTP/3, which is the new version of HTTP which uses QUIC under the hood. I found this <a href="https://web.dev/webtransport/">writeup</a> on web.dev to be a great introduction to WebTransport. The author even answers some FAQ's, including a few about the differences between WebSockets and WebTransport and how the latter may soon replace the former.

As I mentioned, WebTransport relies on QUIC, which avoids head the head-of-line blocking often associated with TCP. If you are unfamiliar, head-of-line blocking occurs when a single lost packet in a stream of TCP packets holds up the rest of the line. Since TCP guarantees in-order delivery of packets, if a number of resources on the web are transported over a single TCP connection, a single lost packet from resource A may hold up packets being sent for resource B. In short, QUIC avoids this issue by issuing each resource a "stream", preventing the loss of packets from one resource from holding up packets from another resource. Furthermore, these streams use UDP instead of TCP, eliminating costly handshakes (hence the 0-RTT associated with QUIC). WebTransport uses these streams for either uni-direction or bi-directional communication streams.

# WebTransport and Go
GoLang (ie. Go) is a programming language that has long been on my list to learn, and during this past holiday break I finally had the chance to start working through <a href="https://quii.gitbook.io/learn-go-with-tests/">Learn Go with Tests</a>. As such, I figured that this would be a fantastic opportunity to put my learning to the test.