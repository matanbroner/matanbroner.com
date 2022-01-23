---
layout: post
title:  "[WIP] Learning about WebTransport with Go" 
date:   2022-01-23 11:20:00 +0700
categories: go webtransport quic
---
> Note: This post is a work in progress. It is subject to change.

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
**Warning: The WebTransport protocol is still in development, and is not yet ready for use in production.**

GoLang (ie. Go) is a programming language that has long been on my list to learn, and during this past holiday break I finally had the chance to start working through <a href="https://quii.gitbook.io/learn-go-with-tests/">Learn Go with Tests</a>. As such, I figured that this would be a fantastic opportunity to put my learning to the test.
Luckily, others have already taken on implementing WebTransport in various languages. The <a href="https://github.com/aiortc/aioquic">aioquic</a> library for Python implements both HTTP/3 and WebTransport streams. Also, the folks behind <a href="https://centrifugal.github.io/centrifugo/">Centrifugal</a> have a great <a href="https://centrifugal.github.io/centrifugo/blog/quic_web_transport/">writeup</a> on their own experiments with WebTransport streams in Go. I used both of these sources in my own learning.

## Setting up our project
We begin with initializing a Go project in a directory `webtransport-go`.

```bash
$ go mod init github.com/matanbroner/webtransport-go
```

Closely following the example of the Centrifugal writeup, I am going to use the <a href="https://github.com/lucas-clemente/quic-go">`quic-go`</a> library to set up a QUIC server. We will define some basic configurations we will need for the server, a shell for a "start" function, and our main function.

```go
package main

import (
	"github.com/lucas-clemente/quic-go"
)

// Configuration for the server
type Config struct {
	Host                 string
	Port                 string
	CertificatePath      string
	KeyPath              string
	AllowedAccessOrigins []string
}

type QuicServer struct {
	config Config
}

func (server *QuicServer) Start() {
	// TODO: Implement
}

func main() {
	config := Config{
		Host:                 "localhost",
		Port:                 "4433",
		CertificatePath:      "quic_cert.pem",
		KeyPath:              "quic_key.pem",
		AllowedAccessOrigins: []string{"localhost"},
	}
	server := QuicServer{config: config}
	server.Start()
}
```

Everything should look fairly straightforward. We define a host, port, and hosts which can access our server. What may seem off is the certificate and key configurations. We have yet to create a TLS certificate-key pair, let's do that now. We will use the `openssl` library, so be sure to install it first.

```bash
$ openssl genrsa -des3 -passout pass:x -out quic_server.pass.key 2048
$ openssl rsa -passin pass:x -in quic_server.pass.key -out quic_key.pem
$ rm quic_server.pass.key
$ openssl req -new -key quic_server.key -out quic_server.csr # Use 'localhost' as the common name
$ openssl x509 -req -sha256 -days 365 -in quic_server.csr -signkey quic_server.key -out quic_cert.pem
```

And with that, we can begin to make our server actually do something interesting, as well as using our `quic-go` library.
Let's jump back into our `Start()` function.

```go
func (server *QuicServer) TLSConfig() *tls.Config {
	// TODO: Implement
}

func (server *QuicServer) handleSession(session quic.Session) {
	// TODO: Implement
}

func (server *QuicServer) Start() error {
	var addr string = server.config.Host + ":" + server.config.Port
	// A Listener for incoming QUIC connections
	listener, err := quic.ListenAddr(addr, server.TLSConfig(), nil)
	if err != nil {
		return err
	}
	for {
		session, err := listener.Accept(context.Background())
		if err != nil {
			return err
		}
		go func() {
			server.handleSession(session)
		}()
	}
}
```

Our listener will accept connections within a top level context (ie. `context.Background()`). From there, assuming no errors we will handle the session.
At this point we need to set up our function `TLSConfig()`, returning a configuration object using the TLS certificate-key pair we generated earlier.

```go
func (server *QuicServer) TLSConfig() *tls.Config {
	cert, err := tls.LoadX509KeyPair(server.config.CertificatePath, server.config.KeyPath)
	if err != nil {
		log.Fatal(err)
	}
	return &tls.Config{
		Certificates: []tls.Certificate{cert},
		// We identify ourselves as a HTTP/3 based client
        // Note that according to the official WebTransport over QUIC specification (QuicTransport), this should be "wq-vvv-01"
        // Our test client will be a Google Chrome WebTransport over HTTP/3 client, using QUIC under the hood
		NextProtos: []string{"h3"},
	}
}
```

# Conclusion

Feel free to play around with my code yourself and make improvements. The repository is linked below.
<div class="github-card" data-github="matanbroner/webtransfer-go" data-width="400" data-height="155" data-theme="default"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>