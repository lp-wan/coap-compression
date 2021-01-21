---
stand_alone: true
ipr: trust200902
docname: draft-ietf-lpwan-coap-static-context-hc-18
cat: std
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
- ins: R. Andreasen
  name: Ricardo Andreasen
  org: Universidad de Buenos Aires
  street: Av. Paseo Colon 850
  city: C1063ACV Ciudad Autonoma de Buenos Aires
  country: Argentina
  email: randreasen@fi.uba.ar
normative:  
  RFC2119:
  RFC5116:
  RFC7252:
  RFC7967:
  RFC7641:
  RFC7959:
  RFC8174:
  RFC8613:
  RFC8724: 
  


--- abstract

   This draft defines how to compress the Constrained Application Protocol (CoAP) using the Static Context Header Compression (SCHC). 
   SCHC is a header compression mechanism adapted for Constrained Devices. 
   SCHC uses a static description of the header to reduce the header's redundancy and size.
   While RFC 8724 describes the SCHC compression and fragmentation framework,
   and its application for IPv6/UDP headers, this document applies SCHC for CoAP headers. The CoAP header structure differs from 
   IPv6 and UDP since CoAP uses a flexible header with a variable number of options, themselves of variable length. The CoAP protocol 
   messages format is asymmetric: the request messages have a header format different from the one in the response messages. This
   specification gives guidance on applying SCHC to flexible headers and how to leverage the asymmetry for more efficient compression Rules.



--- middle

# Introduction {#Introduction}

CoAP {{RFC7252}} is a command/response protocol designed for micro-controllers with a small RAM and ROM and optimized for REST-based (Representative state transfer) services. Although the Constrained Devices leads the CoAP design, a CoAP header's size is still too large for LPWAN (Low Power Wide Area Networks).
SCHC header compression over CoAP header is required to increase performance or use CoAP over LPWAN technologies.

The {{RFC8724}} defines SCHC, a header compression mechanism for the LPWAN network based on a static context. Section 5 of the {{RFC8724}} explains where compression and decompression occur in the architecture. The SCHC compression scheme assumes as a prerequisite that both end-points know the static context before transmission. The way the context is configured, provisioned, or exchanged is out of this document's scope.

CoAP is an application protocol, so CoAP compression requires installing common Rules between the two SCHC instances. SCHC compression may apply at two different levels: at IP and UDP in the LPWAN network and another at the application level for CoAP. These two compressions may be independent. Both follow the same principle described in {{RFC8724}}. As different entities manage the CoAP compression at different levels, the SCHC Rules driving the compression/decompression are also different. The {{RFC8724}} describes how to use SCHC for IP and UDP headers.	This document specifies how to apply SCHC compression to CoAP headers.

SCHC compresses and decompresses headers based on common contexts between Devices. SCHC context includes multiple Rules. Each Rule can match the header fields to specific values or ranges of values. If a Rule matches, the matched header fields are replaced by the RuleID and the Compression Residue that contains the residual bits of the compression. Thus, different Rules may correspond to different protocol headers in the packet that a Device expects to send or receive.

A Rule describes the packets' entire header with an ordered list of fields descriptions; see section 7 of {{RFC8724}}. Thereby each description contains the field ID (FID), its length (FL), and its position (FP), a direction indicator (DI) (upstream, downstream, and bidirectional), and some associated Target Values (TV). The direction indicator is used for compression to give the best TV to the FID when these values differ in the transmission direction. So a field may be described several times.

A Matching Operator (MO) is associated with each header field description. The Rule is selected if all the MOs fit the TVs for all fields of the incoming header.
A Rule cannot be selected if the message contains an unknown field to the SCHC compressor.

In that case, a Compression/Decompression Action (CDA) associated with each field gives the method to compress and decompress each field. 
Compression mainly results in one of 4 actions:

*  send the field value (value-sent), 
*  send nothing (not-sent), 
*  send some least significant bits of the field (LSB) or, 
*  send an index (mapping-sent).

After applying the compression, there may be some bits to be sent.
These values are called Compression Residue.


SCHC is a general mechanism applied to different protocols, the exact Rules to be used depending on the protocol and the Application.  
Section 10 of the {{RFC8724}} describes the compression scheme for IPv6 and UDP headers. This document targets the CoAP header compression using SCHC.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP 14
{{RFC2119}}{{RFC8174}} when, and only when, they
appear in all capitals, as shown here.

# SCHC Applicability to CoAP 
 
SCHC Compression for CoAP header MAY be done in conjunction with the lower layers (IPv6/UDP) or independently. The SCHC adaptation layers, described in Section 5 of {{RFC8724}}, may be used as shown in {{Fig-SCHCCOAP1}}, {{Fig-SCHCCOAP2}}, and {{Fig-SCHCCOAP3}}.

In the first example, {{Fig-SCHCCOAP1}}, a Rule compresses the complete header stack from IPv6 to CoAP.  In this case, the Device and the NGW perform SCHC C/D (Static Context Header Compression Compressor/Decompressor).  The Application communicating with the Device does not implement SCHC C/D.


~~~~

      (Device)            (NGW)                              (App)

      +--------+                                           +--------+
      |  CoAP  |                                           |  CoAP  |
      +--------+                                           +--------+
      |  UDP   |                                           |  UDP   |
      +--------+     +----------------+                    +--------+
      |  IPv6  |     |      IPv6      |                    |  IPv6  |
      +--------+     +--------+-------+                    +--------+
      |  SCHC  |     |  SCHC  |       |                    |        |
      +--------+     +--------+       +                    +        +
      |  LPWAN |     | LPWAN  |       |                    |        |
      +--------+     +--------+-------+                    +--------+
          ((((LPWAN))))             ------   Internet  ------
~~~~
{: #Fig-SCHCCOAP1 title='Compression/Decompression at the LPWAN boundary.'}  

{{Fig-SCHCCOAP1}} shows the use of SCHC header compression above layer 2 in the Device and the NGW. The SCHC layer receives non-encrypted packets and can apply compression Rules to all the headers in the stack. On the other end, the NGW receives the SCHC packet and reconstructs the headers using the Rule and the Compression Residue. After the decompression, the NGW forwards the IPv6 packet toward the destination. The same process applies in the other direction when a non-encrypted packet arrives at the NGW. Thanks to the IP forwarding based on the IPv6 prefix, the NGW identifies the Device and compresses headers using the Device's Rules. 

In the second example, {{Fig-SCHCCOAP2}}, the SCHC compression is applied in the CoAP layer, compressing the CoAP header independently of the other layers. The RuleID, the Compression Residue, and CoAP payload are encrypted using a mechanism such as DTLS. Only the other end (App) can decipher the information. If needed, layers below use SCHC to compress the header as defined in {{RFC8724}} (represented in dotted lines).

This use case needs an end-to-end context initialization between the Device and the Application. The context initialization is out of the scope of this document.


~~~~
      (Device)            (NGW)                               (App)

      +--------+                                           +--------+
      |  CoAP  |                                           |  CoAP  |
      +--------+                                           +--------+
      |  SCHC  |                                           |  SCHC  |
      +--------+                                           +--------+
      |  DTLS  |                                           |  DTLS  |
      +--------+                                           +--------+
      .  udp   .                                           .  udp   .
      ..........     ..................                    ..........
      .  ipv6  .     .      ipv6      .                    .  ipv6  .
      ..........     ..................                    ..........
      .  schc  .     .  schc  .       .                    .        .
      ..........     ..........       .                    .        .
      .  lpwan .     . lpwan  .       .                    .        .
      ..........     ..................                    ..........
          ((((LPWAN))))             ------   Internet  ------

~~~~
{: #Fig-SCHCCOAP2 title='Standalone CoAP end-to-end Compression/Decompression'} 

The third example, {{Fig-SCHCCOAP3}}, shows the use of Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}}. In this case, SCHC needs two Rules to compress the CoAP header. A first Rule focused on the inner header. The result of this first compression is encrypted using the OSCORE mechanism. Then a second Rule compresses the outer header, including the OSCORE Options.


~~~~

      (Device)            (NGW)                              (App)

      +--------+                                           +--------+
      |  CoAP  |                                           |  CoAP  |
      |  inner |                                           |  inner |
      +--------+                                           +--------+
      |  SCHC  |                                           |  SCHC  |
      |  inner |                                           |  inner |
      +--------+                                           +--------+
      |  CoAP  |                                           |  CoAP  |
      |  outer |                                           |  outer |
      +--------+                                           +--------+
      |  SCHC  |                                           |  SCHC  |
      |  outer |                                           |  outer |
      +--------+                                           +--------+
      .  udp   .                                           .  udp   .
      ..........     ..................                    ..........
      .  ipv6  .     .      ipv6      .                    .  ipv6  .
      ..........     ..................                    ..........
      .  schc  .     .  schc  .       .                    .        .
      ..........     ..........       .                    .        .
      .  lpwan .     . lpwan  .       .                    .        .
      ..........     ..................                    ..........  
          ((((LPWAN))))             ------   Internet  ------


~~~~
{: #Fig-SCHCCOAP3 title='OSCORE compression/decompression.'} 

In the case of several SCHC instances, as shown in {{Fig-SCHCCOAP2}} and {{Fig-SCHCCOAP3}}, the Rules may come from different provisioning domains.

This document focuses on CoAP compression represented in the dashed boxes in the previous figures.



#  CoAP Headers compressed with SCHC

The use of SCHC over the CoAP header uses the same description, and compression/decompression techniques like the one for IP and UDP explained in the {{RFC8724}}. For CoAP, the SCHC Rules description uses the direction information to optimize the compression by reducing the number of Rules needed to compress headers. The field description MAY define both request/response headers and target values in the same Rule, using the DI (direction indicator) to make the difference.

As for other header compression protocols, when the compressor does not find a correct Rule to compress the header, the packet MUST be sent uncompressed using the RuleID dedicated to this purpose. Where the Compression Residue is the complete header of the packet. See section 6 of {{RFC8724}}.

## Differences between CoAP and UDP/IP Compression

CoAP compression differs from IPv6 and UDP compression in the following aspects: 
   
* The CoAP protocol is asymmetric; the headers are different for a request or a response. 
  For example, the URI-Path option is mandatory in the request, and it might not be present in the response. 
  A request might contain an Accept option, and the response might include a Content-Format option. 
  In comparison, IPv6 and UDP returning path swap the value of some fields in the header. 
  However, all the directions have the same fields (e.g., source and destination address fields).

  The {{RFC8724}} defines the use of a direction indicator (DI) in the
  Field Descriptor, which allows a single Rule to process a message
  header differently depending on the direction.

* Even when a field is "symmetric" (i.e., found in both directions), the values carried in each direction are different.
  The compression may use a "match-mapping" MO to limit the range of expected values 
  in a particular direction and reduce the Compression Residue's size.
  Through the direction indicator (DI), a field description in the Rules splits the possible field value into two parts,
  one for each direction. For instance, if a client sends only CON requests, the Type can be elided by compression,
  and the answer may use one single bit to carry either the ACK or RST type.
  The field Code has the same behavior, the 0.0X code format value in the request, and the Y.ZZ code format in the response. 

* In SCHC, the Rule defines the different header fields' length, so SCHC does not need to send it.
  In IPv6 and UDP headers, the fields have a fixed size, known by definition.
  On the other hand, some CoAP header fields have variable lengths, and the Rule description specifies it.
  For example, in a URI-path or URI-query, the Token size may vary from 0 to 8 bytes,
  and the CoAP options use the Type-Length-Value encoding format.

  When doing SCHC compression of a variable-length field,
  Section 7.5.2 from {{RFC8724}} offers the possibility to define a function for the Field length in the Field Description
  to know the length before compression. If the field length is unknown, the Rule will set it as a variable,
  and SCHC will send the compressed field's length in the Compression Residue. 

* A field can appear several times in the CoAP headers.
  It is found typically for elements of a URI (path or queries).
  The SCHC specification {{RFC8724}} allows a Field ID to appear several times in the Rule
  and uses the Field Position (FP) to identify the correct instance, thereby removing the matching operation's ambiguity.

* Field lengths defined in the CoAP protocol can be too large regarding LPWAN traffic constraints.
  For instance, this is particularly true for the Message-ID field and the Token field.
  SCHC uses different Matching operators (MO) to perform the compression. See section 7.4 of {{RFC8724}}.
  In this case, SCHC can apply the Most Significant Bits (MSB) MO to reduce the information carried on LPWANs.

# Compression of CoAP header fields
{: #CoAPcomp}

This section discusses the compression of the different CoAP header fields. The CoAP compression with SCHC follows Section 7.1 of {{RFC8724}}. 

## CoAP version field

CoAP version is bidirectional and MUST be elided during the SCHC compression since it always contains the same value.
In the future, or if a new version of CoAP is defined, new Rules will be needed to avoid ambiguities between versions.

## CoAP type field

The CoAP protocol {{RFC7252}} has four types of messages: two requests (CON, NON), one response (ACK), and one empty message (RST).

The SCHC compression SHOULD elide this field if, for instance, a client is sending only NON or only CON messages. 
For the RST message, SCHC may use a dedicated Rule. For other usages, SCHC can use a "match-mapping" MO.
  
## CoAP code field

The code field is an IANA registry {{RFC7252}}, and it indicates the Request Method used in CoAP. The compression of the CoAP code field follows the same principle as that of the CoAP type field. If the Device plays a specific role, SCHC may split the code values into two fields description, the request codes with the 0 class and the response values. SCHC will use the direction indicator to identify the correct value in the packet.

If the Device only implements a CoAP client, SCHC compression may reduce the request code to the set of requests the client can process.

For known values, SCHC can use a "match-mapping" MO. If SCHC cannot compress the code field, it will send the values in the Compression Residue.
 
## CoAP Message ID field

SCHC can compress the Message ID field with the "MSB" MO and the "LSB" CDA. See section 7.4 of {{RFC8724}}. 

## CoAP Token fields
CoAP defines the Token using two CoAP fields, Token Length in the mandatory header and Token Value directly following the mandatory CoAP header.

SCHC processes the Token length as any header field. If the value does not change, the size can be stored in the TV and elided during the transmission. Otherwise, SCHC will send the token length in the Compression Residue.

For the Token Value, SCHC MUST NOT send it as a variable-length in the Compression Residue to avoid ambiguity with Token Length. Therefore, SCHC MUST use the Token length value to define the size of the Compression Residue. SCHC designates a specific function "tkl" that the Rule MUST use to complete the field description. During the decompression, this function returns the value contained in the Token Length field.

# CoAP options
CoAP defines options placed after the based header in Option Numbers order; see {{RFC7252}}. Each Option instance in a message uses
the format Delta-Type (D-T), Length (L), Value (V). The SCHC Rule builds the description of the option by using in the Field ID the Option Number built from D-T; in TV, the Option Value; and the Option Length uses section 7.4 of {{RFC8724}}. When the Option Length has a well-known size, the Rule may stock the length value. Therefore, SCHC compression does not send it. Otherwise, SCHC Compression carries the length of the Compression Residue, in addition to the Compression Residue value.

CoAP requests and responses do not include the same options. So Compression Rules may reflect this asymmetry by tagging the direction indicator.

Note that length coding differs between CoAP options and SCHC variable size Compression Residue.

The following sections present how SCHC compresses some specific CoAP options.

If CoAP introduces a new option, the SCHC Rules MAY be updated, and the new Field ID description MUST be assigned to allow its compression.
Otherwise, if no Rule describes this new option, the SCHC compression is not achieved, and SCHC sends the CoAP header without compression. 

## CoAP Content and Accept options.

If the client expects a single value, it can be stored in the TV and elided during the transmission. 
Otherwise, if the client expects several possible values, a "match-mapping" SHOULD be used to limit the Compression Residue's size. 
If not, SCHC has to send the option value in the Compression Residue (fixed or variable length).


## CoAP option Max-Age, Uri-Host, and Uri-Port fields

SCHC compresses these three fields in the same way. When the value of these options is known, SCHC can elide these fields.
If the option uses well-known values, SCHC can use a "match-mapping" MO. Otherwise, SCHC will use "value-sent" MO, and the Compression Residue will send these options' values.

## CoAP option Uri-Path and Uri-Query fields

The Uri-Path and Uri-Query fields are repeatable options; this means that in the CoAP header, they may appear several times with different values. SCHC Rule description uses the Field Position (FP) to distinguish the different instances in the path. 

To compress repeatable field values, SCHC may use a "match-mapping" MO to reduce the size of variable Paths or Queries. In these cases, to optimize the compression, several elements can be regrouped into a single entry. The Numbering of elements does not change, and the first matching element sets the MO comparison.

~~~~~ 
   +--------+---+--+--+--------+-------------+------------+
   | Field  |FL |FP|DI| Target |  Matching   |     CDA    |
   |        |   |  |  | Value  |  Operator   |            |
   +--------+---+--+--+--------+-------------+------------+
   |Uri-Path|   | 1|up|["/a/b",|match-mapping|mapping-sent|
   |        |   |  |  |"/c/d"] |             |            |
   |Uri-Path|var| 3|up|        |ignore       |value-sent  |
   +--------+---+--+--+--------+-------------+------------+


~~~~~
{: #Fig--complex-path title="complex path example"}

In {{Fig--complex-path}}, SCHC can use a single bit in the Compression Residue to code one of the two paths. 
If regrouping were not allowed, 2 bits in the Compression Residue would be needed. SCHC sends the third path element as a variable size in the Compression Residue.


### Variable-length Uri-Path and Uri-Query

When SCHC creates the Rule, the length of URI-Path and URI-Query may be known. Nevertheless, SCHC MUST set the field length to variable, 
and the unit to bytes. 

SCHC compression can use the MSB MO to a Uri-Path or Uri-Query element. However, attention to the length is important because the MSB value is in bits, and the size MUST always be a multiple of 8 bits.

The length sent at the beginning of a variable-length Compression Residue indicates the LSB's size in bytes. 

For instance, for a CORECONF path /c/X6?k="eth0" the Rule description can be:

~~~~~ 
   +-------------+---+--+--+--------+---------+-------------+
   | Field       |FL |FP|DI| Target | Match   |     CDA     |
   |             |   |  |  | Value  | Opera.  |             |
   +-------------+---+--+--+--------+---------+-------------+
   |Uri-Path     |  8| 1|up|"c"     |equal    |not-sent     |
   |Uri-Path     |var| 2|up|        |ignore   |value-sent   |
   |Uri-Query    |var| 1|up|"k="    |MSB(16)  |LSB          |
   +-------------+---+--+--+--------+---------+-------------+
~~~~~
{: #Fig-CoMicompress title='CORECONF URI compression'}

{{Fig-CoMicompress}} shows the Rule description for a URI-Path and a URI-Query. SCHC compresses the first part of the URI-Path with a "not-sent" CDA.
SCHC will send the second element of the URI-Path with the length (i.e., 0x2 X 6) followed by the query option (i.e., 0x05 "eth0").


### Variable number of Path or Query elements

SCHC fixed the number of Uri-Path or Uri-Query elements in a Rule at the Rule creation time.
If the number varies, SCHC SHOULD create several Rules to cover all the possibilities.
Another one is to define the length of Uri-Path to variable and sends a Compression Residue with a length of 0 to indicate that this Uri-Path is empty. However, this adds 4 bits to the variable Compression Residue size. See section 7.5.2 {{RFC8724}}.

## CoAP option Size1, Size2, Proxy-URI and Proxy-Scheme fields

The SCHC Rule description MAY define sending some field values by setting the TV to "not-sent," MO to "ignore," and CDA to "value-sent." A Rule MAY also use a "match-mapping" when there are different options for the same FID. Otherwise, the Rule sets the TV to the value, MO to "equal," and CDA to "not-sent."


## CoAP option ETag, If-Match, If-None-Match, Location-Path, and Location-Query fields

A Rule entry cannot store these fields' values. The Rule description MUST always send these values in the Compression Residue. 


# SCHC compression of CoAP extension RFCs

## Block

When a packet uses a Block {{RFC7959}} option, SCHC compression MUST send its content in the Compression Residue. 
The SCHC Rule describes an empty TV with a MO set to "ignore" and a CDA to "value-sent."
Block option allows fragmentation at the CoAP level that is compatible with SCHC fragmentation.
Both fragmentation mechanisms are complementary, and the node may use them for the same packet as needed. 

## Observe

The {{RFC7641}} defines the Observe option. The SCHC Rule description will not define the TV, but MO to "ignore," and the CDA to "value-sent." SCHC does not limit the maximum size for this option (3 bytes). To reduce the transmission size, either the Device implementation MAY limit the delta between two consecutive values, or a proxy can modify the increment.

Since the Observe option MAY use an RST message to inform a server that the client does not require the Observe response, a specific SCHC Rule SHOULD exist to allow the message's compression with the RST type.


## No-Response

The {{RFC7967}} defines a No-Response. Different behaviors exist while using this option to limit the responses made by a server to a request. If both ends know the value, then the SCHC Rule will describe a TV to this value, with a MO set to "equal" and CDA set to "not-sent."

Otherwise, if the value is changing over time, the SCHC Rule will set the MO to "ignore" and CDA to "value-sent." The Rule may also use a "match-mapping" to compress this option. 

## OSCORE
{: #Sec-OSCORE}

OSCORE {{RFC8613}} defines end-to-end protection for CoAP messages.
This section describes how SCHC Rules can be applied to compress OSCORE-protected messages.

~~~~

      0 1 2 3 4 5 6 7 <--------- n bytes ------------->
     +-+-+-+-+-+-+-+-+---------------------------------
     |0 0 0|h|k|  n  |      Partial IV (if any) ...    
     +-+-+-+-+-+-+-+-+---------------------------------
     |               |                                |
     |<--  CoAP   -->|<------ CoAP OSCORE_piv ------> |
        OSCORE_flags 

      <- 1 byte -> <------ s bytes ----->
     +------------+----------------------+-----------------------+
     | s (if any) | kid context (if any) | kid (if any)      ... |
     +------------+----------------------+-----------------------+
     |                                   |                       |
     | <------ CoAP OSCORE_kidctx ------>|<-- CoAP OSCORE_kid -->|

~~~~
{: #Fig-OSCORE-Option title='OSCORE Option'} 

The {{Fig-OSCORE-Option}} shows the OSCORE Option Value encoding defined in Section 6.1 of {{RFC8613}}, where the first byte specifies the Content of the OSCORE options using flags. The three most significant bits of this byte are reserved and always set to 0. Bit h, when set, indicates the presence of the kid context field in the option. Bit k, when set, indicates the presence of a kid field. The three least significant bits n indicate the length of the piv (Partial Initialization Vector) field in bytes. When n = 0, no piv is present.

The flag byte is followed by the piv field, kid context field, and kid field in this order, and if present, 
the kid context field's length is encoded in the first byte denoting by 's' the length of the kid context in bytes.

To better perform OSCORE SCHC compression, the Rule description needs to identify the OSCORE Option and the fields it contains. Conceptually, it discerns up to 4 distinct pieces of information within the OSCORE option: the flag bits, the piv, the kid context, and the kid.  The SCHC Rule splits into four field descriptions the OSCORE option to compress them:


*  CoAP OSCORE_flags,
*  CoAP OSCORE_piv,
*  CoAP OSCORE_kidctx,
*  CoAP OSCORE_kid.

{{Fig-OSCORE-Option}} shows the OSCORE Option format with those four fields superimposed on it.
Note that the CoAP OSCORE_kidctx field directly includes the size octet s. 

# Examples of CoAP header compression

## Mandatory header with CON message

In this first scenario, the SCHC Compressor at the Network Gateway side 
receives a POST message from an Internet client, which is immediately acknowledged by the Device. 
{{Fig-CoAP-header-1}} describes the SCHC Rule descriptions for this scenario.


~~~~
RuleID 1
+-------------+--+--+--+------+---------+-------------++------------+
| Field       |FL|FP|DI|Target| Match   |     CDA     ||    Sent    |
|             |  |  |  |Value | Opera.  |             ||   [bits]   |
+-------------+--+--+--+------+---------+-------------++------------+
|CoAP version | 2| 1|bi|  01  |equal    |not-sent     ||            |
|CoAP Type    | 2| 1|dw| CON  |equal    |not-sent     ||            |
|CoAP Type    | 2| 1|up|[ACK, |         |             ||            |
|             |  |  |  | RST] |match-map|matching-sent|| T          |
|CoAP TKL     | 4| 1|bi| 0    |equal    |not-sent     ||            |
|CoAP Code    | 8| 1|bi|[0.00,|         |             ||            |
|             |  |  |  | ...  |         |             ||            |
|             |  |  |  | 5.05]|match-map|matching-sent||  CC CCC    |
|CoAP MID     |16| 1|bi| 0000 |MSB(7 )  |LSB          ||        M-ID|
|CoAP Uri-Path|var 1|dw| path |equal 1  |not-sent     ||            |
+-------------+--+--+--+------+---------+-------------++------------+



~~~~
{: #Fig-CoAP-header-1 title='CoAP Context to compress header without Token'}


In this example, SCHC compression elides the version and the Token Length fields. The 26 method and response codes defined in {{RFC7252}} has been shrunk to 5 bits using a "match-mapping" MO. The Uri-Path contains a single element indicated in the TV and elided with the CDA "not-sent."  

SCHC Compression reduces the header sending only the Type, a mapped code, and the least significant bits of Message ID (9 bits in the example above). 

Note that a client located in an Application Server sending a request to a server located in the Device may not be compressed through this Rule since the MID will not start with 7 bits equal to 0. A CoAP proxy placed before the SCHC C/D can rewrite the message ID to fit the value and match the Rule.



## OSCORE Compression
{: #Sec-OSCORE-Examples}

OSCORE aims to solve the problem of end-to-end encryption for CoAP messages. Therefore, the goal is to hide as much as possible the message
while still enabling proxy operation.

Conceptually this is achieved by splitting the CoAP message into an Inner Plaintext and Outer OSCORE Message. The Inner Plaintext contains sensitive information that is not necessary for proxy operation. However, it is part of the message that can be encrypted until it
reaches its end destination. The Outer Message acts as a shell matching the regular CoAP message format and includes all Options and information 
needed for proxy operation and caching. {{Fig-inner-outer}} illustrates this analysis.

The CoAP protocol arranges the options into one of 3 classes; each granted a specific type of protection by the protocol:

 * Class E: Encrypted options moved to the Inner Plaintext,
 * Class I: Integrity-protected options included in the AAD for the encryption of the Plaintext but otherwise left untouched in the Outer Message,
 * Class U: Unprotected options left untouched in the Outer Message.

These classes point out that the Outer option contains the OSCORE Option and that the message is OSCORE protected; this option carries the information necessary to retrieve the Security Context. The end-point will use this Security Context to decrypt the message correctly.

~~~~
 
                      Original CoAP Packet
                   +-+-+---+-------+---------------+
                   |v|t|TKL| code  |  Msg Id.      |
                   +-+-+---+-------+---------------+....+
                   | Token                              |
                   +-------------------------------.....+
                   | Options (IEU)            |
                   .                          .
                   .                          .
                   +------+-------------------+ 
                   | 0xFF |
                   +------+------------------------+
                   |                               |
                   |     Payload                   |
                   |                               |
                   +-------------------------------+      
                          /                \ 
                         /                  \
                        /                    \
                       /                      \
     Outer Header     v                        v  Plaintext
  +-+-+---+--------+---------------+          +-------+
  |v|t|TKL|new code|  Msg Id.      |          | code  |
  +-+-+---+--------+---------------+....+     +-------+-----......+
  | Token                               |     | Options (E)       |
  +--------------------------------.....+     +-------+------.....+
  | Options (IU)             |                | OxFF  |
  .                          .                +-------+-----------+
  . OSCORE Option            .                |                   |
  +------+-------------------+                | Payload           |
  | 0xFF |                                    |                   |
  +------+                                    +-------------------+

~~~~
{: #Fig-inner-outer title='A CoAP packet is split into an OSCORE outer and plaintext'}

{{Fig-inner-outer}} shows the packet format for the OSCORE Outer header and Plaintext. 

In the Outer Header, the original header code is hidden and replaced by a default dummy value. As seen in Sections 4.1.3.5 and 4.2 of {{RFC8613}}, the message code is replaced by POST for requests and Changed for responses when CoAP is not using the Observe option. If CoAP uses Observe, the OSCORE message code is replaced by FETCH for requests and Content for responses.

The first byte of the Plaintext contains the original packet code, followed by the message code, the class E options, and, if present, the original message Payload preceded by its payload marker.

An AEAD algorithm now encrypts the Plaintext. This integrity protects the Security Context parameters and, eventually, any class I options from the Outer Header. The resulting Ciphertext becomes the new payload of the OSCORE message, as illustrated in {{Fig-full-oscore}}.

As defined in {{RFC5116}}, this Ciphertext is the encrypted Plaintext's concatenation of the authentication tag. Note that Inner Compression only affects the Plaintext before encryption. Thus only the first variable-length of the Ciphertext can be reduced. The authentication tag is fixed in length and is considered part of the cost of protection.

~~~~
 
   
     Outer Header                           
  +-+-+---+--------+---------------+          
  |v|t|TKL|new code|  Msg Id.      |          
  +-+-+---+--------+---------------+....+     
  | Token                               |     
  +--------------------------------.....+     
  | Options (IU)             |               
  .                          .               
  . OSCORE Option            .               
  +------+-------------------+               
  | 0xFF |                                  
  +------+---------------------------+
  |                                  |
  | Ciphertext: Encrypted Inner      |
  |             Header and Payload   |
  |             + Authentication Tag |
  |                                  |
  +----------------------------------+
   
~~~~
{: #Fig-full-oscore title='OSCORE message'}

The SCHC Compression scheme consists of compressing both the Plaintext before encryption and the resulting OSCORE message after encryption, see {{Fig-OSCORE-Compression}}.

The OSCORE message translates into a segmented process where SCHC compression is applied independently in 2 stages, each with its corresponding set of Rules, with the Inner SCHC Rules and the Outer SCHC Rules. This way, compression is applied to all fields of the original CoAP message.   

Note that since the corresponding end-point can only decrypt the Inner part of the message, this end-point will also have to implement Inner SCHC Compression/Decompression.

~~~~
     Outer Message                             OSCORE Plaintext
  +-+-+---+--------+---------------+          +-------+
  |v|t|TKL|new code|  Msg Id.      |          | code  |
  +-+-+---+--------+---------------+....+     +-------+-----......+
  | Token                               |     | Options (E)       |
  +--------------------------------.....+     +-------+------.....+
  | Options (IU)             |                | OxFF  |
  .                          .                +-------+-----------+
  . OSCORE Option            .                |                   |
  +------+-------------------+                | Payload           |
  | 0xFF |                                    |                   |
  +------+------------+                       +-------------------+
  |  Ciphertext       |<---------\                      |
  |                   |          |                      v
  +-------------------+          |             +-----------------+
          |                      |             |   Inner SCHC    |
          v                      |             |   Compression   |
    +-----------------+          |             +-----------------+
    |   Outer SCHC    |          |                     |
    |   Compression   |          |                     v
    +-----------------+          |             +-------+
          |                      |             |RuleID |
          v                      |             +-------+-----------+
    +--------+             +------------+      |Compression Residue|
    |RuleID' |             | Encryption | <--  +----------+--------+
    +--------+-----------+ +------------+      |                   |
    |Compression Residue'|                     | Payload           |
    +-----------+--------+                     |                   |
    |  Ciphertext        |                     +-------------------+
    |                    |     
    +--------------------+
      
~~~~
{: #Fig-OSCORE-Compression title='OSCORE Compression Diagram'}

## Example OSCORE Compression

This section gives an example with a GET Request and its consequent Content
Response from a Device-based CoAP client to a cloud-based CoAP server.
The example also describes a possible set of Rules for the Inner and Outer SCHC
Compression. A dump of the results and a contrast between SCHC + OSCORE
performance with SCHC + COAP performance is also listed. This example gives an approximation of the
cost of security with SCHC-OSCORE.

Our first CoAP message is the GET request in {{Fig-GET-temp}}.

~~~~
Original message:
=================
0x4101000182bb74656d7065726174757265

Header:
0x4101
01   Ver
  00   CON
    0001   TKL
        00000001   Request Code 1 "GET"

0x0001 = mid
0x82 = token

Options:
0xbb74656d7065726174757265
Option 11: URI_PATH
Value = temperature

Original msg length:   17 bytes.
~~~~
{: #Fig-GET-temp title='CoAP GET Request'}

Its corresponding response is the CONTENT Response in {{Fig-CONTENT-temp}}.

~~~~
Original message:
=================
0x6145000182ff32332043

Header:
0x6145
01   Ver
  10   ACK
    0001   TKL
        01000101 Successful Response Code 69 "2.05 Content"

0x0001 = mid
0x82 = token

0xFF  Payload marker
Payload:
0x32332043

Original msg length:   10
~~~~
{: #Fig-CONTENT-temp title='CoAP CONTENT Response'}

The SCHC Rules for the Inner Compression include all fields already present in a regular CoAP message. The methods described in {{CoAPcomp}} apply to these fields. As an example, see {{Fig-Inner-Rules}}.

~~~~
 RuleID 0
+--------------+--+--+--+-----------+---------+---------++------+
| Field        |FL|FP|DI|  Target   |    MO   |    CDA  || Sent |
|              |  |  |  |  Value    |         |         ||[bits]|
+--------------+--+--+--+-----------+---------+---------++------+
|CoAP Code     | 8| 1|up| 1         |  equal  |not-sent ||      |
|CoAP Code     | 8| 1|dw|[69,       |         |         ||      |
|              |  |  |  |132]       |match-map|mapp-sent|| c    |
|CoAP Uri-Path |88| 1|up|temperature|  equal  |not-sent ||      |
+--------------+--+--+--+-----------+---------+---------++------+
~~~~
{: #Fig-Inner-Rules title='Inner SCHC Rules'}

{{Fig-Inner-Compression-GET}} shows the Plaintext obtained for the example GET request. The packet follows the process of Inner Compression and Encryption until the payload. The outer OSCORE Message adds the result of the Inner process. 

In this case, the original message has no payload, and its resulting Plaintext compressed up to only 1 byte (size of the RuleID). The AEAD algorithm preserves this length in its first output and yields a fixed-size tag. SCHC cannot compress the tag, and the OSCORE message must include it without compression.
The use of integrity translates into an overhead in total message length, limiting the amount of compression that can be achieved and plays into the cost of adding security to the exchange.

~~~~
   ________________________________________________________
  |                                                        |
  | OSCORE Plaintext                                       |
  |                                                        |
  | 0x01bb74656d7065726174757265  (13 bytes)               |
  |                                                        |
  | 0x01 Request Code GET                                  |
  |                                                        |
  |      bb74656d7065726174757265 Option 11: URI_PATH      |
  |                               Value = temperature      |
  |________________________________________________________|

                              |
                              |
                              | Inner SCHC Compression
                              |
                              v
                _________________________________
               |                                 |
               | Compressed Plaintext            |
               |                                 |
               | 0x00                            |
               |                                 |
               | RuleID = 0x00 (1 byte)          |
               | (No Compression Residue)        |
               |_________________________________|

                              |
                              | AEAD Encryption
                              |  (piv = 0x04)
                              v
         _________________________________________________
        |                                                 |
        |  encrypted_plaintext = 0xa2 (1 byte)            |
        |  tag = 0xc54fe1b434297b62 (8 bytes)             |
        |                                                 |
        |  ciphertext = 0xa2c54fe1b434297b62 (9 bytes)    |
        |_________________________________________________|

~~~~
{: #Fig-Inner-Compression-GET title='Plaintext compression and encryption for GET Request'}


{{Fig-Inner-Compression-CONTENT}} shows the process for the example CONTENT Response. The Compression Residue is 1 bit long.
Note that since SCHC adds padding after the payload, this misalignment causes the hexadecimal code from the payload to differ from the original, even if SCHC cannot compress the tag. The overhead for the tag bytes limits the SCHC's performance but brings security to the transmission.

~~~~
   ________________________________________________________
  |                                                        |
  | OSCORE Plaintext                                       |
  |                                                        |
  | 0x45ff32332043  (6 bytes)                              |
  |                                                        |
  | 0x45 Successful Response Code 69 "2.05 Content"        |
  |                                                        |
  |     ff Payload marker                                  |
  |                                                        |
  |       32332043 Payload                                 |
  |________________________________________________________|

                              |
                              |
                              | Inner SCHC Compression
                              |
                              v
         _____________________________________________
        |                                             |
        | Compressed Plaintext                        |
        |                                             |
        | 0x001919902180 (6 bytes)                    |                               
        |                                             |
        |   00 RuleID                                 |
        |                                             |
        |  0b0 (1 bit match-map Compression Residue)  |
        |       0x32332043 >> 1 (shifted payload)     |
        |                        0b0000000 Padding    |
        |_____________________________________________|

                              |
                              | AEAD Encryption
                              |  (piv = 0x04)
                              v
     _________________________________________________________
    |                                                         |
    |  encrypted_plaintext = 0x10c6d7c26cc1 (6 bytes)         |
    |  tag = 0xe9aef3f2461e0c29 (8 bytes)                     |
    |                                                         |
    |  ciphertext = 0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes) |
    |_________________________________________________________|

~~~~
{: #Fig-Inner-Compression-CONTENT title='Plaintext compression and encryption for CONTENT Response'}

The Outer SCHC Rules ({{Fig-Outer-Rules}}) must process the OSCORE Options fields. {{Fig-Protected-Compressed-GET}} and {{Fig-Protected-Compressed-CONTENT}} shows a dump of the OSCORE Messages generated from the example messages. They include the Inner Compressed Ciphertext in the payload. These are the messages that have to be compressed by the Outer SCHC Compression.


~~~~
Protected message:
==================
0x4102000182d8080904636c69656e74ffa2c54fe1b434297b62
(25 bytes)

Header:
0x4102
01   Ver
  00   CON
    0001   TKL
        00000010   Request Code 2 "POST"

0x0001 = mid
0x82 = token

Options:
0xd8080904636c69656e74 (10 bytes)
Option 21: OBJECT_SECURITY
Value = 0x0904636c69656e74
          09 = 000 0 1 001 Flag byte
                   h k  n
            04 piv
              636c69656e74 kid


0xFF  Payload marker
Payload:
0xa2c54fe1b434297b62 (9 bytes)
~~~~
{: #Fig-Protected-Compressed-GET title='Protected and Inner SCHC Compressed GET Request'}


~~~~
Protected message:
==================
0x6144000182d008ff10c6d7c26cc1e9aef3f2461e0c29
(22 bytes)

Header:
0x6144
01   Ver
  10   ACK
    0001   TKL
        01000100   Successful Response Code 68 "2.04 Changed"

0x0001 = mid
0x82 = token

Options:
0xd008 (2 bytes)
Option 21: OBJECT_SECURITY
Value = b''

0xFF  Payload marker
Payload:
0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes)
~~~~
{: #Fig-Protected-Compressed-CONTENT title='Protected and Inner SCHC Compressed CONTENT Response'}

For the flag bits, some SCHC compression methods are useful, depending on the Application. The most straightforward alternative is to
provide a fixed value for the flags, combining MO "equal" and CDA "not-sent." 
This SCHC definition saves most bits but could prevent flexibility. Otherwise, SCHC could use a "match-mapping" MO to choose from several configurations for the exchange. If not, the SCHC description may use an "MSB" MO to mask off the three hard-coded most significant bits.

Note that fixing a flag bit will limit CoAP Options choice that can be used in the exchange since their values are dependent on specific options.

The piv field lends itself to having some bits masked off with "MSB" MO and "LSB" CDA. This SCHC description could be useful in applications where the message frequency is low such as LPWAN technologies. 
Note that compressing the sequence numbers may reduce the maximum number of sequence numbers used in an exchange. 
Once the sequence number exceeds the maximum value, the OSCORE keys need to be re-established.

The size s included in the kid context field MAY be masked off with "LSB" CDA. The rest of the field could have additional bits masked off or have the whole field fixed with MO "equal" and CDA "not-sent." The same holds for the kid field.

{{Fig-Outer-Rules}} shows a possible set of Outer Rules to compress the Outer Header. 

~~~~
RuleID 0
+------------------+--+--+--+--------------+-------+--------++------+
| Field            |FL|FP|DI|    Target    |   MO  |   CDA  || Sent |
|                  |  |  |  |    Value     |       |        ||[bits]|
+------------------+--+--+--+--------------+-------+--------++------+ 
|CoAP version      | 2| 1|bi|      01      |equal  |not-sent||      |
|CoAP Type         | 2| 1|up|      0       |equal  |not-sent||      |
|CoAP Type         | 2| 1|dw|      2       |equal  |not-sent||      |
|CoAP TKL          | 4| 1|bi|      1       |equal  |not-sent||      |
|CoAP Code         | 8| 1|up|      2       |equal  |not-sent||      |
|CoAP Code         | 8| 1|dw|      68      |equal  |not-sent||      |
|CoAP MID          |16| 1|bi|     0000     |MSB(12)|LSB     ||MMMM  |
|CoAP Token        |tkl 1|bi|     0x80     |MSB(5) |LSB     ||TTT   |
|CoAP OSCORE_flags | 8| 1|up|     0x09     |equal  |not-sent||      |
|CoAP OSCORE_piv   |var 1|up|     0x00     |MSB(4) |LSB     ||PPPP  |
|COAP OSCORE_kid   |var 1|up|0x636c69656e70|MSB(52)|LSB     ||KKKK  |
|COAP OSCORE_kidctx|var 1|bi|     b''      |equal  |not-sent||      |
|CoAP OSCORE_flags | 8| 1|dw|     b''      |equal  |not-sent||      |
|CoAP OSCORE_piv   |var 1|dw|     b''      |equal  |not-sent||      |
|CoAP OSCORE_kid   |var 1|dw|     b''      |equal  |not-sent||      |
+------------------+--+--+--+--------------+-------+--------++------+
~~~~
{: #Fig-Outer-Rules title='Outer SCHC Rules'}


The Outer Rule of {{Fig-Outer-Rules}} is applied to the example GET Request and CONTENT Response. 
{{Fig-Compressed-GET}} and {{Fig-Compressed-CONTENT}} show the resulting messages.

~~~~
Compressed message:
==================
0x001489458a9fc3686852f6c4 (12 bytes)
0x00 RuleID
    1489 Compression Residue
        458a9fc3686852f6c4 Padded payload

Compression Residue:
0b 0001 010 0100 0100 (15 bits -> 2 bytes with padding)
    mid tkn piv  kid

Payload
0xa2c54fe1b434297b62 (9 bytes)

Compressed message length: 12 bytes
~~~~
{: #Fig-Compressed-GET title='SCHC-OSCORE Compressed GET Request'}

~~~~
Compressed message:
==================
0x0014218daf84d983d35de7e48c3c1852 (16 bytes)
0x00 RuleID
    14 Compression Residue
      218daf84d983d35de7e48c3c1852 Padded payload
Compression Residue:
0b0001 010 (7 bits -> 1 byte with padding)
  mid  tkn

Payload
0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes)

Compressed msg length: 16 bytes
~~~~
{: #Fig-Compressed-CONTENT title='SCHC-OSCORE Compressed CONTENT Response'}

In contrast, comparing these results with what would be obtained by SCHC
compressing the original CoAP messages without protecting them with OSCORE is done
by compressing the CoAP messages according to the SCHC Rules in {{Fig-NoOsc-Rules}}.

~~~~
RuleID 1
+---------------+--+--+--+-----------+---------+-----------++-------+
| Field         |FL|FP|DI|  Target   |   MO    |     CDA   ||  Sent |
|               |  |  |  |  Value    |         |           || [bits]|
+---------------+--+--+--+-----------+---------+-----------++-------+ 
|CoAP version   | 2| 1|bi|    01     |equal    |not-sent   ||       |
|CoAP Type      | 2| 1|up|    0      |equal    |not-sent   ||       |
|CoAP Type      | 2| 1|dw|    2      |equal    |not-sent   ||       |
|CoAP TKL       | 4| 1|bi|    1      |equal    |not-sent   ||       |
|CoAP Code      | 8| 1|up|    2      |equal    |not-sent   ||       |
|CoAP Code      | 8| 1|dw| [69,132]  |match-map|map-sent   ||C      |
|CoAP MID       |16| 1|bi|   0000    |MSB(12)  |LSB        ||MMMM   |
|CoAP Token     |tkl 1|bi|    0x80   |MSB(5)   |LSB        ||TTT    |
|CoAP Uri-Path  |88| 1|up|temperature|equal    |not-sent   ||       |
+---------------+--+--+--+-----------+---------+-----------++-------+
~~~~
{: #Fig-NoOsc-Rules title='SCHC-CoAP Rules (No OSCORE)'}

{{Fig-NoOsc-Rules}} Rule yields the SCHC compression results in {{Fig-GET-temp-no-oscore}} for request, and
{{Fig-CONTENT-temp-no-oscore}} for the response.

~~~~

Compressed message:
==================
0x0114
0x01 = RuleID

Compression Residue:
0b00010100 (1 byte)

Compressed msg length: 2

~~~~
{: #Fig-GET-temp-no-oscore title='CoAP GET Compressed without OSCORE'}

~~~~


Compressed message:
==================
0x010a32332043
0x01 = RuleID

Compression Residue:
0b00001010 (1 byte)

Payload
0x32332043

Compressed msg length: 6


~~~~
{: #Fig-CONTENT-temp-no-oscore title='CoAP CONTENT Compressed without OSCORE'}

As can be seen, the difference between applying SCHC + OSCORE as compared to 
regular SCHC + COAP is about 10 bytes.

# IANA Considerations

This document has no request to IANA.

# Security considerations {#SecConsiderations}

The use of SCHC header compression over CoAP header fields allow the compression of the header information only. The SCHC header compression itself does not increase or reduce the level of security in the communication. When the connection does not use any security protocol as OSCORE, DTLS, or other, it is necessary to use a layer two security.

If LPWAN is the layer two technology, the use of SCHC over the CoAP protocol keeps valid the Security Considerations of SCHC header compression {{RFC8724}}.  When using another layer two, integrity protection is mandatory.

The use of SCHC when CoAP uses OSCORE keeps valid the security considerations defined in {{RFC8613}}. 

DoS attacks are possible if an intruder can introduce a corrupted SCHC compressed packet onto the link and cause excessive resource consumption at the decompressor. However, an intruder having the ability to add corrupted packets at the link layer raises additional security issues than those related to header compression.

SCHC compression returns variable-length Compression Residues for some CoAP fields. In the compressed header, 
the length sent is not the original header field length but the Compression Residue's length that is transmitted. So If a corrupted packet comes to the decompressor with a longer or shorter length than the original header, SCHC decompression will detect an error and drop the packet.

Using SCHC over the OSCORE headers, OSCORE MUST consider the Initialization Vector (IV) size carefully in the Compression Residue.  A Compression Residue size obtained with an "LSB" CDA over the IV impacts the compression efficiency and the frequency that the Device will renew its key. This operation requires several exchanges and is energy-consuming. 

SCHC header and compression Rules MUST remain tightly coupled. Otherwise, an encrypted Compression Residue may be decompressed differently by the receiver. Any update in the context Rules on one side MUST trigger the OSCORE keys re-establishment. 

# Acknowledgements

The authors would like to thank (in alphabetic order): Christian Amsuss, Dominique Barthel, Carsten Bormann, Theresa Enghardt, Thomas Fossati, Klaus Hartke, Benjamin Kaduk, Francesca Palombini, Alexander Pelov, Goran Selander and Eric Vyncke. 





--- back



