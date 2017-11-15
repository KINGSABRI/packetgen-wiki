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

See also http://rubydoc.info/gems/packetgen/PacketGen/Header/IP.

### TCP header

A TCP header consists of:

```
                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------------------------------+-------------------------------+
|          Source Port          |       Destination Port        |
+-------------------------------+-------------------------------+
|                       Sequence Number                         |
+---------------------------------------------------------------+
|                      Acknowledge Number                       |
+-------+-----+-+-+-+-+-+-+-+-+-+-------------------------------+
|  Data | RSV |N|C|E|U|A|P|R|S|F|                               |
| Offset|0 0 0|S|W|C|R|C|S|S|Y|I|          Window Size          |
|       |     | |R|E|G|K|H|T|N|N|                               |
+-------+-----+-+-+-+-+-+-+-+-+-+-------------------------------+
|           Checksum            |  Urgent Pointer (if URG set)  |
+-------------------------------+-------------------------------+
|                   Options (if data offset > 5)                |
+---------------------------------------------------------------+
```

* 16-bit source port (`TCP#sport`),
* 16-bit destination port (`TCP#dport`),
* 32-bit sequence number (`TCP#seqnum`),
* 32-bit acknowledge number (`TCP#acknum`),
* a 16-bit field (`TCP#u16`) composed og:
  * 4-bit data offset (`TCP#data_offset`),
  * 3-bit reserved field (`TCP#reserved`),
  * 9 1-bit flags (`TCP#flags` or `TCP#flag_ns`, `TCP#flag_cwr`, `TCP#flag_ece`,
    `TCP#flag_urg`, `TCP#flag_ack`, `TCP#flag_psh`, `TCP#flag_rst`, `TCP#flag_syn` and
    `TCP#flag_fin`),
* 16-bit window size (`TCP#window`),
* 16-bit checksum (`TCP#checksum`),
* 16-bit urgent pointer (`TCP#urg_pointer`),
* an optional options field (`TCP#options`),
* and a body (`TCP#body`).

A TCP header may be built this way:

```
pg> PacketGen::Header::TCP.new(dport: 80, sport: 32768 + rand(2**15))
=> ---- PacketGen::Header::TCP ------------------------------------------
             Int16        sport: 43959      (0xabb7)
             Int16        dport: 80         (0x0050)
             Int32       seqnum: 1555880791 (0x5cbcdb57)
             Int32       acknum: 0          (0x00000000)
             Int16          u16: 20480      (0x5000)
                    data_offset: 5          (0x5)
                       reserved: 0
                          flags: .........
             Int16       window: 0          (0x0000)
             Int16     checksum: 0          (0x0000)
             Int16  urg_pointer: 0          (0x0000)
           Options      options: 

```

If not specified, `seqnum` is a random value.

A TCP over IP packet may be created this way:

```ruby
pkt = PacketGen.gen('IP', src: '127.0.0.1', dst: '127.0.0.1').
                add('TCP', dport: 80, sport: 44158, flag_syn: true)
pkt.is?('IP')          # => true
pkt.is?('TCP')         # => true
pkt.tcp               # => PacketGen::Header::TCP
pkt.tcp.sport         # => 44158
pkt.tcp.dport         # => 80
pkt.tcp.seqnum        # => Integer
pkt.tcp.acknum        # => Integer
pkt.tcp.u16           # => 0x5002
pkt.tcp.data_offset   # => 5
pkt.tcp.flags         # => 0x002
pkt.tcp.flag_ns?      # => false
pkt.tcp.flag_cwr?     # => false
pkt.tcp.flag_ece?     # => false
pkt.tcp.flag_urg?     # => false
pkt.tcp.flag_ack?     # => false
pkt.tcp.flag_psh?     # => false
pkt.tcp.flag_rst?     # => false
pkt.tcp.flag_syn?     # => true
pkt.tcp.flag_fin?     # => false
pkt.tcp.window        # => Integer
pkt.tcp.checksum      # => Integer
pkt.tcp.urg_pointer   # => Integer
pkt.tcp.options       # => PacketGen::Header::TCP::Options

pkt.tcp.flag_rst = true
pkt.tcp.sport = 44444
```

`checksum` field may be computed by `Header::TCP#calc_sum`. All checksum and length fields
from this packet may by computed at once using `pkt.calc`:

```
pg> pkt.calc
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 45         (0x002d)
             Int16           id: 25958      (0x6566)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 64         (0x40)
              Int8     protocol: 6          (0x06)
             Int16     checksum: 5987       (0x1763)
              Addr          src: 127.0.0.1
              Addr          dst: 127.0.0.1
            String      options: ""
---- PacketGen::Header::TCP ------------------------------------------
             Int16        sport: 44444      (0xad9c)
             Int16        dport: 80         (0x0050)
             Int32       seqnum: 3904665160 (0xe8bc7648)
             Int32       acknum: 0          (0x00000000)
             Int16          u16: 20486      (0x5006)
                    data_offset: 5          (0x5)
                       reserved: 0
                          flags: ......RS.
             Int16       window: 0          (0x0000)
             Int16     checksum: 63218      (0xf6f2)
             Int16  urg_pointer: 0          (0x0000)
           Options      options:
```

TCP options may be easily added:

```
pg> pkt.tcp.options << { opt: 'MSS', value: 1250 }
pg> pkt.tcp.options << { opt: 'NOP' }
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 45         (0x002d)
             Int16           id: 25958      (0x6566)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 64         (0x40)
              Int8     protocol: 6          (0x06)
             Int16     checksum: 5987       (0x1763)
              Addr          src: 127.0.0.1
              Addr          dst: 127.0.0.1
            String      options: ""
---- PacketGen::Header::TCP ------------------------------------------
             Int16        sport: 44444      (0xad9c)
             Int16        dport: 80         (0x0050)
             Int32       seqnum: 3904665160 (0xe8bc7648)
             Int32       acknum: 0          (0x00000000)
             Int16          u16: 20486      (0x5006)
                    data_offset: 5          (0x5)
                       reserved: 0
                          flags: ......RS.
             Int16       window: 0          (0x0000)
             Int16     checksum: 63218      (0xf6f2)
             Int16  urg_pointer: 0          (0x0000)
           Options      options: MSS:1250,NOP
```

As `TCP::Options` is a subclass of `Array`, all array methods may be used:

```ruby
pkt.tcp.options.each do |opt|
  # opt class is a subclass from Header::TCP::Option
  p opt
  puts "kind: #{opt.kind}"
  puts "length: #{opt.length}"
  puts "value: #{opt.value.inspect}"
end
```

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
