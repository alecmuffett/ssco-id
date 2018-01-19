---
title: Self-Signed TLS Certificates for Onion Services
abbrev: I-D
docname: draft-muffett-tls-ssconion-00
date: 2018-01-18
category: info
ipr: trust200902

author:
 -
    ins: A. Muffett
    name: Alec Muffett
    organization: (To Be Decided)
    email: alec.muffett@gmail.com

normative:
  RFC1034:
  RFC1123:
  RFC2119:
  RFC5280:
  RFC7686:

informative:

--- abstract

A proposal for standardization of self-signed TLS certificates for use
with ".onion" names as-defined in {{RFC7686}}, for onion services.

--- middle

# Introduction

## Terminology

In this document, the key words "MUST", "MUST NOT", "REQUIRED",
"SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" are to be interpreted as described in BCP 14, RFC 2119
{{RFC2119}}.

In this document, ".onion name" is to be interpreted as described in
section 1 of {{RFC7686}}.

In this document "base .onion name" is interpreted to mean the
lowercase canonical representation of a valid .onion name, shorn of
any/all subdomain labels and of the ".onion" label; for example the
base .onion name of "www.foo.facebookcorewwwi.onion" is
"facebookcorewwwi".

In this document "wildcard .onion name" is interpreted to mean a name
which comprises three or more labels, where:

* the leftmost label is wholly an asterisk ("*")
* the rightmost label is "onion"
* the second-to-rightmost label is a "base .onion name"
* if the asterisk comprising the leftmost label were replaced with the
  single character "A", the resulting whole would conform with DNS
  name syntax as defined in Section 3.5 of {{RFC1034}} and Section 2.1
  of {{RFC1123}}.

# Self-Signed TLS Certificates for Onion Services

## Rationale

Tor .onion names are self-asserted cryptographic identifiers which
enable one form of communication on the Tor network.

Possession of the private key for a given .onion name provides defacto
"domain ownership" of that .onion name, and also the ability to
communicate on the Tor network as that .onion name.

Given the self-asserted (and even sometimes ephemeral) nature of
.onion names, combined with the proliferation of protocols which
mandate HTTPS, the requirement for involvement of a Certificate
Authority to issue TLS certificates for a .onion name is a burden.

Therefore: we propose a standard for Self-Signed TLS Certificates for
Onion Services, which Tor-enabled clients may optionally honour as
being equally trustworthy to a CA-supplied DV certificate.

This proposal does not replace certification of a .onion name as
currently, or in future, provided under CABForum BRs; in fact we
appeal for extension of CABForum BRs to include DV certification for
"Next Generation" Onion addresses.

Instead: this standard provides an additional specification to offer
ad-hoc HTTPS capability to enabled clients in limited circumstances.

## Certificate Fields

All fields that are described in this section MUST be present for a
"Self-Signed TLS Certificate for Onion Services" to be valid.

All fields which are not described in this section MUST NOT exist in
such a certificate.

### Signature Algorithm

The signature algorithm must conform with {{RFC5280}} section 4.1.1.2

#### Signature Algorithm Considerations

Tor implements perfect forward security at the transport layer; in the
spirit of this, and given the narrow and legacy-free nature of clients
which will implement this standard, it would appear retrogressive to
implement anything that does not also meet that functionality.

Therefore we propose that all certificates under this standard SHOULD
be signed with ECDSA.  TODO: Can we tighten this up to kill sucky
algorithms?

### Signature Value

The Signature Value field MUST conform with {{RFC5280}} section 4.1.1.3

### Version

The version field MUST conform with {{RFC5280}} section 4.1.2.1

### Serial Number

The Serial Number MUST conform with {{RFC5280}} section 4.1.2.2

From section 4.1.2.2, "The serial number MUST be a positive integer
... It MUST be unique for each certificate ... MUST NOT use
serialNumber values longer than 20 octets"

In the spirit of that specification, we propose that the serial number
be constructed as the hex digest of:

Digest ( BaseOnionName NotBefore NotAfter )

...where the following applies:

* Digest is MD5
* BaseOnionName is the "base .onion name" (qv) of the Subject
  CommonName
* NotBefore and NotAfter are from the certificate Validity and are
  represented in ASCII GeneralizedTime Zulu

Thus a serial number for a certificate valid from midnight 1 January
2018 UTC to 13:14:15 on 30 June 2019 UTC for
"www.facebookcorewwwi.onion" would be:

md5("facebookcorewwwi20180101000000Z20190630131415Z")

= A8:E2:59:98:F0:2B:E1:40:F6:D2:18:29:8B:68:BB:EA

#### Serial Number Considerations

* should we roll the public key into the digest, somehow?
* should we digest with 20-octet-truncated SHA256 instead?
* should we roll all of the SANs through the digest, instead?

### Issuer

The CommonName for the Issuer MUST be the 29-character ASCII string:
"Self-Signed Onion Certificate".

Other Issuer fields MUST conform with {{RFC5280}} section 4.1.2.4
conforming to CA/B-Forum BRs for DV Certificates.

### Validity

The certificate validity MUST conform with {{RFC5280}} section 4.1.2.5

* TODO: should we set a max 1-year validity, or less, to assure
  ongoing housekeeping and freshness?

### Subject

The CommonName for the Subject MUST be identical to the first DNS Name
from the list of Subject Alternative Names.

All other Subject fields MUST conform with {{RFC5280}} section
4.1.2.6, and MUST also conform with extant CA/B-Forum BRs for DV
Certificate Subjects.

### Subject Public Key Info

The Subject Public Key MUST conform with {{RFC5280}} section 4.2.1.7

#### Subject Public Key Info Considerations

* Per discussion for Signature Algorithm above, it is RECOMMENDED that
  the Subject Public Key Algorithm should be "Elliptic Curve Public
  Key" of length 256.
* TODO: It would be nice to tighten this up a bit.

### Extensions

The Extensions MUST contain a Subject Alternative Name extension.

All other Extensions MUST be excluded.

#### Subject Alternative Names

Requirements:

* The Subject Alternative Names MUST only contain "DNS names".
* At least one "DNS name" MUST be given.
* The given "DNS names" MAY be ".onion names".
* The given "DNS names" MAY be "wildcard .onion names"
* The given "DNS names" MUST NOT be other than a ".onion name" or "wildcard .onion name".

#### Subject Alternative Names Considerations

* Should we set a maximum number?

## Security Considerations

### Transitive Trust

General-purpose HTTP/HTTPS is being extended such that a existing
browser connection to "www.foo.com" will be reused for requests to
"www.bar.com" in the instance that "www.bar.com" matches a Subject
Alternative Name that was provided by the TLS certificate which
authenticated "www.foo.com".

This arrangement, under a realm of self-signed certificates, would
present an obvious security risk of violation of trust roots in Onion
networking.

Therefore, for the avoidance of this and other similar future security
issues:

* A valid Self-Signed TLS Certificate for Onion Services MUST only
  cite a solitary "base .onion name" amongst the Subject Alternative
  Names that it lists.
* Clients which support Self-Signed TLS Certificates for Onion
  Services MUST NOT trust certificates that cite more than one "base
  .onion name".

TODO: if this risk is mitigated elsewhere in the specs, we could maybe
nuke this section; the risk is in implementations which might unwisely
trust a Self-Signed TLS Certificate for Onion Services. The above does
appear to be a desirably robust solution, however.

### Onion Name Types

At the time of writing, "base .onion names" are of two types:

* "old" .onion names, being encodings of truncated SHA-1 hashes
* "new" or "next generation" names, being encodings of X25519 keys and metadata

This standard SHALL NOT prohibit generation of Self-Signed TLS
Certificates for Onion Services for either of these, nor for future,
types of .onion name.

Clients SHOULD implement fine-grained checks to distingush, and to
eventually deprecate, legacy types of .onion name.
