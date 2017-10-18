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

## Add a new header type, in more detail

### Define a header class
A new header class should inherit from [`PacketGen::Header::Base`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/Base) class (or from `PacketGen::Header::ASN1Base` but this is off topic). This base class implements minimal API to parse a header from binary string, or to generate binary string from a header class.

`PacketGen::Header::Base` inherits from  [`PacketGen::Types::Fields`](http://www.rubydoc.info/gems/packetgen/PacketGen/Types/Fields), which is a class to define headers or anything else with a binary format containing multiple fields.

Here, magical method is `define_field`. This method define a field and its name. You may set as fields as you need. A type may be a predefined type from `PacketGen::Types`, or any other `PacketGen::Types::Fields` subclass:

```ruby
class CustomField < PacketGen::Types::Fields
  # add a 16-bit integer field
  define_field :one, PacketGen::Types::Int16
  # add another one
  define_field :two, PacketGen::Types::Int16
end

class ExampleHeader < PacketGen::Header::Base
  # define a first 32-bit integer field
  define_field :first, PacketGen::Types::Int32
  # add a custom field
  define_field :custom, CustomField
  # add a 8-bit integer field
  define_field :int8, PacketGen::Types::Int8

  # then bit fields on int8 field
  #  ack and error are boolean flags, as no size is specified (default to 1)
  #  rsv is a 4-bit field
  #  type is a 2-bit field
  define_bit_fields_on :int8, :ack, :error, :rsv, 4, :type, 2
end
```

Here, some example to access these fields:

```ruby
example = ExampleHeader.new

example.first        #=> Integer
example.first = 0x12345678

example.custom       #=> CustomField
example.custom.one   #=> Integer

example.ack?         #=> Boolean
example.ack = true

example.type         #=> Integer
example.type = 1
```

### Add some methods to a header class
Work in Progess...