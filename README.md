# uott

Simple and stupid transparent proxy for UDP applications.

## Introduction

`uott` (UDP Over TCP Transport) is a simple and stupid proxy allowing to send UDP
datagrams over TCP streams.

There are some cases when somebody *really* needs to send UDP datagrams but it
is impossible to send them directly. This is not so uncommon, especially when
debugging UDP applications, for example:

1. You want to interact with some remote UDP application from your local
   machine, but the remote application listens loopback device only, making it
   unreachable from the external networks. If you can run TCP applications
   listening on some public interface of that remote machine, you can succeed
   with `uott`.

1. You want to interact with some remote UDP application, but the UDP datagrams
   cannot be passed through the network for some reason (port is filtered / UDP
   is filtered / UDP datagrams are dropped due to fragmentation / ...). If you
   still can create TCP connections in that network, you can succeed with
   `uott`.

1. You want to interact with some remote UDP application, but the application
   protocol is not protected, and there is a public network segment on the path.
   If you can forward TCP ports in a secure way (using SSH port forwarding or
   similar) to protect the channel on the public path, you can succeed with
   `uott`.

### Algorithm

The algorithm of `uott` is very simple. First of all, the `uott-proxy` service
is started on the remote side. After that, the `uott-client` service is launched
on the local side. The client service establishes TCP connection with the proxy
and opens a UDP socket for the clients.

#### Client loop

For each UDP message got from the UDP socket, the client checks the sources
*address:port* of the received datagram. The client accounts the *address:port*
pairs and assign a unique number (*tag*) for each pair. This *tag* is used then
to deliver the UDP response to the original address. Then the UDP message's
content is packed in UOTT message, which contains some magic number, *tag*,
datagram length and the actual UDP payload. This UOTT message is serialized to
the TCP stream and processed by the proxy service.

#### Proxy loop

The proxy service listens the TCP stream from the client and deserializes the
UDP messages using *tag* and length encoded in the stream. For each new *tag*
the proxy creates a new UDP socket. The socket is then used to send UDP
datagrams, corresponding to this *tag*, to the remote UDP service. The answers
from the remote UDP service are serialized again and sent over the TCP stream
back to the client.

#### Diagram

There will be a nice picture soon.

### Pros/Cons

Let's describe the advantages and the drawbacks of `uott`.

**Pros**

* `uott` has no dependencies except the Python itself. Python is very common
  now, so `uott` can be used in many cases with ease.
* `uott` is very simple. Like many UNIX tools, it solves one single task. The
  module is < 10 files, < 500 lines of code, easy to audit.
* `uott` does not require any root privileges (until you want it to listen
  privileged ports), so you can run it on remote machines as a regular user.

**Cons**

* `uott` does not protect its own TCP streams. If you need a protection, try to
  use `uott` in combination with SSH port forwarding. This is the cost of being
  simple and stupid, because data protection is a really complicated topic.
* `uott` keeps all the UDP sockets on the proxy side for the whole session even
  when they are not used. You can reach the OS limit of opened sockets very
  quickly. In this case you may need a more complex solution like a VPN tunnel.
  This is the cost of being simple and stupid, because a trivial proxy cannot
  investigate general UDP content to understand a UDP connection status.

### Alternatives

## Usage

### Disclaimer

### Case 1: UDP application on a remote's loopback

### Case 2: secure UDP tunnel

### Case 3: UDP port forwarding for ADB
