---
title: "SSH Support of ML-DSA"
abbrev: "SSH ML-DSA"
category: info

docname: draft-sfluhrer-ssh-mldsa-06
submissiontype: IETF
consensus: true
date:
v: 3
area: "sec"
workgroup: "sshm"
stand_alone: true
ipr: trust200902
keyword:
 - ML-DSA
coding: UTF-8
venue:
  group: "Secure Shell Maintenance"
  type: "Security Area"
  mail: "ssh@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ssh/"
  github: "sfluhrer/ssh-mldsa"
  latest: "https://sfluhrer.github.io/ssh-mldsa/draft-sfluhrer-ssh-mldsa.html"

author:
  - fullname: "Scott Fluhrer"
    organization: Cisco Systems
    email: "sfluhrer@cisco.com"
  - fullname: "Ryan Appel"
    organization: Bank of America
    email: "ryan.appel@bofa.com"

normative:

  FIPS204:
    target: https://doi.org/10.6028/NIST.FIPS.204
    title: Module-Lattice-Based Digital Signature Standard
    seriesinfo:
      "NIST": "FIPS 204"
    date: August 2024

informative:

  RFC4250:
  RFC4251:
  RFC4253:
  RFC4255:
  RFC9881:
  IANA-SSH:
    target: https://www.iana.org/assignments/ssh-parameters
    title: Secure Shell (SSH) Protocol Parameters
  IANA-SSHFP:
    target: https://www.iana.org/assignments/dns-sshfp-rr-parameters)
    title: DNS SSHFP Resource Record Parameters
  I-D.ietf-lamps-pq-composite-sigs:
     target: https://datatracker.ietf.org/doc/draft-ietf-lamps-pq-composite-sigs/19/
     title: Composite Module-Lattice-Based Digital Signature Algorithm (ML-DSA) for use in X.509 Public Key Infrastructure

--- abstract

   This document describes the use of ML-DSA digital
   signatures in the Secure Shell (SSH) protocol.

--- middle

# Introduction

   A Cryptographically Relevant Quantum Computer (CRQC) is a quantum computer with sufficient compute capability and
   stability that it is able to break traditional asymmetric cryptographic algorithms: e.g RSA, ECDSA; which are
   currently the only authentication algorithms available for SSH. NIST has recently published the post-quantum
   cryptography (PQC) algorithm known as ML-DSA [FIPS204] which is a digital signature algorithm.

   This document describes how to use this algorithm for authentication within SSH [RFC4251], as a replacement for the
   traditional signature algorithms (e.g. RSA, ECDSA).

## Background on ML-DSA

   ML-DSA (as specified in FIPS 204) is a signature algorithm that is believed to be secure against attackers who have a
   CRQC available to them. There are three parameter sets defined for it which belong to a respective Security Category
   of 2, 3, and 5 (ML-DSA-44, ML-DSA-65, and ML-DSA-87). In addition, for each defined parameter set, there are two
   versions, the 'pure' version (where ML-DSA computes the hash internally of the message and then signs the internally
   computed hash), and a 'prehashed' version (where ML-DSA signs a hash that was computed outside of ML-DSA). For this
   protocol, we will always use the pure version.

   In addition, ML-DSA also has a 'context' input, which is a short string that is common to the sender and the
   receiver. It is intended to allow for domain separation between separate uses of the same public key. This protocol
   always uses an empty (zero length) context.

   FIPS 204 also allows ML-DSA to be run in either determanistic or 'hedged' mode (where randomness is applied during the signature generation operation).
   We recommend that implementations use hedged mode, as it blocks certain side channel attacks.
   However, as both are interoperable, we do not place any requirement on which is used within this protocol.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

   The descriptions of key and signature formats use the notation
   introduced in [RFC4251], Section 3, and the string data type from
   [RFC4251], Section 5.  Identifiers and terminology from ML-DSA
   [FIPS204] are used throughout the document.

# Public Key Algorithms

   This document describes three public key algorithms for use with SSH, as
   per [RFC4253], Section 6.6, corresponding to the three parameter sets of ML-DSA.
   The names of the algorithm are "ssh-mldsa-44", "ssh-mldsa-65" and "ssh-mldsa-87", to match the level 2, 3 and 5 parameter sets [FIPS204].
   These algorithm support only signing and not encryption.

   The below table lists the public key sizes and the signature size (in bytes) for the three parameter sets.

   | Public Key Algorithm Name | Public Key Size | Signature Size |
   | ------------------------- | --------------- | -------------- |
   | ssh-mldsa-44              | 1312            | 2420           |
   | ssh-mldsa-65              | 1952            | 3309           |
   | ssh-mldsa-87              | 2592            | 4627           |

# Public Key Format

   The key format for all three parameter sets have the following encoding:

   string "ssh-mldsa-44" (or "ssh-mldsa-65" or "ssh-mldsa-87")

   string key

   Here, 'key' is the public key described in [FIPS204].

# Signature Algorithm

   Signatures are generated according to the procedure in Section 5.2 [FIPS204], using the "pure" version of ML-DSA,
   with an empty context string.

# Signature Format

   The "ssh-mldsa" signature format has the following encoding:

   string "ssh-mldsa-44" (or "ssh-mldsa-65" or "ssh-mldsa-87")

   string signature

   Here, 'signature' is the signature produced in accordance with the previous section.

# Verification Algorithm

   Signatures are verified according to the procedure in
   [FIPS204], Section 5.3, using the "pure" version of ML-DSA, with an empty context string.

# SSHFP DNS Resource Records

   Usage and generation of the SSHFP DNS resource record is described in [RFC4255].  This section illustrates the
   generation of SSHFP resource records for ML-DSA keys, and this document also specifies the corresponding code point
   to "SSHFP RR Types for public key algorithms" in the "DNS SSHFP Resource Record Parameters" IANA registry
   [IANA-SSHFP].

   The generation of SSHFP resource records keys for ML-DSA is described as follows.

   The encoding of ML-DSA public keys is described in [FIPS204].

   The SSHFP Resource Record for an ML-DSA key fingerprint (with a SHA-256 fingerprint) would, for example, be:

   pqserver.example.com. IN SSHFP TBD 2 (
                    a87f1b687ac0e57d2a081a2f28267237
                    34d90ed316d2b818ca9580ea384d9240 )

   Replace TBD with the value eventually allocated by IANA.

# IANA Considerations

  This document augments the Public Key Algorithm Names in [RFC4250], Section 4.11.3.

   IANA is requested to add the following entries to "Public Key Algorithm Names" in the "Secure Shell (SSH) Protocol
   Parameters" registry [IANA-SSH]:

   | Public Key Algorithm Name | Reference |
   | ------------------------- | --------- |
   | ssh-mldsa-44              | THIS-RFC  |
   | ssh-mldsa-65              | THIS-RFC  |
   | ssh-mldsa-87              | THIS-RFC  |

   IANA is requested to add the following entries to "SSHFP RR Types for public key algorithms" in the "DNS SSHFP
   Resource Record Parameters" registry [IANA-SSHFP]:

   | Value | Description | Reference |
   | ----- | ----------- | --------- |
   | TBD1  | ML-DSA-44   | THIS RFC  |
   | TBD2  | ML-DSA-65   | THIS RFC  |
   | TBD3  | ML-DSA-87   | THIS RFC  |

# Security Considerations

   The security considerations in [RFC4251], Section 9 apply to all SSH implementations, including those using ML-DSA.

   The security considerations in ML-DSA [FIPS204] apply to all uses of ML-DSA, including those in SSH.

   Cryptographic algorithms and parameters are usually broken or weakened over time.  Implementers and users need to
   continuously re-evaluate that cryptographic algorithms continue to provide the expected level of security.

<!-- Start of Appendices -->
--- back

# Storage of the ML-DSA Private Key

   In general, many protocols have defined exactly the way in which a private key is to be represented when stored. This
   has required all parties involved to implement the same mechanism in which to represent the private key if they
   desire to have a compliant implementation. This has almost never been the case for SSH, which has allowed different
   implementations to represent the private key in any way which made sense to the application. This has also almost
   never been a problem, as the chosen representation of the private key was nearly always interoperable with different
   implementations allowing users to "port" their `implementation style 1` keys to `implementation style 2` or vice
   versa.

   Depending on the way in which an ML-DSA key is represented between implementations could possibly prevent the
   conversion of one format to another. As such, this document does not limit the mechanism in which an implementation
   chooses to represent the private key, but does make a suggestion for interoperability reasons.

   As discussed in section 3.6.3 of [FIPS204], The ML-DSA seed value (denoted in the document as ξ) can be stored rather
   than the full expanded private/public key. This decision is one that offers a trade off of storage space vs
   computation. If one were to store the private key, the amount of storage space required ranges from 2560 bytes - 4896
   bytes (depending of course on the parameter set chosen). Although by today's general computation standards, these
   storage requirements are very minimal (an average picture for example is 10KB - 10MB), for space constrained devices
   (i.e. smart cards), this can severely limit the number of ML-DSA keys that can be stored on such a device.
   Representing the private/public keypair with the seed however limits the storage requirements to just 32 bytes --
   around the size of an ECDSA key on the P-256 curve. The computational tradeoff here is of course that if you want to
   produce a signature from an ML-DSA private key, you first have to re-expand the private key from the seed value to
   the full-expanded private key before producing the signature, every time (or minimally cache the expanded private key
   in primary memory for the length of time intended to produce signatures, which again for constrained devices, is
   limited). This computation is not much of a slow down from loading in the private key from secondary storage, but it
   is still extra computation that has to be done, every time.

   As such, an ML-DSA private key SHOULD be represented as the seed value (ξ) as denoted in [FIPS204] for the purposes of
   interoperability between all devices and implementations. An ML-DSA private key MAY be represented as the full key,
   and if desired MAY be represented by a concatenation between the seed value and the expanded private key so that
   implementations can take advantage of interoperability and conversion, but also take advantage of not re-expanding
   the key each time, with the disadvantage being of course that the storage requirements for the keys as listed in the
   last paragraph are increased by 32 bytes.

   Please note that PKCS#8 encoding of the ML-DSA private key is not finalized (as of the writing of this document), but
   there is a document currently in draft [I-D.ietf-lamps-pq-composite-sigs] as well as [RFC9881], which have suggested
   and used the ML-DSA seed value in the PKCS8 format. If implementations wish to utilize PKCS8 for their implementation
   and serialization / deserialization of the private key, it is likely that storage in seed format will be a
   requirement.

# Acknowledgments
{:numbered="false"}

   The text of draft-josefsson-ssh-sphincs was used as a template for this document.
