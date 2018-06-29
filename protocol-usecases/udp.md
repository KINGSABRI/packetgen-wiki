# UDP

A UDP header consists of:

```text
                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------------------------------+-------------------------------+
|          Source Port          |       Destination Port        |
+-------------------------------+-------------------------------+
|            Length             |           Checksum            |
+-------------------------------+-------------------------------+
```

* 16-bit source port \(`UDP#sport`\),
* 16-bit destination port \(`UDP#dport`\),
* 16-bit length \(`UDP#length`\),
* 16-bit checksum \(`UDP#checksum`\),
* and a body \(`UDP#body`\).

A UDP header may be built this way:

```text
pg> PacketGen::Header::UDP.new(dport: 53, sport: 65053)
=> ---- PacketGen::Header::UDP ------------------------------------------
             Int16        sport: 65053      (0xfe1d)
             Int16        dport: 53         (0x0035)
             Int16       length: 8          (0x0008)
             Int16     checksum: 0          (0x0000)
```

A UDP over IP packet may be created this way:

```ruby
pkt = PacketGen.gen('IP', src: '127.0.0.1', dst: '127.0.0.1').
                add('UDP', dport: 80, sport: 44158, body: 'abcd')
pkt.is?('IP')         # => true
pkt.is?('UDP')        # => true
pkt.udp               # => PacketGen::Header::UDP
pkt.udp.sport         # => 65053
pkt.udp.dport         # => 53
pkt.udp.length        # => Integer
pkt.udp.checksum      # => Integer
pkt.udp.body          # => "abcd"
pkt.body              # => "abcd"

pkt.udp.sport = 44444
```

`checksum` and `length` fields may be computed by `Header::UDP#calc_sum` and `Header::UDP#calc_length` respectively. All checksum and length fields from this packet may by computed at once using `pkt.calc`:

```text
pg> pkt.calc
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 32         (0x0020)
             Int16           id: 39945      (0x9c09)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 64         (0x40)
              Int8     protocol: 17         (0x11)
             Int16     checksum: 57537      (0xe0c1)
              Addr          src: 127.0.0.1
              Addr          dst: 127.0.0.1
            String      options: ""
---- PacketGen::Header::UDP ------------------------------------------
             Int16        sport: 44444      (0xad9c)
             Int16        dport: 80         (0x0050)
             Int16       length: 12         (0x000c)
             Int16     checksum: 36640      (0x8f20)
---- Body ------------------------------------------------------------
 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
----------------------------------------------------------------------
 61 62 63 64                                      abcd
----------------------------------------------------------------------
```

See also [http://rubydoc.info/gems/packetgen/PacketGen/Header/UDP](http://rubydoc.info/gems/packetgen/PacketGen/Header/UDP).

