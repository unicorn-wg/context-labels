---
title: "Using Context Strings in Digital Signatures"
abbrev: "Signature Context"
docname: draft-thomson-saag-sig-context-latest
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


--- abstract

A single digital signature key is often relied upon to produce signatures that
have different semantics.  This produces a potential problem whereby signatures
with different intended uses can be confused.  The addition of context strings
in signatures removes this problem.

--- middle

# Introduction

There are a handful of cryptographic primitives that are used to build
protocols that are used on the Internet: digital signatures, key derivation
algorithms, message integrity codes, etc..  These protocols are generally
developed in isolation from each other, with minimal effort to ensure that data
structures used in one protocol do not have plausible interpretations in other
protocols.  This gives an opportunity for cross-protocol attacks, wherein a
well-behaved participant in one protocol can be abused by an attacker to create
a cryptographic object that, when interpreted by a different protocol,
introduces a vulnerability.  Including a unique protocol-specific context label
as input to all cryptographic operations prevents a cryptographic object
created in one protocol from being interpreted in the context of a different
protocol.  To avoid breaking existing protcols, only new constructs can be
given such context labels as they are added to protocols, but cross-protocol
attacks will be avoided between primitives/protocols that do use context
lables.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC2119].

# Signature Functions with Context

The following signature schemes define an explicit context string argument:

* Ed448 [I-D.irtf-cfrg-eddsa] defines a `context` argument.

* HKDF [RFC5869] specifies an `info` argument to the HKDF-Expand function.


# Generic Signature with Context {#sig-context}

Many pre-existing signature schemes do not define an explicit context string.
This document defines a method for using context strings in existing signature
functions.

Given a signature function S that takes a key K and message M as a sequence of
octets, this section defines a signature with context function Sc.  The
signature with context function takes three arguments, K, M, and a context
string C as a sequence of octets and is defined as:

~~~ inline
   Sc(K, M, C) = S(K, C || M)
~~~

That is, the signature is changed to accept a message that is the concatenation
of the context and the message.

This scheme MUST be used with:

* RSA (both PKCS#1 and PSS) [RFC3447]

* ECDSA [X9.62]

* HMAC [RFC2104]  ???

* Ed25519 [I-D.irtf-cfrg-eddsa]


# Recommendations for Signature Context Strings

In order to avoid attacks that permit use of signatures for purposes other than
intended, the context C MUST NOT be a prefix of any other signature context
string.

New specifications defining context strings for use in signing, SHOULD select
context strings that end with a single zero-valued octet and do not contain any
other zero-valued octets.  Context strings SHOULD be at least 12 octets in
length.


# IANA Considerations

This document establishes "Signature Context String" registry.

Entries in this registry contain the following fields:

Context string:

: A sequence of octets between 1 and 255 octets in length

Specification:

: A reference to a specification describing the use of the context string.

Context strings in this registry MUST NOT be a prefix of any other context
string in the registry.  For example, if 0x0100 is registered, then a
registration for 0x01 or 0x010000 MUST be rejected.

A context string that is at least 12 octets in length and contains exactly one
zero-valued octet at the end can be registered on a First-Come, First-Served
basis [RFC5226].  Context strings that do not meet these requirements require
Expert Review [RFC5226].


# Security Considerations

Derp, derp, derp.


# Acknowledgments

Benjamin Kaduk contributed to this document.
