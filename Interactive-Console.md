PacketGen provides an interactive console: `pgconsole`. It may be used to quickly write and test some pieces of code.

`pgconsole` uses IRB, or Pry if it is installed. It includes [PacketGen](http://www.rubydoc.info/gems/packetgen/PacketGen) class methods and [PacketGen::Utils](http://www.rubydoc.info/gems/packetgen/PacketGen/Utils) ones to simplify access to them.

## Quick access to PacketGen methods
`gen`, `parse`, `capture`, `read`, `write` and `default_iface` are quickly accessible:

```
pg> pkt = gen('Eth', src: '00:00:00:00:00:01', dst: '00:00:00:00:00:02')
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::Eth ------------------------------------------
           MacAddr          dst: 00:00:00:00:00:02
           MacAddr          src: 00:00:00:00:00:01
             Int16    ethertype: 0          (0x0000)
pg>
pg> parse(pkt.to_s, first_header: 'Eth')
=> -- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::Eth ------------------------------------------
           MacAddr          dst: 00:00:00:00:00:02
           MacAddr          src: 00:00:00:00:00:01
             Int16    ethertype: 0          (0x0000)
pg> write 'packet.pcapng', [pkt]
pg> pkts = read('packet.pcapng')
=> [-- PacketGen::Packet -------------------------------------------------
---- PacketGen::Header::Eth ------------------------------------------
           MacAddr          dst: 00:00:00:00:00:02
           MacAddr          src: 00:00:00:00:00:01
             Int16    ethertype: 0          (0x0000)
]
pg>
pg> capture { |pkt| p pkt }
```

## local configuration
`pgconsole` provides quick access to local network configuration through `config`, a [PacketGen::Config](http://www.rubydoc.info/gems/packetgen/PacketGen/Config) object:

```
$ pgconsole
pg> config.default_iface
=> "eth0"
pg> config.hwaddr
=> "7a:fb:fc:fd:fe:ff"
pg> config.ipaddr
=> "192.168.0.2"
pg> config.ip6addr
=> ["fe80::e255:ffff:275c:9bee%eth0"]
pg> config.ipaddr('lo')
=> "127.0.0.1"
```

This local configuration may be used to forge packets:

```
pg> pkt = gen('IP', src: config.ipaddr, dst: '8.8.8.8')
pg> pkt.recalc
pg> pkt.to_w
=> 20
```

## Utils
To ease tests, mMethods from `PacketGen::Utils` module are quickly accessible from `pgconsole`:
```
pg> PacketGen::Utils.arp '192.168.0.1'
=> "aa:bb:cc:dd:ee:ff"
pg> arp '192.168.0.1'
=> "aa:bb:cc:dd:ee:ff"
```

Utils methods are:
* [`arp`](http://www.rubydoc.info/gems/packetgen/PacketGen/Utils#arp-class_method)
  to get MAC address for given IP address,
* [`arp_spoof`](http://www.rubydoc.info/gems/packetgen/PacketGen/Utils#arp_spoof-class_method)
  to do ARP spoofing.
* [`mitm`](http://www.rubydoc.info/gems/packetgen/PacketGen/Utils#mitm-class_method) to do a Man-In-The-Middle attack (on local network only).
