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


--- abstract

A single digital signature key is often relied upon to produce signatures that
have different semantics.  This produces a potential problem whereby signatures
with different intended uses can be confused.  The addition of context strings
in signatures removes this problem.


--- middle

# Introduction

Digital signatures are used in a range of different contexts.  These uses are
often developed in isolation, which leads to the potential for data structures
that are used in one protocol having plausible interpretations in other
protocols.  This gives an opportunity for cross-protocol attacks, wherein a
well-behaved participant in one protocol can be coerced into creating a
cryptographic object that, when interpreted by a different protocol, introduces
a vulnerability.

Reuse of signing keys across different contexts is strongly discouraged.
However, in some cases, use of the same key across contexts might be
unavoidable.  For example, the same key might need to be used in multiple
versions of the same protocol, or a protocol might define multiple uses for
signatures using the same key.

Including a unique protocol- and usage- specific context label as input to all
signing operations prevents a signature created in one context from being
successfully validated in a different context.

This document describes a uniform approach for the inclusion of context labels
and a registry for unique labels.

Existing protocols might already include a unique context label.  This document
collects some of these existing labels into the context label registry.


## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC2119].


# Signature Functions with Context

The following cryptographic primitives define an explicit argument for
identifying a context:

* Ed448 and Ed448ph [I-D.irtf-cfrg-eddsa] define a `context` argument.

* HKDF [RFC5869] specifies an `info` argument to the HKDF-Expand function.


# Generic Signature with Context {#sig-context}

Many pre-existing signature schemes do not define an explicit context label.
This document defines a method for using context labels in existing signature
functions.

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


# Recommendations for Signature Context Labels

In order to avoid attacks that permit use of signatures for purposes other than
intended, the context label C MUST NOT be a prefix of any other signature context
label.

New specifications defining context labels for use in signing, SHOULD select
context labels that end with a single zero-valued octet and do not contain any
other zero-valued octets.  Context labels SHOULD be at least 12 octets in
length.


# IANA Considerations

This document establishes "Signature Context String" registry.

Entries in this registry contain the following fields:

Context label:

: A sequence of octets between 1 and 255 octets in length, displayed as a
  hexadecimal string

String:

: An optional, informative ASCII representation of the context label

Specification:

: A reference to a specification describing the use of the context label

Context labels in this registry MUST NOT be a prefix of any other context label
in the registry.  For example, if 0x01ab00 is registered, then a registration for
0x01 or 0x01ab007c MUST be rejected.

A context label that is at 12 octets or more in length and contains exactly one
zero-valued octet at the end can be registered on a First-Come, First-Served
basis [RFC5226].  Context strings that do not meet these requirements require
Expert Review [RFC5226].


# Security Considerations

Derp, derp, derp.


# Acknowledgments

This document originated from hallway discussions at IETF 95; thank
you to those who helped spark the idea.
