# hdlcpp

[![Build Status](https://travis-ci.org/bang-olufsen/hdlcpp.svg?branch=master)](https://travis-ci.org/bang-olufsen/hdlcpp) [![License](https://img.shields.io/badge/license-MIT_License-blue.svg?style=flat)](LICENSE)

Hdlcpp is a header-only C++11 framing protocol optimized for embedded communication. It is the successor of the [yahdlc](https://github.com/bang-olufsen/yahdlc) implementation written in C. Hdlcpp uses the [HDLC](https://en.wikipedia.org/wiki/High-Level_Data_Link_Control) asynchronous framing format for handling data integrity and retransmissions.

## Usage

Hdlcpp requires that a transport read and write function is supplied as e.g. a lambda for the actual data transport (like a standard socket interface). Hereby Hdlcpp can be used for any kind of transport (serial, network etc.). It is also possible to increase the buffer size for encoding/decoding data, write timeout and number of write retries.

```
hdlcpp = std::make_shared<Hdlcpp::Hdlcpp>(
    [this](unsigned char *data, unsigned short length) { return serial->read(data, length); },
    [this](const unsigned char *data, unsigned short length) { return serial->write(data, length); },
    bufferSize, writeTimeout, writeRetries);
```

To read and write data using Hdlcpp the read and write functions are used. These could again e.g. be used as lambdas functions to a protocol implementation (layered architecture).

```
protocol = std::make_shared<Protocol>(
    [this](unsigned char *data, unsigned short length) { return hdlcpp->read(data, length); },
    [this](const unsigned char *data, unsigned short length) { return hdlcpp->write(data, length); });
```

## Implementation

The supported frames are limited to DATA (I-frame with Poll bit), ACK (S-frame Receive Ready with Final bit) and NACK (S-frame Reject with Final bit). All DATA frames are acknowledged or negative acknowledged. The Address and Control fields uses the 8-bit format which means that the highest sequence number is 7. The FCS field is 16-bit.

#### Acknowledge of frame

![](https://bang-olufsen.gravizo.com/svg?%3B%0A%40startuml%3B%0Ahide%20footbox%3B%0AA%20-%3E%20B:%20DATA%20[sequence%20number%20=%201]%3B%0AB%20-%3E%20A:%20DATA%20[sequence%20number%20=%204]%3B%0AB%20-%3E%20A:%20ACK%20[sequence%20number%20=%202]%3B%0AA%20-%3E%20B:%20ACK%20[sequence%20number%20=%205]%3B%0A%40enduml)

#### Negative acknowledge of frame

![](https://bang-olufsen.gravizo.com/svg?%3B%0A%40startuml%3B%0Ahide%20footbox%3B%0AA%20-%3E%20B:%20DATA%20[sequence%20number%20=%201]%3B%0AB%20-%3E%20A:%20NACK%20[sequence%20number%20=%201]%3B%0AA%20-%3E%20B:%20DATA%20[sequence%20number%20=%201]%3B%0A%40enduml)

#### Acknowledge frame lost

![](https://bang-olufsen.gravizo.com/svg?%3B%0A%40startuml%3B%0Ahide%20footbox%3B%0AA%20-%3E%20B:%20DATA%20[sequence%20number%20=%201]%3B%0AB%20-%3Ex%20A:%20ACK%20[sequence%20number%20=%202]%3B%0A...%20Timeout%20...%3B%0AA%20-%3E%20B:%20DATA%20[sequence%20number%20=%201]%3B%0A%40enduml)
