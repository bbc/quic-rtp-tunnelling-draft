---
title: "QRT: QUIC RTP Tunnelling"
abbrev: "QRT"
docname: draft-hurst-quic-rtp-tunnelling-latest
category: info

ipr: trust200902
area: General
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: S. Hurst
    name: Sam Hurst
    organization: BBC Research & Development
    email: sam.hurst@bbc.co.uk

normative:
  QUIC-TRANSPORT:
    title: "QUIC: A UDP-Based Multiplexed and Secure Transport"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-transport-29
    author:
      -
        ins: J. Iyengar
        name: Jana Iyengar
        org: Google
        role: editor
      -
        ins: M. Thomson
        name: Martin Thomson
        org: Mozilla
        role: editor

  QUIC-RECOVERY:
    title: "QUIC Loss Detection and Congestion Control"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-recovery-29
    author:
      -
        ins: J. Iyengar
        name: Jana Iyengar
        org: Google
        role: editor
      -
        ins: I. Swett
        name: Ian Swett
        org: Google
        role: editor

  QUIC-DATAGRAM:
    title: "An Unreliable Datagram Extension to QUIC"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-datagram-00
    author:
      -
        ins: T. Pauly
        name: Tommy Pauly
        org: Apple Inc.
        role: editor
      -
        ins: E. Kinnear
        name: Eric Kinnear
        org: Apple Inc.
        role: editor
      -
        ins: D. Schinazi
        name: David Schinazi
        org: Google LLC
        role: editor

informative:
  QUIC-TLS:
    title: "Using Transport Layer Security (TLS) to Secure QUIC"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-tls-29
    author:
      -
        ins: M. Thomson
        name: Martin Thomson
        org: Mozilla
        role: editor
      -
        ins: S. Turner, Ed.
        name: Sean Turner
        org: sn3rd
        role: editor


--- abstract

QUIC is a UDP-based transport protocol for stream-orientated, congestion-controlled, secure,
multiplexed data transfer. RTP carries real-time data between endpoints, and the accompanying
control protocol RTCP allows monitoring and control of the transfer of such data. With RTP and RTCP
being agnostic to the underlying transport protocol, it would be possible to multiplex both the RTP
and associated RTCP flows into a single QUIC connection to take advantage of QUIC features such as
the low latency setup and strong TLS-based security.

--- middle

# Introduction

The Real-time Transport Protocol (RTP) {{!RFC3550}} provides end-to-end network transport functions
suitable for application transmitting real-time data, such as audio and video data, over multicast
or unicast network services for the purposes of telephony, video streaming, conferencing and other
real-time applications.

The QUIC transport protocol is a UDP-based stream-orientated and encrypted transport protocol aimed
at offering an improvement path over the common combination of TCP and TLS for web applications.
Compared with TCP+TLS, QUIC offers much reduced connection set-up times, improved stream
multiplexing aware congestion control, and the ability to perform connection migration. QUIC offers
two modes of data transfer:

* Reliable transfer using STREAM frames, as specified in {{QUIC-TRANSPORT}}, {{QUIC-RECOVERY}}, etc.
* Unreliable transfer using DATAGRAM extension frames, as specified in {{QUIC-DATAGRAM}}.

RTP has traditionally been run over UDP or DTLS to achieve timely but unreliable data transfer. For
use cases such as real-time audio and video transmission, the underlying media codecs can be
considered in part fault tolerant to an unreliable transport mechanism, with missing data from the
stream resulting in glitches in the media presentation, such as missing video frames or gaps in
audio playback. By purposely using an unreliable transport mechanism, applications can minimise the
added latency that would otherwise result from managing the large packet reception buffers needed to
account for network reordering or transport protocol retransmission.


## Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

Packet and frame diagrams in this document use the format described in {{QUIC-TRANSPORT}}.

## Definitions

* Endpoint: host capable of being a participant in a QRT session.

* QRT session: A QUIC connection carrying one or more RTP sessions, each with or without an
accompanying RTCP channel. Logically, this aligns with the RTP concept of a "multimedia session".

* Client: The endpoint which initiates the QUIC connection

* Server: The endpoint which accepts the incoming QUIC connection

# Use Cases for an RTP Mapping over QUIC

The following sections describe some possible use cases for an RTP Mapping over QUIC, hereafter QRT.
The examples were chosen to illustrate some basic concepts, and is both not an exhaustive list of
possible use cases nor a limitation on what QRT may be used for.

## Current Affairs Contribution Feed {#contrib-feed}

TODO Some words about contrib feeds

## Audio and Video Conference Via a Central Server {#teleconference}

TODO Some words about videoconferences

# QRT Sessions {#qrt-session}



# RTP Sessions {#rtp-session}

QRT allows multiple RTP sessions to be carried in a single QRT session. Each RTP session is operated
independently of all the others, and individually discriminated by an RTP session flow identifier,
as described below in {{flow-identifier}}.

RTP packets are carried in QUIC `DATAGRAM` frames, as described in {{QUIC-DATAGRAM}}. QUIC allows
multiple QUIC frames to be carried within a single QUIC packet, so multiple RTP packets for one (or
more) RTP sessions may therefore be carried in a single QUIC packet, subject to the network path
MTU. If multiple RTP packets are to be carried within a single QUIC packet, then all but the final
`DATAGRAM` frame must specify the length of the datagram, as the RTP packet header does not provide
its own length field. It is therefore assumed that if a `DATAGRAM` frame is received without a
Length field, then this QUIC frame extends to the end of the QUIC packet.

## RTP Session Flow Identifier {#flow-identifier}

The flow of packets belonging to an RTP session are identified by way of an RTP Session Flow
Identifier header carried in the `DATAGRAM` frame payload before each RTP packet. This flow
identifier is encoded as a variable-length integer, as defined in {{QUIC-TRANSPORT}}.

~~~
QRT Datagram Payload {
  Flow Identifier (i),
  RTP Packet (..)
}
~~~
{: #fig-qrt-datagram-payload title="QRT Datagram Payload"}

### Assigning RTP Session Flow Identifiers {#assign-rtp-session-flow-identifiers}

{{!RFC3550}} specifies that in traditional RTP, RTP sessions are distinguished by pairs of transport
addresses. However as QUIC allows for connections to migrate between transport address associations,
and as we wish to multiplex multiple RTP session flows over a single RTP session, this profile of
RTP amends this statement and instead distinguishes between RTP sessions by the way of a flow
identifier.

The RTP Session Flow Identifier is a 62-bit unsigned integer between the values of 0 and 2^62 - 1.
The flow identifier is always allocated by the initial sender of an RTP session, starting from the
smallest available number to the sender and increasing with each new RTP session to be sent.

Similar to QUIC stream IDs, QRT splits these flow identifiers into four categories based on the
value of the two lowest order bits. In order to mitigate against potential race conditions between a
QUIC client and server attempting to allocate the same new flow identifier, the second least
significant bit (0x2) of the flow identifier indicates the the initiator of the RTP session.
Client-initiated RTP sessions will have this bit set to 0, while server-initiated RTP sessions will
have this bit set to 1.

The least significant bit (0x1) of the flow identifier distinguishes between an RTP packet flow.
`DATAGRAM` frames which carry RTP packet flows will have this bit set to 0 (and as such be an
even-numbered flow identifier), and `DATAGRAM` frames which carry RTCP packet flows will have this
bit set to 1 (odd-numbered flow identifier). Carriage of RTCP packets is discussed further in
{{rtcp-mapping}}.

| Bits | Flow identifier category                            |
|:-----|:----------------------------------------------------|
| 0x0  | RTP packet flow for a client-initiated RTP session  |
| 0x1  | RTCP packet flow for a client-initiated RTP session |
| 0x2  | RTP packet flow for a server-initiated RTP session  |
| 0x3  | RTCP packet flow for a server-initiated RTP session |
{: #flow-identifier-categories title="RTP session flow identifer categories"}

For example, the first RTP session that a QUIC client creates would use a flow identifier of 0. The
third RTP session that a QUIC server creates would use a flow identifier of 10. The equivalent RTCP
flows would be carried on flow identifiers 1 and 11 respectively.

Unlike QUIC stream IDs, all flow identifiers can be used for bidirectional communication - the flow
identifier only discriminates on the first participant to send packet flows to an RTP session.

Synchronization sources are always unique between RTP sessions identified by an RTP session flow
identifier.

> **Authors' Note:** The authors welcome comments on whether a state model of RTP session flows
would be beneficial. Currently, once an RTP session has been opened by an initiator, it is then
considered an extant RTP session and implementations would have to keep any resources allocated to
that RTP session until the QRT session is complete.

## RTCP Mapping {#rtcp-mapping}

An RTP session may have RTCP packet flows associated with it. These flows are carried on a separate
RTP session flow identifier, as described in {{assign-rtp-session-flow-identifiers}}. The session
flow identifier is always the value of the RTP session flow identifier + 1, regardless of which
endpoint in a QRT session is the first to send an RTCP packet. For example, for a server-initiated
RTP packet flow with a flow identifier of 18, even if the first RTCP packet exchanged between the
two endpoints is a Receiver Report from the client, the RTCP flow identifier would take the value
19.

As RTCP packets contain a length field in their header, implementations MAY combine several RTCP
packets pertaining to the same RTP session into a single `DATAGRAM` frame. Additionally,
implementations MAY choose to carry these RTCP packets each in their own `DATAGRAM` frame.

### Restricted RTCP Packet Types {#restricted-rtcp}

> **Authors' Note:** I have specifically avoided calling this section "Prohibited RTCP packet types"
for the time being, so as to not unnecessarily exclude the carriage of these packet types for the
purposes of experimentation. Similarly, most statements below use SHOULD NOT instead of MUST NOT.
The authors welcome comments on whether the document should prohibit the sending of some or all of
these packet types.

In order to reduce duplication, the following RTCP packet types SHOULD NOT be sent in a QRT session:

* The "Generic NACK" packet defined in {{!RFC4585}} states that Generic NACK feedback SHOULD NOT be
used if the underlying transport protocol is capable of providing similar feedback information to
the sender. As all `DATAGRAM` frames are ACK-eliciting, QUIC already fulfils this requirement.

* The "Loss RLE" Extended Report (XR) packet defined in {{!RFC3611}} contains information that
should already be known to both ends of the QUIC connection by means of the loss detection mechanism
specified in {{QUIC-RECOVERY}}.

* The "Receiver Reference Time" and "Delay Since Last Receiver Report" Extended Report packet types
defined in {{!RFC3611}} SHOULD NOT be used if both endpoints in a QRT session are using the QUIC
spin bit to calculate Round-Trip Time (RTT) between endpoints as specified in {{rtt-spin}}.

* The "Port Mapping" packet type defined in {{!RFC6284}} is used to negotiate UDP port pairs for the
carriage of RTP and RTCP packets to peers. This does not apply in a QRT session, as the QUIC
connection manages the UDP port association(s), and as such this packet type SHOULD NOT be used.

# Calculating Round-Trip Time Using The Spin Bit {#rtt-spin}

{{QUIC-TRANSPORT}} includes a mechanism that allows latency monitoring for a connection, not just by
QUIC endpoints but also passively by observation points on the network path. QRT implementations
SHOULD support the spin bit and actively use it to calculate the Round-Trip Time between a client
and server, and make this data available to the RTP layer.

> **Authors' Note:** It occurs to me that there is no means for the server to indicate what amount
of idle or processing time was incurred before the packet that flips the spin bit is received. For
example, a sender may be pacing packets out at a defined rate so as to not overwhelm the network,
so maybe this isn't such a good idea - I welcome comments on the subject.

# Protocol Identifier {#protocol-identifier}

The QRT protocol specified in this document is identified by the application-layer protocol
negotiation (ALPN) {{!RFC7301}} identifier "qrt".

## Draft Version Identification

Only implementations of the final, published RFC can identify themselves as "qrt". Until such an RFC
exists, implementations MUST NOT identify themselves using this string. Implementations of draft
versions of the protocol MUST add the string "-h" and the corresponding draft number to the
identifier. For example, draft-hurst-quic-rtp-tunnelling-03 is identified using the string
"qrt-h03".

Non-compatible experiments that are based on these draft versions MUST append the string "-" and an
experiment name to the identifier. For example, an experimental implementation based on
draft-hurst-quic-rtp-tunnelling-03 which uses extension features not registered with the appropriate
IANA registry might identify itself as "qrt-h03-extension-foo". Note that any label MUST conform to
the "token" syntax defined in Section 3.2.6 of [RFC7230]. Experimenters are encouraged to coordinate
their experiments.

# Discoverability of QRT Sessions {#discoverability}

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
