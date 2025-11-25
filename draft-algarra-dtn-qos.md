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
  RFC9172: https://www.rfc-editor.org/info/rfc9172
  RFC8949: https://www.rfc-editor.org/info/rfc8949
--- abstract

This document defines a Quality of Service (QoS) Extension Block for the Bundle Protocol Version 7 (BPv7). The QoS Extension Block enables users to request and indicate QoS parameters, such as traffic prioritization, retransmission, latest-only delivery, and bundle retention preference. The purpose of this extension is to enhance the efficiency of bundle forwarding and delivery according to user requirements while maintaining interoperability with existing BP implementations.

--- middle

# Introduction

This document describes the structure of an extension block for Bundle Protocol Version 7 (BPv7) as defined in [RFC9171]. It can be added to a bundle to signal user-requested QoS parameters which indicate to the nodes how to handle the bundle based on the desired QoS characteristics.

QoS mechanisms enhance the efficiency of communications over
constrained networks by prioritizing traffic, reducing unnecessary retransmissions, and enabling context-aware bundle handling. In environments like space, latencies are high, contacts intermittent and bandwidth and storage limited, making QoS signaling essential to effective resource management.

BPv7 does not include intrinsic QoS signaling, so there is therefore a need for a standardized manner for QoS preferences to be indicated. The QoS parameters supported by this specification include:

- Traffic Priority: bundle forwarding precedence.
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

### Traffic Priority

Traffic Priority (value 0 in the QoS Parameter Registry in Section 5.1) determines the forwarding order when multiple bundles are queued.
A new IANA registry, "QoS Traffic Priority", is defined for the registration of traffic prioritization classes and the associated values; see Section 5.2.
Bundles marked as expedited (value 0 in the QoS Traffic Priority Registry in Section 5.2) MUST be forwarded before any other bundles from the same user.
Bundles marked normal (values 1 to 4 in the QoS Traffic Priority Registry in Section 5.2) MUST be forwarded before bulk bundles from that user.
Bundles marked bulk (values 5 to 8 in the QoS Traffic Priority Registry in Section 5.2) MUST NOT be forwarded if expedited or normal bundles from the user are present.
In the case of several bundles with the same priority class and the same sub-priority class, they SHOULD be forwarded in a First-In, First-Out (FIFO) basis.
In the case of several bundles with the same priority class but different sub-priority classes, priority MUST decrease in ascending order.
A volume-based fair scheduling algorithm MAY be used to avoid data starvation.
Priorities SHOULD be used to inform the selection of parameters for underlying convergence layers if those implement priority schemes.
Bundles without a priority marking MUST be treated as having value 5 (default bulk).

### Retransmission

The retransmission parameter (value 1 in the QoS Parameter Registry in Section 5.1) defines whether an underlying protocol stack that provides retransmission shall be used.
A new IANA registry, "QoS Retransmission Requirement", is defined for the registration of retransmission requirements and the associated values; see Section 5.3.
Bundles marked as retransmission required (value 1 in the QoS Retransmission Requirement in Section 5.3) MUST be dispatched via a protocol stack which applies retransmission in case of data loss.
Bundles marked as retransmission if possible (value 2 in the QoS Retransmission Requirement in Section 5.3) MAY be dispatched via a protocol stack which applies retransmission in case of data loss if currently available, and via a protocol stack which doesn’t if none is available.
Bundles marked as no retransmission allowed (value -1 in the QoS Retransmission Requirement in Section 5.3) MUST be dispatched via a protocol stack which does not apply retransmission in case of data loss.
Bundles marked as no retransmission if possible (value -2 in the QoS Retransmission Requirement in Section 5.3) SHOULD be dispatched via a protocol stack which does not apply retransmission in case of data loss if currently available, and via a protocol stack which does if none is available.
Bundles without a retransmission marking or where the indication is unspecified (value 0 in the QoS Retransmission Requirement in Section 5.3), the implementation SHOULD use the default option available.

### Latest-Only Delivery

Latest-only delivery (value 2 in the QoS Parameter Registry in Section 5.1) prevents the transmission of old information if newer information is available.
A new IANA registry, "QoS Latest-Only Delivery", is defined for the registration of retransmission requirements and the associated values; see Section 5.4.
Bundles marked as all (value 0 in the QoS Latest-Only Delivery Registry in Section 5.4) MUST proceed as usual.
Bundles marked as latest-only (value 1 in the QoS Latest-Only Delivery Registry in Section 5.4) MUST be discarded if a bundle with the same source node ID, destination endpoint ID, and a newer creation time is available.
Bundles without a lates-only marking MUST be treated as having value 0 (all).
The latest bundle MUST be forwarded at the time the oldest discarded bundle would have been forwarded.

### Bundle Retention

The bundle retention parameter (value 3 in the QoS Parameter Registry in Section 5.1) indicates the deletion order if a node runs out of storage for the user.
A new IANA registry, "QoS Bundle Retention", is defined for the registration of retransmission requirements and the associated values; see Section 5.5.
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

IANA is requested to create a new registry entitled "QoS Parameters", in the existing "Bundle Protocol" registry group.  The registration procedures for this registry, using terms defined in {{!RFC8126}}, is:

| Values | Registration Procedure |
|:-------:|:------|
| 0..11 | Standards Action |
| 12..23 | Private Use |
| 24..2^64-1 | Reserved for future expansion |
{: #tab-qos-parameters-reg align="left" title="QoS Parameters registration policies"}

The initial values for the registry are:

| Type | Parameter | Reference |
|:------:|:------------|:------|
| 0 | [Traffic Priority](#traffic-priority)  | This document |
| 1 | [Retransmission Requirement](#retransmission)  | This document |
| 2 | [Latest-only Delivery](#latest-only-delivery) | This document |
| 3 | [Bundle Retention](#bundle-retention) | This document |
{: #tab-qos-parameters-vals align="left" title="QoS Parameters initial values"}

## QoS Traffic Priority Registry

IANA is requested to create a new registry entitled "QoS Traffic Priority", in the existing "Bundle Protocol" registry group.  The registration procedures for this registry, using terms defined in {{!RFC8126}}, is:

| Values | Registration Procedure |
|:-------:|:------|
| 0..15 | Standards Action |
| 16..23 | Private Use |
| 24..2^64-1 | Reserved for future expansion |
{: #tab-traffic-priority-reg align="left" title="QoS Traffic Priority registration policies"}

The initial values for the registry are:

| Values | Priority Class | Reference |
|:------:|:-----|:------|
| 0 | Expedited | This document |
| 1..4 | Normal | This document |
| 5..8 | Bulk | This document |
{: #tab-traffic-priority-vals align="left" title="QoS Traffic Priority initial values"}

## QoS Retransmission Requirement Registry

IANA is requested to create a new registry entitled "QoS Retransmission Requirement", in the existing "Bundle Protocol" registry group.  The registration procedures for this registry, using terms defined in {{!RFC8126}}, is:

| Values | Retransmission Requirement |
|:-----------:|:-----------|
| -2^64+1..-24 | Reserved for future expansion |
| -24..-13 | Private Use |
| -12..12 | Standards Action |
| 13..23 | Private Use |
| 24..2^64-1 | Reserved for future expansion |
{: #tab-retrans-req-reg align="left" title="QoS Retransmission Requirement registration policies"}

The initial values for the registry are:

| Value | Retransmission Requirement | Reference |
|:------:|:-----|:------|
| -2 | No Retransmission If Possible | This document |
| -1 | No Retransmission Allowed | This document |
| 0 | Reserved | This document |
| 1 | Retransmission Required | This document |
| 2 | Retransmission If Possible | This document |
{: #tab-retrans-req-vals align="left" title="QoS Retransmission Requirement initial values"}

## QoS Latest-Only Delivery Registry

IANA is requested to create a new registry entitled "QoS Latest-Only Delivery", in the existing "Bundle Protocol" registry group.  The registration procedures for this registry, using terms defined in {{!RFC8126}}, is:

| Values | Registration Procedure |
|:-------:|:------|
| 0..12 | Standards Action |
| 13..23 | Private Use |
| 24..2^64-1 | Reserved for future expansion |
{: #tab-delivery-reg align="left" title="QoS Latest-Only Delivery registration policies"}

The initial values for the registry are:

| Value | Information Validity | Reference |
|:-----------:|:-----------|:---------|
| 0 | All | This document |
| 1 | Latest-Only | This document |
{: #tab-delivery-vals align="left" title="QoS Latest-Only Delivery initial values"}

## QoS Bundle Retention Registry

IANA is requested to create a new registry entitled "QoS Bundle Retention", in the existing "Bundle Protocol" registry group.  The registration procedures for this registry, using terms defined in {{!RFC8126}}, is:

| Values | Registration Procedure |
|:-------:|:------|
| 0..23 | Standards Action |
| 24..2^64-1 | Reserved for future expansion |
{: #tab-bundle-ret-reg align="left" title="QoS Bundle Retention registration policies"}

The initial values for the registry are:

| Value | Bundle Retention Class | Reference |
|:-----------:|:-----------|:---------|
| 0 | Class 0 | This document |
| 1 | Class 1 | This document |
| 2 | Class 2 | This document |
| 3 | Class 3 | This document |
| 4 | Class 4 | This document |
| 5 | Class 5 | This document |
| 6 | Class 6 | This document |
| 7 | Class 7 | This document |
| 8 | Class 8 | This document |
| 9 | Class 9 | This document |
| 10 | Class 10 | This document |
| 11 | Class 11 | This document |
| 12 | Class 12 | This document |
| 13 | Class 13 | This document |
| 14 | Class 14 | This document |
| 15 | Class 15 | This document |
| 16 | Class 16 | This document |
| 17 | Class 17 | This document |
| 18 | Class 18 | This document |
| 19 | Class 19 | This document |
| 20 | Class 20 | This document |
| 21 | Class 21 | This document |
| 22 | Class 22 | This document |
| 23 | Class 23 | This document |
{: #tab-bundle-ret-vals align="left" title="QoS Bundle Retention initial values"}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
