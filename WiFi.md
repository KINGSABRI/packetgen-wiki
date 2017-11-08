# WiFi

PacketGen can handle wifi packets thanks to `PacketGen::Header::Dot11` classes.

## Create wifi packets

As `PacketGen::Header::Dot11` is an abstract class it should not be used directly.
Instead, `PacketGen::Header::Dot11::Control`, `PacketGen::Header::Dot11::Management` and `PacketGen::Header::Dot11::Data` should be used.

Creation of protected frames is not supported yet.

### Create control frames
Control frames may be created this way:

```ruby
pkt = PacketGen.gen('Dot11::Control', subtype: 13) # Ack control frame
pkt.dot11_control     # => PacketGen::Header::Dot11::Control
```

### Create management frames
Management frames may be created this way:

```ruby
pkt = PacketGen.gen('Dot11::Management')
pkt.dot11_management     # => PacketGen::Header::Dot11::Management
```

Management frames are usually specialized. By example, you may want to create an AssociationRequest frame:

```ruby
pkt = PacketGen.gen('Dot11::Management')
pkt.add('Dot11::AssoReq')
pkt.dot11_assoreq        # => PacketGen::Header::Dot11::AssoReq
```

Management frames also may contain some elements (see IEEE 802.11 standard):

```ruby
# add a SSID to AssociationRequest frame
el1 = PacketGen::Header::Dot11::Element.new(type: 'SSID', value: 'My SSID')
pkt.dot11_assoreq.elements << el1

# And also add supported rates
el2 = PacketGen::Header::Dot11::Element.new(type: 'Rates', value: supported_rates)
pkt.dot11_assoreq.elements << el2
```

### Create data frames
Data frames may be created this way (LLC and SNAP layers are needed):

```ruby
pkt = PacketGen.gen('Dot11::Data', mac1: receiver, mac2: transmitter, mac3: destmac).
                add('LLC').add('SNAP').add('IP', src: sourceip, dst: destip)
pkt.dot11_data     # => PacketGen::Header::Dot11::Data
# #dot11 is a shortcut for #dot11_data
pkt.dot11          # => PacketGen::Header::Dot11::Data
# access to IP datagram
pkt.ip             # => PacketGen::Header::IP
```

## Send wifi packets

To send a Dot11 packet, simply do:

```ruby
pkt = PacketGen.gen('RadioTap').
                add('Dot11::Management', mac1: clientaddr, mac2: bssid, mac3: bssid).
                add('Dot11::DeAuth', reason: 7)
# compute all checksum and length
pkt.calc
pkt.to_w('wlan0')
```
RadioTap header is needed to send packets on the air.

Before sending a packet, you always should call `PacketGen::Packet#calc` to automatically set lengths and checksums in packets. This is not done automatically to let you send malformed packets on "wire" for network test purpose.

## Capture and parse wifi packets

Capturing and parsing Dot11 packets is supported by `PacketGen.capture`, `PacketGen.read` and `PacketGen.parse`.

Captured packets may contain a header before Dot11 one: a `PPI` or a `RadioTap`, depending on your network interface's driver.

### Capturing wifi packets

```ruby
PacketGen.capture(iface: 'wlan0') do |packet|
  # Here packet should be instances of PacketGen::Packet with a Dot11 header
  do_stuffs_with(packet)
end
```
### Parsing wifi packets in general
Parsing wifi packets is also supported from reading from a PCAP (or PCAP-ng) file, or from parsing a binary string.

## Frame Control Sequence

FCS is the CRC of the Dot11 frame. By default, PacketGen, sets a FCS field in all Dot11
frames, and try to parse it when parsing or capturing.

But, some drivers remove FCS field when capturing, and/or do not support setting it.
You may deactivate FCS in Dot11, for all packets:

```ruby
PacketGen::Header::Dot11.has_fcs = false
```

## See also
API documentation for [PacketGen::Header::Dot11](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/Dot11.html)
