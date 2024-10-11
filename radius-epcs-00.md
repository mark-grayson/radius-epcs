---
title: RADIUS attributes for National Security and Emergency Preparedness Service
abbrev: RADIUS EPCS
docname: draft-gundavelli-radepcs-00
date: 2024-10-11
category: std
submissiontype: IETF

ipr: trust200902
area: general
workgroup: RADEXT Working Group

keyword:
  - Internet-Draft
  - RADIUS
  - EPCS
  - Emergency
  - Priority

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:

-
  ins: S. Gundavelli
  name: Sri Gundavelli
  organization: Cisco Systems
  street: 170 West Tasman Drive
  city: San Jose
  code: 95134
  country: US
  email: sgundave@cisco.com

-
  ins: S. Das
  name: Subir Das
  organization: Peraton Labs
  country: US
  email: sdas@peratonlabs.com

-
   ins: M. Grayson
   name: Mark Grayson
   organization: Cisco Systems
   street: 10 New Square Park
   city: Feltham
   code: TW14 8HA
   country: UK
   email: mgrayson@cisco.com

normative:
  RFC2119:
  RFC8174:

informative:
  RFC2865:
  CSRC8-WG4:
    title: REPORT ON 911 SERVICE OVER WI-FI
    target: https://www.fcc.gov/sites/default/files/CSRIC8-Report-911overWi-Fi032123.pdf
    author:
      ins: COMMUNICATIONS SECURITY, RELIABILITY, AND INTEROPERABILITY COUNCIL VIII WG4
    date: 2023-03
  CSRC5-WG8:
    title: PRIORITY SERVICES
    target: https://www.fcc.gov/sites/default/files/CSRIC5-WG8-FinalReport031517.pdf
    author:
      ins: COMMUNICATIONS SECURITY, RELIABILITY, AND INTEROPERABILITY COUNCIL V WG8
    date: 2017-03
  FCC:
    title: Review of Rules and Requirements For Priority Services
    target: https://docs.fcc.gov/public/attachments/FCC-22-36A1_Rcd.pdf
    author:
      ins: Federal Communications Comission
    date: 2022-05-19

--- abstract

This document describes RADIUS attributes for supporting
authorization of Emergency Preparedness Communication
Service (EPCS), enabling users to benefit from
access to Wi-Fi based Multimedia Priority Services
Communications.

--- middle

Introduction        {#intro}
============

National Security/Emergency Preparedness priority communications services are operational in various countries around the world. While these have traditionally been only supported over cellular networks, the recent addition of Emergency Preparedness Communication Service (EPCS) to Wi-Fi 7 means that Wi-Fi now supports Multimedia Priority Services Communications (MPS). This specifies preferred or prioritized channel access (PCA) over public/private Wi-Fi networks in emergency service (ES) scenarios.

Whereas Wi-Fi 7 defines the interactions between the Wi-Fi Access Point (AP) and the Wi-Fi Station device for prioritized channel access, it does not provide an end-to-end architecture for enabling the EPCS on a per device basis.

CISRCVIII WG4 recommendations for FCC provide opportunities for regulatory bodies, like the FCC, to assess and promote Wi-Fi's role in emergency services {{CSRC8-WG4}}. This involves shaping rules, clarifying obligations, and exploring methods for handling Wi-Fi-enabled 911 calls, ultimately enhancing Wi-Fi's role in critical communications.

Earlier deliverables from CISRCV WG8 have defined requirements for the role of the network within the priority services platform to enforce priority levels on traffic associated with priority users {{CSRC5-WG8}}. There are number of factors critical to this enforcement function:

* The priority user must be authenticated and authorized to receive priority treatment
* The network must be able to uniquely identify priority user traffic and associate the authorized level of priority to that traffic
* The network must then have prioritization means to apply to the identified traffic.
* For cases where networks interconnect, traffic prioritization indicators must be securely passed to interconnected networks for downstream prioritization.

This document specifies an extension to the Remote Authentication
Dial-In User Service (RADIUS) protocol {{RFC2865}} that enables
a Wi-Fi NAS to indicate it is EPCS capable, as well as enabling
a RADIUS server to indicate to a NAS that an authenticated user
is authorized to receive the EPCS service.

Requirements Language          {#Requirements}
-----------
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL",
"SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT
RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

Terminology          {#Terminology}
-----------

Authorizing agent:

  Refers to a Federal or State entity that authenticates, evaluates, and makes
  recommendations to DHS regarding the assignment of priority levels.

Service user:

  Mmeans an individual or organization to whom or which a priority access assignment
  has been made.

Wireless service provider:

  Refers to a provider of a wireless communications
  service or Internet Protocol-based service, including commercial or private mobile service. The
  term includes agents of the licensed provider and resellers of wireless service.

Overiew of EPCS Provisioning {#Provisioning}
===============

This section provides background on how EPCS service is currently authorized
and how that can be adapted to support Wi-Fi use cases. The service user
engages with an authorizing agent to request the provision of priority service
on a service provider device. The service provider, in this case a cellular
operator, will receive an indication from the authorizing agent that a
subscriber has been authorized for priority services and will stored such
an indication in the Home Subscriber Server (HSS).

In the case that the cellular operator is also operating as a Wi-Fi Ideneity Provider,
the operator can mirror the authorization information in their Wi-Fi AAA system, as
illustrated in {{figprovisioning}}.

~~~~~~~~~~

+---------+    +------------+    +--------------+    +-----------+
|  Wi-Fi  |    |EPCS Enabled|    |   Cellular   |    | Priority  |
|Priority |    |Wi-Fi Access|    |Operator/Wi-Fi|    |  Service  |
| Service |    |  Network   |    |   Identity   |    |Authorizing|
|  User   |    |            |    |   Provider   |    |  Entity   |
+---------+    +------------+    +--------------+    +-----------+
     |               |                   |                 |
     |1. Standard service user authorization for priority  |
     |   Service (unchanged)             |                 |
     |<===================================================>|
     |               |                   |                 |
     |               |                   |2. Receive Priority
     |               |                   |   Authorization |
     |               | 3. Mirror HSS     |   (unchanged)   |
     |               |    Authorization  |<===============>|
     |               |    in Wi-Fi AAA   |                 |
     |               |         //========|                 |
     |               |         ||        |                 |
     |               |         \\=======>|                 |
     |               |                   |                 |
     |               |                   |                 |

~~~~~~~~~~
{: #figprovisioning title="Provisioning Priority Services over Wi-fi"}


EPCS Authorization RADIUS Exchange {#Authorization}
===============

The top level RADIUS Authorization flows for an EPCS Service User is
illustrated in {{figauthorization}}.

~~~~~~~~~~

+---------+    +------------+    +--------------+    +-----------+
|  Wi-Fi  |    |EPCS Enabled|    |   Cellular   |    | Priority  |
|Priority |    |Wi-Fi Access|    |Operator/Wi-Fi|    |  Service  |
| Service |    |  Network   |    |   Identity   |    |Authorizing|
|  User   |    |            |    |   Provider   |    |  Entity   |
+---------+    +------------+    +--------------+    +-----------+
     |               |                   |                 |
     |1. EAP-Identity/                   |                 |
     |   Request     |                   |                 |
     |<==============|                   |                 |
     |2. EAP-Identity/                   |                 |
     |   Response    |                   |                 |
     |==============>|3. RADIUS Access-Request             |
     |               |   EPCS-Capable-Indication           |
     |               |==================>|                 |
     |               |                   |                 |
     |       4. EAP Dialogue             |                 |
     |<=================================>|5. Match user with
     |               |                   |   priority      |
     |               |                   |   subscription  |
     |               |                   |======\          |
     |               |                   |      ||         |
     |               |                   |<=====/          |
     |               |6. RADIUS Access-Accept              |
     |               |   EPCS-Subscription-Info            |
     |               |<==================|                 |
     |7. EAP-SUCCESS |                   |                 |
     |<==============|                   |                 |
     |               |                   |                 |

~~~~~~~~~~
{: #figauthorization title="Authorizing Priority Services over Wi-fi"}


RADIUS EPCS Attributes {#attributes}
==================

EPCS-Capable-Indication {#EPCSCI}
--------------

Description

The EPCS-Capable-Indication (TBA1) Attribute allows a RADIUS
NAS to indicate to a RADIUS server that it is EPCS capable.

One EPCS-Capable-Indication Attribute MAY be included in
an Access-Request packet.

A summary of the EPCS-Capable-Indication Attribute format is
shown in {{attr-epcs-cap}}. The fields are transmitted from left to right.

~~~~~~~~~~
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |  Length       |           Value               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

~~~~~~~~~~
{: #attr-epcs-cap title="Encoding EPCS-Capable-Indication Attribute"}

Type

>> TBA1

Length

>> 6 octet

Data Type

>> Integer

Value

>  The field is 2 octets, containing a 16-bit unsigned integer that
represents whether the NAS supports EPCS and how the EPCS service
can be invoked. This document defines three values used with the
EPCS-Capable-Indication attribute:

>>>    0 &nbsp; &nbsp; The NAS does not support EPCS.

>>>    1 &nbsp; &nbsp; The provider of the NAS has elected to provide
        support for priority services to authorized users at all times {{FCC}}.

>>>    2 &nbsp; &nbsp; The provider of the NAS has elected to provide
        support for priority services on a per EPCS-flow basis.


EPCS-Subscription-Info {#EPCSSI}
--------------

Description

The EPCS-Subscription-Info (TBA2) Attribute allows a RADIUS
Server to indicate to a NAS that a user is authorized
to receive priority servie.

One EPCS-Subscription-Info Attribute MAY be included in
an Access-Accept packet and its presence indicates the
authenticated user is authorized to receive priority service.

A summary of the EPCS-Subscription-Infon Attribute format is
shown in {{attr-epcs-sub}}. The fields are transmitted from left to right.

~~~~~~~~~~
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |  Length       |        Priority-level         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

~~~~~~~~~~
{: #attr-epcs-sub title="Encoding EPCS-Subscription-Info Attribute"}

Type

>> TBA2

Length

>> 6 octet

Data Type

>> Integer

Value

>  The field is 2 octets, containing a 16-bit unsigned integer that
represents whether the priority level associated with the user's
subscription.


Security Considerations {#Security}
==================

To be completed.


IANA Considerations {#IANA}
==================

To be completed.

--- back



# Acknowledgements {#Acknowledgements}
{: numbered="false"}

The authors would like to thank all the members of the Wireless Boradband Alliance's
Mission Critical & Emergency Services (e911) project.
