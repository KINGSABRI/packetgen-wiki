# PacketGen Structure

PacketGen uses 3 primary concepts:

* a packet is an object describing a network packet,
* a header is an object describing a network protocol,
* a field is of a basic or composed type. A composed type is a type based on one or

  more others composed types or basic types.

## Packets

PacketGen is packet centric, so sessions or fragmentation are not handled. Thus, a packet may not contain all data necessary to interpret it.

A packet \([`PacketGen::Packet`](http://www.rubydoc.info/gems/packetgen/PacketGen/Packet) class\) is merely a container for headers. It also has a body to handle data of most inner protocol.

A packet consists of:

* an array containing headers \(`PacketGen::Packet#headers`\),
* a body \(`PacketGen::Packet#body`, which is a shortcut to last header's body\).

Packet class also provides methods to interact with packets:

* parsing packets from binary string,
* reading packets from PCAP and PCAP-NG files,
* capturing packets from a network interface,
* writing packets to PCAP-NG files,
* helpers methods to:
  * calculate all length and checksum fields among headers,
  * serialize packet to binary data,
  * encapsulate a packet in another,
  * decapsulate some headers from a packet to a new packet.

## Headers

Most of headers are based on PacketGen types. They contain fields. Each field is defined from a type.

Some headers may contain others headers. Such headers should have a `#body` field to handle inner headers.

Some protocols use length fields and/or checksum fields. To permit computation of these fields at once through `PacketGen::Packet#calc`, these fields should be named `#length` and `#checksum`, respectively.

Most of PacketGen header classes inherit from [`PacketGen::Header::Base`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/Base) class. This class implements [minimal API](https://github.com/sdaubert/packetgen/wiki/Create-Custom-Protocol#header-minimal-api) needed to parse packets and add headers to packets.

## Types

### Basic types

Basic types are types used to construct headers or composed types. Basic types are listed in table below.

| Type | Description |
| :---: | :--- |
| `Types::Int8` | 8-bit integer |
| `Types::Int8Enum` | 8-bit enumerated integer |
| `Types::Int16`, `Types::Int16be` | 16-bit big-endian integer |
| `Types::Int16Enum`, `Types::Int16beEnum` | 16-bit big-endian enumerated integer |
| `Types::Int16le` | 16-bit little-endian integer |
| `Types::Int16leEnum` | 16-bit little-endian enumerated integer |
| `Types::Int32`, `Types::Int32be` | 32-bit big-endian integer |
| `Types::Int32Enum`, `Types::Int32beEnum` | 32-bit big-endian enumerated integer |
| `Types::Int32le` | 32-bit little-endian integer |
| `Types::Int32leEnum` | 32-bit little-endian enumerated integer |
| `Types::Int64`, `Types::Int64be` | 64-bit big-endian integer |
| `Types::Int64le` | 64-bit little-endian integer |
| `Types::String` | binary string |
| `Types::CString` | null-terminated string |
| `Types::IntString` | binary string prepended with its field |
| `Types::Array` | container for types. May contain multiple values of a single type |

### Composed Types

Composed types are some PacketGen default types built from basic ones. These types are commonly used to define headers:

| Type | Description |
| :---: | :--- |
| `Types::Fields` | a container to concatenate multiple fields of different types together |
| `Types::TLV` | Type-Length-Value type |
| `Types::OUI` | Organizationally Unique Identifier |

Some headers also define commonly used types:

| Type | Description |
| :---: | :--- |
| `Header::Eth::MacAddr` | Ethernet MAC address |
| `Header::IP::Addr` | IPv4 address |
| `Header::IPv6::Addr` | IPv6 address |

