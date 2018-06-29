# IPv6

## IPv6

A IPv6 header consists of a set of fields:

```text
                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------+---------------+---------------------------------------+
|VERSION| Traffic Class |             Flow Label                |
+-------+---------------+-------+---------------+---------------+
|         Payload Length        |  Next Header  |   Hop-Limit   |
+-------------------------------+---------------+---------------+
|                                                               |
|                         Source Address                        |
|                           (128 bits)                          |
|                                                               |
+---------------------------------------------------------------+
|                                                               |
|                      Destination Address                      |
|                           (128 bits)                          |
|                                                               |
+---------------------------------------------------------------+
```

* 4-bit protocol version \(`IPV6#version`\). Attended value is 6 for IPv6,
* 8-bit traffic class \(\`IPV6\#traffic\_class\),
* 20-bit flow label \(`IPv6#flow_label`\),
* 16-bit payload length \(`IPV6#length`\), size of IPV6 payload, excluding the header,
* 8-bit next header \(`IPV6#next`\) to indicate upper protocol \(6 for TCP, and 17 for UDP

  by example, but this may also be IPv6 options\),

* 8-bit hop-limit \(`IPV6#hop`\),
* 128-bit source IPV6 address \(`IPV6#src`\),
* 128-bit destination IPV6 address \(`IPV6#dst`\),
* a body \(`IPV6#body`\) containing data conveyed by IPv6.

A IPV6 header may be built this way:

```text
pg> PacketGen::Header::IPv6.new(hop: 32, src: '::1', dst: '::1')
=> ---- PacketGen::Header::IPv6 -----------------------------------------
             Int32          u32: 1610612736 (0x60000000)
                        version: 6
                         tclass: 0          (0x00)
                     flow_label: 0          (0x00000)
             Int16       length: 0          (0x0000)
              Int8         next: 0          (0x00)
              Int8          hop: 32         (0x20)
              Addr          src: ::1
              Addr          dst: ::1
```

A IPv6 packet may be created this way:

```ruby
pkt = PacketGen.gen('IPV6', ttl: 32, src: '10.0.0.1', dst: '20.0.0.1', flag_df: true)
pkt.is? 'IPVv'          # => true
pkt.ipv6                # => PacketGen::Header::IPv6
pkt.ipv6.version        # => 6
pkt.ipv6.tclass         # => Integer
pkt.ipv6.flow_label     # => Integer
pkt.ipv6.length         # => 0
pkt.ipv6.next           # => Integer
pkt.ipv6.hop            # => 32
pkt.ipv6.dst            # => '::1'

pkt.ipv6.src = '::2'
pkt.ipv6.src            # => '::2'

pkt.body = 'This is a body'
pkt.length            # => 0
```

As you can see, `length` is not automatically set to correct value. But it may be set easily with `IPv6#calc_length`. `Packet#calc` may be used too: it automatically call `#calc_sum` and `#calc_length` on all headers responding to them:

```text
pg> pkt.calc
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IPv6 -----------------------------------------
             Int32          u32: 1610612736 (0x60000000)
                        version: 6
                         tclass: 0          (0x00)
                     flow_label: 0          (0x00000)
             Int16       length: 14         (0x000e)
              Int8         next: 0          (0x00)
              Int8          hop: 32         (0x20)
              Addr          src: ::2
              Addr          dst: ::1
---- Body ------------------------------------------------------------
 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
----------------------------------------------------------------------
 54 68 69 73 20 69 73 20 61 20 62 6f 64 79        This is a body
----------------------------------------------------------------------
```

&lt;&lt;&lt;&lt;&lt;&lt;&lt; HEAD:use-cases/IPv6.md

### TCP and UDP headers to IPv6

TCP and UDP headers may be added to a IPv6 packet:

```ruby
pg> pkt = PacketGen.gen('IPv6', src: '::1', dst: '::1')
pg> pkt.add('TCP', dport: 80, sport: 51234)
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IPv6 -----------------------------------------
             Int32          u32: 1610612736 (0x60000000)
                        version: 6
                         tclass: 0          (0x00)
                     flow_label: 0          (0x00000)
             Int16       length: 0          (0x0000)
              Int8         next: 6          (0x06)
              Int8          hop: 64         (0x40)
              Addr          src: ::1
              Addr          dst: ::1
---- PacketGen::Header::TCP ------------------------------------------
             Int16        sport: 51234      (0xc822)
             Int16        dport: 80         (0x0050)
             Int32       seqnum: 6082437    (0x005ccf85)
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

## See also [http://rubydoc.info/gems/packetgen/PacketGen/Header/IPv6](http://rubydoc.info/gems/packetgen/PacketGen/Header/IPv6).

See also [http://rubydoc.info/gems/packetgen/PacketGen/Header/IPv6](http://rubydoc.info/gems/packetgen/PacketGen/Header/IPv6).

> > > > > > > 4a29f7680eb289e9cefb0a2055a9e76eef6c22c7:protocol-usecases/ipv6.md

