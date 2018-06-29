# Create Custom Protocol

Since v1.1.0, PacketGen allows you adding your own header classes.

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

A new header class should inherit from [`PacketGen::Header::Base`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/Base) class \(or from `PacketGen::Header::ASN1Base` but this is off topic\). This base class implements minimal API \(see [below](create-custom-protocol.md#header-minimal-api)\) to parse a header from binary string, or to generate binary string from a header class.

`PacketGen::Header::Base` inherits from [`PacketGen::Types::Fields`](http://www.rubydoc.info/gems/packetgen/PacketGen/Types/Fields), which is a class to define headers or anything else with a binary format containing multiple fields.

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

### Builtin Types

To create new Fields or Header classes, PacketGen provides some base types:

* integers:
  * `Int8`,
  * big-endian integers: `Int16` or `Int16be`, `Int32` or `Int32be` and `Int64` or `Int64be`,
  * little-endian integers: `Int16le`, `Int32le` and `Int64le`,
* enumerated integers: `Int8Enum`, `Int16Enum`, `Int16beEnum`, `Int16leEnum`, `Int32Enum`, `Int32beEnum` and `Int32leEnum`,
* strings: `String`, `IntString` \(a string with a length field at the beginning\)

  and `CString` \(null-terminated string\),

* arrays: `Array` to define set of fields,
* `TLV` to define fields as set of type-length-value,
* `OUI` \(_Organizationally Unique Identifier_\).

All these types are defined under `PacketGen::Types` namespace.

### Add some methods to a header class

By default, all fields will have accessors with the good type. By example, a `Types::Int32` field may be accessed as an Integer, a `Types::String` may be accessed as a String, etc.

But, for some reason, you may need to add another accessor, or a method to compute some protocol data.

To do that, you have to understanf Fields model. A field may be accessed through its accessor, as already seen. But it may also be accessed through Fields' hash.

Fields class defines `#[]` method to access to field object by its name. By example, with our previously defined ExampleHeader:

```ruby
example.first     #=> Integer
example[:first]   #=> PacketGen::Types::Int32
```

Sometimes, you will need to access real field object. All field objects have common methods:

* `#read` reads a binary string to set object,
* `#to_s` gives binary string from object,
* `#sz` gives binary size.

As an example of method to a header class, we will define one to calculate `first` field, which value should be size of custom field:

```ruby
class ExampleHeader
  # compute first field
  def calc_first
    self[:first].read self[:custom].sz
  end
end
```

## Add a new ASN.1 header

Some header may be defined using ASN.1 notation, like SNMP.

To define such a header, use [`PacketGen::Header::ASN1Base`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/ASN1Base) as base class:

```ruby
class SNMP < PacketGen::Header::ASN1Base
  sequence :message,
            content: [enumerated(:version, value: 'v2c',
                                 enum: { 'v1' => 0, 'v2c' => 1, 'v2' => 2, 'v3' =>}),
                      octet_string(:community, value: 'public'),
                      model(:data, PDUClass)]

  define_attributes :version, :community
end
```

This definition uses lots of stuffs from [`rasn1`](https://github.com/sdaubert/rasn1/wiki) gem:

* `sequence` defines a ASN.1 Sequence named `message`. This sequence contains:
  * an Enumerated named `version`,
  * an Octet String named `community`,
  * and a PDUClass \(subclass of [`RASN1::Model`](http://www.rubydoc.info/gems/rasn1/RASN1/Model)\) named `data` and not defined here.

`define_attributes` is a helper method to declare attributes from some ASN.1 fields. This helps to mimic standard header behaviour.

## Packet magics

`PacketGen::Packet` defines some magics using specific header methods.

If your header class has a checksum field and/or a length field, `Packet` provides magic for them. You have to define a `#calc_checksum` and/or a `calc_length` which appropriatly set checksum and/or length fields respectively. Then `Packet#calc_checksum` and `Packet#calc_length` will calculate all checksum and length fields in all headers, including yours.

If your header class is not an application layer one, you should define a `body` field of type `PacketGen::Types::String`. This will allow `Packet#parse` to automagically parse headers embedded in yours. Same magic will happen for `Packet#to_s`, `Packet#encapsulate`, `Packet#decapsulate` and `Packet#add`.

## Header minimal API

`PacketGen::Header::Base` and `PacketGen::Header::ASN1Base` are provided to simplify writing of new headers. But they may not be so useful for some protocol types.

So, here is minimal API needed by PacketGen to handle a header class.

A header MUST have accessors:

* `#packet` : get/set packet to which header belongs.

A header MUST respond to:

* `#protocol_name`: get protocol name, usually class name as a String, without

   module path,

* `#method_name`: get method name, usually same as protocol name but downcase.

  This name is used as accessor from packet to access header object,

* `#read`: method to parse binary string and decode header,
* `#parse?`: return `true` if decoded header is correct. Used when guessing if

  header may be decoded from binary string. An example of use if checking first

  4-bit field for IP version in a IP/IPv6 header.

