---
title: "Deadline Aware Streams in MP-QUIC"
category: info

docname: draft-tjohn-quic-mpquic-deadline-aware-streams-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "QUIC"
keyword:
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

author:
 -
    fullname: "Tony John"
    organization: "Otto-von-Guericke University Magdeburg"
    email: "tony.john@ovgu.de"
 -
    fullname: "Till-Frederik Riechard"
    organization: "Otto-von-Guericke University Magdeburg"
    email: "riechard@ovgu.de"

normative:
   QUIC-TRANSPORT: rfc9000
   QUIC-TLS: rfc9001
   MP-QUIC: I-D.draft-ietf-quic-multipath

informative:
   DMTP: DOI.10.23919/IFIPNetworking57963.2023.10186417


--- abstract

This document proposes deadline aware streams to be added to Multipath QUIC in order to be able to deliver deadline sensitive information via QUIC over multiple paths.

--- middle

# Introduction

Deadline-aware streams allow applications to specify deadlines for data transmission on specific streams. This enables the transport layer to make scheduling and retransmission decisions that aim to meet these deadlines, optimizing for latency-sensitive applications.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Motivation

The Multipath Extension of QUIC {{MP-QUIC}} enhances performance by utilizing multiple paths simultaneously, but it currently lacks mechanisms to guarantee data delivery within specific timeframes. Given the increasing demand for real-time applications such as teleoperation, live video streaming, and online gaming, there's a growing need for transport protocols that can efficiently handle strict latency requirements. Introducing deadline-aware streams to {{MP-QUIC}} could enable applications to meet those stringent latency constraints, optimizing for low-latency and high-reliability scenarios.

Additionally, the ability to have multiple paths using the same 4-tuple opens up the possibility of leveraging paths from path-aware networks like SCION, source routing, and others. This expands the pool of available paths beyond traditional IPv4 and IPv6 routes, potentially increasing the effectiveness of deadline-aware mechanisms like those proposed in the Deadline-aware Multipath Transport Protocol {{DMTP}}.

# Signaling Deadlines

To signal deadlines, endpoints use the DEADLINE_CONTROL frame (see {{deadline-control-frame}}). This frame associates a specific deadline with a stream, indicating the relative time by which the data should be delivered.

# Deadline Semantics

- Deadline Representation: Deadlines are represented as a relative time in milliseconds from the time the frame is sent.
- Stream Association: A deadline applies to a specific stream identified by its Stream ID
- Transport Behavior: Upon receiving a DEADLINE_CONTROL frame, the transport layer SHOULD attempt to schedule and retransmit packets carrying data for the specified stream to meet the indicated deadline.
- Retransmissions and Scheduling: Endpoints MAY implement custom schedulers and congestion controllers optimized for deadline-aware traffic, such as those based on DMTP concepts.

# Handling Missed Deadline

If the transport layer determines that the deadline cannot be met, it MAY choose to:

- Discard the data associated with the deadline-aware stream.
- Inform the application of the missed deadline.
- Continue delivering the data if it is still deemed useful.

The specific behavior is implementation-specific and MAY be configurable by the application.

## Custom Scheduler and Congestion Controller for Deadline-Aware Streams

Implementations that support deadline-aware streams SHOULD provide a custom scheduler and congestion controller that prioritize packets based on their deadlines. This includes:

- Path Selection: Dynamically selecting paths that are most likely to meet the deadlines based on real-time metrics like latency, bandwidth, and packet loss.
- Packet Prioritization: Prioritizing packets from streams with earlier deadlines.
- Adaptive Retransmissions: Deciding whether to retransmit lost packets based on their remaining time before the deadline.
- Forward Error Correction (FEC): Optionally integrating FEC mechanisms to reduce the need for retransmissions in networks with random losses.
- Deadline Miss Handling: Informing the application when a deadline cannot be met, allowing it to take appropriate action.

# Extension of Multipath Extension of QUIC

This extension is based on and aims to extend {{MP-QUIC}}. The contents of this extension will be specified in this section.

## Handshake Negotiation and Transport Parameter {#nego}

This extension defines a new transport parameter, used to negotiate the use of deadline-aware streams during the connection handshake, as specified in {{QUIC-TRANSPORT}}. The new transport parameter is defined as follows:

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

# Security Considerations

This extension retains all the security features and considerations of {{QUIC-TRANSPORT}}, {{QUIC-TLS}} and {{MP-QUIC}}.

# IANA Considerations

This document defines a new transport parameter for the negotiation of enable multiple paths for QUIC, and three new frame types. The draft defines provisional values for experiments, but we expect IANA to allocate short values if the draft is approved.

The following entry in {{transport-parameters}} should be added to the "QUIC Transport Parameters" registry under the "QUIC Protocol" heading.

Value                                         | Parameter Name.   | Specification
----------------------------------------------|-------------------|-----------------
TBD                                           | enable_deadline_aware_streams  | {{nego}}
{: #transport-parameters title="Addition to QUIC Transport Parameters Entries"}

The following frame type defined in {{frame-types}} should be added to
the "QUIC Frame Types" registry under the "QUIC Protocol" heading.

Value                                              | Frame Name          | Specification
---------------------------------------------------|---------------------|-----------------
TBD                                                  | DEADLINE_CONTROL | {{deadline-control-frame}}
{: #frame-types title="Addition to QUIC Frame Types Entries"}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
