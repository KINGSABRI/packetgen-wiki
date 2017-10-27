PacketGen provides simple ways to generate, send and capture network packets.

## Usage

### Easily create packets
```ruby
PacketGen.gen('IP')             # generate a IP packet object
PacketGen.gen('TCP')            # generate a TCP over IP packet object
PacketGen.gen('IP').add('TCP')  # the same
PacketGen.gen('Eth')            # generate a Ethernet packet object
PacketGen.gen('IP').add('IP')   # generate a IP-in-IP tunnel packet object

# Generate a IP packet object, specifying addresses
PacketGen.gen('IP', src: '192.168.1.1', dst: '192.168.1.2')

# get binary packet
PacketGen.gen('IP').to_s
```

### Send packets on wire
```ruby
# send Ethernet packet
PacketGen.gen('Eth', src: '00:00:00:00:00:01', dst: '00:00:00:00:00:02').to_w
# send IP packet
PacketGen.gen('IP', src: '192.168.1.1', dst: '192.168.1.2').to_w
# send forged IP packet over Ethernet
PacketGen.gen('Eth', src: '00:00:00:00:00:01', dst: '00:00:00:00:00:02').
          add('IP').to_w('eth1')
```

### Parse packets from binary data
```ruby
packet = PacketGen.parse(binary_data)
```

### Capture packets from wire
```ruby
# Capture packets from first network interface, action from a block
PacketGen.capture do |packet|
  do_stuffs_with_packet
end

# Capture some packets, and act on them afterward
# will return when 10 packets will be captured
packets = PacketGen.capture(iface: 'eth0', max: 10)

# Use filters
packets = PacketGen.capture(iface: 'eth0', filter: 'ip src 1.1.1.2', max: 1)
```

### Easily manipulate packets
```ruby
# access header fields
pkt = PacketGen.gen('IP').add('TCP')
pkt.ip.src = '192.168.1.1'
pkt.ip(src: '192.168.1.1', ttl: 4)
pkt.tcp.dport = 80

# access header fields when multiple header of one kind exist
pkt = PacketGen.gen('IP').add('IP')
pkt.ip.src = '192.168.1.1'  # set outer src field
pkt.ip(2).src = '10.0.0.1'  # set inner src field

# test packet types
pkt = PacketGen.gen('IP').add('TCP')
pkt.is? 'TCP'   # => true
pkt.is? 'IP'    # => true
pkt.is? 'UDP'   # => false

# encapulsate/decapsulate packets
pkt2 = PacketGen.gen('IP')
pkt2.encapsulate pkt                   # pkt2 is now a IP/IP/TCP packet
pkt2.decapsulate(pkt2.ip)              # pkt2 is now inner IP/TCP packet
```

### Read/write PcapNG files
```ruby
# read a PcapNG file, containing multiple packets
packets = PacketGen.read('file.pcapng')
packets.first.udp.sport = 65535
# write only one packet to a PcapNG file
pkt.write('one_packet.pcapng')
# write multiple packets to a PcapNG file
PacketGen.write('more_packets.pcapng', packets)
```

## See also

API documentation: http://www.rubydoc.info/gems/packetgen
