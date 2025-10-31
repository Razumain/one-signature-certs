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
  RFC5035:
  RFC5280:
  RFC5652:
  RFC7515:
  RFC8174:
  RFC9608:
  XMLDSIG11:
    title: "XML Signature Syntax and Processing Version 1.1"
    author:
      -
        ins: D. Eastlake
        name: Donald Eastlake
      -
        ins: J. Reagle
        name: Joseph Reagle
      -
        ins: D. Solo
        name: David Solo
      -
        ins: F. Hirsch
        name: Frederick Hirsch
      -
        ins: M. Nystrom
        name: Magnus Nystrom
      -
        ins: T. Roessler
        name: Thomas Roessler
      -
        ins: K. Yiu
        name: Kelvin Yiu
    date: 2013-04-11
    seriesinfo:
      "W3C": "Proposed Recommendation"
  ISOPDF2:
    title: "Document management -- Portable document format -- Part 2: PDF 2.0"
    author:
      org: ISO
    date: 2017-07
    seriesinfo:
      "ISO": "32000-2"
  XADES:
    title: "Electronic Signatures and Infrastructures (ESI); XAdES digital signatures; Part 1: Building blocks and XAdES baseline signatures"
    author:
      org: ETSI
    date: 2024-07
    seriesinfo:
      "ETSI": "EN 319 132-1 v1.3.1"
  PADES:
    title: "Electronic Signatures and Infrastructures (ESI); PAdES digital signatures; Part 1: Building blocks and PAdES baseline signatures"
    author:
      org: ETSI
    date: 2024-01
    seriesinfo:
      "ETSI": "EN 319 142-1 v1.2.1"
  CADES:
    title: "Electronic Signatures and Infrastructures (ESI); CAdES digital signatures; Part 1: Building blocks and CAdES baseline signatures"
    author:
      org: ETSI
    date: 2021-10
    seriesinfo:
      "ETSI": "EN 319 122-1 v1.2.1"

informative:
  RFC9321:

...

--- abstract

This document defines a profile for one-time-use certificates that are issued for a single signing operation. Each certificate is created at the time of signing and bound to the signed content. The associated signing key is immediately destroyed after use, and the certificate never expires and is never revoked. This simplifies long-term validation by removing the need for revocation or expiration handling.

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


    name           id-pe-signedDocumentBinding
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

The bindingType field defines how the data to be signed (dataTbsHash) is derived from the signed document.
This field identifies a deterministic procedure for selecting the portion of the signed content that is included in the hash computation.
When the field is omitted, the rules for the default binding type apply.

The purpose of the dataTbsHash value is to bind the certificate to the document being signed, not to protect the document’s integrity.
The integrity of the signed content is provided by the signature itself.
If any byte of the signed document is modified, the calculated hash will no longer match the certificate.
Therefore, the dataTbsHash enables validators and relying parties to confirm that the certificate was issued for the exact content that was signed.

Validators SHOULD verify that the signed document matches the certificate’s binding information.
This verification is not required for the signature to validate successfully but provides an additional safeguard against misuse or substitution of certificates.

### Default Binding

Identifier: "default"

The default binding applies when the bindingType field is absent or set to "default".
In this case, the dataTbsHash value is the hash of the exact data that is hashed and signed by the signature format in use.

Examples include:
- For XML Signatures {{XMLDSIG11}}, the hash of the SignedInfo element.
- For CMS Signatures {{RFC5652}}, the DER-encoded SignedAttributes structure.
- For other formats, the data structure input directly to the signature algorithm.

This bindingType MUST NOT be used when the data to be signed includes either the signer certificate itself or a hash of the signer certificate. This includes JWS and COSE signed documents that can include signer certificates in the protected header. JWS signatures {{RFC7515}} MUST use the "jws" bindingType and COSE signatures {{RFC8152}} MUST use the "cose" binding type.

This document defines a set of bindingType identifiers. Additional bindingType identifiers MAY be defined by future specifications.

### CAdES Binding

Identifier: "cades"

For CMS {{RFC5652}} or ETSI CAdES {{CADES}} signatures incorporating SigningCertificate or SigningCertificateV2 attributes {{RFC5035}} in signedAttrs,
the dataTbsHash value is computed over the DER encoding of SignerInfo excluding any instances of SigningCertificate or SigningCertificateV2 attributes from the SignedAttributes set.

This bindingType also applies to PDF {{ISOPDF2}} and ETSI PAdES {{PADES}} signed documents when applicable due to its use of CMS for signing.

### XAdES Binding

Identifier: "xades"

For ETSI XML Advanced Electronic Signatures {{XADES}}, the dataTbsHash value is computed over the canonicalized SignedInfo element,
with any Reference elements whose Type attribute equals "http://uri.etsi.org/01903#SignedProperties" removed prior to hashing.
This ensures that the SignedProperties element, which may contain references to the signing certificate, does not create a circular dependency. Extraction of the Reference element MUST be done by removing only the characters from the leading <Reference> tag up to and including the ending </Reference> tag, preserving all other bytes of SignedInfo unchanged, including any white space or line feeds.

Note: This operation is purely textual and does not require XML parsing beyond locating the tag boundaries.

### JWS Binding

Identifier: "jws"

For JSON Web Signatures (JWS) {{RFC7515}}, the dataTbsHash value is computed over the payload only.
The protected header and any unprotected header parameters MUST NOT be included in the hash calculation.

This exclusion avoids circular dependencies where certificate data may appear in the protected header.

### COSE Binding

Identifier: "cose"

For COSE signatures {{RFC8152}}, the dataTbsHash value is computed over the payload only.
The protected header and any unprotected header parameters MUST NOT be included in the hash calculation.

This exclusion avoids circular dependencies where certificate data may appear in the protected header.


# ASN.1 Module

    <CODE BEGINS>
       SignedDocumentBindingExtn
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
         bindingType     UTF8String OPTIONAL }

       -- signedDocumentBinding Certificate Extension OID

       id-pe-signedDocumentBinding OBJECT IDENTIFIER ::= { id-pe TBD }

       END
     <CODE ENDS>

# Security Considerations

TODO Security Considerations. Including text on reliance on certificates without revocation.

# IANA Considerations

TBD IANA registry for bindingType identifiers


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
