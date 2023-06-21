---
title: "A Profile for Binding Messages (BMs)"
abbrev: "Binding Messages"
category: std

docname: draft-xu-fc-binding-message-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
# keyword:
#  - next generation
#  - unicorn
#  - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "BasilGuo/fc-binding-message"
  latest: "https://BasilGuo.github.io/fc-binding-message/draft-xu-fc-binding-message.html"

author:
  -
      fullname: Ke Xu
      org: Tsinghua University
      city: Beijing
      country: China
      email: xuke@tsinghua.edu.cn
  -
      fullname: Xiaoliang Wang
      org: Tsinghua University
      city: Beijing
      country: China
      email: wangxiaoliang0623@foxmail.com
  -
      fullname: Yangfei Guo
      org: Zhongguancun Labratory
      city: Beijing
      country: China
      email: guoyangfei@zgclab.edu.cn
  -
      fullname: Jiangou Zhan
      org: Tsinghua University
      city: Beijing
      country: China
      email: "904542587@qq.com" # TODO: use your edu.cn email
  -
      fullname: Jianping Wu
      org: Tsinghua University
      city: Beijing
      country: China
      email: jianping@cernet.edu.cn

normative:
    RFC3779:
    RFC4271:
    RFC5652:
    RFC6480:
    RFC6485:
    RFC6488:
    RFC7908:

informative:
    FC:
      title: "A Profile for Forwarding Commitments (FCs)"
      target: "https://github.com/BasilGuo/fc-signed-object/"
      date: Jun. 2023
      author:
          - name: Ke Xu
          - name: Xiaoliang Wang
          - name: Yangfei Guo
          - name: Jiangou Zhan
          - name: Jianping Wu
    X.680:
      title: "Information technology -- Abstract Syntax Notation One (ASN.1): Specification of basic notation"
      target: "https://itu.int/rec/T-REC-X.680-202102-I/en"
      date: Feb. 2021
      author:
        - ins: ITU-T
      seriesinfo: "Recommendation ITU-T X.680"
    X.690:
      title: "Information technology - ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)"
      target: "[https://itu.int/rec/T-REC-X.680-202102-I/en](https://www.itu.int/rec/T-REC-X.690-202102-I/en)"
      date: Feb. 2021
      author:
        - ins: ITU-T
      seriesinfo: "Recommendation ITU-T X.690"

--- abstract

This document defines a standard profile for Binding Messages (BM) used in Resource Public Key Infrastructure (RPKI). A BM is a digitally signed object that provides a means of verifying that an IP address prefix announced from `AS a` to `AS b` and selected by `AS b`. When validated, a BM's eContent can be used for the detection and mitigation of route hijacking and provide protection for the AS_PATH attribute in BGP-UPDATE.


--- middle

# Introduction

The Border Gateway Protocol (BGP) {{RFC4271}} was designed with no mechanisms to validate the security of BGP attributes. There are two types of BGP security issues, BGP Hijacks and BGP Route Leaks {{!RFC7908}}, plagues Internet security.

The primary purpose of the Resource Public Key Infrastructure (RPKI) is to improve routing security (See {{RFC6480}} for more information). As part of this system, a mechanism is needed to allow a BGP router to select and verify that a prefix is announced from and followed the path that it said itself. And the router could also filter traffic according to this policy in the data plane. A Binding Message provides this function.

Binding Message (BM) is a signed object that binds the IP prefix with its announced AS's Paths, eventually, it could compose, verify, and protect the forwarding path of traffic. It should together use with Forwarding Commitments {{FC}} which is a signed object used in the control plane to protect BGP routing. BM is used for concatenating all ASes on the announced path. It means that FC should provide the relationship between one IP prefix in a BGP-UPDATE message and the original announced AS, while BM should list all the generated FCs for one IP prefix in a BGP-UPDATE message. The generated Binding Message is split into two parts, one for the BM body which is a newly signed object, and the other for the FCList which includes the BMID as well as a series of FCID {{FC}}. The FCList would be sent to all ASes in the reverse direction of related IP prefix propagation.

The BM uses the template for RPKI digitally signed objects {{RFC6488}} for the definition of a Cryptographic Message Syntax (CMS) {{RFC5652}} wrapper for the BM content as well as a generic validation procedure for RPKI signed objects.  As BM needs to be validated with RPKI certificates issued by the current infrastructure, we assume the mandatory-to-implement algorithms in {{RFC6485}} or its successor.

To complete the specification of the BM (see {{Section 4 of RFC6488}}), this document defines:

1.  The object identifier (OID) that identifies the BM signed object. This OID appears in the eContentType field of the encapContentInfo object as well as the content-type signed attribute within the signerInfo structure.
2.  The ASN.1 syntax for the BM content, which is the payload signed by the BGP speaker. The BM content is encoded using the ASN.1 {{X.680}} Distinguished Encoding Rules (DER) {{X.690}}.
3.  The steps required to validate a BM beyond the validation steps specified in {{RFC6488}}.



## Requirements Language

{::boilerplate bcp14-tagged}

# The BM Content-Type

The content-type for a BM is defined as BindingMessage and has the numerical value of 1.2.840.113549.1.9.16.1.(TBD).

This OID MUST appear both within the eContentType in the encapContentInfo object as well as the content-type signed attribute in the signerInfo object (see {{RFC6488}}).

# The BM eContent

The content of a BM identifies a binding message and forwarding binding that an AS has selected one AS path to announce to other nodes upon receiving a BGP-UPDATE message. Other ASes that are on-path can validate the BM and perform path verification for traffic forwarding based on the AS-path information. Off-path ASes can utilize this BM for collaborative filtering. a BM is an instance of BindingMessageAttestation, formally defined by the following ASN.1 {{X.680}} module:

~~~~~~
RPKI-BM-2023
  { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
     pkcs-9(9) smime(16) modules(0) id-mod-rpki-BM-2023(TBD) }

DEFINITIONS EXPLICIT TAGS ::=
BEGIN

IMPORTS
  CONTENT-TYPE
  FROM CryptographicMessageSyntax-2010 -- RFC 6268
    { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
      pkcs-9(9) smime(16) modules(0) id-mod-cms-2009(58) } ;

id-ct-BM OBJECT IDENTIFIER ::= { iso(1) member-body(2) us(840)
    rsadsi(113549) pkcs(1) pkcs-9(9) id-smime(16) id-ct(1) bm(TDB) }

ct-BM CONTENT-TYPE ::=
    { TYPE ForwardingCommitmentAttestation IDENTIFIED BY id-ct-BM }

BindingMessageAttestation ::= SEQUENCE {
    version [0]         INTEGER DEFAULT 0,
    asidS               ASID,
    prefixS             SEQUENCE (SIZE(1..MAX)) OF Prefix,
    prefixD             SEQUENCE (SIZE(1)) OF Prefix,
    bm                  BindingMessage
}

ASID ::= INTEGER (0..4294967295)

Prefix ::= SEQUENCE {
    afi                 AFI,
    address             IPAddress,
    prefixLength        INTEGER (SIZE(0..128))}

AFI ::= OCTET STRING (SIZE(2))

IPAddress ::= BIT STRING (SIZE(0..128))

BindingMessage ::= SEQUENCE {
    id                  BIT STRING
    signature           BIT STRING }

END
~~~~~~
{: #fig-eContentFC title="eContent of BM signed object"}

Note that this content appears as the eContent within the encapContentInfo (see {{RFC6488}}).

## version

The version number of the BindingMessageAttestation MUST be 0.

## asidS

The asidS field contains the AS number of the AS that generates the BM signed object.

## Type Prefix

Within the Prefix structure, the prefix field encodes the set of IP address prefixes.

### Element afi

Within the Prefix structure, afi contains the Address Family Identifier of an IP address family. This specification only supports IPv4 and IPv6. Therefore, afi MUST be either 0001 or 0002.

### Element address

The address field contains the IP address.

### Element prefixLength

The prefixLength field contains the length of the IP address prefix.

## prefixS

This field is a set of IP prefixes that belongs to the BM generating AS according to its local routing policy. Thus, these IP prefixes are the BM generating AS would like to use when it communicates with the prefix in the BGP-UPDATE message.

## prefixD

This field is only one IP prefix that contains in the BGP-UPDATE message and binds an FC signed object.


## Type BindingMessage

Within the BindingMessage structure, the bm field encodes the binding message generated by this current AS and will be validated by other AS.

### Element id

The id field contains the hash of the previous AS-path in the associated BGP-UPDATE plus the self AS number and above prefixD filed.

### Element signature

The signature field is a signature signed by the BGP speaker who issues this BM. This field should also sign the contents of FCList which would not be synchronized by RPKI.


#  BM Validation

Before a relying party can sign a new BM to announce it has trusted and selected the forwarding path, the relying party MUST first validate the BM signed object itself. To validate a BM, the relying party MUST perform all the validation checks specified in [RFC6488] as well as the following additional BM-specific validation step.

<!-- TODO: I don't know if it should be prefixS or prefixD. Need further discussion. -->
The IP Address Delegation extension [RFC3779] is present in the end-entity (EE) certificate (contained within the BM), and the IP address prefix (just only prefixS) in the BM payload is contained within the set of IP addresses specified by the EE certificate's IP Address Delegation extension.
The EE certificate's IP Address Delegation extension MUST NOT contain "inherit" elements described in [RFC3779].
The Autonomous System Identifier Delegation Extension described in [RFC3779] is also used in Binding Message and MUST be present in the EE certificate.


#  BM-based BGP Path Verification

## FCList Generation

FCList contains a BMID as well as a set of FCIDs that relate to a Binding Message. FCList has a one-to-one relationship with BM. That means the BGP router generates BM and FCList as a pair, but they would not all be placed at RPKI.

As it is private data that consist of the AS_PATH information, FCList would not be synchronized through RPKI. Otherwise, adversarial ASes may lunch an attack and get the path information.

It SHOULD use TCP and Transport Layer Security (TLS) or other end-to-end communication techniques to send the FCList to on-path ASes in an off-pathway. And these ASes MUST store receiving FCLists until RPKI is synchronized and relative BM is verified.

<!-- We MAY need another draft to describe it -->
The process of FCList is TBD.

The generation time of FCList is the same as the BM generation time.

By FCList, we also could get a path-aware network.

##  BM Generation

When one AS establishes a connection with its peer, it announces its route to this peer. When one BGP router receives one BGP-UPDATE message, it would generate the FC and BM signed object for each IP prefix in the BGP-UPDATE message. The generation time of BM is the same as FC, they are both generated after the process of BGP path selection.

The process of FC generation is in {{FC}} from which one can get more information.

The process of the generator of BM SHOULD be split into two parts,

- generation of BM which is the signed object,
- and generation of FCList which contains a BMID and a set of FCIDs.

Taking the following topology as an example, there are 4 ASes and the topology of them is in a line as {{fig-example}} shows.

~~~~~~
+----------+     +----------+     +----------+     +----------+
| AS 65536 | --- | AS 65537 | --- | AS 65538 | --- | AS 65539 |
+----------+     +----------+     +----------+     +----------+
~~~~~~
{: #fig-example title="A topology example of 3 AS"}

Suppose that the simplified RIB table of AS 65536 is as follows. That means there is 1 BGP-UPDATE that is announced from AS 65539 to AS 65538, 1 BGP-UPDATE that is announced from AS 65538 to AS 65537, and 1 BGP-UPDATE that is announced from AS 65537 to AS 65536.


~~~~~~
Network           Next Hop        Path
2001:db8:a::/48   ::              i
2001:db8:b::/48   2001:db8:a::1   65537 i
2001:db8:c::/48   2001:db8:a::1   65537 65538 i
2001:db8:d::/48   2001:db8:a::1   65537 65538 65539 i
~~~~~~
{: #fig-rib-of-AS65536 title="Simplified RIB table of AS 65536"}

Let's take the announcement of the prefix 2001:db8:d::/48 as an example.

First, the BGP router of AS 65539 generates one FC `FC_{65539, 65538}` when it announces the prefix to AS 65538.

Then, the BGP router in AS 65538 would generate one FC `FC_{65539, 65538, 65537}`, one BM, and one FCList after receiving the BGP-UPDATE message. The newly generated FC and BM would be placed at the RPKI repository, but the FCList which contains the ASID of AS 65538 and the FCID of the FC, i.e. `{BMID1, {FC_{65539, 65538}_id}}`, would be sent to the ASes on the BGP-UPDATE propagation path, which means the FCList would be sent to AS 65539. And the BGP-UPDATE would also announce to AS 65537. For BM generated by the BGP router of AS 65538, it fills the eContent of BM signed object as follows:

~~~~~~
BM_{65538, 65539}:
  version: 0
  asidS: 65538
  prefixS:
    afi: 2,
    address: 2001:db8:c::
    prefixLength: 48
  prefixD:
    afi: 2
    address: 2001:db8:d::
    prefixLength: 48
  BindingMessage:
    id: HASH({65539}, 65538, 2001:db8:d::/48) -- as BMID1
    signature: SIGNATURE1
~~~~~~


Next, the BGP router in AS 65537 would generate one FC `FC_{65539, 65538, 65537, 65536}`, one BM, and one FCList after receiving the BGP-UPDATE message. The newly generated FC and BM would be placed at the RPKI repository, but the FCList which contains the ASID of AS 65537 and the FCIDs of the two FCs, i.e. `{BMID2, {FC_{65539, 65538}_id, FC_{65539, 65538, 65537}_id}`, would be sent to the ASes on the BGP-UPDATE propagation path, which means the FCList would be sent to AS 65538 and AS 65539 independently. And the BGP-UPDATE would also announce to AS 65536. For BM generated by the BGP router of AS 65537, it fills the eContent of BM signed object as follows:

~~~~~~
BM_{65537, 65538, 65539}:
  version: 0
  asidS: 65537
  prefixS:
    afi: 2,
    address: 2001:db8:b::
    prefixLength: 48
  prefixD:
    afi: 2
    address: 2001:db8:d::
    prefixLength: 48
  BindingMessage:
    id: HASH({65539, 65538}, 65537, 2001:db8:d::/48) -- as BMID2
    signature: SIGNATURE2
~~~~~~

Finally, the BGP router in AS 65536 would generate one BM and one FCList after receiving the BGP-UPDATE message. The newly generated BM would be placed at the RPKI repository, but the FCList which contains the ASID of AS 65536 and the FCIDs of the three FCs, i.e. `{BMID3, {FC_{65539, 65538}_id, FC_{65539, 65538, 65537}_id, FC_{65539, 65538, 65537, 65536}_id}`, would be sent to the ASes on the BGP-UPDATE propagation path, which means the FCList would be sent to AS 65537, AS 65538, and AS 65539. For BM generated by the BGP router of AS 65537, it fills the eContent of BM signed object as follows:

~~~~~~
BM_{65536, 65537, 65538, 65539}:
  version: 0
  asidS: 65536
  prefixS:
    afi: 2,
    address: 2001:db8:a::
    prefixLength: 48
  prefixD:
    afi: 2
    address: 2001:db8:d::
    prefixLength: 48
  BindingMessage:
    id: HASH({65539, 65538, 65537}, 65536, 2001:db8:d::/48) -- as BMID3
    signature: SIGNATURE3
~~~~~~

## Filtering Rules Generation

When one AS receives one BGP-UPDATE, it MUST do as usual: filter the BGP route using its local policy, scratch the BGP route to its Route Information Base (RIB) table, generate Forwarding Information Base (FIB) table, and send it out as per the routing policy.

When the BM and FC signed objects synchronize from the RPKI repository, the BGP speaker MUST first verify FCs. According to the verified FCs, the router then could verify the BMs. It SHOULD find FCList by the id field of one BM. Then it gets all the FCIDs in the FCList. Based on previous verified FCs, it could know which prefix is received as it announces.

It RECOMMENDs deleting all the invalid RIB items to instruct the traffic forwarding. If all the FCs and BMs are valid, the forwarding plane would be a valid one, and source validation and path verification are complete.


# IANA Considerations

## SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1)

 Please add the id-mod-rpki-fc-2023 to the SMI Security for S/MIME Module Identifier (1.2.840.113549.1.9.16.0) registry (https://www.iana.org/assignments/smi-numbers/smi-numbers.xml#security-smime-0) as follows:

~~~~~~
Decimal    Description                   Specification
---------------------------------------------------------------------
TBD        id-mod-rpki-bm-2023           [RFC-to-be]
~~~~~~

## SMI Security for S/MIME CMS Content Type registry

Please add the FC to the SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1) registry (https://www.iana.org/assignments/smi-numbers/smi-numbers.xml#security-smime-1) as follows:

~~~~~~
Decimal     Description                  Specification
---------------------------------------------------------------------
TBD         id-ct-BM                     [RFC-to-be]
~~~~~~

## RPKI Signed Object registry

Please add Binding Message to the RPKI Signed Object registry (https://www.iana.org/assignments/rpki/rpki.xhtml#signed-objects) as follows:

~~~~~~
Name        OID                          Specification
---------------------------------------------------------------------
Binding
Message  1.2.840.113549.1.9.16.1.TBD  [RFC-to-be]
~~~~~~

--- back

## TCP Port registry

This mechanism requires a TCP port to listen to and receive the FCList message from the on-path ASes.

TBD.

# Security Consideration

TBD.

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
