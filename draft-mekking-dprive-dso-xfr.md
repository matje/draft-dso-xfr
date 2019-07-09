%%%
title           = "DNS Zone Transfer using DNS Stateful Operations"
abbrev          = "DSO-XFR"
workgroup       = "DNS Operations"
area            = "Operations and Management"
submissiontype  = "IETF"
ipr             = "trust200902"
date            = 2018-10-19T14:59:17Z
keyword         = [
    "DNS",
    "XFR",
    "DSO",
]

[seriesInfo]
name            = "Internet-Draft"
value           = "draft-mekking-dnsop-dso-xfr"
status          = "standard"

[[author]]
initials        = "W.M."
surname         = "Mekking"
fullname        = "Matthijs Mekking"
organization    = "ISC"
 [author.address]
 email          = "matthijs@isc.org"
  [author.address.postal]
  street        = "950 Charter St"
  city          = "Redwood City"
  region        = "CA"
  code          = "94063"
  country       = "USA"

[[author]]
initials        = "S"
surname         = "Dickinson"
fullname        = "Sara Dickinson"
organization    = "Sinodun Internet Technologies"
abbrev          = "Sinodun"
 [author.address]
 email          = "sara@sinodun.com"
  [author.address.postal]
  street        = "Magadalen Centre"
  region        = "Oxford Science Park"
  city          = "Oxford"
  code          = "OX4 4GA"
  country       = "United Kingdom"

%%%

.# Abstract

This document introduces DNS zone transfers using DNS stateful
operations (DSO) [@!RFC8490].

{mainmatter}


# Introduction

DNS zones are distributed by more than one name server.  Secondary
servers acquire zone updates from a primary server using the DNS
zone transfer protocol (AXFR [@!RFC5936] or IXFR [@!RFC1995]).  The
existing mechanisms for zone transfers have some drawbacks:

  * Zone transfers are inefficient in communicating changed portions
    of a zone.  IXFR was meant to solve that problem but turns out
    to be very inefficient in combination with DNSSEC
    [@!I-D.ietf-mekking-ixfr].

  * Zone transfers are communicated unencrypted and that is a
    privacy vulnerability for the holder of the zone
    [@!I-D.ietf-hzpa-dprive-xfr-over-tls].

This document introduces a new way to do zone transfers and uses
DNS Stateful Operations (DSO) [@!RFC8490].  This means the zone
transfers will use either DNS-over-TCP [@!RFC1035] [@!RFC7766]
or DNS-over-TLS [@!RFC7858] as transport protocol.  Both transports
can be used to benefit from the more efficient DSO zone transfer
mechanism.  Only DNS-over-TLS will help you mitigate against the
privacy vulnerabilities mentioned above.


# Terminology

Section 3 of RFC 8490 [@!RFC8490] describes DSO terminology.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**,
**SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**,
and **OPTIONAL** in this document are to be interpreted as described in
[@!RFC2119].


# Overview

A secondary server can subscribe for zone transfers by connecting
to a primary server and sending DSO messages indicating the zones
it wants to receive transfers for. 


# Protocol Operation

The DSO Zone Transfer protocol is a session-oriented protocol. 

DSO Zone Transfer clients and servers MUST support DSO.

## Establishing a Connection

A DSO Zone Transfer client will start a DSO Zone Transfer session
by establishing a connection to the server.  In most zone transfer
setups it is known in advance that both client and server support
DSO.  However it may also be common that some actors do support
DSO and some actors don't.  Therefore it is RECOMMENDED to use
explicit DSO session establishment.

The DSO session is established over a connection by the client sending
a DSO request message.  For the purpose of zone transfers this
is a DSO Zone Transfer Subscribe request message (#subscribexfr).

It SHOULD be followed with a DSO Keepalive request message to agree
on a long-lived session.

## Subscribing to DSO Zone Transfers

A client that is interested in zone transfers can subscribe to
that zone with a DSO Zone Transfer Subscribe request message.
This request is encoded in a DSO message.  This document defines
a Primary TLV for DSO Zone Transfer Subscribe message (SUBSCRIBE-XFR).

A server MUST NOT send a REQUEST-XFR message over an existing session
from a client.

### SUBSCRIBE-XFR Request Message {#subscribexfr}

A SUBSCRIBE-XFR request message begins with the DSO 12-byte header
followed by the SUBSCRIBE-XFR Primary TLV. The header fields MUST be
set as described in the DSO specification. Since this is a request
message the MESSAGE ID field MUST be set to a unique nonzero value
(the initiator is not currently using for any other active operation
on this connection). 

The DSO-TYPE is SUBSCRIBE-XFR.

The DSO-LENGTH is the length of the DSO-DATA that follows which
specifies the zone and class that the initiator wants to get zone
transfers for.

The DSO-DATA for a SUBSCRIBE-XFR request message MUST contain exactly
one NAME and CLASS. CLASS IN (1) MUST be supported.

[MM] Discuss: Should we allow multiple zones in one REQUEST-XFR?

### SUBSCRIBE-XFR Response Message

Each SUBSCRIBE-XFR request generates exactly one SUBSCRIBE-XFR response
from the server. A response with matching MESSAGE ID, and RCODE set to
NOERROR (0) indicates that the request was successful.

A SUBSCRIBE-XFR response may be followed by one or more optional TLVs,
such as a Retry Delay TLV.

A SUBSCRIBE-XFR response message MUST NOT include a SUBSCRIBE-XFR TLV.

In the SUBSCRIBE-XFR response the RCODE indicates whether or not the
subscription was accepted.  Anything other than a NOERROR response
indicates a failure.  RCODE values are defined in RFC 8490 [@!RFC8490].

## Providing DSO Zone Transfers

Once a zone transfer connection has been successfully established, the
server generates PUSH-XFR messages to send to the client as appropriate.

For each zone, the first PUSH-XFR message will likely contain the full
zone contents, similar to getting an AXFR.

Other zone changes are provided in subsequent PUSH-XFR messages.

### PUSH-XFR Message

A PUSH-XFR request message begins with the DSO 12-byte header followed by
the PUSH-XFR primary TLV.  This is a DSO unidirectional message, so the
MESSAGE ID field MUST be zero.  There is no client response to a PUSH-XFR
message.

The DSO-TYPE is PUSH-XFR.

[MM: Discuss: How does a server know that a secondary was successful in
applying zone changes?

## Refreshing zones with DSO

The SOA record has a REFRESH Field that indicates when a zone should be
refreshed.  If no PUSH-XFR messages have occurred when the refresh time
has passed, the client can initiate a REFRESH-XFR message. 

# IANA considerations

To do.


# Security considerations

To do.


{backmatter}
