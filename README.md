## What is PacketGen?

PacketGen is a pure ruby library to handle network packets.

### Create network packets

PacketGen makes creating network packets in a ruby way easy. Multiple layers may
be put together easily, and depending data from one layer to another is
 automatically updated. You even may create erroneous low-level packets to stress
 your  network devices.
 
 ### Send packets on network
 
 Home-made packets may be easily sent over the network. Packets may be
sent at levels 2 or 3 from OSI model. Erroneous packets may be sent (by example,
a IP packet with bad checksum) to test network devices.

### Capture packets from network

PacketGen provides an easy way to capture network packets, as Wireshark does,
but these packets may be processed with ruby code, in a ruby way. Capture is
always done on level 2 from OSI model.
