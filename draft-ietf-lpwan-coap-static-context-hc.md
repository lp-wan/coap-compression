---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-coap-static-context-hc-01
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
title: LPWAN Static Context Header Compression (SCHC) for CoAP
abbrev: LPWAN CoAP compression
wg: lpwan Working Group
date: 2016-12
author:
- ins: A. Minaburo
  name: Ana Minaburo
  org: Acklio
  street: 2bis rue de la Chataigneraie
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ana@ackl.io
- ins: L. Toutain
  name: Laurent Toutain
  org: Institut MINES TELECOM ; TELECOM Bretagne
  street:
  - 2 rue de la Chataigneraie
  - CS 17607
  city: 35576 Cesson-Sevigne Cedex
  country: France
  email: Laurent.Toutain@telecom-bretagne.eu
normative:
  rfc4944:
  rfc3095:
  rfc4997:
  rfc5225:
  rfc6282:
  rfc7252:
  rfc7967:
  rfc1332:
  rfc7641:
  I-D.toutain-lpwan-ipv6-static-context-hc:
  I-D.ietf-core-comi:

--- abstract

This draft discusses the way SCHC header compression can be applied to CoAP 
headers in an LPWAN flow regarding the generated traffic.
CoAP protocol differs from IPv6 and UDP protocols because the CoAP Header 
has a flexible header due to variable options.  Another important difference is 
the asymmetric format in the header information used in the request and 
the response packets. This draft shows that the Client and the Server do not 
uses the same fields and how the SCHC header compression can be used.


--- middle

# Introduction {#Introduction}

{{I-D.toutain-lpwan-ipv6-static-context-hc}} defines a header compression
   mechanism for LPWAN network based on a static context. Where the context is
   said static since the element values composing the context are not
   learned during the packet exchanges but are previously defined.  The
   context(s) is(are) known by both ends before transmission. 
  
   A context is composed of a set of rules (contexts) that are referenced by Rule IDs 
   (identifiers).  A rule describes the header fields
   with some associated Target Values (TV). A Matching Operator (MO) is
   associated to each header field description.  The rule is selected if all the MOs fit
   the TVs.  In that case, a Compression Decompression Function (CDF)
   associated to each field defines the link between the compressed and
   decompressed value for each of the header fields.

   This draft discusses the way SCHC can be applied to CoAP headers, how to
   extend MOs to match a specific element when several fields of the same
   type are presented in the header.  It also introduces the notion of
   bidirectional or unidirectional (upstream and downstream) fields.


#  CoAP Compressing

CoAP {{RFC7252}} is an implementation of the REST architecture for constrained 
devices. Gateway
between CoAP and HTTP can be easily built since both protocols uses the same
address space (URL), caching mechanisms and methods.

Nevertheless, if limited, the size of a CoAP header may be
   too large for LPWAN constraints and some compression may be
   needed to reduce the header size.  CoAP compression is not
   straightforward. Some differences between IPv6/UDP and CoAP can be
   highlighted. CoAP differs from IPv6 and UDP protocols in the following   
   aspects:


* IPv6 and UDP are symmetrical protocols. The same fields are found in the
  request and in
  the response, only position in the header may change (e.g. source and destination
  fields). A CoAP request is different from  an response. For example, the URI-path
  option is mandatory in the request and is not found in the response. 

* CoAP also obeys to the client/server paradigm and the compression rate can
  be different if the request is issued from a LPWAN node or from an non LPWAN
  device. For instance a Thing (ES) aware of LPWAN constraints can generate a 1 byte token, but
  a regular CoAP cleint will certainly send a larger token to the Thing.

* In IPv6 and UDP header fields have a fixed size. In CoAP, Token size
  may vary from 0 to 8 bytes, length is given by a field in the header. More
  systematically, the CoAP options are described using the Type-Length-Value. 
  When applying SCHC header compression, the token size is not known at
      the rule creation, the sender and the receiver must agree on its
      compressed size.

* The options type in CoAP is not defined with the same value.  The
      Delta TLV coding makes that the type is not independent of
      previous option and may vary regarding the options contained in
      the header.


## CoAP behavior

A LPWAN node can either be a client or a server and sometimes both.
   In the client mode, the LPWAN node sends request to a server and
   expects an answer or acknowledgements.  Acknowledgements can be at 2
   different levels:

* In the transport level, a CON message is acknowledged by an ACK message.
      A NON confirmable message is not acknowledged at all.


* In REST level, a REST request is acknowledged by an "error" code.
      The {{RFC7967}} defines an option to limit the number of
      acknowledgements.



Note that acknowledgement can be optimized and a REST level acknowledgement
can be used as a transport level acknowledgement.


## CoAP protocol analysis

CoAP header format defines the following fields:

* version (2 bits): this field can be elided during the SCHC  compresssion

* type (2 bits). It defines the type of the transport messages, 4
      values are defined, regarding the type of exchange. If only NON
      messages are sent or CON/ACK messages, this field can be reduced
      to 0 or 1 bit.


* token length (4 bits).  The standard allows up to 8 bytes for a
      token.  If a fixed token size is chosen, then this field can
      be elided.  If some variation in length are needed then 1 or 2
      bits could be enough for most of LPWAN applications.


* code (8 bits).  This field codes the request and the response
      values. In CoAP these values are represented in a more compact way 
      then the coding used in
      HTTP, but the coding is not optimal.


* message id (16 bits). This value of this header field is used to acknowledge CON
  frames. The size of this field is computed to allow the anticipation
  (how many frames can be sent without acknowledgement). When a value
  is used, the {{RFC7252}} defines the time before it can be reused
  without ambiguities. This size defined may be too large for a LPWAN node
  sending or receiving few messages a day.

* Token (0 to 8 bytes). Token header field is used to identify active flows. 
  Regarding the usage for LPWAN 
  (stability  in time and limited number), a short token (1 Byte or less) can be
  enough.

* options are coded using delta-TLV. The delta-T depends on previous values,
  length is encoded inside the option. The {{RFC7252}} distinguishes
  repeatable options that can appear several times in the header.
  Among them we can distinguish:

  * list options which appear several time in the header but are exclusive such
    as the Accept option.

  * cumulative options which appear several times in the header but are part of
    a more generic value such as Uri-Path and Uri-Query. In that case, some elements
    may not change during the Thing lifetime and other may change at each request. 
    For instance CoMi {{I-D.ietf-core-comi}} defines the following path /c/X6?k="eth0", 
    where the first path element "c" does not change, the second element can vary
    over time with a different length (it represents the base64 enconding of a SID) and
    the query string can also vary over time. 

  For a given flow some value options are stable through time. Observe,
  ETag, If-Match, If-None-Match and Size varies in each message. 


The CoAP protocol must not be altered by the compression/decompression phase,
but if no semantic is attributed to a value, it may be changed during this
phase. For instance, the compression phase may reduce the size of a token
but must maintain its unicity. The decompressor will not be able to restore
the original value but the behavior will remain the same. If no special semantic
is assigned to the token, this will be transparent. If a special semantic
is assigned to the token, its compression may not be possible.


#SCHC rules for CoAP header compression

This draft refines the rules definition by adding the direction of the message, 
from the Thing point of view (uplink, downlink or bidirectional). It does not 
introduce new Machting Operator or new Compression Decompression Function, but add
some possibility to check one particular element when  several of them are
 present at the  same time.

A rule can contain CoAP and IPv6/UDP entries. In that case, IPv6/UDP entries are
tagged bidirectional.

## Directional Rules

By default, an entry in a rule is bidirectional which means that it can be applied
either on the uplink or downlink headers. By specifying the direction, the LC will
take into account  the specific  field only if the direction match. 

If the Thing is a client, the URI-Path option is only present on request and not 
on the response. Therefore, the exact matching principle to select a rule cannot apply.

Some options are marked unidirectional, the value (uplink or downlink) depends of the 
scenario. A Uri-Path option will be marked uplink if the Thing acts as a client and downlink
if the Thing acts as a server. If the Thing acts both as client and server, two different
rules will be defined.


## Matching Operator {#URI-Example}

The Matching Operator behavior has not changed, but the value must take a position value, 
if the entry is repeated :

~~~~
      FID          TV      MO             CDF
      
      URI-Path     foo     equal 1      not-sent
      URI-Path     bar     equal 2      not-sent
~~~~
{: #Fig-MOposition title='Position entry.'}

For instance, the  rule {{Fig-MOposition}} matches with /foo/bar, but not /bar/foo.

The position is added after the natural argument of the MO, for example MSB (4,3) 
indicates a most significant bit matching of 4 bits in a field located in position 3. 

##Compressed field length

When the length is not clearly indicated in the rule, the value length must be sent with the
field data, which means for CoAP to send directly the CoAP option where the delta-T is
set to 0. 

For the CoMi path  /c/X6?k="eth0" the rule can be set to:

~~~~~~ 
      FID          TV      MO             CDF
      
      URI-Path     c       equal 1      not-sent
      URI-Path             ignore 2     value-sent
      URI-Query    k=      MSB (16, 1)  value-sent 
      
~~~~~
{: #Fig-CoMicompress title='CoMi URI compression'}

{{Fig-CoMicompress}} shows the parsing and the compression of the URI. where c is not sent.
The second element is sent with the length (i.e. 0x02 X 6) followed by the query option 
(i.e. 0x08 k="eth0").

\[\[NOTE we don't process URI with a multiple number of path element ??\]\].  

# Application to CoAP header fields

   This section lists the different CoAP header fields and how they can
   be compressed.

## CoAP version field

This field is bidirectional.

   This field contains always the same value, therefore the TV may be 1, the MO
   is set to "equal" and the CDF is set to "not-sent"

## CoAP type field

This field is bidirectional or undirectional.

Several strategies can be applied to this field regarding the values used:

* if only one type is sent, for example NON message, its transmission can be avoided. 
  TV is set to the value, MO is set to "equal" and CDF is set to "not-sent".
  
* if two values are sent, for example CON and ACK and RST is not used, this field can 
  be reduced to one bit. TV is set to a matching value {CON: 0, ACK: 1}, MO is set
  to match-mapping and CDF is set to mapping-sent.
  
* It is also possible avoid transmission of this field by marking it unidirectional.
  In one direction, the TV is set to CON, MO is set to "equal" and CDF is set to "not-sent".
  On the other direction, the TV is set to ACK, the MO is set to "equal" and the CDF is 
  set to "not-sent".
  
* Otherwise TV is not set, MO is set to "ignore" and CDF is set to "value-sent".


##CoAP token length field

This field is bi-directional.

Several strategies can be applied to this field regarding the values:

* no token or a wellknown length, the transmission can be avoided. TV is set to the
  length, the MO is set to "equal" and CDF is set to "not-sent"
  
* The length is variable from one message to another. TV is not set, MO is set to 
  "ignore" and CDF is set to "value-sent". The size of the sent value  must be known by 
  ends. The  size may be 4 bits. The receiver must take into account this value to 
  retrieve the token. A CoAP proxy may be used before the compression to reduce the 
  field size.

  
##CoAP code field

This field is unidirectional. The client and the server do not use the 
same values.

The CoAP code field defines a tricky way to ensure compatibility with
HTTP values.  Nevertheless only 21 values are defined by {{RFC7252}}
compared to the 255 possible values.  So, it could efficiently be
coded on 5 bits. The number of code may vary over time, some new codes
may be introduced or some applications use a limited number of values. 

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

 The field can be treated differently in upstream than in downstream.
   If the Thing is a client an entry can be set on the uplink message with a
   code matching for 0.0X values and another for downlink values for Y.ZZ
   codes.  It is the opposite if the thing is a server.


 
## CoAP Message ID field

This field is bidirectional.

Message ID is used for two purposes:

* To acknowledge a CON message with an ACK.

* To avoid duplicate messages. 

In LPWAN, since a message can be received by several radio gateway, some 
LPWAN technologies include a sequence number in L2 to avoid duplicate frames. Therefore
if the message does not need to be acknowledged (NON or RST message), the Message
ID field can be avoided. In that case TV is not set, MO is set to ignore and CDF
is set to "not-sent". The decompressor can generate a number. 

\[\[Note; check id this field is not used by OSCOAP .\]\]

To optimize information sent on the LPWAN, shorter values may be used during the 
exchange, but Message ID values generated a common CoAP implementation will not take 
into account this limitation. Before the compression, a proxy may be needed to 
reduce the size. In that case, the TV is set to 0x0000, MO is set to "MSB(l)" 
and CDF is set to "LSB(16-l)", where "l" is the size of the compressed header.

Otherwise if no compression is needed the  TV is not set, 
MO is set to ignore and CDF is set to "not-sent". 

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

Otherwise the TV is not set, the MO is set to ignore and CDF is set to "value-sent". 

The decompression may know the length of the token field from the token length field.

##CoAP option Content-format field.

This field is unidirectional and must not be set to bidirectional in a rule entry.
It is used only by the server to inform the client about of the payload type and is never found in client
requests.

If the value is known by both sides, the TV contains that value and MO is set to
"equal" and the CDF is set to "not-sent".

Otherwise the TV is not set, MO is set to "ignore" and CDF is set to "value-sent"

A mapping list can also be used to reduce the size.

##CoAP option Accept field

This field is unidirectional and must not be set to bidirectional in a rule entry.
It is used only by the client to inform of the possible payload type and is never 
found in server response.

The number of accept options is not limited and can vary regarding the usage. To
be selected a rule must contain the exact number about accept options with their positions.

if the accept value must be sent, the TV contains that value, MO is set to "ignore x" 
where "x" is the accept option's position and CDF is set to value-sent. Since the value length
is not known, it must be sent as a CoAP TLV with delta-T set to 0. 

Otherwise the TV is not set, MO is set to "equal x" where x is the accept option's position
and CDF is set to "not-sent"

\[\[note: it could be more liberal and do not provide the same order after decompression\]\]


##CoAP option Max-Age field

This field is unidirectional and must not be set to bidirectional in a rule entry.
It is used only by the server to inform of the caching duration and is never found in client
requests.

If the duration is known by both ends, the TV is set with this duration, the MO is set
to "equal" and the CDF is set to "not-sent".

Otherwise the TV is not set, the MO is set to "ignore" and the CDF is set to "value-sent".
Since the value length is not known, it must be sent as a CoAP TLV with delta-T set to 0. 


\[\[note: we can reduce (or create a new option) the unit to minute, 
second is small for LPWAN \]\]

##CoAP option Uri-Host and Uri-Port fields

This fields are unidirectional and must not be set to bidirectional in a rule entry.
They are  used only by the client to access to a specific server and are never found 
in server response.

For each option, if the value is known by both ends, the TV is set with this value, 
the MO is set
to "equal" and the CDF is set to "not-sent".

Otherwise the TV is not set, the MO is set to "ignore" and the CDF is set to "value-sent".
Since the value length is not known, it must be sent as a CoAP TLV with delta-T set to 0. 


##CoAP option Uri-Path and Uri-Query fields

This fields are unidirectional and must not be set to bidirectional in a rule entry.
They are used only by the client to access to a specific resource and are never found 
in server response.

Path and Query option may have different formats. {{URI-Example}} gives some examples. 

If the URI path as well as the query is composed of 2 or more elements, then
the position must be set in the MO parameters. 

If a Path or Query element is stable over the time, then TV is set with its value, MO is
set to "equal x" where x is the position in the Path or the Query and CDF is set to "not-sent". 

Otherwise if the value varies over time, TV is not set, MO is set to "ignore x" 
where x is the position in the Path or in the Query and CDF is set to "value-sent". Since
the value length is not known, it must be sent as a CoAP TLV with deltaT set to 0.

A Mapping list can be used to reduce size of variable Paths or Queries. In that case, to
optimize the compression, several elements can be regrouped into a single entry. 
Numbering of elements do not change, MO comparison is set with the first element 
of the matching.

For instance, the following Path /foo/bar/variable/stable can leads to the rule defined 
{{Fig--complex-path}}.

~~~~
      FID        TV              MO                   CDF
      
      URI-Path   {"/foo/bar":1,  match-mapping 1      mapping-sent
                  "/bar/foo":2}  
      URI-Path                   ignore 3             value-sent
      URI-Path     stable        equal 4              not-sent
~~~~
{: #Fig--complex-path title="complex path example"}
 

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

{{RFC7641}} defines the Observe option. The TV is not set, MO is set to "ignore" and the
CDF is set to "value-sent". SCHC does not limit the maximum size for this option (3 bytes).
To reduce the transmission size either the Thing implementation should limit the value 
increase or a proxy can be used limit the increase.

Since RST message may be sent to inform a server that the client do not require Observe
response, a rule must allow the transmission of this message.

## No-Response

{{RFC7967}}  defines an No-Response option limiting the responses made by a server to 
a request. If the value is not by both ends, then TV is set to this value, MO is 
set to "equal" and CDF is set to "not-sent".

Otherwise, if the value is changing over time, TV is not set, MO is set to "ignore" and
CDF to "value-sent". A matching list can also be used to reduce the size. 


# Examples of CoAP header compression

## Mandatory header with CON message

In this first scenario, the LPWAN compressor receives from outside client
a POST message, which is immediately acknowledged by the Thing. For this simple
scenario, the rules are described {{Fig-CoAP-header-1}}.


~~~~
 rule id 1
+-------------+------+---------+-------------+-----+----------------+
| Field       |TV    |MO       |CDF          |dir  | Sent           |
+=============+======+=========+=============+=====+================+
|CoAP version | 01   |equal    |not-sent     |bi   |                |
|CoAP Type    |      |ignore   |value-sent   |bi   |TT              |
|CoAP TKL     | 0    |equal    |not-sent     |bi   |                |
|CoAP Code    | ML1  |match-map|matching-sent|bi   |  CC CCC        |
|CoAP MID     | 0000 |MSB(7 )  |LSB(9)       |bi   |         M-ID   |
|CoAP Uri-Path| path |equal 1  |not-sent     |down |                |
+-------------+------+---------+-------------+-----+----------------+
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
                   End System               LPWA LC
                        |                     |
                        |        rule id=1    |<----------------------
                        |<--------------------| +-+-+--+----+--------+
  <-------------------- |  TTCC CCCM MMMM MMMM| |1|0| 4|0.01| 0x0034 |
 +-+-+--+----+--------+ |  0000 0010 0011 0100| |  0xb4   p    a   t |
 |1|0| 1|0.01| 0x0034 | |                     | |  h   |
 |  0xb4   p    a   t | |                     | +------+
 |  h   |               |                     |     
 +------+               |                     |   
                        |                     |    
                        |                     |    
----------------------->|       rule id=1     |
+-+-+--+----+--------+  |-------------------->|
|1|2| 0|2.05| 0x0034 |  |  TTCC CCCM MMMM MMMM|------------------------>
+-+-+--+----+--------+  |  1001 1000 0011 0100| +-+-+--+----+--------+
                        |                     | |1|2| 0|2.05| 0x0034 |
                        v                     v +-+-+--+----+--------+

~~~~
{: #Fig-CoAP-3 title='Compression with global addresses'}

The message can be further optimized by setting some fields unidirectional, as
described in {{Fig-CoAP-header-1-direc}}. Note that Type is no more sent in the
compressed format, Compressed Code size in not changed in that example (8 values 
are needed to code all the requests and 21 to code all the responses in the matching list 
 {{Fig--example-code-mapping}})

~~~~
 rule id 1
+-------------+------+---------+-------------+---+----------------+
| Field       |TV    |MO       |CDF          |dir| Sent           |
+=============+======+=========+=============+===+================+
|CoAP version | 01   |equal    |not-sent     |bi |                |
|CoAP Type    | CON  |equal    |not-sent     |dw |                |
|CoAP Type    | ACK  |equal    |not-sent     |up |                |
|CoAP TKL     | 0    |equal    |not-sent     |bi |                |
|CoAP Code    | ML2  |match-map|matching-sent|dw |CCCC C          |
|CoAP Code    | ML3  |match-map|matching-sent|up |CCCC C          |
|CoAP MID     | 0000 |MSB(5)   |LSB(11)      |bi |       M-ID     |
|CoAP Uri-Path| path |equal 1  |not-sent     |dw |                |
+-------------+------+---------+-------------+---+----------------+

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
 rule id 3
+-------------+------+---------+-------------+---+----------------+
| Field       |TV    |MO       |CDF          |dir| Sent           |
+=============+======+=========+=============+===+================+
|CoAP version | 01   |equal    |not-sent     |bi |                |
|CoAP Type    | CON  |equal    |not-sent     |up |                |
|CoAP Type    | ACK  |equal    |not-sent     |dw |                |
|CoAP TKL     | 1    |equal    |not-sent     |bi |                |
|CoAP Code    | POST |equal    |not-sent     |up |                |
|CoAP Code    | 0.00 |equal    |not-sent     |dw |                |
|CoAP MID     | 0000 |MSB(8)   |LSB(8)       |bi |MMMMMMMM        |
|CoAP Token   |      |ignore   |send-value   |up |TTTTTTTT        |
|CoAP Uri-Path| /c   |equal 1  |not-sent     |dw |                |
|CoAP Uri-query ML4  |equal 1  |not-sent     |dw |P               |
|CoAP Content | X    |equal    |not-sent     |up |                |
+-------------+------+---------+-------------+---+----------------+

 rule id 4
+-------------+------+---------+-------------+---+----------------+
| Field       |TV    |MO       |CDF          |dir| Sent           |
+=============+======+=========+=============+===+================+
|CoAP version | 01   |equal    |not-sent     |bi |                |
|CoAP Type    | CON  |equal    |not-sent     |dw |                |
|CoAP Type    | ACK  |equal    |not-sent     |up |                |
|CoAP TKL     | 1    |equal    |not-sent     |bi |                |
|CoAP Code    | 2.05 |equal    |not-sent     |dw |                |
|CoAP Code    | 0.00 |equal    |not-sent     |up |                |
|CoAP MID     | 0000 |MSB(8)   |LSB(8)       |bi |MMMMMMMM        |
|CoAP Token   |      |ignore   |send-value   |dw |TTTTTTTT        |
|COAP Accept  | X    |equal    |not-sent     |dw |                |
+-------------+------+---------+-------------+---+----------------+


alternative rule:

 rule id 4
+-------------+------+---------+-------------+---+----------------+
| Field       |TV    |MO       |CDF          |dir| Sent           |
+=============+======+=========+=============+===+================+
|CoAP version | 01   |equal    |not-sent     |bi |                |
|CoAP Type    | ML1  |equal    |match-sent(1)|bi |t               |
|CoAP TKL     | 1    |equal    |not-sent     |bi |                |
|CoAP Code    | ML2  |equal    |match-sent(1)|up | cc             |
|CoAP Code    | ML3  |equal    |match-sent(2)|dw | cc             |
|CoAP MID     | 0000 |MSB(8)   |LSB(8)       |bi |MMMMMMMM        |
|CoAP Token   |      |ignore   |send-value   |dw |TTTTTTTT        |
|CoAP Uri-Path| /c   |equal 1  |not-sent     |dw |                |
|CoAP Uri-query ML4  |equal 1  |not-sent     |dw |P               |
|CoAP Content | X    |equal    |not-sent     |up |                |
|COAP Accept  | x    |equal    |not-sent     |dw |                |
+-------------+------+---------+-------------+---+----------------+

ML1 {CON:0, ACK:1} ML2 {POST:0, 0.00: 1} ML3 {2.05:0, 0.00:1}
ML4 {NULL:0, k=AS:1, K=AZE:2}

~~~~







--- back