---
description: Simple Network Management Protocol (SNMP)
---

# SNMP

SNMP is quite different from others protocols: it uses [ASN.1](https://en.wikipedia.org/wiki/ASN.1) data structures. So, to understand SNMP serialization/deserialization, you need to understand ASN.1.

## ASN.1

ASN.1 is an interface description language for defining data structures that can be serialized and deserialized in a standard and cross-platform way. It is broadly used in telecommunications and computer networking, and also in cryptography \(X.509 certificates are defined with ASN.1, for example\).

ASN.1 is a notation independent of the way data are encoded. Encoding are specified in Encoding Rules.

The most used encoding rules are BER \(Basic Encoding Rules\) and DER \(Distinguished Encoding Rules\). Both are quite equivalent, but the latter is specified to guaranty uniqueness of encoding. But, in fact, there are just little differences between them.

ASN.1 provides basic objects, such as: integers, many kinds of strings, floats, booleans. It also provides some container objects \(sequences and sets\). They all are defined in Universal class. A protocol may defined others objects, which will be grouped in the Context class.For example, SNMP defines GetRequest-PDU object in this class. They also exist Private and Application classes.

Each basic object has a tag, used by the encoding rules. For example, Boolean has tag value 1, Integer has tag value 2. In context class, tags begin at 0xA0. For example, in SNMP context, 0xA0 tag is a GetRequest-PDU.

Others objects may be constructed from those basic ones using Sequences and Sets. Sequences are like ruby arrays. Sets are arrays limited to a unique object type.

Finally, a ASN.1 object is a tree, whose leafs are basic types, and non-leaf nodes are Sets or Sequences.

### ASN.1 in PacketGen

In PacketGen, ASN.1 objetcs are handled using `rasn1` gem. This gem defines basic ASN.1 objets and provides ways to decode and encode data in DER and BER encodings.

### rasn1 gem

rasn1 gem provides a `RASN1::Model` class to define complex ASN.1 objects. See [Rasn1 wiki](https://github.com/sdaubert/rasn1/wiki) for a simple example.

## SNMP

In PacketGen, `SNMP` header inherits from `PacketGen::Header::ASN1Base`, which inherits from `RASN1::Model`. `Header::ASN1Base` provides \[\[Header minimal API\|Create Custom Protocol\#Header minimal API\]\].

Some ASN1. objets are also defined in `PacketGen::Header::SNMP` namespace:

* [`PDUs`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/PDUs),

  which is a CHOICE between all SNMP PDUs,

* [`GetRequest`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/GetRequest),

  which is the model of a SNMP Get request,

* [`GetNextRequest`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/GetNextRequest),
* [`GetResponse`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/GetResponse),
* [`SetRequest`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/SetRequest),
* [`Trapv1`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/Trapv1),

  which is Trap PDU for SNMPv1,

* [`Bulk`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/Bulk),
* [`InformRequest`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/InformRequest),
* [`Trapv2`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/Trapv2),

  which is Trap PDU for SNMPv2,

* [`Report`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/Report),
* [`VariableBindings`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/VariableBindings),

  which is a SEQUENCE OF \(an array of\) `VarBind`. This class is used in PDU classes,

* [`VarBind`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP/VarBind),

  which is an association between a name \(as an OBJECT ID\) and a value \(its type

    depends on its name\).

### SNMP base class

[`SNMP`](http://www.rubydoc.info/gems/packetgen/PacketGen/Header/SNMP) class is a simple ASN.1 object defined like this:

```ruby
class SNMP < ASN1Base
  sequence :message,
           content: [enumerated(:version, value: 'v2c',
                                enum: { 'v1' => 0, 'v2c' => 1, 'v2' => 2, 'v3' => 3 }),
                     octet_string(:community, value: 'public'),
                     model(:data, PDUs)]
end
```

which is equivalent to this ASN.1 definition:

```text
Message ::= SEQUENCE {
                version INTEGER { version-1(0), version-2c(1), version-2(2), version-3(3) },
                community OCTET STRING,
                data ANY        -- Here is PDU
            }
```

All objects may be accessed to `#[]` accessor:

```ruby
snmp = PacketGen::Header::SNMP.new
snmp[:version]             # => RASN1::Types::Enumerated
snmp[:version].to_i        # => 1
snmp[:version].value       # => "v2c"
snmp[:community]           # => RASN1::Types::OctetString
snmp[:community].value     # => "public"
snmp[:data]                # => PacketGen::Header::SNMP::PDUs
```

For convenience, these upper fields are accessible through accessors, as for others headers:

```ruby
snmp.version = 0
snmp.version           # => "v1"
snmp.version = "v3"

snmp.community         # => "public"
snmp.commutiyy = 'private'

snmp.data             # => PacketGen::Header::SNMP::PDUs
```

### SNMP::PDUs class

In RFC, PDUs is defined as:

```text
PDUs ::= CHOICE {
           get-request      [0] IMPLICIT PDU,
           get-next-request [1] IMPLICIT PDU,
           get-response     [2] IMPLICIT PDU,
           set-request      [3] IMPLICIT PDU,
           snmpV1-trap      [4] IMPLICIT PDU,
           get-bulk-request [5] IMPLICIT PDU,
           inform-request   [6] IMPLICIT PDU,
           snmpV2-trap      [7] IMPLICIT PDU,
           report           [8] IMPLICIT PDU
         }
```

`SNMP::PDUs` class is defined as a subclass of `RASN1::Model`, and as a CHOICE.

Setting header type \(or PDU type\) may be done this way:

```ruby
smp.data.root                # access to root ASN.1 object in PDU class
                             # => RASN1::Types::CHOICE
snmp.data.root.chosen = 0    # Choose first CHOICE from PDUs: SNMP::GetRequest
snmp.data.root.chosen_value  # => PacketGen::Header::SNMP::GetRequest
```

As RASN1::Model may delegate some methods to its root object, we can simplify previous code:

```ruby
smp.data.root             # access to root ASN.1 object in PDU class
                          # => RASN1::Types::CHOICE
snmp.data.chosen = 0      # Choose first CHOICE from PDUs: SNMP::GetRequest
snmp.data.chosen_value    # => PacketGen::Header::SNMP::GetRequest
# or even simpler
snmp.pdu                  # => PacketGen::Header::SNMP::GetRequest
```

### SNMP::GetRequest class

GetRequest PDU is defined as:

```text
GetRequest-PDU ::= [0] IMPLICIT PDU

PDU ::= SEQUENCE {
            request-id INTEGER (-214783648..214783647),

            error-status                -- sometimes ignored
                INTEGER {
                    noError(0),
                    tooBig(1),
                    noSuchName(2),      -- for proxy compatibility
                    badValue(3),        -- for proxy compatibility
                    readOnly(4),        -- for proxy compatibility
                    genErr(5),
                    noAccess(6),
                    wrongType(7),
                    wrongLength(8),
                    wrongEncoding(9),
                    wrongValue(10),
                    noCreation(11),
                    inconsistentValue(12),
                    resourceUnavailable(13),
                    commitFailed(14),
                    undoFailed(15),
                    authorizationError(16),
                    notWritable(17),
                    inconsistentName(18)
                },

            error-index                 -- sometimes ignored
                INTEGER (0..max-bindings),

            variable-bindings           -- values are sometimes ignored
                VarBindList
        }
```

So a `GetRequest` object has these accessors:

```ruby
snmp = PacketGen::Header::SNMP.new
snmp.data.chosen = 0                 # GetRequest PDU

# accessors
snmp.pdu[:id]            # request-id (RASN1::Types::Integer)
snmp.pdu[:error]         # error, with possible values from SNMP::ERRORS (RASN1::Types::Enumerated)
snmp.pdu[:error_index]   # error-index (RASN1::Types::Integer)
snmp.pdu[:varbindlist]   # variable-bindings (SNMP::VariableBindings)
```

You may add some `VarBind`:

```ruby
# name is an OBJECT ID, and there is no value in a GetRequest (value set to NULL)
snmp.pdu[:varbindlist] << { name: '1.3.6.1.2.1.1.5.0' }
snmp.pdu[:varbindlist].value[0]              # => PacketGen::Header::SNMP::VarBind
snmp.pdu[:varbindlist].value[0][:name]       # =>  RASN1::Types::ObjectId
snmp.pdu[:varbindlist].value[0][:name].value # =>  '1.3.6.1.2.1.1.5.0'
```

### SNMP::GetNextRequest class

`GetNextRequest` is a subclass of `GetRequest`, with only a different PDU identifier.

Creating a `GetNextRequest` may be done this way:

```ruby
snmp = PacketGen::Header::SNMP.new
snmp.data.chosen = 1         # GetNextRequest PDU
snmp.pdu                     # => PacketGen::Header::SNMP::GetNextRequest
```

### SNMP::GetResponse class

`GetResponse` is a subclass of `GetRequest`, with only a different PDU identifier.

Creating a `GetResponse` may be done this way:

```ruby
snmp = PacketGen::Header::SNMP.new
snmp.data.chosen = 2         # GetResponse PDU
snmp.pdu                     # => PacketGen::Header::SNMP::GetResponse
```

### SNMP::SetRequest class

`GetResponse` is a subclass of `GetRequest`, with only a different PDU identifier.

Creating a `GetResponse` may be done this way:

```ruby
snmp = PacketGen::Header::SNMP.new
snmp.data.chosen = 3         # SetRequest PDU
snmp.pdu                     # => PacketGen::Header::SNMP::SetRequest
```

### SNMP::Trapv1 class

`Trapv1` PDU is defined as:

```text
Trap-PDU ::= [4] IMPLICIT SEQUENCE {
                        enterprise OBJECT IDENTIFIER,
                        agent-addr NetworkAddress,
                        generic-trap      -- generic trap type
                            INTEGER {
                                coldStart(0),
                                warmStart(1),
                                linkDown(2),
                                linkUp(3),
                                authenticationFailure(4),
                                egpNeighborLoss(5),
                                enterpriseSpecific(6)
                            },
                        specific-trap INTEGER,
                        time-stamp TimeTicks,
                        variable-bindings VarBindList
                 }
```

So a `GetRequest` object has these accessors:

```ruby
snmp = PacketGen::Header::SNMP.new
snmp.data.chosen = 4                 # Trapv1 PDU

# accessors
snmp.pdu[:enterprise]       # => RASN1::Types::ObjectId
snmp.pdu[:agent_addr]       # => RASN1::Types::OctetString
snmp.pdu[:generic_trap]     # => RASN1::Types::Enumerated
snmp.pdu[:specific_trap]    # => RASN1::Types::Integer
snmp.pdu[:timestamp]        # => RASN1::Types::Integer
snmp.pdu[:varbindlist]      # => PacketGen::Header::SNMP::VariableBindings
```

`:generic_trap` may take these values:

* `cold_start` or `0`,
* `warm_start` or `1`,
* `link_down` or `2`,
* `link_up` or `3`,
* `auth_failure` or `4`,
* `egp_neighbor_loss` or `5`,
* `specific` or `6`.

### SNMP::Bulk class

Create a Bulk PDU:

```ruby
bulk = PacketGen::Header::SNMP.new
bulk.data.chosen = 5
```

Bulk accessors:

```ruby
bulk.pdu[:id]                # => RASN1::Types::Integer
bulk.pdu[:non_repeaters]     # => RASN1::Types::Integer
bulk.pdu[:max_repetitions]   # => RASN1::Types::Integer
bulk.pdu[:varbindlist        # => PacketGen::Header::SNMP::VariableBindings
```

### SNMP::InformRequest class

Create a InformRequest PDU:

```ruby
ir = PacketGen::Header::SNMP.new
ir.data.chosen = 6
```

InformRequest is a subclass of GetRequest, so it has the same accessors.

### SNMP::Trapv2 class

Create a Trapv2 PDU:

```ruby
ir = PacketGen::Header::SNMP.new
ir.data.chosen = 7
```

Trapv2 is a subclass of GetRequest, so it has the same accessors.

### SNMP::Report class

Create a Report PDU:

```ruby
ir = PacketGen::Header::SNMP.new
ir.data.chosen = 8
```

Report is a subclass of GetRequest, so it has the same accessors.

