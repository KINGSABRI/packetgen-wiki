# ICMP

A ICMP header consists of:

```text
                     1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+---------------+-------------------------------+
|     Type      |      Code     |           Checksum            |
+---------------+---------------+-------------------------------+
```

* 8-bit type \(`ICMP#type`\),
* 8-bit code \(`ICMP#code`\),
* 16-bit checksum \(`ICMP#checksum`\),
* and a body \(`ICMP#body`\).

A ICMP header may be built this way:

```text
pg> PacketGen::Header::ICMP.new(type: 8, code: 0, body: 'PING!')
=> ---- PacketGen::Header::ICMP -----------------------------------------
              Int8         type: 8          (0x08)
              Int8         code: 0          (0x00)
             Int16     checksum: 0          (0x0000)
```

A ICMP packet may be created this way:

```ruby
pkt = PacketGen.gen('IP', src: '127.0.0.1', dst: '127.0.0.1').
                add('ICMP', type: 8, code: 0, body: 'PING!')
pkt.is?('IP')         # => true
pkt.is?('ICMP')       # => true
pkt.icmp              # => PacketGen::Header::ICMP
pkt.icmp.type         # => Integer
pkt.icmp.code         # => Integer
pkt.icmp.body         # => "PING!"
pkt.body              # => "PING!"

pkt.udp.type = 0 # This is now a pong!
```

As usual, `checksum` field may be computed by `Header::ICMP#calc_sum`. All checksum and length fields from a packet may by computed at once using `pkt.calc`:

```text
pg> pkt.calc
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 29         (0x001d)
             Int16           id: 26634      (0x680a)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 64         (0x40)
              Int8     protocol: 1          (0x01)
             Int16     checksum: 5332       (0x14d4)
              Addr          src: 127.0.0.1
              Addr          dst: 127.0.0.1
            String      options: ""
---- PacketGen::Header::ICMP -----------------------------------------
              Int8         type: 0          (0x08)
              Int8         code: 0          (0x00)
             Int16     checksum: 16495      (0x406f)
---- Body ------------------------------------------------------------
 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
----------------------------------------------------------------------
 50 49 4e 47 21                                   PING!
----------------------------------------------------------------------
```

See also [http://rubydoc.info/gems/packetgen/PacketGen/Header/ICMP](http://rubydoc.info/gems/packetgen/PacketGen/Header/ICMP).

