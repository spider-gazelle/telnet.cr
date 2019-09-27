# Telnet

[![Build Status](https://travis-ci.org/spider-gazelle/telnet.cr.svg?branch=master)](https://travis-ci.org/spider-gazelle/telnet.cr)


Telnet protocol support for Crystal Lang. Does not implement transport, just the raw protocol.


Usage
=====

There are three processes that need to implemented to talk to a Telnet device or service.

* Incoming data needs to be buffered, this handles protocol specific operations and unescapes incomming messages.
* A callback for sending protocol level responses to the far-end
* Preparing a message for transmission

```crystal
    require "telnet"

    telnet = Telnet.new do |cmd_response|
      # This is a response to a request from the far end
      # it should be transferred to the remote
    end

    # Buffer incoming data for unescaping and command processing
    clear_text = telnet.buffer("a string or slice / Bytes")

    # Prepare data for sending to the remote (i.e. the application level protocol)
    # The prepare function adds the appropriate line endings, as per the telnet session negotiations
    encoded_response = telnet.prepare("hello")
```

An example with the IO implemented properly

```crystal
    require "telnet"

    socket = TCPSocket.new("192.168.0.60", 23)

    telnet = Telnet.new do |cmd_response|
      # This is a response to a request from the far end
      socket.write cmd_response
    end

    # Buffer incoming data for unescaping and command processing
    encoded = socket.gets
    clear_text = telnet.buffer(encoded)

    # Prepare data for sending to the remote
    encoded_response = telnet.prepare("hello")
    socket.write encoded_response
```

If you are sending binary data to the remote you'll want to escape it

```crystal
    data = Bytes[3, 255, 4, 40, 255, 30, 20]
    encoded_response = telnet.prepare(data, escape: true)
    socket.write encoded_response

    encoded_response # => Bytes[3, 255, 255, 4, 40, 255, 255, 30, 20, 13, 10]
```
