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

This documents defines a standard profile for Binding Messages (BM) used in Resource Public Key Infrastructure (RPKI). A BM is a digitally signed object that provides a means of verifying that an IP address prefix announced from `AS a` to `AS b` and selected by `AS b`. When validated, a BM's eContent can be used for detection and mitigation of route hijacking and provide protection for the AS_PATH attribute in BGP-UPDATE.


--- middle

# Introduction

The Border Gateway Protocol (BGP) {{RFC4271}} was designed with no mechanisms to validate the security of BGP attributes. There are two types of BGP security issues, BGP Hijacks and BGP Route Leaks {{!RFC7908}}, plagues Internet security.

The primary purpose of the Resource Public Key Infrastructure (RPKI) is to improve routing security (See {{RFC6480}} for more information). As part of this system, a mechanism is needed to allow a BGP router to select and verify that a prefix is announced from and followed the path that it said itself. And the router could also filter traffic according to this policy in the data plane. A Binding Message provides this function.

Binding Message (BM) is a signed object that binds the IP prefix with its announced AS's Paths, eventually, it could compose, verify, and protect the forwarding path of traffic. It should together use with Forwarding Commitments {{FC}} which is a signed object used in the control plane to protect BGP routing. BM is used for concatenating all ASes on the announced path. It means that FC should provide the relationship between one IP prefix in a BGP-UPDATE message and the original announced AS, while BM should list all the generated FCs for one IP prefix in a BGP-UPDATE message. The generated Binding Message is split into two parts, one for the BM body (BM-body) which is a newly signed object, and the other for the BM message (BM-msg) which would be sent to all ASes in the reverse direction of related IP prefix propagation.

The BM-body uses the template for RPKI digitally signed objects {{RFC6488}} for the definition of a Cryptographic Message Syntax (CMS) {{RFC5652}} wrapper for the BM-body content as well as a generic validation procedure for RPKI signed objects.  As BM-body needs to be validated with RPKI certificates issued by the current infrastructure, we assume the mandatory-to-implement algorithms in {{RFC6485}}, or its successor.

To complete the specification of the FC (see {{Section 4 of RFC6488}}), this document defines:

1.  The object identifier (OID) that identifies the FC signed object. This OID appears in the eContentType field of the encapContentInfo object as well as the content-type signed attribute within the signerInfo structure.
2.  The ASN.1 syntax for the BM-body content, which is the payload signed by the BGP speaker. The FC content is encoded using the ASN.1 {{X.680}} Distinguished Encoding Rules (DER) {{X.690}}.
3.  The steps required to validate a BM-body beyond the validation steps specified in {{RFC6488}}.



## Requirements Language

{::boilerplate bcp14-tagged}




#  BM Validation



#  BM based BGP Path Verification

##  BM Generation

When one BGP router recives one BGP-UPDATE message, it would generate the FC and BM signed object. The process of FC generation is in {{!FC}} that one can get more information. The process of generator of BM SHOULD be split into two part,

- generation of BM body.
- and generation of BM message.

The generation time of BM is the same as FC, they are both generated after the process of BGP path selection.

## BM Verification and Path Verification

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
