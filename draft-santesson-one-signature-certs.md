---
title: "One Signature Certificates"
abbrev: "OSC"
ipr: "trust200902"
category: std

docname: draft-santesson-one-signature-certs-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
date: 2025-10-30
consensus: true
v: 3
area: Security
wg: LAMPS
keyword:
 - certificate
 - signature
 - X.509
venue:
  github: "Razumain/one-signature-certs"

author:
  -
    ins: S. Santesson
    name: Stefan Santesson
    org: IDsec Solutions AB
    abbrev: IDsec Solutions
    street: Forskningsbyn Ideon
    city: Lund
    code: "223 70"
    country: SE
    email: sts@aaa-sec.com

normative:
  RFC2119:
  RFC5280:
  RFC8174:
  RFC9608:

informative:
  RFC9321:

...

--- abstract

Electronic signatures based on multipurpose expiring certificates that may be revoked introduce many challenges when a signed document has to be maintained over time. Solutions include timestamping, storing revocation records and advanced verification procedures. This draft introduces a new type of certificates for a key that is used and bound to just one signing instance. The certificate is created at the time of signing, and the signature key is destroyed after signing is completed. These certificates never expire and are never revoked, which drastically reduce the burden of future signature validation.

--- middle

# Introduction

The landscape of server-based signing services has changed over the last decades. During the past years one type of signature service has gained a lot of traction where the signing key and signing certificate are created for each instance of signing, rather than re-using a static key and certificate over time.

Some reasons why this type of signature services has been successful are:

- The certificate will always have a predictable validity time from the time of signing
- The time of signing is guaranteed by the certificate issue date
- The identity information in the certificate can be adapted to the signing context for each instance of signing
- Revocation of signing certificates is practically non-existent despite many years of operation and millions of signatures.
- Service providers are not bound to using the signature service where the signer's key and certificate is located, but can choose one signature service it integrates with that holds no pre-stored user keys and certificates.

While this type of signature service solves a lot of problems, it still suffers from the complexity caused by expiring signing certificates. One solution to this problem is the Signature Validation Token (SVT) {{RFC9321}} where future validation can rely on a previous successful validation rather than making a new re-validation based on aging data.

This document takes this one step further and allows future re-validation at any time in the future as long as trust in the CA certificate can be established.

## Basic features

One signature certificates have the following common characteristics:

- They never expire
- They are never revoked
- They are bound to a specific document content
- They assert that the corresponding private key was destroyed after signing

### Revocation

Traditional certificates that are re-used over time have many legitimate reasons for revocation, such as if the private key is lost or compromised. This can lead to large volumes of revocation data.

The fact that the same key is used many times exposes the key for the risk of unauthorized usage or theft. When many objects are signed with the same key, the risk of exposure and the number of affected signed documents upon revocation increases, unless properly timestamped and properly verified.

When a signing key is used only once, that risk of exposure is drastically reduced, and it has been shown that most usages of dedicated keys and certificates no longer require revocation.

No CA can guarantee that a certificate is correctly issued. What the CA does is to attest that a certain procedure was followed when the certificate was issued, and the certificate itself is an attestation that this process was followed successfully when the signature was created. Certificates issued according to this profile therefore only attest to the validity at the time of issuance and signing, rather than a retroactive state at the time of validation. This profile is intended for those applications where this declaration of validity is relevant and useful.

Applications that require traditional revocation checking that provides the state at the time of validation MUST NOT use this profile.

An example usage where this is useful is in services where the signed document is stored as an internal evidence record, such as when a Tax agency allows citizens to sign their tax declarations. This record is then pulled out and used only in case of a dispute where the identified signer challenges the signature. A revocation service is less likely to contribute to this process. If the challenge is successful, the signed document will be removed without affecting any other signed documents in the archive.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Certificate content

Conforming certificates SHALL meet all requirements of this section.

Certificates MUST indicate that a certificate has no well-defined expiration date by setting the notAfter field to the GeneralizedTime value 99991231235959Z, as defined in {{RFC5280}}.

Certificates MUST include the id-ce-noRevAvail extension in compliance with {{RFC9608}}, indicating that this certificate is not supported by any revocation mechanism.

Certificates MUST include the signedDocumentBinding extension, binding the certificate to a specific signed content.

## The signedDocumentBinding extension

The signedDocumentBinding extension binds a certificate to a specific signed content. When present, conforming CAs SHOULD mark this extension as non-critical.


    name           id-ce-signedDocumentBinding
    OID            { id-pe TBD }
    syntax         SignedDocumentBinding
    criticality    SHOULD be FALSE

    SignedDocumentBinding ::= SEQUENCE {
    dataTbsHash     OCTET STRING,
    hashAlg         OBJECT IDENTIFIER,
    bindingType     UTF8String OPTIONAL }


The dataTbsHash field SHALL contain a hash of the data to be signed.

The hashAlg field SHALL contain the OID of the hash algorithm used to generate the dataTbsHash value.

The bindingType field MAY contain an optional identifier that defines how the data to be signed is derived from the document to be signed.
When omitted, data to be signed is identical to the data signed by the generated signature. Some signature standards, such as CMS and ETSI signature profiles includes an option or requirement to sign the signer certificate. This reference to the signer certificate MUST be omitted from the hashed dataTbsHash calculation to avoid circular dependency. The process to exclude the signer certificate is defined by the bindingType identifier

## Defined bindingType identifiers

TODO Define identified procedures for handling CMS ESSCertV2 and ETSI profiles XAdES, JAdES and PAdES. (CAdES is equal to ESSCertV2)


# ASN.1 Module

    <CODE BEGINS>
       NoRevAvailExtn
         { iso(1) identified-organization(3) dod(6) internet(1)
           security(5) mechanisms(5) pkix(7) id-mod(0)
           id-mod-signedDocumentBinding(TBD) }

       DEFINITIONS IMPLICIT TAGS ::=
       BEGIN

       IMPORTS
         EXTENSION, id-pkix, id-pe
         FROM PKIX-CommonTypes-2009  -- RFC 5912
           { iso(1) identified-organization(3) dod(6) internet(1)
             security(5) mechanisms(5) pkix(7) id-mod(0)
             id-mod-pkixCommon-02(57) } ;

       -- signedDocumentBinding Certificate Extension

       ext-SignedDocumentBinding EXTENSION ::= {
         SYNTAX SignedDocumentBinding
         IDENTIFIED BY id-pe-signedDocumentBinding }

       SignedDocumentBinding ::= SEQUENCE {
         dataTbsHash     OCTET STRING,
         hashAlg         OBJECT IDENTIFIER,
         bindingType     UTF8String }

       -- signedDocumentBinding Certificate Extension OID

       id-pe-signedDocumentBinding OBJECT IDENTIFIER ::= { id-pe TBD }

       END
     <CODE ENDS>

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
