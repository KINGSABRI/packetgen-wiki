# ICMPv6

IPv6 comes with a new version of ICMP: ICMPv6 \(yes!\).

ICMPv6 has the same header than ICMP, but type and code have different meanings. And ICMPv6 has not the same protocol number than ICMP.

In fact, ICMPv6 supports much more features than ICMP, but at packet level, there is no such differences.

Adding a ICMPv6 header to an IPv6 packet is easy:

```text
pg> pkt = PacketGen.gen('IPv6', src: '::1', dst: '::1')
pg> pkt.add('ICMPv6', type: 128, code: 0, body: 'ping')
pg> pkt.calc
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IPv6 -----------------------------------------
             Int32          u32: 1610612736 (0x60000000)
                        version: 6
                         tclass: 0          (0x00)
                     flow_label: 0          (0x00000)
             Int16       length: 8          (0x0008)
              Int8         next: 58         (0x3a)
              Int8          hop: 64         (0x40)
              Addr          src: ::1
              Addr          dst: ::1
---- PacketGen::Header::ICMPv6 ---------------------------------------
              Int8         type: 128        (0x80)
              Int8         code: 0          (0x00)
             Int16     checksum: 41194      (0xa0ea)
---- Body ------------------------------------------------------------
 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
----------------------------------------------------------------------
 70 69 6e 67                                      ping
----------------------------------------------------------------------
```

See also [http://rubydoc.info/gems/packetgen/PacketGen/Header/ICMPv6](http://rubydoc.info/gems/packetgen/PacketGen/Header/ICMPv6).

