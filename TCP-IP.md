This page describes the use of PacketGen with TCP/IP stack.

## Mastering IP stack protocols

The IP stack (or TCP/IP stack) is a network stack based on IP protocol, which is a
network level (level 3 in [OSI model](https://en.wikipedia.org/wiki/OSI_model)).

IP is itself often used with a higher protocol (transport level, or level 4):

* Transmission Control Protocol (TCP), which provides reliable, ordered, and error-checked
  delivery of a stream of octets between applications running on hosts communicating by an
  IP network,
* User Datagram Protocol (UDP), which is simpler than TCP: this is a connection-less
  protocol with a minimum of protocol mechanisms. It provides checksums for data integrity
  but is not reliable, there is no guarantee of ordering delivery.

### IP header

A IP header consists of a set of fields:

```
                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------+-------+---------------+-------------------------------+
|VERSION|  IHL  |      TOS      |          Total Length         |
+-------+-------+---------------+-----+-------------------------+
|              ID               |flags|    Fragment Offset      |
+---------------+---------------+-----+-------------------------+
|      TTL      |   Protocol    |          Checksum             |
+---------------+---------------+-------------------------------+
|                         Source Address                        |
+---------------------------------------------------------------+
|                      Destination Address                      |
+---------------------------------------------------------------+
|                      Options (if IHL > 5)                     |
+---------------------------------------------------------------+
```

* 4-bit protocol version (`IP#version`). Attended value is 4 for IPv4,
* 4-bit IP header length (`IP#ihl). Size of IP header in 32-bit words. As a IP header
  is 20 bytes long, this value should be 5. It may be greater if header has options,
  but options are not supported by PacketGen,
* 8-bit type of service (`IP#tos`),
* 16-bit total length (`IP#length`), size of IP packet, including the header,
* 16-bit ID (`IP#id`),
* 16-bit fragment word (`IP#frag`), used for IP fragmentation, composed of:
  * 3 1-bit flags:
    * a reserved bit (`IP#flag_rsv`),
    * don't fragment flag (`IP#flag_df`) to forbid fragmentation of this packet,
    * more fragment flag (`IP#flag_mf`) to indicate a fragment, which is not the last
      one, of a IP packt,
  * a 13-bit fragment offset (`IP#fragment_offset`),
* 8-bit time to live (`IP#ttl`),
* 8-bit protocol (`IP#protocol`) to indicate upper protocol (6 for TCP, and 17 for UDP
  by example),
* 16-bit checksum (`IP#checksum`) of all IP packet,
* 32-bit source IP address (`IP#src`),
* 32-bit destination IP address (`IP#dst`),
* a body (`IP#body`) containing data conveyed by IP.

A IP header may be built this way:

```
pg> PacketGen::Header::IP.new(ttl: 32, src: '10.0.0.1', dst: '20.0.0.1', flag_df: true)
=> ---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 20         (0x0014)
             Int16           id: 33533      (0x82fd)
             Int16         frag: 16384      (0x4000)
                          flags: DF
                    frag_offset: 0          (0x0000)
              Int8          ttl: 32         (0x20)
              Int8     protocol: 0          (0x00)
             Int16     checksum: 0          (0x0000)
              Addr          src: 10.0.0.1
              Addr          dst: 20.0.0.1
```

If not specified, `id` is a random number.

A IP packet may be created this way:

```ruby
pkt = PacketGen.gen('IP', ttl: 32, src: '10.0.0.1', dst: '20.0.0.1', flag_df: true)
pkt.is? 'IP'          # => true
pkt.ip                # => PacketGen::Header::IP
pkt.ip.version        # => 4
pkt.ip.length         # => 20
pkt.ip.id             # => a 16-bit random value
pkt.ip.frag           # => 0x4000
pkt.ip.df?            # => true
pkt.ip.mf?            # => false
pkt.ip.frag_offset    # => 0
pkt.ip.ttl            # => 32
pkt.ip.protocol       # => 0
pkt.ip.checksum       # => 0
pkt.ip.dst            # => '20.0.0.1'

pkt.ip.src = '20.0.0.2'
pkt.ip.src            # => '20.0.0.2'

pkt.body = 'This is a body'
pkt.length            # => 20
```

As you can see, `checksum` and `length` are not automatically set to correct values. But
they may be set easily with `IP#calc_sum`, which computes checksum, and `IP#calc_length`
which set correct length (taking care of body length). `Packet#calc` may be used too:
it automatically call `#calc_sum` and `#calc_length` on all headers responding to them:

```
pg> pkt.calc
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 34         (0x0022)
             Int16           id: 4640       (0x1220)
             Int16         frag: 16384      (0x4000)
                          flags: DF
                    frag_offset: 0          (0x0000)
              Int8          ttl: 32         (0x20)
              Int8     protocol: 0          (0x00)
             Int16     checksum: 8378       (0x20ba)
              Addr          src: 20.0.0.2
              Addr          dst: 20.0.0.1
---- Body ------------------------------------------------------------
 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
----------------------------------------------------------------------
 54 68 69 73 20 69 73 20 61 20 62 6f 64 79        This is a body
----------------------------------------------------------------------
```

### TCP header

work in progress

### UDP header

work in progress

### ICMP header

work in progress

## Mastering IPv6 stack protocols

### IPv6 header

work in progress

### TCP and UDP headers

work in progress

### ICMPv6 header

## Sending packets

IP packets may be sent at link level (Ethernet, IEEE 802.11) or at network one (IP).

So, if you want to generate IP packets and let your OS route them, create packets at
IP level:

```ruby
pkt = PacketGen.gen('IP', src: '192.168.1.1', '192.168.1.101').add('TCP', dport: 80)
pkt.body = some_string_containing_http_content
pkt.to_w
```

But you may still use link level:

```ruby
# Ethernet
pkt = PacketGen.gen('Eth', src: '00:01:02:03:04:05', dst: '06:07:08:09:0a:0b')
pkt.add('IP', src: '192.168.1.1', '192.168.1.101').add('TCP', dport: 80)
pkt.body = some_string_containing_http_content
pkt.calc
pkt.to_w

# IEEE 802.11
pkt = PacketGen.gen('RadioTap').add('Dot11::Data, src: '00:01:02:03:04:05', dst: '06:07:08:09:0a:0b')
pkt.add('LLC').add('SNAP')
pkt.add('IP', src: '192.168.1.1', '192.168.1.101').add('TCP', dport: 80)
pkt.body = some_string_containing_http_content
pkt.to_w('wlan0', calc: true)
```

## Capturing packets

Captured packets will always have a link level header:

* `Eth` one for packets captured from an ethernet interface,
* `RadioTap` or `Dot11` one for packets captured from a IEEE 802.11 interface.
  If RadioTap header is present, il will always be followed by a Dot11 one.
