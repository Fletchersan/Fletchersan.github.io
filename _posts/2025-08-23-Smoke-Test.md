---
layout: post
title: "Smoke Test (Secretly a socket appreciation post)"
date: 2025-08-23
categories: [networking, go, tcp, sockets, protohackers]
tags: [tcp-echo, networking, go-programming, system-programming]
excerpt: "A deep dive into building a TCP echo server from scratch, understanding sockets, and implementing the RFC 862 echo service in Go."
---

# [The problem](https://protohackers.com/problem/0)
> Create a TCP echo server that can handle multiple connections.
### Understanding the problem
* What is the input: 
	* A TCP Connection
* What is the requirement
	- Whatever data you receive send it back unmodified
	- Do not mangle binary data
	- Be able to handle at least 5 simultaneous clients
	- On reaching end of file on receiving side and having sent back all the data received, close the socket so that the client knows you have finished.
	- Basically implement the TCP Echo Service from [RFC 862](https://www.rfc-editor.org/rfc/rfc862.html).
- What is the unknown (for me personally)
	- TCP
	- Multiple client handling
	- Sockets
	- Go (the language I have chosen to do this in :') )
As it may or may not be apparent I'm woefully unqualified to attempt this problem, no worries though, we shall wing it and know what we need to know by the end of it.

# The Plan
## Approach 1: Quick and Dirty, and shouldn't be legal but it is :)
Use a library in go to do this for me, shouldn't be that hard to build and deploy.
### How to setup Go 
- it's been 2 years since i last touched this classless, java disguised travesty, so understandably I remember next to nothing. Oh well, we shall burn this bridge when we get to it, which is now I guess.
- I already had it setup apparently, it's an old version and I honestly can't be bothered to uninstall, remove the old installation dir from the path and install the latest version, so we're going to hope for the  best, I'm using `go1.23.4`, I don't even know how old this is, but oh well.
- So we move on yay!

### What do we need.
1. Some sort of tcp library, as far as i remember there's a socket type connection with a 3 way handshake, and since I have to return data exactly I'd better store it somewhere, we can probably use a buffer type of object for that.
2. Looking at wikipedia each tcp packet has a source and destination port, so I need to listen at a port, makes sense.
3. How would I be able to handle multiple connections in on one port?
	1. This is a concern, but let's be able to handle a single connection. 

With all of these information let's get started.
1. What library can we use: Go has a ["net"](https://go.dev/pkg/net/?m=old#Listener) package used for networking, which conveniently handles tcp implementation. 
Sadly for the first step my search ends here, the documentation had exactly what I needed down to the echo service and go routines to be able to handle multiple (duh, should have been obvious to me for handling multiple connections)

## Approach 2: Maybe try and see if it's possible (for me, for now) to not use direct libraries?
1. There are two main objects we need to be concerned with
	1. Listener to listen to any objects on the correct port
	2. Connection to setup and handle the data coming for an object
	3. Now each connection requires a session setup (`syn`, `synack` handshake)
		1. I **don't** want to handle the session layer, so is there a way to just handle the tcp logic?
		2. I need more information on how the tcp connection works, so I can choose on what I want to implement and what to use from a library, because [boredom and drudgery are evil](https://www.catb.org/~esr/faqs/hacker-howto.html#believe3),
### Listener
> TLDR: this is a socket (that is bound to a port) and waits by the door and listens for requests and then creates connection

Now what does this really mean?
1. The socket is bound to a port, a port is a transport layer concept, till the network layer the only information we have are mac addresses and IP addresses, which means the TCP protocol needs the transport layer (idk why I'm explaining this, but bears repeating imo)
2. The sole purpose of this listener is to listen at a port for any incoming `syn` requests, respond to them with `syn ack` and then send `ack` requests
3. After the handshake is complete a new socket is created to handle the request, which is a connection object.
   > Side note: This connection object is also a socket which is listening on the same port. So how do we know which connection is communicating with which sender if all of them communicate with the same port? This is handled based on the sender's address, we quite literally have a map of the socket to the sender address (huh, who knew hash tables had a use besides throwing them at a leetcode problem and hoping it works)

All of this is accomplished using 4 main syscalls -> `Socket`, `Bind`, `Listen` and `Accept`.

Now since all of our communication is based on sockets. We **need** to be pretty sure about what they are, so buckle up kids. We're veering off the road into Socket Land, a magical place all about sockets (Full disclosure, this may be pretty boring, hence an over the top intro, don't blame me later on).

![Sockets Meme]({{ site.baseurl }}/assets/sockets-meme.png)


# Sockets
A socket is an API/construct to allow two processes to talk to each other. Why do we need them though, don't pipes do the same thing? Yes, they do, but only for processes that are related (parent child, or sibling processes). Sockets allow unrelated processes to communicate, they don't even need to be on the same machine!

Now you can say that the term API or construct is too vague for me to create a dedicated section in this article for them. Here's the thing though, this is a networking post, and socket implementation is something highly platform dependent. We can rely on the wisdom of the kernel gods and say since everything is a file, this must be too. So for our intents and purposes we can use the read, write and close to interact with a socket, which is in turn represented by a file descriptor.

Now the syscalls in the context of what are sockets:
1. `Socket`: Defines a socket, this returns a file descriptor that represents the socket.
2. `Bind`: Associates the socket with a particular port in the namespace, without this a socket is just a construct that you can read write too, but unless it's bound to a port, how will it be known which port to send data to.
3. `Listen`: Sets the socket in a listening state, unless a socket is actively listening it won't register and process the data being sent, this in the context of our TCP implementation handles the queuing of the incoming requests for us to accept them.
4. `Accept`: This basically pulls the first connection request from the queue in the listening socket and accepts it and creates a new connection socket, this is where the actual TCP data communication will take place after the handshake.


Now what happens when we accept a socket, how do we handle multiple sockets? One for listening and creating new connections and all the sockets that are created for the connections.

We use the `Select` sys call, this basically takes a set of file descriptors and lets you know which ones are ready for some sort of I/O operation like read, write.

This allows us to iterate over these "ready" file descriptors and handle the connections.
>[!warning] Select has a fatal flaw, it can only at max handle 1024 file descriptors, this is pitifully small by the standard of any modern application. Which is why most implementations use `poll` or `epoll` instead of select.

### Operating on a connection
A connection is a socket where the handshake has been done and we can send and receive data, since this is an echo server, we simply send back the data. And once again since sockets can be handled as files using read and write was enough for my use case.


# Appendix
## [Go Implementation](https://github.com/Fletchersan/protohackers/blob/main/0-smoke-test/less-packages/echo_tcp.go)

```go
package main

import (
	"fmt"
	"log"

	"golang.org/x/sys/unix"
)

func server(port int) {
	socketFD, err := unix.Socket(unix.AF_INET, unix.SOCK_STREAM, unix.IPPROTO_IP)

	if err != nil {
		log.Fatal("Socket: ", err)
	}
	listen_on := [4]byte{0, 0, 0, 0}
	socketAddr := unix.SockaddrInet4{
		Port: port,
		Addr: listen_on,
	}

	err = unix.Bind(socketFD, &socketAddr)
	if err != nil {
		log.Fatal("Listen: ", err)
	}
	err = unix.Listen(socketFD, 10)
	if err != nil {
		log.Fatal("Listen: ", err)
	}
	log.Printf("Listening on port %d", port)
	var activeFDSet, readFDSet unix.FdSet
	FDZero(&activeFDSet)
	FDSet(socketFD, &activeFDSet)
	fdAddrMap := make(map[int]unix.Sockaddr, unix.FD_SETSIZE)
	for {
		readFDSet = activeFDSet
		_, err := unix.Select(unix.FD_SETSIZE, &readFDSet, nil, nil, nil)
		if err != nil {
			log.Fatal("Select: ", err)
		}
		for i := 0; i < unix.FD_SETSIZE; i++ {
			if isPosSet(i, &readFDSet) {
				if i == socketFD {
					newConn, newConnAddr, err := unix.Accept(socketFD)
					if err != nil {
						log.Fatal("Accept: ", err)
					}
					FDSet(newConn, &activeFDSet)
					fdAddrMap[newConn] = newConnAddr
				} else {
					msg := make([]byte, 4096)
					msgSize, err := unix.Read(i, msg)
					if err != nil {
						log.Print("Recvfrom: ", err)
						FDClr(i, &activeFDSet)
						delete(fdAddrMap, i)
						unix.Close(i)
						continue
					}
					_, err = unix.Write(i, msg[:msgSize])
					if err != nil {
						log.Print("Sendmsg: ", err)
					}
					if msgSize == 0 {
						FDClr(i, &activeFDSet)
						delete(fdAddrMap, i)
						unix.Close(i)
					}
				}
			}
		}
	}
}

func FDZero(fdSet *unix.FdSet) {
	for i := 0; i < len(fdSet.Bits); i++ {
		fdSet.Bits[i] = 0
	}
}

func FDSet(pos int, fdSet *unix.FdSet) {
	if pos < 0 || pos >= unix.FD_SETSIZE {
		return
	}
	index := pos / 64
	bit := pos % 64
	if index < len(fdSet.Bits) {
		fdSet.Bits[index] |= (1 << bit)
	}
}

func FDClr(pos int, fdSet *unix.FdSet) {
	if pos < 0 || pos >= unix.FD_SETSIZE {
		return
	}
	index := pos / 64
	bit := pos % 64
	if index < len(fdSet.Bits) {
		fdSet.Bits[index] &^= (1 << bit)
	}
}

func isPosSet(pos int, fdSet *unix.FdSet) bool {
	if pos < 0 || pos >= unix.FD_SETSIZE {
		return false
	}
	index := pos / 64
	bit := pos % 64
	if index < len(fdSet.Bits) {
		return fdSet.Bits[index]&(1<<bit) != 0
	}
	return false
}

func main() {
	fmt.Println("Hello World!")
	server(8080)
}
```


# Sources:
1. https://www.gnu.org/software/libc/manual/html_node/index.html
2. https://gist.github.com/vomnes/be42868583db5812b7266b2f45262dca