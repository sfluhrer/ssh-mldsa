---
title: "SSH Support of ML-DSA"
abbrev: "SSH ML-DSA"
category: info

docname: draft-sfluhrer-ssh-mldsa-latest
submissiontype: IETF
consensus: true
date:
v: 3
area: "sec"
workgroup: "SSHM"
stand_alone: true
ipr: trust200902
keyword:
 - ML-DSA
coding: UTF-8
venue:
  group: "Secure Shell Maintenance"
  type: "Security Area"
  mail: "cfrg@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ssh/"
  github: "sfluhrer/ssh-mldsa-support"
  latest: "https://sfluhrer.github.io/ikev2-ssh-support/draft-sfluhrer-ikev2-ssh-support.html"

author:
 -
    fullname: "Scott Fluhrer"
    organization: Cisco Systems
    email: "sfluhrer@cisco.com"

normative:

  FIPS204:
    target: https://doi.org/10.6028/NIST.FIPS.204
    title: Module-Lattice-Based Digital Signature Standard
    seriesinfo:
      "NIST": "FIPS 204"
    date: August 2024

informative:

  RFC2119:
  RFC4250:
  RFC4251:
  RFC4255:
  RFC8174:
  IANA-SSHFP:
    target: https://www.iana.org/assignments/dns-sshfp-rr-parameters)
    title: DNS SSHFP Resource Record Parameters

--- abstract---

   This document describes the use of ML-DSA digital
   signatures in the Secure Shell (SSH) protocol.
  
--- middle

# Introduction

   A Cryptographically Relevant Quantum Computer (CRQC) could break
   traditional asymmetric cryptograph algorithms: e.g RSA, ECDSA; which
   are widely deployed authentication options of IKEv2.
   NIST has recently published the postquantum digitial signature algorithm ML-DSA [FIPS204].

   This document describes how to use this algorithm for authentication within SSH [RFC4251], as a replacement for the traditional signature algorithms (RSA, ECDSA).

## Background on ML-DSA

   ML-DSA (as specified in FIPS 204) is a signature algorithm that is believed to be secure against attackers who have a Quantum Computer available to them.
   There are three strengths defined for it (with the parameter sets being known as ML-DSA-44, ML-DSA-65 and ML-DSA-87).
   In addition, for each defined parameter set, there are two versions, the 'pure' version (where ML-DSA directly signs the message) and a 'prehashed' version (where ML-DSA signs a hash that was computed outside of ML-DSA).
   For this protocol, we will always use the pure version.

   In addition, ML-DSA also has a 'context' input, which is a short string that is common to the sender and the recceiver.
   It is intended to allow for domain separation between separate uses of the same public key.

   FIPS 204 also allows ML-DSA to be run in either determanistic or 'hedged' mode (where randomness is applied to the signature operation).
   We place no requirement on which is used; the implementation should select based on the quality of their random number source.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

   The descriptions of key and signature formats use the notation
   introduced in [RFC4251], Section 3, and the string data type from
   [RFC4251], Section 5.  Identifiers and terminology from ML-DSA
   [FIPS204] are used throughout the document.

# Requirement Language

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in
   BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all
   capitals, as shown here.

# Public Key Algorithms

   This document describes three public key algorithms for use with SSH, as
   per [RFC4253], Section 6.6, corresponding to the three parameter sets of ML-DSA.
   The names of the algorithm are "ssh-ml-dsa-44", "ssh-ml-dsa-65" and "ssh-ml-dsa-87", to match the level 2, 3 and 5 parameter sets [FIPS204].
   These algorithm only support signing and not encryption.

# Public Key Format

   The key format for all three parameter sets have the following encoding:

   string "ssh-ml-dsa-44" (or "ssh-ml-dsa-65" or "ssh-ml-dsa-87")

   string key

   Here, 'key' is the public key described in [FIPS204].

 # Signature Algorithm

   Signatures are generated according to the procedure in Section 5.2
   [FIPS204], using the "pure" version of ML-DSA, with an empty context string.

# Signature Format

   The "ssh-ml-dsa" key format has the following encoding:

   string "ssh-ml-dsa-44" (or "ssh-ml-dsa-65" or "ssh-ml-dsa-87")

   string signature

   Here, 'signature' is the signature produced in accordance with the
   previous section.

# Verification Algorithm

   Signatures are verified according to the procedure in
   [FIPS204], Section 5.3, using the "pure" version of ML-DSA, with an empty context strong.

# SSHFP DNS Resource Records

   Usage and generation of the SSHFP DNS resource record is described in
   [RFC4255].  This section illustrates the generation of SSHFP resource
   records for ML-DSA keys, and this document also specifies
   the corresponding code point to "SSHFP RR Types for public key
   algorithms" in the "DNS SSHFP Resource Record Parameters" IANA
   registry [IANA-SSHFP].

   The generation of SSHFP resource records keys for ML-DSA is
   described as follows.

   The encoding of ML-DSA public keys is described in [FIPS204].

   The SSHFP Resource Record for an ML-DSA key fingerprint
   (with a SHA-256 fingerprint) would, for example, be:

   pqserver.example.com. IN SSHFP TBD 2 (
                    a87f1b687ac0e57d2a081a2f28267237
                    34d90ed316d2b818ca9580ea384d9240 )

   Replace TBD with the value eventually allocated by IANA.
 
# IANA Considerations

   This document augments the Public Key Algorithm Names in [RFC4250], Section 4.11.3.

   IANA is requested to add the following entries to "Public Key Algorithm
   Names" in the "Secure Shell (SSH) Protocol Parameters" registry
   [IANA-SSH]:

   | Public Key Algorithm Name | Key Size | Signature Size | Reference |
   | ------------------------- | -------- | -------------- | --------- |
   | ssh-ml-dsa-44             | 1312     | 2420           | THIS-RFC  |
   | ssh-ml-dsa-65             | 1952     | 3309           | THIS-RFC  |
   | ssh-ml-dsa-87             | 2592     | 4627           | THIS-RFC  |

   IANA is requested to add the following entries to "SSHFP RR Types for
   public key algorithms" in the "DNS SSHFP Resource Record Parameters"
   registry [IANA-SSHFP]:

   | Value | Description | Reference |
   | ----- | ----------- | --------- |
   | TBD1  | ML-DSA-44   | THIS RFC  |
   | TBD2  | ML-DSA-65   | THIS RFC  |
   | TBD3  | ML-DSA-87   | THIS RFC  |

# Security Considerations

   The security considerations in [RFC4251], Section 9 apply to all SSH
   implementations, including those using ML-DSA.

   The security considerations in ML-DSA [NIST.FIPS.204] apply to all
   uses of ML-DSA, including those in SSH.

   Cryptographic algorithms and parameters are usually broken or
   weakened over time.  Implementers and users need to continously re-
   evaluate that cryptographic algorithms continue to provide the
   expected level of security.
 
--- back

# Acknowledgments
{:numbered="false"}

   The text of draft-josefsson-ssh-sphincs was used as a template for this document.
