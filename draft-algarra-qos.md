---
title: "Quality of Service Extension Block for Bundle Protocol"
abbrev: "qos"
category: std

docname: draft-algarra-dtn-qos-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: Internet
workgroup: dtn
keyword:
  - dtn
  - bp
  - qos
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
  - fullname: Your Name Here
    organization: Your Organization Here
    email: your.email@example.com

normative:
  RFC9171: https://www.rfc-editor.org/info/rfc9171
  RFC2119: https://www.rfc-editor.org/info/rfc2119
  RFC8174: https://www.rfc-editor.org/info/rfc8174
  RFC9172: https://www.rfc-editor.org/info/rfc9172
  RFC8949: https://www.rfc-editor.org/info/rfc8949
---
# Abstract

This document defines a Quality of Service (QoS) Extension Block for the Bundle Protocol Version 7 (BPv7).The QoS Extension Block enables users to request and indicate QoS parameters, such as traffic prioritization, retransmission, latest-only delivery, and bundle retention preference.The purpose of this extension is to enhance the efficiency of bundle forwarding and delivery according to user requirements while maintaining interoperability with existing BP implementations.

--- middle

# Introduction

This document describes the structure of an extension block for Bundle Protocol Version 7 (BPv7) as defined in [RFC9171]. It can be added to a bundle to signal user-requested QoS parameters which indicate to the nodes how to handle the bundle based on the desired QoS characteristics.

QoS mechanisms enhance the efficiency of communications over
constrained networks by prioritizing traffic, reducing unnecessary retransmissions, and enabling context-aware bundle handling. In environments like space, latencies are high, contacts intermittent and bandwidth and storage limited, making QoS signaling essential to effective resource management.

BPv7 does not include intrinsic QoS signaling, so there is therefore a need for a standardized manner for QoS preferences to be indicated. The QoS parameters supported by this specification include:
- Traffic Prioritization: bundle forwarding precedence.
- Retransmission: desired use of reliable or unreliable convergence layers and underlying protocols.
- Latest-Only Delivery: indication to drop outdated information from the same flow.
- Bundle Storage: deletion order preference when node storage is full.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Description

## Definitions

- User: entity or set of entities operating under the same SLA using the network for sending data from a single or multiple sources to a single or multiple destinations.
- Priority class: level of urgency assigned to a bundle, helping to determine the order in which it should be forwarded.
- Sub-priority class: secondary level of prioritization used to further classify bundles within a main priority class, adding granularity based on relative urgency within a priority class.
- Transmission volume: aggregate total size of bundles transmitted across a link, measured in bytes.

## Extension Block Description

The User QoS Extension Block (UQEB) MUST follow the canonical bundle block format defined in [RFC9171].
The UQEB MUST be added only by the source node of a bundle.
The UQEB MUST NOT be modified by any other node.
Integrity and authentication MUST be ensured for the extension block. A Bundle Integrity Block (BIB) as defined in [RFC9172] MAY be used, unless an external system can guarantee the same results.

There will be an image here.

The block type code MUST be assigned.
The block processing control flags Bit 1, Bit 2 and Bit 4 MAY be set to ‘0’ to allow bundle nodes not supporting this extension block to pass it transparently.
The block-type-specific data MUST be encoded as a definite-length CBOR [RFC8949] byte string containing a definite-length CBOR map, with each map key identifying a QoS parameter, and the associated value specifying its setting.
A new IANA registry, "UQEB Parameters", is defined for the registration of QoS parameters and the associated values; see Section 5.1.
Each QoS parameter MAY appear if needed, but does not need to.
Each QoS parameter MUST NOT appear more than once.

There will be an image here.

## QoS Parameters

### Traffic Prioritization

Traffic prioritization (value 0 in the QoS Parameter Registry in Section 5.1) determines the forwarding order when multiple bundles are queued. 
A new IANA registry, "Traffic Prioritization", is defined for the registration of traffic prioritization classes and the associated values; see Section 5.2.
Bundles marked as expedited (value 0 in the Traffic Prioritization Registry in Section 5.2) MUST be forwarded before any other bundles from the same user.
Bundles marked normal (values 1 to 4 in the Traffic Prioritization Registry in Section 5.2) MUST be forwarded before bulk bundles from that user.
Bundles marked bulk (values 5 to 8 in the Traffic Prioritization Registry in Section 5.2) MUST NOT be forwarded if expedited or normal bundles from the user are present.
In the case of several bundles with the same priority class and the same sub-priority class, they SHOULD be forwarded in a First-In, First-Out (FIFO) basis. 
In the case of several bundles with the same priority class but different sub-priority classes, priority MUST decrease in ascending order.
A volume-based fair scheduling algorithm MAY be used to avoid data starvation.
Priorities SHOULD be used to inform the selection of parameters for underlying convergence layers if those implement priority schemes.
Bundles without a priority marking MUST be treated as having value 5 (default bulk).

### Retransmission

The retransmission parameter (value 1 in the QoS Parameter Registry in Section 5.1) defines whether an underlying protocol stack that provides retransmission shall be used.
A new IANA registry, "Retransmission", is defined for the registration of retransmission requirements and the associated values; see Section 5.3.
Bundles marked as retransmission required (value 1 in the Retransmission Registry in Section 5.3) MUST be dispatched via a protocol stack which applies retransmission in case of data loss.
Bundles marked as retransmission if possible (value 2 in the Retransmission Registry in Section 5.3) MAY be dispatched via a protocol stack which applies retransmission in case of data loss if currently available, and via a protocol stack which doesn’t if none is available.
Bundles marked as no retransmission allowed (value -1 in the Retransmission Registry in Section 5.3) MUST be dispatched via a protocol stack which does not apply retransmission in case of data loss.
Bundles marked as no retransmission if possible (value -2 in the Retransmission Registry in Section 5.3) SHOULD be dispatched via a protocol stack which does not apply retransmission in case of data loss if currently available, and via a protocol stack which does if none is available.
Bundles without a retransmission marking or where the indication is unspecified (value 0 in the Retransmission Registry in Section 5.3), the implementation SHOULD use the default option available.

### Latest-Only Delivery

Latest-only delivery (value 2 in the QoS Parameter Registry in Section 5.1) prevents the transmission of old information if newer information is available.
A new IANA registry, "Latest-Only Delivery", is defined for the registration of retransmission requirements and the associated values; see Section 5.4.
Bundles marked as all (value 0 in the Latest-Only Delivery Registry in Section 5.4) MUST proceed as usual.
Bundles marked as latest-only (value 1 in the Latest-Only Delivery Registry in Section 5.4) MUST be discarded if a bundle with the same source node ID, destination endpoint ID, and a newer creation time is available.
Bundles without a lates-only marking MUST be treated as having value 0 (all).
The latest bundle MUST be forwarded at the time the oldest discarded bundle would have been forwarded.

### Bundle Retention

The bundle retention parameter (value 3 in the QoS Parameter Registry in Section 5.1) indicates the deletion order if a node runs out of storage for the user.
A new IANA registry, "Bundle Retention", is defined for the registration of retransmission requirements and the associated values; see Section 5.5.
Bundles marked as a certain class MUST NOT be deleted if bundles with a higher class value are in the storage.
In case of several bundles of the same class, deletion MUST be performed starting with the shortest time to live left.
Bundles without a bundle retention marking MUST be treated as having value 12.



# Security Considerations

## Concerns About the Manipulation of the Extension Block

There is a concern that intermediate nodes may intentionally or unintentionally alter the information contained in the UQEB. This may result in bundles being forwarded in the incorrect order, or dropped inappropriately. Such manipulation could even be used to deny service to legitimate traffic or to gain preferential treatment for unauthorized bundles.
Therefore, the integrity and authenticity protection of the UQEB  MUST be ensured.
BPSec as specified in [RFC9172] SHOULD be used, unless an external system can provide the same degree of protection.

## Concerns About Malicious Behavior

The UQEB indicated desired handling and forwarding behavior, which could be exploited by malicious actors to monopolize network resources. For example, a malicious node could mark all bundles as expedited in order to monopolize link capacity, or repeatedly request retransmission in extremely asymmetric links, congesting the network segment.
To prevent this, implementations SHOULD include mechanisms such as authentication of the originating node, access controls that restrict use of certain QoS parameters, and rate-limiting on users marked as suspicious of being malicious or abusive.
Nodes SHOULD also apply local or network checks to determine whether a requesting user is authorized to receive particular QoS capabilities. If a UQEB is found to violate policy constraints, implementations SHOULD ignore the requested handling.

# IANA Considerations

The following sections detail the creation of five new IANA registries:

## QoS Parameters Registry

| QoS Parameter | Key |
| ----------- | ----------- |
| Traffic Prioritization | 0 |
| Retransmission | 1 |
| Latest-Only Delivery | 2 |
| Bundle Retention | 3 |
| Reserved for Future Expansion | 4..11 |
| Private Use | 12..23 |
| Reserved for Future Expansion | 24..2^64-1 |

## Traffic Prioritization Registry

| Priority Class | Range |
| ----------- | ----------- |
| Expedited | 0 |
| Normal | 1..4 |
| Bulk | 5..8 |
| Reserved for Future Expansion | 9..15 |
| Private Use | 16..23 |
| Reserved for Future Expansion | 24..2^64-1 |

## Retransmission Registry

| Retransmission Requirement | Range |
| ----------- | ----------- |
| Reserved for Future Expansion | -2^64+1..-24 |
| Private Use | -24..-13 |
| Reserved for Future Expansion | -12..-3 |
| No Retransmission If Possible | -2 |
| No Retransmission Allowed | -1 |
| Reserved | 0 |
| Retransmission Required| 1 |
| Retransmission If Possible| 2 |
| Reserved for Future Expansion | 3..12 |
| Private Use | 13..23 |
| Reserved for Future Expansion | 24..2^64-1 |

## Latest-Only Delivery Registry

| Information Validity | Range |
| ----------- | ----------- |
| All | 0 |
| Latest-Only | 1 |
| Reserved for Future Expansion | 2..12 |
| Private Use | 13..23 |
| Reserved for Future Expansion | 24..2^64-1 |

## Bundle Retention Registry

| Information Validity | Range |
| ----------- | ----------- |
| Class 0 | 0 |
| Class 1 | 1 |
| Class 2 | 2 |
| … | … |
| Class 22 | 22 |
| Class 23 | 23 |
| Reserved for Future Expansion | 24..2^64-1 |

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
