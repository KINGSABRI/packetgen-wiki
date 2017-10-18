Since v1.1.0, PacketGen permits adding your own header classes.

## Quick start
To add a new/custom header, you first have to define the new header class. For example:

```ruby
module MyModule
 class MyHeader < PacketGen::Header::Base
   define_field :field1, PacketGen::Types::Int32   
   define_field :field2, PacketGen::Types::Int32   
 end
end
```

Then, class must be declared to PacketGen:

```ruby
PacketGen::Header.add_class MyModule::MyHeader
```

Finally, bindings must be declared:

```ruby
# bind MyHeader as IP protocol number 254 (needed by Packet#parse and Packet#add)
PacketGen::Header::IP.bind_header MyModule::MyHeader, protocol: 254
```

And use it:

```ruby
pkt = Packet.gen('IP').add('MyHeader', field1: 0x12345678)
pkt.myheader.field2.read 0x01
```
