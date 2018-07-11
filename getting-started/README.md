# Getting Started

## Install PacketGen

Use rubygem:

```text
$ gem install packetgen
```

or use bundler. Put this line in you `Gemfile`:

```ruby
gem packetgen
```

## Starting PacketGen

The easiest way to start PacketGen is using interactive console. To send packets, root privileges are needed. In a terminal, do:

```text
$ sudo pgconsole

pgconsole uses IRB

pg>
```

## Interactive use

### First step

Build a packet and play with it:

```text
pg> pkt = gen('IP', ttl: 1)
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 20         (0x0014)
             Int16           id: 10172      (0x27bc)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 1          (0x01)
              Int8     protocol: 0          (0x00)
             Int16     checksum: 0          (0x0000)
              Addr          src: 127.0.0.1
              Addr          dst: 127.0.0.1

pg> pkt.ip.src
=> "127.0.0.1"
pg> pkt.ip.ttl
=> 1
pg> pkt.ip.dst = '192.168.0.1'
=> "192.168.0.1"
pg> pkt.ip.src = '192.168.0.200'
=> "192.168.0.200"
pg> pkt.ip
=> ---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 20         (0x0014)
             Int16           id: 10172      (0x27bc)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 1          (0x01)
              Int8     protocol: 0          (0x00)
             Int16     checksum: 0          (0x0000)
              Addr          src: 192.168.0.200
              Addr          dst: 192.168.0.1

pg>
```

Here, `gen` \(a shortcut to `PacketGen.gen`\) generate a `PacketGen::Packet` object with a IP header.

Then, IP header is accessed and/or modified through `#ip` method, which returns a `PacketGen::Header::IP` object \(mapping of a IP header\).

### Put layers together

To add layers to a packet, `PacketGen::Packet#add` method should be used. Adding a header on a packet may update fields from underlying packet. Here, adding a TCP header to our IP packet will update IP protocol field to 6 \(TCP protocol number\):

```text
pg> pkt.add('TCP')
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 20         (0x0014)
             Int16           id: 10172      (0x27bc)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 1          (0x01)
              Int8     protocol: 6          (0x06)
             Int16     checksum: 0          (0x0000)
              Addr          src: 192.168.0.200
              Addr          dst: 192.168.0.1
---- PacketGen::Header::TCP ------------------------------------------
             Int16        sport: 0          (0x0000)
             Int16        dport: 0          (0x0000)
             Int32       seqnum: 4174616899 (0xf8d39943)
             Int32       acknum: 0          (0x00000000)
             Int16          u16: 20480      (0x5000)
                    data_offset: 5          (0x5)
                       reserved: 0
                          flags: .........
             Int16       window: 0          (0x0000)
             Int16     checksum: 0          (0x0000)
             Int16  urg_pointer: 0          (0x0000)
           Options      options: 

pg>
```

`#add` may be chained:

```text
pg> pkt = gen('Eth').add('IP', src: '1.1.1.1', dst: '2.2.2.2').add('TCP', dport: 80)
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::Eth ------------------------------------------
           MacAddr          dst: 00:00:00:00:00:00
           MacAddr          src: 00:00:00:00:00:00
             Int16    ethertype: 2048       (0x0800)
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 20         (0x0014)
             Int16           id: 12588      (0x312c)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 64         (0x40)
              Int8     protocol: 6          (0x06)
             Int16     checksum: 0          (0x0000)
              Addr          src: 1.1.1.1
              Addr          dst: 2.2.2.2
---- PacketGen::Header::TCP ------------------------------------------
             Int16        sport: 0          (0x0000)
             Int16        dport: 80         (0x0050)
             Int32       seqnum: 4168248929 (0xf8726e61)
             Int32       acknum: 0          (0x00000000)
             Int16          u16: 20480      (0x5000)
                    data_offset: 5          (0x5)
                       reserved: 0
                          flags: .........
             Int16       window: 0          (0x0000)
             Int16     checksum: 0          (0x0000)
             Int16  urg_pointer: 0          (0x0000)
           Options      options: 

pg>
```

You may also add whatever you want as packet body:

```text
pg> pkt.body = "GET / HTTP1.0\r\n\r\n"
=> "GET / HTTP1.0\r\n\r\n"
pg> pkt
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::Eth ------------------------------------------
           MacAddr          dst: 00:00:00:00:00:00
           MacAddr          src: 00:00:00:00:00:00
             Int16    ethertype: 2048       (0x0800)
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 20         (0x0014)
             Int16           id: 31729      (0x7bf1)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 64         (0x40)
              Int8     protocol: 6          (0x06)
             Int16     checksum: 0          (0x0000)
              Addr          src: 1.1.1.1
              Addr          dst: 2.2.2.2
---- PacketGen::Header::TCP ------------------------------------------
             Int16        sport: 0          (0x0000)
             Int16        dport: 80         (0x0050)
             Int32       seqnum: 394494115  (0x178380a3)
             Int32       acknum: 0          (0x00000000)
             Int16          u16: 20480      (0x5000)
                    data_offset: 5          (0x5)
                       reserved: 0
                          flags: .........
             Int16       window: 0          (0x0000)
             Int16     checksum: 0          (0x0000)
             Int16  urg_pointer: 0          (0x0000)
           Options      options: 
---- Body ------------------------------------------------------------
 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
----------------------------------------------------------------------
 47 45 54 20 2f 20 48 54 54 50 31 2e 30 0d 0a 0d  GET / HTTP1.0...
 0a                                               .
----------------------------------------------------------------------
pg>
```

### Generate binary data and read packets

From a packet, you may generate binary data which will be sent on network. You may also parse binary data to create packets:

```text
pg> str = pkt.to_s
=> "\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\b\x00E\x00\x00\x14{\xF1\x00\x00@\x06\x00\x00\x01\x01\x01\x01\x02\x02\x02\x02\x00\x00\x00P\x17\x83\x80\xA3\x00\x00\x00\x00P\x00\x00\x00\x00\x00\x00\x00GET / HTTP1.0\r\n\r\n"
pg> parse(str)
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::Eth ------------------------------------------
           MacAddr          dst: 00:00:00:00:00:00
           MacAddr          src: 00:00:00:00:00:00
             Int16    ethertype: 2048       (0x0800)
---- PacketGen::Header::IP -------------------------------------------
              Int8           u8: 69         (0x45)
                        version: 4
                            ihl: 5
              Int8          tos: 0          (0x00)
             Int16       length: 20         (0x0014)
             Int16           id: 31729      (0x7bf1)
             Int16         frag: 0          (0x0000)
                          flags: none
                    frag_offset: 0          (0x0000)
              Int8          ttl: 64         (0x40)
              Int8     protocol: 6          (0x06)
             Int16     checksum: 0          (0x0000)
              Addr          src: 1.1.1.1
              Addr          dst: 2.2.2.2
---- PacketGen::Header::TCP ------------------------------------------
             Int16        sport: 0          (0x0000)
             Int16        dport: 80         (0x0050)
             Int32       seqnum: 394494115  (0x178380a3)
             Int32       acknum: 0          (0x00000000)
             Int16          u16: 20480      (0x5000)
                    data_offset: 5          (0x5)
                       reserved: 0
                          flags: .........
             Int16       window: 0          (0x0000)
             Int16     checksum: 0          (0x0000)
             Int16  urg_pointer: 0          (0x0000)
           Options      options: 
---- Body ------------------------------------------------------------
 00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
----------------------------------------------------------------------
 47 45 54 20 2f 20 48 54 54 50 31 2e 30 0d 0a 0d  GET / HTTP1.0...
 0a                                               .
----------------------------------------------------------------------
```

### Read and write files

You may read packets from PCAP or PCAP-NG files:

```text
pg> array_of_packets = read('file.pcap')
```

You also may write packets to a file \(only PCAP-NG is supported\):

```text
pg> write('file.pcapng', array_of_packets)
```

Or write a single packet to a file:

```text
pg> packet.to_f('file.pcapng')
```

### Send packets

Sending a packet is as easy as:

```text
pg> pkt.to_w
```

The packet will be send on your first network interface. You also may choose interface on which sends packet:

```text
pg> pkt.to_w('eth1')
```

In general, packets are erroneous because some fields are not properly set. To easily fix that, use `PacketGen::Packet#calc`, which will calculate all calculatable fields \(for now: length and checksum ones\):

```text
pg> pkt.calc
pg> pkt.to_w
```

Of course, this is to you to put correct values for addresses or ports, by example.

### Capture packets

You may capture packets to post-process them:

```text
pg> packets = capture(iface: 'eth0', max: 50, timeout: 10)
```

This command will capture at most 50 packets from eth0, during at most 10 seconds.

You also may process them on the fly:

```text
pg> capture(iface: 'eth0', max: 5, timeout: 10) do |pkt|
pg*     p pkt
irb(#<PgConsole:0x0055d20a9812a0>):004:1> end
```

Captured packets may be filtered using a tcpdump filter:

```text
pg> packets = capture(iface: 'eth0', max: 50, filter: 'ip dst 192.168.1.1')
```

## Go further

Read others pages from this wiki.

[API documentation](http://www.rubydoc.info/gems/packetgen/) also gives all methods for `PacketGen::Packet` and for all header classes.

