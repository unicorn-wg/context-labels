---
title: "Using Context Labels for Domain Separation of Cryptographic Objects"
abbrev: "Context Labels"
docname: draft-thomson-saag-context-labels-latest
date: 2016
category: std
ipr: trust200902

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
 -
    ins: M. Thomson
    name: Martin Thomson
    org: Mozilla
    email: martin.thomson@gmail.com
 -
    ins: D. Gillmor
    name: Daniel Kahn Gillmor
    org: ACLU
    email: dkg@fifthhorseman.net
 -
    ins: B. Kaduk
    name: Benjamin Kaduk
    org: Unaffiliated
    email: kaduk@mit.edu


normative:
  RFC2119:
  RFC2104:
  RFC3447:
  RFC5226:
  RFC5869:
  I-D.irtf-cfrg-eddsa:
  X9.62:
     title: "Public Key Cryptography For The Financial Services Industry: The Elliptic Curve Digital Sig
nature Algorithm (ECDSA)"
     author:
       - org: ANSI
     date: 1998
     seriesinfo: ANSI X9.62

informative:
  RFC5246:
  I-D.ietf-tls-tls13:


--- abstract

A single cryptographic key is sometimes relied upon to produce muliple
cryptographic artifacts that each have different semantics.  This produces a
potential problem whereby artifacts with different intended uses can be
confused.  The addition of context labels removes this problem.


--- middle

# Introduction

The same cryptographic primitive can be used in a range of different contexts.
These uses are often developed in isolation, which leads to the potential for
data structures that are used in one protocol having plausible interpretations
in other protocols.  This gives an opportunity for cross-protocol attacks,
wherein a well-behaved participant in one protocol can be coerced into creating
a cryptographic object that, when interpreted by a different protocol,
introduces a vulnerability.

Reuse of the same key in multiple contexts is strongly discouraged.  However, in
some cases, use of the same key might be unavoidable.  For example, the same key
might need to be used in multiple versions of the same protocol, or a protocol
might define multiple uses for a particular type of key.

Including a unique protocol- and usage- specific context label as input to a
cryptographic operation prevents objects created in one context from being
mistakenly used in a different context.

This document describes a uniform approach for the inclusion of context labels
and a registry for unique labels.  It covers the use of these labels in digital
signatures, key derivation functions (KDFs), and message authentication codes
(MACs).

Existing protocols might already include a unique context label.  This document
collects some of these existing labels into the context label registry.


## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC2119].


# Existing Functions with Context Labels

The following cryptographic primitives define an explicit argument for
identifying a context:

* Ed448 and Ed448ph [I-D.irtf-cfrg-eddsa] define a `context` argument.

* HKDF [RFC5869] specifies an `info` argument to the HKDF-Expand function.


# Generic Signature or MAC Function with Context {#context}

Many pre-existing signature and MAC schemes do not define an explicit context
label.  This document defines a new signature function that adds a context label
to an existing function.

Given a signature function S that takes a key K and message M as a sequence of
octets, a signature with context function Sc is defined.  The signature with
context function Sc takes three arguments, K, M, and a context label C as a
sequence of octets and is defined as:

~~~ inline
   Sc(K, M, C) = S(K, C || M)
~~~

That is, the signature is changed to accept a message that is the concatenation
of the context label and the message.

This scheme MUST be used with:

* RSA (both PKCS#1 and PSS) [RFC3447]

* ECDSA [X9.62]

* HMAC [RFC2104]

* Ed25519 and Ed25519ph [I-D.irtf-cfrg-eddsa]


# Recommendations for Context Labels {#rec}

In order to avoid attacks that permit use of a cryptographic object for purposes
other than intended, a context label C MUST NOT be a prefix of any other
context label.

New specifications defining context labels SHOULD select context labels that end
with a single zero-valued octet and do not contain any other zero-valued octets.
Context labels SHOULD be at least 12 octets in length.


# IANA Considerations

This document establishes a "Cryptographic Context Label" registry.

Entries in this registry contain the following fields:

Context Label:

: A sequence of octets between 1 and 255 octets in length, displayed as a
  hexadecimal string

String:

: An optional, informative ASCII representation of the context label

Specification:

: A reference to a specification describing the use of the context label

Context labels in this registry MUST NOT be a prefix of any other context label
in the registry.  For example, if 0x01ab00 is registered, then a registration for
0x01 or 0x01ab007c MUST be rejected.

A context label that is 12 octets or more in length and contains exactly one
zero-valued octet at the end can be registered on a First-Come, First-Served
basis [RFC5226].  Context labels that do not meet these requirements require
Expert Review [RFC5226].

The initial contents of this registry are included in {{ok}}.


# Security Considerations

In general, it is best to limit any cryptographic material to being used for a
single purpose.


--- back

# Existing Protocols with Context Labels {#ok}

| Context label | String | Specification |
|---|
| 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 54 4c 53 20 31 2e 33 2c 20 73 65 72 76 65 72 20 43 65 72 74 69 66 69 63 61 74 65 56 65 72 69 66 79 00 | (64 spaces)TLS 1.3, server CertificateVerify\0 | [I-D.ietf-tls-tls13] |
| 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 54 4c 53 20 31 2e 33 2c 20 63 6c 69 65 6e 74 20 43 65 72 74 69 66 69 63 61 74 65 56 65 72 69 66 79 00 | (64 spaces)TLS 1.3, client CertificateVerify\0 | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 78 70 61 6e 64 65 64 20 73 74 61 74 69 63 20 73 65 63 72 65 74 | TLS 1.3, expanded static secret | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 78 70 61 6e 64 65 64 20 65 70 68 65 6d 65 72 61 6c 20 73 65 63 72 65 74 | TLS 1.3, expanded ephemeral secret | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 74 72 61 66 66 69 63 20 73 65 63 72 65 74 | TLS 1.3, traffic secret | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 72 65 73 75 6d 70 74 69 6f 6e 20 6d 61 73 74 65 72 20 73 65 63 72 65 74 | TLS 1.3, resumption master secret | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 78 70 6f 72 74 65 72 20 6d 61 73 74 65 72 20 73 65 63 72 65 74 | TLS 1.3, exporter master secret | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 61 72 6c 79 20 68 61 6e 64 73 68 61 6b 65 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 63 6c 69 65 6e 74 20 77 72 69 74 65 20 6b 65 79 | TLS 1.3, early handshake key expansion, client write key | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 61 72 6c 79 20 68 61 6e 64 73 68 61 6b 65 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 73 65 72 76 65 72 20 77 72 69 74 65 20 6b 65 79 | TLS 1.3, early handshake key expansion, server write key | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 61 72 6c 79 20 68 61 6e 64 73 68 61 6b 65 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 63 6c 69 65 6e 74 20 77 72 69 74 65 20 69 76 | TLS 1.3, early handshake key expansion, client write iv | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 61 72 6c 79 20 68 61 6e 64 73 68 61 6b 65 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 73 65 72 76 65 72 20 77 72 69 74 65 20 69 76 | TLS 1.3, early handshake key expansion, server write iv | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 61 72 6c 79 20 61 70 70 6c 69 63 61 74 69 6f 6e 20 64 61 74 61 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 63 6c 69 65 6e 74 20 77 72 69 74 65 20 6b 65 79 | TLS 1.3, early application data key expansion, client write key | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 61 72 6c 79 20 61 70 70 6c 69 63 61 74 69 6f 6e 20 64 61 74 61 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 73 65 72 76 65 72 20 77 72 69 74 65 20 6b 65 79 | TLS 1.3, early application data key expansion, server write key | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 61 72 6c 79 20 61 70 70 6c 69 63 61 74 69 6f 6e 20 64 61 74 61 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 63 6c 69 65 6e 74 20 77 72 69 74 65 20 69 76 | TLS 1.3, early application data key expansion, client write iv | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 65 61 72 6c 79 20 61 70 70 6c 69 63 61 74 69 6f 6e 20 64 61 74 61 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 73 65 72 76 65 72 20 77 72 69 74 65 20 69 76 | TLS 1.3, early application data key expansion, server write iv | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 68 61 6e 64 73 68 61 6b 65 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 63 6c 69 65 6e 74 20 77 72 69 74 65 20 6b 65 79 | TLS 1.3, handshake key expansion, client write key | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 68 61 6e 64 73 68 61 6b 65 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 73 65 72 76 65 72 20 77 72 69 74 65 20 6b 65 79 | TLS 1.3, handshake key expansion, server write key | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 68 61 6e 64 73 68 61 6b 65 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 63 6c 69 65 6e 74 20 77 72 69 74 65 20 69 76 | TLS 1.3, handshake key expansion, client write iv | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 68 61 6e 64 73 68 61 6b 65 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 73 65 72 76 65 72 20 77 72 69 74 65 20 69 76 | TLS 1.3, handshake key expansion, server write iv | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 61 70 70 6c 69 63 61 74 69 6f 6e 20 64 61 74 61 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 63 6c 69 65 6e 74 20 77 72 69 74 65 20 6b 65 79 | TLS 1.3, application data key expansion, client write key | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 61 70 70 6c 69 63 61 74 69 6f 6e 20 64 61 74 61 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 73 65 72 76 65 72 20 77 72 69 74 65 20 6b 65 79 | TLS 1.3, application data key expansion, server write key | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 61 70 70 6c 69 63 61 74 69 6f 6e 20 64 61 74 61 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 63 6c 69 65 6e 74 20 77 72 69 74 65 20 69 76 | TLS 1.3, application data key expansion, client write iv | [I-D.ietf-tls-tls13] |
| 54 4c 53 20 31 2e 33 2c 20 61 70 70 6c 69 63 61 74 69 6f 6e 20 64 61 74 61 20 6b 65 79 20 65 78 70 61 6e 73 69 6f 6e 2c 20 73 65 72 76 65 72 20 77 72 69 74 65 20 69 76 | TLS 1.3, application data key expansion, server write iv | [I-D.ietf-tls-tls13] |

Note that in the above table, the following categories of entry do not conform
with the guidance in {{rec}}:

* Labels for the TLS 1.3 HKDF input


# Existing Protocols without Context Labels {#bad}

TLS versions 1.2 [RFC5246] and earlier do not use context labels for signatures
though the use of the pseudorandom function (PRF) uses version-agnostic labels.


# Acknowledgments

This document originated from hallway discussions at IETF 95; thank
you to those who helped spark the idea.
