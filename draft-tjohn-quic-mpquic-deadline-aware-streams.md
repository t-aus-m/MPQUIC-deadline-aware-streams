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
   MP-QUIC: I-D.draft-ietf-quic-multipath

informative:


--- abstract

This document proposes deadline aware streams to be added to Multipath QUIC in order to be able to deliver deadline sensitive information via QUIC over multiple paths.

--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Extension of {{MP-QUIC}}

## Handshake Negotiation and Transport Parameter

This extension defines a new transport parameter, used to negotiate the use of deadline-aware streams during the connection handshake, as specified in {{QUIC-TRANSPORT}}. The new transport parameter is defined as follows:

- enable_deadline_aware_streams (value TBD): A zero-length value that, if present, indicates that the endpoint supports deadline-aware streams.

Endpoints negotiate the use of deadline-aware streams by including the enable_deadline_aware_streams transport parameter in the handshake. Both endpoints MUST include this transport parameter to enable the use of deadline-aware streams. If an endpoint receives a DEADLINE_CONTROL frame without having negotiated support, it MUST treat this as a connection error of type PROTOCOL_VIOLATION



# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
