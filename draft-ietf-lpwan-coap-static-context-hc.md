---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-coap-static-context-hc-03
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'

title: LPWAN Static Context Header Compression (SCHC) for CoAP
abbrev: LPWAN CoAP compression
wg: lpwan Working Group
author:
- ins: A. Minaburo
  name: Ana Minaburo
  org: Acklio
  street: 1137A avenue des Champs Blancs
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ana@ackl.io
- ins: L. Toutain
  name: Laurent Toutain
  org: Institut MINES TELECOM; IMT Atlantique
  street:
  - 2 rue de la Chataigneraie
  - CS 17607
  city: 35576 Cesson-Sevigne Cedex
  country: France
  email: Laurent.Toutain@imt-atlantique.fr
normative:
  rfc7252:
  rfc7967:
  rfc7641:
  I-D.ietf-lpwan-ipv6-static-context-hc:

--- abstract

This draft defines the way SCHC header compression can be applied to CoAP 
headers.
CoAP header structure differs from IPv6 and UDP protocols since the CoAP Header 
is flexible header with a variable number of options themself of a variable length. 
Another important difference is 
the asymmetry in the header information used for request and 
response messages. Most of the compression mechanisms have been introduced in 
{{I-D.ietf-lpwan-ipv6-static-context-hc}}, this document explains how to use the SCHC compression
for CoAP.

--- middle

# Introduction {#Introduction}

CoAP {{rfc7252}} is an implementation of the REST architecture for constrained 
devices. Nevertheless, if limited, the size of a CoAP header may be
   too large for LPWAN constraints and some compression may be
   needed to reduce the header size. 
   
{{I-D.ietf-lpwan-ipv6-static-context-hc}} defines a header compression
   mechanism for LPWAN network based on a static context. The context is
   said static since the field description composing the Rules and the context are not
   learned during the packet exchanges but are previously defined.  The
   context(s) is(are) known by both ends before transmission. 
  
   A context is composed of a set of rules that are referenced by Rule IDs 
   (identifiers).  A rule contains an ordered list of the fields descriptions containing à field ID (FID), its length (FL)
   and its position (FP) when repeated differs from 1, a direction indicator (DI) (upstream, downstream and bidirectional)
   and some associated Target Values (TV) which are expected in the message header. A Matching Operator (MO) is
   associated to each header field description. The rule is selected if all the MOs fit
   the TVs.  In that case, a Compression/Decompression Action (CDA)
   associated to each field defines the link between the compressed and
   decompressed value for each of the header fields. 
   
This document describes how the rules can be applied to CoAP flows. 
   Compression of the CoAP header may be done in conjunction with the 
   above layers (IPv6/UDP) or independantly.
    

#  CoAP Compression with SCHC

CoAP differs from IPv6 and UDP protocols on the following aspects: 
   
* IPv6 and UDP are symmetrical protocols. The same fields are found in the
  request and in the response, only the location in the header may vary 
  (e.g. source and destination fields). A CoAP request is different from a response. 
  For example, the URI-path option is mandatory in the request and is not found in the response, 
  a request may contain an Accept option and the response a Content option.
  
  {{I-D.ietf-lpwan-ipv6-static-context-hc}} allows to use a message direction (DI) when
  processing the rule. 
  
* Even when a field is "symmetric" (i.e. found in both directions) the values carried are
  different. For instance the Type field will contain a CON value in the request and a
  ACK or RST value in the response. Exploiting the asymmetry in compression will allow to 
  send no bit in the compressed request and a single bit in the answer. Same behavior can be 
  applied to the CoAP Code field (O.OX code are present in the request and Y.ZZ in the answer).
  The direction allows to split in two parts the possible values for each direction. 
  
  In combination
  with the mapping list, the size of the compression residu can be reduced.

* In IPv6 and UDP header fields have a fixed size. In CoAP, Token size
  may vary from 0 to 8 bytes, length is given by a field in the header. More
  systematically, the CoAP options are described using the Type-Length-Value. 
  When applying SCHC header compression. 
  
  {{I-D.ietf-lpwan-ipv6-static-context-hc}}
  offers the possibility to define a function for the Field Length in the Field Description.
  
* In CoAP headers a field can be duplicated several times, for instances, elements of an URI 
  (path or queries). The position defined in a rule, associated to a Field ID, can be used to 
  identify the proper element.

  {{I-D.ietf-lpwan-ipv6-static-context-hc}} allows a Field id to appears several times in the
  rule, the Field Position (FP) removes ambiguities for the matching operation. 


* CoAP also obeys to the client/server paradigm and the compression rate can
  be different if the request is issued from an LPWAN node or from an non LPWAN
  device. For instance a Device (Dev) aware of LPWAN constraints can generate a 1 byte token, but
  a regular CoAP client will certainly send a larger token to the Thing. SCHC compression
  will not modify the values to offer a better compression rate. Nevertheless a proxy placed
  before the compressor may change some field values to offer a better compression rate and 
  maintain the necessary context for interoperability with existing CoAP implementations.
  

# Compression of CoAP header fields

This section discusses of the compression of the different CoAP header fields. These are just
examples. The compression should take into account the nature of the traffic and not 
only the field values.

## CoAP version field (2 bits)

This field is bidirectional and must be elided during the SCHC compression, since it always
contains  the same value. In the future, if new version of CoAP are defined, new rules will 
have to defined leading to no ambiguities between versions. 

## CoAP type field

{{rfc7252}} defines 4 types of messages: CON, NON, ACK and RST. The latter two ones are a response of the two first ones. If the device plays a specific role, a rule can exploit these property with the mapping list: [CON, NON] for one direction and [ACK, RST] for the other direction. Compression residue is reduced to 1 bit. 

The field must elided if for instance the client is sending only NON or CON messages. 

In any case, a rule must be define to carry RST to a client.
  
##CoAP code field

This field is bidirectional, but compression can be enhanced using DI. 

The CoAP Code field defines a tricky way to 
ensure compatibility with
HTTP values.  Nevertheless only 21 values are defined by {{rfc7252}}
compared to the 255 possible values. 



~~~~

     +------+------------------------------+-----------+
     | Code | Description                  | Mapping   |
     +------+------------------------------+-----------+
     | 0.00 |                              |  0x00     |
     | 0.01 | GET                          |  0x01     |
     | 0.02 | POST                         |  0x02     |
     | 0.03 | PUT                          |  0x03     |
     | 0.04 | DELETE                       |  0x04     |
     | 0.05 | FETCH                        |  0x05     |
     | 0.06 | PATCH                        |  0x06     |
     | 0.07 | iPATCH                       |  0x07     |
     | 2.01 | Created                      |  0x08     |
     | 2.02 | Deleted                      |  0x09     |
     | 2.03 | Valid                        |  0x0A     |
     | 2.04 | Changed                      |  0x0B     |
     | 2.05 | Content                      |  0x0C     |
     | 4.00 | Bad Request                  |  0x0D     |
     | 4.01 | Unauthorized                 |  0x0E     |
     | 4.02 | Bad Option                   |  0x0F     |
     | 4.03 | Forbidden                    |  0x10     |
     | 4.04 | Not Found                    |  0x11     |
     | 4.05 | Method Not Allowed           |  0x12     |
     | 4.06 | Not Acceptable               |  0x13     |
     | 4.12 | Precondition Failed          |  0x14     |
     | 4.13 | Request Entity Too Large     |  0x15     |
     | 4.15 | Unsupported Content-Format   |  0x16     |
     | 5.00 | Internal Server Error        |  0x17     |
     | 5.01 | Not Implemented              |  0x18     |
     | 5.02 | Bad Gateway                  |  0x19     |
     | 5.03 | Service Unavailable          |  0x1A     |
     | 5.04 | Gateway Timeout              |  0x1B     |
     | 5.05 | Proxying Not Supported       |  0x1C     |
     +------+------------------------------+-----------+

~~~~
{: #Fig--example-code-mapping title="Example of CoAP code mapping"}
   
{{Fig--example-code-mapping}} gives a possible mapping, it can be changed 
to add new codes or reduced if some values are never used by both ends. 
It could efficiently be coded on 5 bits. 

Even if the number of code can be increase 
with other RFC, implementations may use a limited number of values, which can
help to reduce the number of bits sent on the LPWAN.

The number of code may vary over time, some new codes
may be introduced or some applications use a limited number of values. 

The client and the server do not use the 
same values. This asymmetry can be exploited to reduce the size sent on 
the LPWAN. 

The field can be treated differently in upstream than in downstream.
If the Thing is a client an entry can be set on the uplink message with a
code matching for 0.0X values and another for downlink values for Y.ZZ
codes.  It is the opposite if the thing is a server.

If the ES always sends or receives requests with the same method, the Code
field can be elided. The entry below shows a rule for a client sending only 
GET request.

~~~~~~
FID  FL FP DI  TV  MO     CDA    Sent
code 8  1  up GET equal not-sent
~~~~~~

If the client may send different methods, a matching-list can be applied. For
table {{Fig--example-code-mapping}}, 3 bits are necessary, but it could be less 
if fewer methods are used. Example below gives an example where the ES is a server
and receives only GET and POST requests.

~~~~~~
FID  FL FP DI Target Value    MO            CDA       Sent
code 8  1  dw [0.01, 0.02] match-mapping mapping-sent [1]
~~~~~~

The same approach can be applied to responses. 
 
## CoAP Message ID field

This field is bidirectional.

Message ID is used for two purposes:

* To acknowledge a CON message with an ACK.

* To avoid duplicate messages. 

In LPWAN, since a message can be received by several radio gateway, some 
LPWAN technologies include a sequence number in L2 to avoid duplicate frames. Therefore
if the message does not need to be acknowledged (NON or RST message), the Message
ID field can be avoided.

~~~~~~
FID FL FP DI TV   MO     CDA    Sent
Mid 8  1  bi    ignore not-sent
~~~~~~

The decompressor must generate a value.

\[\[Note; check id this field is not used by OSCOAP .\]\]

To optimize information sent on the LPWAN, shorter values may be used during the 
exchange, but Message ID values generated a common CoAP implementation will not take 
into account this limitation. Before the compression, a proxy may be needed to 
reduce the size. 

~~~~~~
FID FL FP DI   TV      MO    CDA   Sent
Mid 8  1  bi 0x0000 MSB(12) LSB(4) [4]
~~~~~~

Otherwise if no compression is possible, the field has to be sent

~~~~~~
FID FL FP DI TV   MO       CDA    Sent
Mid 8  1  bi    ignore value-sent [8]
~~~~~~


## CoAP Token field

This field is bi-directional.

Token is used to identify transactions and varies from one
   transaction to another.  Therefore, it is usually necessary to send
   the value of the token field on the LPWAN network.  The optimization will occur 
   by using small
   values.

Common CoAP implementations may generate large tokens, even if shorter tokens could 
be used regarding the LPWAN characteristics. A proxy may be needed to reduce 
the size of the token before compression. 

The size of the compress token sent is known by a combination of the Token Length field
and the rule entry. For instance, with the entry below:

~~~~~~
FID   FL FP DI  TV   MO       CDA    Sent
tkl   4  1  bi   2  equal   not-sent    
token 8  1  bi 0x00 MSB(12) LSB(4)   [4]
~~~~~~

The uncompressed token is 2 bytes long, but the compressed size will be 4 bits.

# CoAP options

##CoAP option Content-format field.

This field is unidirectional and must not be set to bidirectional in a rule entry.
It is used only by the server to inform the client about of the payload type and is 
never found in client requests.

If single value is expected by the client, the TV contains that value and MO is set to
"equal" and the CDF is set to "not-sent". The examples below describe the rules for an
ES acting as a server.

~~~~~~
FID     FL FP DI  TV    MO     CDA    Sent
content 16 1  up value equal not-sent
~~~~~~

If several possible value are expected by the client, a matching-list can be used.

~~~~~~
FID     FL FP DI   TV         MO           CDA       Sent
content 16 1  up [50, 41] match-mapping mapping-sent [1]
~~~~~~

Otherwise the value can be sent.The value-sent CDF in the compressor do not send the 
option type and the decompressor reconstruct it regarding the position in the rule.

~~~~~~
FID     FL FP DI   TV   MO     CDA       Sent
content 16 1  up      ignore value-sent [0-16]
~~~~~~

##CoAP option Accept field

This field is unidirectional and must not be set to bidirectional in a rule entry.
It is used only by the client to inform of the possible payload type and is never 
found in server response.

The number of accept options is not limited and can vary regarding the usage. To
be selected a rule must contain the exact number about accept options with their 
positions. Since the order in which the Accept value are sent, the position order 
can be modified. The rule below 

~~~~~~
FID    FL FP DI  TV   MO    CDA    Sent
accept 16 1  up  41  egal not-sent
accept 16 2  up  50  egal not-sent
~~~~~~~ 

will be selected only if two accept options are in the CoAP header if this order. 

The rule below:

~~~~~~
FID    FL FP DI  TV  MO     CDA    Sent
accept 16 0  up  41 egal  not-sent
accept 16 0  up  50 egal  not-sent
~~~~~~~ 

will accept a-only CoAP messages with 2 accept options, but the order will not influence
the rule selection. The decompression will reconstruct the header regarding the rule order.

Otherwise a matching-list can be applied to the different values, in that case the order 
is important to recover the appropriate value and the position must be clearly indicate.

~~~~~~ 
FID    FL FP DI    TV       MO             CDA     Sent
accept 16 1  up [50,41] match-mapping mapping-sent  [1]
accept 16 2  up [50,61] match-mapping mapping-sent  [1]
accept 16 3  up [61,71] match-mapping mapping-sent  [1]
~~~~~~

Finally, the option  can be  explicitly sent.

~~~~~~
FID    FL FP DI  TV    MO       CDA     Sent
accept    1  up      ignore  value-sent
~~~~~~


##CoAP option Max-Age field, CoAP option Uri-Host and Uri-Port fields

This field is unidirectional and must not be set to bidirectional in a rule entry.
It is used only by the server to inform of the caching duration and is never 
found in client
requests.

If the duration is known by both ends, value can be elided on the LPWAN.

A matching list can be used if some wellknown values are defined.

Otherwise the option length and value can be sent on the LPWAN. 


\[\[note: we can reduce (or create a new option) the unit to minute, 
second is small for LPWAN \]\]

#CoAP option Uri-Path and Uri-Query fields

This fields are unidirectional and must not be set to bidirectional in a rule entry.
They are used only by the client to access to a specific resource and are never found 
in server response.

The Matching Operator behavior has not changed, but the value must take a position value, 
if the entry is repeated :

~~~~
FID      FL FP DI  TV    MO      CDA    Sent
URI-Path    1  up  foo  equal  not-sent
URI-Path    2  up  bar  equal  not-sent
~~~~
{: #Fig-MOposition title='Position entry.'}

For instance, the  rule {{Fig-MOposition}} matches with /foo/bar, but not /bar/foo.

When the length is not clearly indicated in the rule, the value length must be sent with the
field data, which means for CoAP to send directly the CoAP option with length and value.

For instance for a CoMi path /c/X6?k="eth0" the rule can be set to:

~~~~~ 
FID       FL FP DI    TV     MO        CDA     Sent
URI-Path     1  up    c     equal    not-sent
URI-Path     2  up         ignore   value-sent 
URI-Query    1  up    k=   MSB (16)    LSB 
~~~~~
{: #Fig-CoMicompress title='CoMi URI compression'}

{{Fig-CoMicompress}} shows the parsing and the compression of the URI. where c is not sent.
The second element is sent with the length (i.e. 0x2 X 6) followed by the query option 
(i.e. 0x05 "eth0").

A Mapping list can be used to reduce size of variable Paths or Queries. In that case, to
optimize the compression, several elements can be regrouped into a single entry. 
Numbering of elements do not change, MO comparison is set with the first element 
of the matching.

~~~~~ 
FID       FL FP DI    TV         MO        CDA    Sent
URI-Path     1  up  {0:"/c/c",  equal   not-sent
                     1:"/c/d"
URI-Path     3  up             ignore   value-sent
URI-Query    1  up   k=       MSB (16)     LSB 
~~~~~
{: #Fig--complex-path title="complex path example"}

For instance, the following Path /foo/bar/variable/stable can leads to the rule defined 
{{Fig--complex-path}}.

 

##CoAP option Proxy-URI and Proxy-Scheme fields

These fields are unidirectional and must not be set to bidirectional in a rule entry.
They are used only by the client to access to a specific resource and are never found 
in server response.

If the field value must be sent, TV is not set, MO is set to "ignore" and CDF is set
to "value-sent. A mapping can also be used.

Otherwise the TV is set to the value, MO is set to "equal" and CDF is set to "not-sent"

## CoAP option ETag, If-Match, If-None-Match, Location-Path and Location-Query fields

These fields are unidirectional.

These fields values cannot be stored in a rule entry. They must always be sent with the
request. 

\[\[Can include OSCOAP Object security in that category \]\]

# Other RFCs

## Block

Block option should be avoided in LPWAN. The minimum size of 16 bytes can be incompatible
with some LPWAN technologies. 

\[\[Note: do we recommand LPWAN fragmentation since the smallest value of 16 is too big?\]\]

## Observe

{{rfc7641}} defines the Observe option. The TV is not set, MO is set to "ignore" and the
CDF is set to "value-sent". SCHC does not limit the maximum size for this option (3 bytes).
To reduce the transmission size either the Thing implementation should limit the value 
increase or a proxy can be used limit the increase.

Since RST message may be sent to inform a server that the client do not require Observe
response, a rule must allow the transmission of this message.

## No-Response

{{rfc7967}}  defines an No-Response option limiting the responses made by a server to 
a request. If the value is not by both ends, then TV is set to this value, MO is 
set to "equal" and CDF is set to "not-sent".

Otherwise, if the value is changing over time, TV is not set, MO is set to "ignore" and
CDF to "value-sent". A matching list can also be used to reduce the size. 

# Protocol analysis

# Examples of CoAP header compression

## Mandatory header with CON message

In this first scenario, the LPWAN compressor receives from outside client
a POST message, which is immediately acknowledged by the Thing. For this simple
scenario, the rules are described {{Fig-CoAP-header-1}}.


~~~~
 Rule ID 1
+-------------+--+--+--+------+---------+-------------++------------+
| Field       |FL|FP|DI|Target| Match   |     CDA     ||    Sent    |
|             |  |  |  |Value | Opera.  |             ||   [bits]   |
+-------------+--+--+--+------+---------+-------------++------------+
|CoAP version |  |  |bi|  01  |equal    |not-sent     ||            |
|CoAP version |  |  |bi| 01   |equal    |not-sent     ||            |
|CoAP Type    |  |  |bi|      |ignore   |value-sent   ||TT          |
|CoAP TKL     |  |  |bi| 0    |equal    |not-sent     ||            |
|CoAP Code    |  |  |bi| ML1  |match-map|matching-sent||  CC CCC    |
|CoAP MID     |  |  |bi| 0000 |MSB(7 )  |LSB(9)       ||        M-ID|
|CoAP Uri-Path|  |  |dw| path |equal 1  |not-sent     ||            |
+-------------+--+--+--+------+---------+-------------++------------+

~~~~
{: #Fig-CoAP-header-1 title='CoAP Context to compress header without token'}


The version  and Token Length fields are elided. Code has shrunk to 5 bits
using the matching list (as the one given {{Fig--example-code-mapping}}: 0.01 
is value 0x01 and 2.05 is value 0x0c) 
Message-ID has shrunk to 9 bits to preserve alignment on byte boundary. The
most significant bit must be set to 0 through a CoAP proxy. Uri-Path contains
a single element indicated in the matching operator.

{{Fig-CoAP-3}} shows the time diagram of the exchange. A LPWAN Application Server sends
a CON message. Compression reduces the header sending only the Type, a mapped
code and the least 9 significant bits of Message ID. The receiver decompresses
the header. .

The CON message is a request, therefore the LC process to a dynamic
   mapping.  When the ES receives the ACK message, this will not
   initiate locally a  message ID mapping since it is a response.
   The LC receives the ACK and uncompressed it to restore the original
   value.  Dynamic Mapping context lifetime follows the same rules as
   message ID duration.

~~~~
                  End System              LPWA LC
                       |                    |
                       |       rule id=1    |<--------------------
                       |<-------------------| +-+-+--+----+------+
  <------------------- | TTCC CCCM MMMM MMMM| |1|0| 4|0.01|0x0034|
 +-+-+--+----+-------+ | 0000 0010 0011 0100| |  0xb4   p   a   t|
 |1|0| 1|0.01|0x0034 | |                    | |  h   |
 |  0xb4   p   a   t | |                    | +------+
 |  h   |              |                    |     
 +------+              |                    |   
                       |                    |    
                       |                    |    
---------------------->|      rule id=1     |
+-+-+--+----+--------+ |------------------->|
|1|2| 0|2.05| 0x0034 | | TTCC CCCM MMMM MMMM|--------------------->
+-+-+--+----+--------+ | 1001 1000 0011 0100| +-+-+--+----+------+
                       |                    | |1|2| 0|2.05|0x0034|
                       v                    v +-+-+--+----+------+

~~~~
{: #Fig-CoAP-3 title='Compression with global addresses'}

The message can be further optimized by setting some fields unidirectional, as
described in {{Fig-CoAP-header-1-direc}}. Note that Type is no more sent in the
compressed format, Compressed Code size in not changed in that example (8 values 
are needed to code all the requests and 21 to code all the responses in the matching list 
 {{Fig--example-code-mapping}})

~~~~
 Rule ID 2
+-------------+--+--+--+------+---------+------------++------------+
| Field       |FL|FP|DI|Target|    MO   |     CDA    ||    Sent    |
|             |  |  |  |Value |         |            ||   [bits]   |
+-------------+--+--+--+------+---------+------------++------------+
|CoAP version |  |  |bi|01    |equal    |not-sent    ||            |
|CoAP Type    |  |  |dw|CON   |equal    |not-sent    ||            |
|CoAP Type    |  |  |up| ACK  |equal    |not-sent    ||            |
|CoAP TKL     |  |  |bi|0     |equal    |not-sent    ||            |
|CoAP Code    |  |  |dw|ML2   |match-map|mapping-sent||CCCC C      |
|CoAP Code    |  |  |up|ML3   |match-map|mapping-sent||CCCC C      |
|CoAP MID     |  |  |bi|0000  |MSB(5)   |LSB(11)     ||      M-ID  |
|CoAP Uri-Path|  |  |dw|path  |equal 1  |not-sent    ||            |
+-------------+--+--+--+------+---------+------------++------------+

ML1 = {CON : 0, ACK:1} ML2 = {POST:0, 2.04:1, 0.00:3}

~~~~
{: #Fig-CoAP-header-1-direc title='CoAP Context to compress header without token'}



##Complete exchange

In that example, the Thing is using CoMi and sends queries for 2 SID. 

~~~~

  CON 
  MID=0x0012     |                         |
  POST           |                         |   
  Accept X       |                         | 
  /c/k=AS        |------------------------>|
                 |                         |
                 |                         |
                 |<------------------------|  ACK MID=0x0012
                 |                         |  0.00
                 |                         |
                 |                         |
                 |<------------------------|   CON
                 |                         |   MID=0X0034
                 |                         |   Content-Format X
ACK MID=0x0034   |------------------------>|
0.00           

~~~~

~~~~
 Rule ID 3
+--------------+--+--+--+------+--------+-----------++------------+
| Field        |FL|FP|DI|Target|   MO   |     CDA   ||    Sent    |
|              |  |  |  |Value |        |           ||   [bits]   |
+--------------+--+--+--+------+--------+-----------++------------+ 
|CoAP version  |  |  |bi| 01   |equal   |not-sent   ||            |
|CoAP Type     |  |  |up| CON  |equal   |not-sent   ||            |
|CoAP Type     |  |  |dw| ACK  |equal   |not-sent   ||            |
|CoAP TKL      |  |  |bi| 1    |equal   |not-sent   ||            |
|CoAP Code     |  |  |up| POST |equal   |not-sent   ||            |
|CoAP Code     |  |  |dw| 0.00 |equal   |not-sent   ||            |
|CoAP MID      |  |  |bi| 0000 |MSB(8)  |LSB        ||MMMMMMMM    |
|CoAP Token    |  |  |up|      |ignore  |send-value ||TTTTTTTT    |
|CoAP Uri-Path |  |  |dw| /c   |equal 1 |not-sent   ||            |
|CoAP Uri-query|  |  |dw|  ML4 |equal 1 |not-sent   ||P           |
|CoAP Content  |  |  |up| X    |equal   |not-sent   ||            |
+--------------+--+--+--+------+--------+-----------++------------+

 Rule ID 4
+--------------+--+--+--+------+--------+-----------++------------+
| Field        |FL|FP|DI|Target|   MO   |     CDA   ||    Sent    |
|              |  |  |  |Value |        |           ||   [bits]   |
+--------------+--+--+--+------+--------+-----------++------------+ 
|CoAP version  |  |  |bi| 01   |equal    |not-sent  ||            |
|CoAP Type     |  |  |dw| CON  |equal    |not-sent  ||            |
|CoAP Type     |  |  |up| ACK  |equal    |not-sent  ||            |
|CoAP TKL      |  |  |bi| 1    |equal    |not-sent  ||            |
|CoAP Code     |  |  |dw| 2.05 |equal    |not-sent  ||            |
|CoAP Code     |  |  |up| 0.00 |equal    |not-sent  ||            |
|CoAP MID      |  |  |bi| 0000 |MSB(8)   |LSB       ||MMMMMMMM    |
|CoAP Token    |  |  |dw|      |ignore   |send-value||TTTTTTTT    |
|COAP Accept   |  |  |dw| X    |equal    |not-sent  ||            |
+--------------+--+--+--+------+---------+----------++------------+

alternative rule:

 Rule ID 4
+--------------+--+--+--+------+---------+-----------++------------+
| Field        |FL|FP|DI|Target|   MO    |     CDA   ||    Sent    |
|              |  |  |  |Value |         |           ||   [bits]   |
+--------------+--+--+--+------+---------+-----------++------------+ 
|CoAP version  |  |  |bi| 01   |equal    |not-sent   ||            |
|CoAP Type     |  |  |bi| ML1  |match-map|match-sent ||t           |
|CoAP TKL      |  |  |bi| 1    |equal    |not-sent   ||            |
|CoAP Code     |  |  |up| ML2  |match-map|match-sent || cc         |
|CoAP Code     |  |  |dw| ML3  |match-map|match-sent || cc         |
|CoAP MID      |  |  |bi| 0000 |MSB(8)   |LSB        ||MMMMMMMM    |
|CoAP Token    |  |  |dw|      |ignore   |send-value ||TTTTTTTT    |
|CoAP Uri-Path |  |  |dw| /c   |equal 1  |not-sent   ||            |
|CoAP Uri-query|  |  |dw| ML4  |equal 1  |not-sent   ||P           |
|CoAP Content  |  |  |up| X    |equal    |not-sent   ||            |
|COAP Accept   |  |  |dw| x    |equal    |not-sent   ||            |
+--------------+--+--+--+------+---------+-----------++------------+

ML1 {CON:0, ACK:1} ML2 {POST:0, 0.00: 1} ML3 {2.05:0, 0.00:1}
ML4 {NULL:0, k=AS:1, K=AZE:2}

~~~~







--- back
