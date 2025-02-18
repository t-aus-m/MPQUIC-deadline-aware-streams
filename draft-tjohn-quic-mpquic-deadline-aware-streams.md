---
title: "Deadline Aware Streams in MP-QUIC"
category: info # TODO: Figure out correct category

docname: draft-tjohn-quic-mpquic-deadline-aware-streams-latest
submissiontype: IETF # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "QUIC"
keyword: # TODO: Define fitting keywords
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "QUIC"
  type: ""
  mail: "quic@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/quic/"
  github: "t-aus-m/MPQUIC-deadline-aware-streams"
  latest: "https://t-aus-m.github.io/MPQUIC-deadline-aware-streams/draft-tjohn-quic-mpquic-deadline-aware-streams.html"

author: # TODO: Any more authors that need to be added?
 -
    fullname: "Tony John"
    organization: "Otto-von-Guericke University Magdeburg"
    email: "tony.john@ovgu.de"
 -
    fullname: "Till-Frederik Riechard"
    organization: "Otto-von-Guericke University Magdeburg"
    email: "riechard@ovgu.de"

normative:
   QUIC: rfc9000
   QUIC-TLS: rfc9001
   MP-QUIC: I-D.draft-ietf-quic-multipath
   QUIC-AFEC: I-D.draft-dmoskvitin-quic-adaptive-fec
   RFC3339:

informative:
   DMTP: DOI.10.23919/IFIPNetworking57963.2023.10186417


--- abstract

This document proposes the Deadline-aware Multipath Transport Protocol (DMTP) as an extension to the Multipath Extension of QUIC. DMTP leverages multiple paths along adaptive Forward Error Correction (FEC) and smart retransmissions to support real-time applications with strict latency requirements. With this it fills in a niche with increasing demand, that existing multipath protocols fall short on.

--- middle

# Introduction

The Multipath Extension of QUIC {{MP-QUIC}} enhances performance by utilizing multiple paths simultaneously, but it currently lacks mechanisms to guarantee data delivery within specific timeframes. Given the increasing demand for real-time applications such as teleoperation, live video streaming, and online gaming there's a growing need for transport protocols that can efficiently handle strict latency requirements. Introducing DMTP to {{MP-QUIC}} could enable applications to meet those stringent latency constraints, optimizing for low-latency and high-reliability scenarios.

While the implementation of DMTP with implementation-specific APIs would be possible, that approach would likely lack endpoint coordination, because deadlines would not be communicated between different implementations and/or endpoints through the protocol. By introducing a transport parameter (see {{transport-parameter}}) and a custom frame (see {{deadline-control-frame}}), endpoints can negotiate support and exchange deadline information directly within the protocol, enabling coordinated scheduling decisions at the transport layer. Standardizing this mechanism avoids the limitations of implementation-specific solutions, promoting wider adoption and interoperability of DMTP across different implementations of {{MP-QUIC}}.

This draft is based on a conference paper proposing {{DMTP}}.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Design of DMTP

## Custom Scheduler and Congestion Controller for Deadline-Aware Streams

Implementations that support deadline-aware streams SHOULD provide a custom scheduler and congestion controller that prioritize packets based on their deadlines. This includes:

- Path Selection: Dynamically selecting paths that are most likely to meet the deadlines based on real-time metrics like latency, bandwidth, and packet loss.
- Packet Prioritization: Prioritizing packets from streams with earlier deadlines.
- Adaptive Retransmissions: Deciding whether to retransmit lost packets based on their remaining time before the deadline.
- Forward Error Correction (FEC): Optionally integrating FEC mechanisms to reduce the need for retransmissions in networks with random losses.
- Deadline Miss Handling: Informing the application when a deadline cannot be met, allowing it to take appropriate action.

## Deadlines

### Signalling Deadlines

To signal deadlines, endpoints use the DEADLINE_CONTROL frame (see {{deadline-control-frame}}). This frame associates a specific deadline with a stream, indicating the relative time by which the data should be delivered.

### Deadline Semantics

- Deadline Representation: Deadlines are represented as a relative time in milliseconds from the time the frame is sent.
- Stream Association: A deadline applies to a specific stream identified by its Stream ID
- Transport Behavior: Upon receiving a DEADLINE_CONTROL frame, the transport layer SHOULD attempt to schedule and retransmit packets carrying data for the specified stream to meet the indicated deadline.
- Retransmissions and Scheduling: Endpoints MAY implement custom schedulers and congestion controllers optimized for deadline-aware traffic, such as those based on DMTP concepts.

### Handling Missed Deadline

If the transport layer determines that the deadline cannot be met, it MAY choose to:

- Discard the data associated with the deadline-aware stream.
- Inform the application of the missed deadline.
- Continue delivering the data if it is still deemed useful.

The specific behavior is implementation-specific and MAY be configurable by the application.

## Adaptive Forward Error Correction (FEC)

DMTP optionally uses Adaptive FEC as proposed in {{QUIC-AFEC}} to reduce the need for retransmissions in networks with random losses.
When using {{QUIC-AFEC}}, the Tag Type of the FEC_Tag MUST be set to 1 in order for the FEC to work in a {{MP-QUIC}} and DMTP environment. This indicates Long Flow Usage which in turn implies that both source symbol packets and repair symbol packets MUST contain the FEC_Tag frame, which is necessary for matching repair symbol packets to their respective source symbol packets after sending them via different paths.
FEC Packets SHOULD be sent via another path then the source data, it is however RECOMMENDED to send the FEC data via the retransmission path. The coding rate is chosen on a per-stream basis.

## Smart Retransmissions

//: # TODO: Describe the goal and design of smart retransmissions briefly

## Path Metrics

//: # TODO: Describe, which path metrics are necessary for DMTP to work and how they will be gathered

### Per Path Delay

A crucial metric for DMTP to work is the one way delay of each path, as this determines the suitability of that path for reaching the deadline requirement. DMTP is designed to be able to retrieve this metric from a path-aware network like SCION directly. However, if such an underlying network is not present, DMTP is also able to measure the per path delay itself. For this to work effectively, endpoints SHOULD use clock synchronization while communication via DMTP to ensure accuracy of the delay measurements. This proposal adds a new frame type (see {{dmtp-ack-frame}}) to determine the per path delays. Since acknowledgments should be sent back to the sender via a path that is different from the receive path, per path delays have to be probed separately. For this, PING frames are sent regularly via each individual active path.

### Per Path Loss Rate

//: # TODO: Describe how loss rate is determined

# Extension of Multipath Extension of QUIC

This extension is based on and aims to extend {{MP-QUIC}}. The contents of this extension will be specified in this section.

## Handshake Negotiation and Transport Parameter {#transport-parameter}

This extension defines a new transport parameter, used to negotiate the use of deadline-aware streams during the connection handshake, as specified in {{QUIC}}. The new transport parameter is defined as follows:

- enable_deadline_aware_streams (value TBD): A zero-length value that, if present, indicates that the endpoint supports deadline-aware streams.

Endpoints negotiate the use of deadline-aware streams by including the enable_deadline_aware_streams transport parameter in the handshake. Both endpoints MUST include this transport parameter to enable the use of deadline-aware streams. If an endpoint receives a DEADLINE_CONTROL frame without having negotiated support, it MUST treat this as a connection error of type PROTOCOL_VIOLATION

## DEADLINE_CONTROL Frame {#deadline-control-frame}

The DEADLINE_CONTROL frame (type=TBD) is used to signal deadline-awareness for specific streams and to indicate their associated deadlines.

~~~
  DEADLINE_CONTROL Frame {
    Type (i) = TBD,
    Stream ID (i),
    Deadline (i),
  }
~~~

The DEADLINE_CONTROL frame contains the following fields:

Stream ID:
: A variable-length integer indicating the Stream ID to which the deadline applies.

Deadline:
: A variable-length integer representing the relative deadline in milliseconds from the time the frame is sent.
An endpoint sends a DEADLINE_CONTROL frame to indicate that data on the specified stream should be delivered by the given deadline. Upon receiving this frame, the peer MUST attempt to schedule and deliver the data on the specified stream within the indicated deadline.

Usage Constraints:

- Endpoints MUST NOT send the DEADLINE_CONTROL frame unless both endpoints have negotiated support via the enable_deadline_aware_streams transport parameter.
- If an endpoint receives a DEADLINE_CONTROL frame without having negotiated support, it MUST treat it as a connection error of type PROTOCOL_VIOLATION.
- The DEADLINE_CONTROL frame MUST only be sent in 1-RTT packets.

## DMTP_ACK Frame {#dmtp-ack-frame}

The DMTP_ACK frame (type=TBD) is used to acknowledge the reception of a packet and feedback the reception time to the sender. If the received frame contains a PING (type=0x01) frame, the DMTP_ACK frame MUST be sent back on the same path that it was received at. The DMTP_ACK Frame contains the same information as a {{QUIC}} ACK frame, but adds a timestamp to it, in order to communicate if a packet has met its deadline or not.

When using deadline-awareness the receiver SHOULD acknowledge each packet separately.

~~~
  DMTP_ACK Frame {
   Type (i) = TBD,
   Largest Acknowledged (i),
   ACK Delay (i),
   ACK Range Count (i),
   First ACK Range (i),
   ACK Range (..) ...,
   [ECN Counts (..)],
   Timestamp (i),
  }
~~~

The DMTP_ACK frame adds the Timestamp field to the {{QUIC}} ACK frame. It MUST be formatted according to {{RFC3339}} with a resolution down to the nanosecond, i.e. with 9 digits after the decimal point. If an endpoint uses a clock with a lower resolution, the remaining digits SHOULD be padded with zeros.

# API

//: # TODO: Define API

# Security Considerations

This extension retains all the security features and considerations of {{QUIC}}, {{QUIC-TLS}} and {{MP-QUIC}}.
//: # TODO: Add DMTP specific Security Considerations

# IANA Considerations

This document defines a new transport parameter for the negotiation of enable multiple paths for QUIC, and three new frame types. The draft defines provisional values for experiments, but we expect IANA to allocate short values if the draft is approved.

The following entry in {{transport-parameters}} should be added to the "QUIC Transport Parameters" registry under the "QUIC Protocol" heading.

Value                                         | Parameter Name.   | Specification
----------------------------------------------|-------------------|-----------------
TBD                                           | enable_deadline_aware_streams  | {{transport-parameter}}
{: #transport-parameters title="Addition to QUIC Transport Parameters Entries"}

The following frame type defined in {{frame-types}} should be added to
the "QUIC Frame Types" registry under the "QUIC Protocol" heading.

Value                                              | Frame Name          | Specification
---------------------------------------------------|---------------------|-----------------
TBD                                                | DEADLINE_CONTROL    | {{deadline-control-frame}}
TBD                                                | DMTP_ACK            | {{dmtp-ack-frame}}
{: #frame-types title="Addition to QUIC Frame Types Entries"}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
