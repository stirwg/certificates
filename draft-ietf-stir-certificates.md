---
title: "Secure Telephone Identity Credentials: Certificates"
abbrev: STIR Certs
docname: draft-ietf-stir-certificates-latest
date: {DATE}
category: std
ipr: trust200902
area: ART
workgroup: STIR

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: J. Peterson
    name: Jon Peterson
    org: Neustar, Inc.
    abbrev: Neustar
    email: jon.peterson@neustar.biz
  -
    ins: S. Turner
    name: Sean Turner
    org: sn3rd
    email: sean@sn3rd.com

normative:

  ATIS-0300251:
      title: "Codes for Identification of Service Providers for Information Exchange"
      date: 2007
      author:
        org: ATIS Recommendation 0300251
  DSS:
      title: "Digital Signature Standard, version 4"
      date: 2013
      author:
        org: National Institute of Standards and Technology, U.S. Department of Commerce
      seriesinfo:
        NIST: FIPS PUB 186-4
  X.509:
       title: "Information technology - Open Systems Interconnection - The Directory: Public-key and attribute certificate frameworks"
       date: 2012
       author:
         org: ITU-T Recommendation X.509 | ISO/IEC 9594-8
         
  X.680:
        title: "Information Technology - Abstract Syntax Notation One: Specification of basic notation"
        date: 2015
        author:
            org: ITU-T Recommendation X.680 | ISO/IEC 8824-1
  X.681:
        title: "Information Technology - Abstract Syntax Notation One: Information Object Specification"
        date: 2015
        author:
            org: ITU-T Recommendation X.681 | ISO/IEC 8824-2
  X.682:
        title: "Information Technology - Abstract Syntax Notation One: Constraint Specification"
        date: 2015
        author:
            org: ITU-T Recommendation X.682 | ISO/IEC 8824-2
  X.683:
        title: "Information Technology - Abstract Syntax Notation One: Parameterization of ASN.1 Specifications"
        date: 2015
        author:
            org: ITU-T Recommendation X.683 | ISO/IEC 8824-3

informative:

  X.520:
        title: "Information technology - Open Systems Interconnection - The Directory: Selected Attribute Types"
        date: 2012
        author:
            org: ITU-T Recommendation X.520 | ISO/IEC 9594-6 

--- abstract

In order to prevent the impersonation of telephone numbers on the
Internet, some kind of credential system needs to exist that
cryptographically asserts authority over telephone numbers.  This
document describes the use of certificates in establishing authority
over telephone numbers, as a component of a broader architecture for
managing telephone numbers as identities in protocols like SIP.

--- middle

# Introduction

The STIR problem statement {{?RFC7340}} identifies the primary enabler
of robocalling, vishing, swatting and related attacks as the
capability to impersonate a calling party number.  The starkest
examples of these attacks are cases where automated callees on the
PSTN rely on the calling number as a security measure, for example to
access a voicemail system.  Robocallers use impersonation as a means
of obscuring identity; while robocallers can, in the ordinary PSTN,
block (that is, withhold) their caller identity, callees are less
likely to pick up calls from blocked identities, and therefore
appearing to call from some number, any number, is preferable.
Robocallers however prefer not to call from a number that can trace
back to the robocaller, and therefore they impersonate numbers that
are not assigned to them.

One of the most important components of a system to prevent
impersonation is the implementation of credentials which identify the
parties who control telephone numbers.  With these credentials,
parties can assert that they are in fact authorized to use telephony
numbers, and thus distinguish themselves from impersonators unable to
present such credentials.  For that reason the STIR threat model
{{?RFC7375}} stipulates, "The design of the credential system envisioned
as a solution to these threats must, for example, limit the scope of
the credentials issued to carriers or national authorities to those
numbers that fall under their purview."  This document describes
credential systems for telephone numbers based on {{X.509}} version 3
certificates in accordance with {{!RFC5280}}.  While telephone numbers
have long been part of the X.509 standard (X.509 supports arbitrary
naming attributes to be included in a certificate; the
telephoneNumber attribute was defined in the 1988 {{X.520}}
specification) this document provides ways to determine authority
more aligned with telephone network requirements, including extending
X.509 with a Telephone Number Authorization List certificate
extension which binds certificates to asserted authority for
particular telephone numbers, or potentially telephone number blocks
or ranges.

In the STIR in-band architecture specified in
{{?I-D.ietf-stir-rfc4474bis}}, two basic types of entities need access
to these credentials: authentication services, and verification
services (or verifiers).  An authentication service must be operated
by an entity enrolled with the certification authority (CA, see
{{enrollment}}), whereas a verifier need only trust the trust anchor of
the authority, and have a means to access and validate the public
keys associated with these certificates.  Although the guidance in
this document is written with the STIR in-band architecture in mind,
the credential system described in this document could be useful for
other protocols that want to make use of certificates to assert
authority over telephone numbers on the Internet.

This document specifies only the credential syntax and semantics
necessary to support this architecture.  It does not assume any
particular CA or deployment environment.  We anticipate that some
deployment experience will be necessary to determine optimal
operational models.


# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in RFC
2119 {{!RFC2119}}.


# Authority for Telephone Numbers in Certificates

At a high level, this specification details two non-exclusive
approaches that can be employed to determine authority over telephone
numbers with certificates.

The first approach is to leverage the existing subject of the
certificate to ascertain that the holder of the certificate is
authorized to claim authority over a telephone number.  The subject
might be represented as a domain name in the subjectAltName, such as
an "example.net" where that domain is known to relying parties as a
carrier, or represented with other identifiers related to the
operation of the telephone network including Service Provider codes
(SPCs) such as OCNs or SPIDs via the TN Authorization List specified
in this document.  A relying party could then employ an external data
set or service that determines whether or not a specific telephone
number is under the authority of the carrier identified as the
subject of the certificate, and use that to ascertain whether or not
the carrier should have authority over a telephone number.
Potentially, a certificate extension to convey the URI of such an
information service trusted by the issuer of the certificate could be
developed (though this specification does not propose one).
Alternatively, some relying parties could form bilateral or
multilateral trust relationships with peer carriers, trusting one
another's assertions just as telephone carriers in the SS7 network
today rely on transitive trust when displaying the calling party
telephone number received through SS7 signaling.

The second approach is to extend the syntax of certificates to
include a new attribute, defined here as TN Authorization List, which
contains a list of telephone numbers defining the scope of authority
of the certificate.  Relying parties, if they trust the issuer of the
certificate as a source of authoritative information on telephone
numbers, could therefore use the TN Authorization List instead of the
subject of the certificate to make a decision about whether or not
the signer has authority over a particular telephone number.  The TN
Authorization List could be provided in one of two ways: as a literal
value in the certificate, or as a network service that allows relying
parties to query in real time to determine that a telephone number is
in the scope of a certificate.  Using the TN Authorization list
rather than the certificate subject makes sense when, for example,
for privacy reasons, the certificate owner would prefer not to be
identified, or in cases where the holder of the certificate does not
participate in the sort of traditional carrier infrastructure that
the first approach assumes.

The first approach requires little change to existing Public Key
Infrastructure (PKI) certificates; for the second approach, we must
define an appropriate enrollment and authorization process.  For the
purposes of STIR, the over-the-wire format specified in
{{!I-D.ietf-stir-rfc4474bis}} accommodates either of these approaches:
the methods for canonicalizing, signing, for identifying and
accessing the certificate and so on remain the same; it is only the
verifier behavior and authorization decision that will change
depending on the approach to telephone number authority taken by the
certificate.  For that reason, the two approaches are not mutually
exclusive, and in fact a certificate issued to a traditional
telephone network service provider could contain a TN Authorization
List or not, were it supported by the CA issuing the credential.
Regardless of which approach is used, certificates that assert
authority over telephone numbers are subject to the ordinary
operational procedures that govern certificate use per {{RFC5280}}.
This means that verification services must be mindful of the need to
ensure that they trust the trust anchor that issued the certificate,
and that they have some means to determine the freshness of the
certificate (see {{cert-fresh}}).

# Certificate Usage with STIR

{{I-D.ietf-stir-rfc4474bis}} Section 7.4 requires that all credential
systems used by STIR explain how they address the requirements
enumerated below.  Certificates as described in this document address
the STIR requirements as follows:

1.  The URI {{!RFC3986}} schemes permitted in the SIP Identity header
"info" parameter, as well as any special procedures required to
dereference the URIs: while normative text is given below in
Section 7, this mechanism permits the HTTP {{!RFC7230}}, CID and SIP
URI schemes to appear in the "info" parameter.

2.  Procedures required to extract keying material from the resources
designated by the URI: implementations perform no special
procedures beyond dereferencing the "info" URI.  See Section 7.

3.  Procedures used by the verification service to determine the
scope of the credential: this specification effectively proposes
two methods, as outlined in Section 3: one where the subject (or
more properly subjectAltName) of the certificate indicates the
scope of authority through a domain name, and relying parties
either trust the subject entirely or have some direct means of
determining whether or not a number falls under a subject's
authority; and another where an extension to the certificate as
described in Section 9 identifies the scope of authority of the
certificate.

4.  The cryptographic algorithms required to validate the
credentials: for this specification, that means the signature
algorithms used to sign certificates.  This specification
REQUIRES that implementations support both ECDSA with the P-256
curve (see {{DSS}}) and RSA PKCS#1 v1.5 (see {{!RFC3447}} Section 8.2)
for certificate signatures.  Implementers are advised that RS256
is mandated only as a transitional mechanism, due to its
widespread use in existing PKI, but we anticipate that this
mechanism will eventually be deprecated.

5.  Finally, note that all certificates compliant with this
specification:

    *  MUST provide cryptographic keying material sufficient to
generate the ECDSA using P-256 and SHA-256 signatures
necessary to support the ES256 hashed signatures required by
PASSporT {{!I-D.ietf-stir-passport}}, which in turn follows JSON
Web Token (JWT) {{!RFC7519}}.

    *  MUST support both ECDSA with P-256 and RSA PKCS#1 v1.5 for
certificate signature verification.

This document also includes additional certificate-related
requirements:

* See Section 5.1 for requirements related to the certificate
policies extension.

* See Section 7 for requirements related to relying parties
acquiring credentials.

* See Section 10 and Section 10.1 for requirements related to
certificate freshness and the Authority Information Access (AIA)
certificate extension.

# Enrollment and Authorization using the TN Authorization List {#enrollment}

This document covers three models for enrollment when using the TN
Authorization List extension.

The first enrollment model is one where the CA acts in concert with
national numbering authorities to issue credentials to those parties
to whom numbers are assigned.  In the United States, for example,
telephone number blocks are assigned to Local Exchange Carriers
(LECs) by the North American Numbering Plan Administrator (NANPA),
who is in turn directed by the national regulator.  LECs may also
receive numbers in smaller allocations, through number pooling, or
via an individual assignment through number portability.  LECs assign
numbers to customers, who may be private individuals or organizations
- and organizations take responsibility for assigning numbers within
their own enterprise.  This model requires top-down adoption of the
model from regulators through to carriers.  Assignees of E.164
numbering resources participating in this enrollment model should
take appropriate steps to establish trust anchors.

The second enrollment model is a bottom-up approach where a CA
requires that an entity prove control by means of some sort of test,
which, as with certification authorities for web PKI, might either be
automated or a manual administrative process.  As an example of an
automated process, an authority might send a text message to a
telephone number containing a URL (which might be dereferenced by the
recipient) as a means of verifying that a user has control of
terminal corresponding to that number.  Checks of this form are
frequently used in commercial systems today to validate telephone
numbers provided by users.  This is comparable to existing enrollment
systems used by some certificate authorities for issuing S/MIME
credentials for email by verifying that the party applying for a
credential receives mail at the email address in question.

The third enrollment model is delegation: that is, the holder of a
certificate (assigned by either of the two methods above) might
delegate some or all of their authority to another party.  In some
cases, multiple levels of delegation could occur: a LEC, for example,
might delegate authority to a customer organization for a block of
100 numbers used by an IP PBX, and the organization might in turn
delegate authority for a particular number to an individual employee.
This is analogous to delegation of organizational identities in
traditional hierarchical PKIs who use the name constraints extension
{{RFC5280}}; the root CA delegates names in sales to the sales
department CA, names in development to the development CA, etc.  As
lengthy certificate delegation chains are brittle, however, and can
cause delays in the verification process, this document considers
optimizations to reduce the complexity of verification.

Future work might explore methods of partial delegation, where
certificate holders delegate only part of their authority.  For
example, individual assignees may want to delegate to a service
authority for text messages associated with their telephone number,
but not for other functions.

## Constraints on Signing PASSporTs

The public key in the certificate is used to validate the signature
on a JSON Web Token (JWT) {{RFC7519}} that conforms to the conventions
specified in PASSporT {{I-D.ietf-stir-passport}}.  This specification
supports constraints on the JWT claims, which allows the CA to grant
different permissions to certificate holders, for example those
enrolled from proof-of-possession versus delegation.  A Certification
Policy and a Certification Practice Statement {{?RFC3647}} are produced
as part of the normal PKI bootstrapping process, (i.e., the CP is
written first and then the CA says how it conforms to the CP in the
CPS).  A CA that wishes to place constraints on the JWT claims MUST
include the JWT Claim Constraints certificate extension in issued
certificates.  See Section 8 for information about the certificate
extension.

## Certificate Extension Scope and Structure

This specification places no limits on the number of telephone
numbers that can be associated with any given certificate.  Some
service providers may be assigned millions of numbers, and may wish
to have a single certificate that can be applied to signing for any
one of those numbers.  Others may wish to compartmentalize authority
over subsets of the numbers they control.

Moreover, service providers may wish to have multiple certificates
with the same scope of authority.  For example, a service provider
with several regional gateway systems may want each system to be
capable of signing for each of their numbers, but not want to have
each system share the same private key.

The set of telephone numbers for which a particular certificate is
valid is expressed in the certificate through a certificate
extension; the certificate's extensibility mechanism is defined in
{{RFC5280}} but the TN Authorization List extension is specified in
this document.

The subjects of certificates containing the TN Authorization List
extension are typically the administrative entities to whom numbers
are assigned or delegated.  For example, a LEC might hold a
certificate for a range of telephone numbers.  In some cases, the
organization or individual issued such a certificate may not want to
associate themselves with a certificate; for example, a private
individual with a certificate for a single telephone number might not
want to distribute that certificate publicly if every verifier
immediately knew their name.  The certification authorities issuing
certificates with the TN Authorization List extensions may, in
accordance with their policies, obscure the identity of the subject,
though mechanisms for doing so are outside the scope of this
document.

# Provisioning Private Keying Material

In order for authentication services to sign calls via the procedures
described in {{I-D.ietf-stir-rfc4474bis}}, they must hold a private key
corresponding to a certificate with authority over the calling
number.  {{I-D.ietf-stir-rfc4474bis}} does not require that any
particular entity in a SIP deployment architecture sign requests,
only that it be an entity with an appropriate private key; the
authentication service role may be instantiated by any entity in a
SIP network.  For a certificate granting authority only over a
particular number which has been issued to an end user, for example,
an end user device might hold the private key and generate the
signature.  In the case of a service provider with authority over
large blocks of numbers, an intermediary might hold the private key
and sign calls.

The specification RECOMMENDS distribution of private keys through
PKCS#8 objects signed by a trusted entity, for example through the
CMS package specified in {{!RFC5958}}.

# Acquiring Credentials to Verify Signatures {#cert-acquire}

This specification documents multiple ways that a verifier can gain
access to the credentials needed to verify a request.  As the
validity of certificates does not depend on the method of their
acquisition, there is no need to standardize any single mechanism for
this purpose.  All entities that comply with
{{I-D.ietf-stir-rfc4474bis}} necessarily support SIP, and consequently
SIP itself can serve as a way to deliver certificates.
{{I-D.ietf-stir-rfc4474bis}} provides an "info" parameter of the
Identity header which contains a URI for the credential used to
generate the Identity header; {{I-D.ietf-stir-rfc4474bis}} also
requires documents which define credential systems list the URI
schemes that may be present in the "info" parameter.  For
implementations compliant with this specification, three URI schemes
are REQUIRED: the CID URI, the SIP URI, and the HTTP URI.

The simplest way for a verifier to acquire the certificate needed to
verify a signature is for the certificate be conveyed in a SIP
request along with the signature itself.  In SIP, for example, a
certificate could be carried in a multipart MIME body {{?RFC2046}}, and
the URI in the Identity header "info" parameter could specify that
body with a CID URI {{!RFC2392}}.  However, in many environments this is
not feasible due to message size restrictions or lack of necessary
support for multipart MIME.

The Identity header "info" parameter in a SIP request may contain a
URI that the verifier dereferences.  Implementations of this
specification are REQUIRED to support the use of SIP for this
function (via the SUBSCRIBE/NOTIFY mechanism) as well as HTTP and
HTTPS.

Note well that as an optimization, a verifier may have access to a
service, a cache or other local store that grants access to
certificates for a particular telephone number.  However, there may
be multiple valid certificates that can sign a call setup request for
a telephone number, and as a consequence, there needs to be some
discriminator that the signer uses to identify their credentials.
The Identity header "info" parameter itself can serve as such a
discriminator, provided implementations use that parameter as a key
when accessing certificates from caches or other sources.

# JWT Claim Constraints Syntax

Certificate subjects are limited to specific values for PASSporT claims
with the JWT Claim Constraints certificate extension; issuers permit
all claims by omitting the JWT Claim Constraints certificate extension
from the certificate's extension field {{RFC5280}}.  The extension is
non-critical, applicable only to end-entity certificates, and defined
with ASN.1 {{X.680}}{{X.681}}{{X.682}}{{X.683}} later in this section.
The syntax of the claims is given in PASSporT; specifying new claims
follows the procedures in {{I-D.ietf-stir-passport}} (Section 8.3).

This certificate extension is optional, but if present, it constrains
the claims that authentication services may include in the PASSporT
objects they sign.  Constraints are applied by issuers and enforced by
verifiers when validating PASSporT claims as follows:

1.  mustInclude indicates claims that MUST appear in the PASSporT in
addition to iat, orig, and dest.  The baseline claims of PASSporT
("iat", "orig", and "dest") are considered to be permitted by default
and SHOULD NOT be included.  If mustInclude is absent, iat, orig, and
dest MUST appear in the PASSporT.

2.  permittedValues indicates that if the claim name is present, the
claim MUST contain one of the listed values. 

Consider two examples with a PASSporT claim called "confidence" with
values "low", "medium", and  "high":

* If a CA issues to an authentication service a certificate that
contains the mustInclude JWTClaimName "confidence", then an
authentication service MUST include the "confidence" claim in all
PASSporTs it generates; a verification service will treat as invalid
any PASSporT it receives with a PASSporT claim that does not include
the "confidence" claim.

* If a CA issues to an authentication service a certificate that
contains the permittedValues JWTClaimName "confidence" and a permitted
"high" value, then an authentication service will treat as invalid any
PASSporT it receives with  a PASSporT claim that does not include the
"confidence" claim with a "high" value.  

The JWT Claim Constraints certificate extension is identified by the
following object identifier (OID), which is defined under the id-pe
OID arc defined in {{RFC5280}} and managed by IANA (see {{iana}}):

~~~
  id-pe-JWTClaimConstraints OBJECT IDENTIFIER ::= { id-pe 25 }
~~~

The JWT Claim Constraints certificate extension has the following
syntax:

~~~
  JWTClaimConstraints ::= SEQUENCE {
    mustInclude [0] JWTClaimNames OPTIONAL,
      -- The listed claim names MUST appear in the PASSporT in addition
      -- to iat, orig, and dest.  If absent, iat, orig, and dest MUST
      -- appear in the PASSporT.
    permittedValues [1] JWTClaimPermittedValuesList OPTIONAL }
      -- If the claim name is present, the claim MUST contain one of
      -- the listed values.
  ( WITH COMPONENTS { ..., mustInclude PRESENT } |
    WITH COMPONENTS { ..., permittedValues PRESENT } )

  JWTClaimPermittedValuesList ::= SEQUENCE SIZE (1..MAX) OF
                                    JWTClaimPermittedValues

  JWTClaimPermittedValues ::= SEQUENCE {
    claim  JWTClaimName,
    permitted  SEQUENCE SIZE (1..MAX) OF UTF8String }

  JWTClaimNames ::= SEQUENCE SIZE (1..MAX) OF JWTClaimName

  JWTClaimName ::= IA5String
~~~

# TN Authorization List Syntax {#tn-authz-list}

The subjects of certificates containing the TN Authorization List
extension are the administrative entities to whom numbers are
assigned or delegated.  When a verifier is validating a caller's
identity, local policy always determines the circumstances under
which any particular subject may be trusted, but the purpose of the
TN Authorization List extension in particular is to allow a verifier
to ascertain when the CA has designated that the subject has
authority over a particular telephone number or number range.  The
non critical Telephony Number (TN) Authorization List certificate
extension is included in the Certificate's extension field {{RFC5280}}.
The extension is defined with ASN.1 {{X.680}}{{X.681}}{{X.682}}
{{X.683}}.  What follows is the syntax and semantics of the extension.

The subjects of certificates containing the TN Authorization List
extension are the administrative entities to whom numbers are
assigned or delegated.  In an end entity certificate, TN
Authorization List indicates the TNs which the certificate has been
authorized.  In a CA certificate, the TN Authorization List limits
the set of TNs for certification paths that include this certificate.

The Telephony Number (TN) Authorization List certificate extension is
identified by the following object identifier (OID), which is defined
under the id-pe OID arc defined in {{RFC5280}} and managed by IANA (see
{{iana}}):

~~~
  id-pe-TNAuthList OBJECT IDENTIFIER ::= { id-pe 26 }
~~~

The TN Authorization List certificate extension has the following
syntax:

~~~
  TNAuthorizationList ::= SEQUENCE SIZE (1..MAX) OF TNEntry

  TNEntry ::= CHOICE {
    spc   [0] ServiceProviderCode,
    range [1] TelephoneNumberRange,
    one   [2] TelephoneNumber
    }

  ServiceProviderCode ::= IA5String

  -- Service Provider Codes may be OCNs, various SPIDs, or other
  -- SP identifiers from the telephone network

  TelephoneNumberRange ::= SEQUENCE {
    start TelephoneNumber,
    count INTEGER (2..MAX)
    }

  TelephoneNumber ::= IA5String (SIZE (1..15)) (FROM ("0123456789#*"))
~~~

The TN Authorization List certificate extension indicates the
authorized phone numbers for the call setup signer.  It indicates one
or more blocks of telephone number entries that have been authorized
for use by the call setup signer.  There are three ways to identify
the block:

1.  Service Provider Codes as described in this document are a
generic term for the identifiers used to designate service
providers in the telepohone networks today.  In North American
context, these would include Operating Company Numbers (OCNs) as
specified in {{ATIS-0300251}}, related Service Provide Identifiers
(SPIDs), or other similar identifiers for service providers.
SPCs can be used to indirectly name all of the telephone numbers
associated with that identifier for a service provider,

2.  Telephone numbers can be listed in a range (in the
TelephoneNumberRange format), which consists of a starting
telephone number and then an integer count of numbers within the
range, where the valid boundaries of ranges may vary according to
national policies, or

3.  A single telephone number can be listed (as a TelephoneNumber).

Note that because large-scale service providers may want to associate
many numbers, possibly millions of numbers, with a particular
certificate, optimizations are required for those cases to prevent
certificate size from becoming unmanageable.  In these cases, the TN
Authorization List may be given by reference rather than by value,
through the presence of a separate certificate extension that permits
verifiers to either securely download the list of numbers associated
with a certificate, or to verify that a single number is under the
authority of this certificate.  For more on this optimization, see
{{acquire}}.

# Certificate Freshness and Revocation {#cert-fresh}

Regardless of which of the approaches in Section 3 is followed for
using certificates, a certificate verification mechanism is required.
However, the traditional problem of certificate freshness gains a new
wrinkle when using the TN Authorization List extension with telephone
numbers or number ranges (as opposed to SPCs), because verifiers must
establish not only that a certificate remains valid, but also that
the certificate's scope contains the telephone number that the
verifier is validating.  Dynamic changes to number assignments can
occur due to number portability, for example.  So even if a verifier
has a valid cached certificate for a telephone number (or a range
containing the number), the verifier must determine that the entity
that signed is still a proper authority for that number.

To verify the status of such a certificate, the verifier needs to
acquire the certificate if necessary (via the methods described in
Section 7), and then would need to either:

a.  Rely on short-lived certificates and not check the certificate's
status, or

b.  Rely on status information from the authority (e.g., OCSP)

The tradeoff between short lived certificates and using status
information is that the former's burden is on the front end (i.e.,
enrollment) and the latter's burden is on the back end (i.e.,
verification).  Both impact call setup time, but some approaches to
generating a short-lived certificate, like requiring one for each
call, would incur a greater operational cost than acquiring status
information.  This document makes no particular recommndation for a
means of determinate certificate freshness for STIR, as this requires
further study and implementation experience.  Acquiring online status
information for certificates has the potential to disclose private
information {{!RFC7258}} if proper precautions are not taken.  Future
specifications that define certificate freshness mechanisms for STIR
MUST note any such risks and provide countermeasures where possible.

##  Acquiring TN Lists By Reference {#acquire}

One alternative to checking certificate status for a particular
telephone number is simply acquiring the TN Authorization List by
reference, that is, through dereferencing a URL in the certificate,
rather than including the value of the TN Authorization List in the
certificate itself.

Acquiring a list of the telephone numbers associated with a
certificate or its subject lends itself to an application-layer
query/response interaction outside of certificate status, one which
could be initiated through a separate URI included in the
certificate.  The AIA extension (see {{RFC5280}}) supports such a
mechanism: it designates an OID to identify the accessMethod and an
accessLocation, which would most likely be a URI.  A verifier would
then follow the URI to ascertain whether the list of TNs are
authorized for use by the caller.

HTTPS is the most obvious candidate for a protocol to be used for
fetching the list of telephone numbers associated with a particular
certificate.  This document defines a new AIA accessMethod, called
"id-ad-stirTNList", which uses the following AIA OID:

~~~
  id-ad-stirTNList  OBJECT IDENTIFIER ::= { id-ad 14 }
~~~

When the "id-ad-stirTNList" accessMethod is used, the accessLocation
MUST be an HTTPS URI.  The document returned by dereferencing that
URI will contain the complete TN Authorization List (see 
{{tn-authz-list}}) for the certificate.

Delivering the entire list of telephone numbers associated with a
particular certificate will divulge to STIR verifiers information
about telephone numbers other than the one associated with the
particular call that the verifier is checking.  In some environments,
where STIR verifiers handle a high volume of calls, maintaining an
up-to-date and complete cache for the numbers associated with crucial
certificate holders could give an important boost to performance.

# IANA Considerations {#iana}

This document makes use of object identifiers for the TN Certificate
Extension defined in Section 9, the TN by reference AIA access
descriptor defined in Section 10.1, and the ASN.1 module identifier
defined in Appendix A.  It therefore requests that the IANA make the
following assignments:

* JWT Claim Constraints Certificate Extension in the SMI Security
for PKIX Certificate Extension registry:

~~~
    http://www.iana.org/assignments/smi-numbers/smi-numbers.xhtml#smi-
    numbers-1.3.6.1.5.5.7.1
~~~

* TN Certificate Extension in the SMI Security for PKIX Certificate
Extension registry: http://www.iana.org/assignments/smi-numbers/
smi-numbers.xhtml#smi-numbers-1.3.6.1.5.5.7.1

*  TNS by reference access descriptor in the SMI Security for PKIX
Access Descriptor registry: http://www.iana.org/assignments/smi-
numbers/smi-numbers.xhtml#smi-numbers-1.3.6.1.5.5.7.48

*  The TN ASN.1 module in SMI Security for PKIX Module Identifier
registry: http://www.iana.org/assignments/smi-numbers/smi-
numbers.xhtml#smi-numbers-1.3.6.1.5.5.7.0

# Security Considerations

This document is entirely about security.  For further information on
certificate security and practices, see {{RFC5280}}, in particular its
Security Considerations.

# Acknowledgments

Anders Kristensen, Russ Housley, Brian Rosen, Cullen Jennings, Dave
Crocker, Tony Rutkowski, John Braunberger, and Eric Rescorla provided
key input to the discussions leading to this document.  Russ Housley
provided some direct assistance and text surrounding the ASN.1
module.

--- back

# ASN.1 Module

This appendix provides the normative ASN.1 {{X.680}} definitions for
the structures described in this specification using ASN.1, as
defined in {{X.680}} through {{X.683}}.

The modules defined in this document are compatible with the most
current ASN.1 specification published in 2015 (see {{X.680}}, {{X.681}},
{{X.682}}, {{X.683}}).  None of the newly defined tokens in the 2008
ASN.1 (DATE, DATE-TIME, DURATION, NOT-A-NUMBER, OID-IRI, RELATIVE-
OID-IRI, TIME, TIME-OF-DAY)) are currently used in any of the ASN.1
specifications referred to here.

This ASN.1 module imports ASN.1 from {{!RFC5912}}.

~~~

  TN-Module-2016
    { iso(1) identified-organization(3) dod(6) internet(1) security(5)
      mechanisms(5) pkix(7) id-mod(0) id-mod-tn-module(88) }

  DEFINITIONS EXPLICIT TAGS ::= BEGIN

  IMPORTS

  id-ad, id-pe
  FROM PKIX1Explicit-2009  -- From [RFC5912]
    { iso(1) identified-organization(3) dod(6) internet(1) security(5)
      mechanisms(5) pkix(7) id-mod(0) id-mod-pkix1-explicit-02(51) }

  EXTENSION
  FROM PKIX-CommonTypes-2009  -- From [RFC5912]
    { iso(1) identified-organization(3) dod(6) internet(1) security(5)
      mechanisms(5) pkix(7) id-mod(0) id-mod-pkixCommon-02(57) }

  ;

  --
  -- JWT Claim Constraints Certificate Extension
  --

  ext-jwtClaimConstraints EXTENSION  ::= {
    SYNTAX JWTClaimConstraints IDENTIFIED BY id-pe-JWTClaimConstraints
    } 

  id-pe-JWTClaimConstraints OBJECT IDENTIFIER ::= { id-pe 25 }

  JWTClaimConstraints ::= SEQUENCE {
    mustInclude [0] JWTClaimNames OPTIONAL,
      -- The listed claim names MUST appear in the PASSporT in addition
      -- to iat, orig, and dest.  If absent, iat, orig, and dest MUST
      -- appear in the PASSporT.
    permittedValues [1] JWTClaimPermittedValuesList OPTIONAL }
      -- If the claim name is present, the claim MUST contain one of
      -- the listed values.
  ( WITH COMPONENTS { ..., mustInclude PRESENT } |
    WITH COMPONENTS { ..., permittedValues PRESENT } )

  JWTClaimPermittedValuesList ::= SEQUENCE SIZE (1..MAX) Of
                                    JWTClaimPermittedValues
 
  JWTClaimPermittedValues ::= SEQUENCE {
    claim  JWTClaimName,
    permitted  SEQUENCE SIZE (1..MAX) OF UTF8String }
 
  JWTClaimNames ::= SEQUENCE SIZE (1..MAX) OF JWTClaimName

  JWTClaimName ::= IA5String

  --
  -- Telephone Number Authorization List Certificate Extension
  --

  ext-tnAuthList  EXTENSION  ::= {
    SYNTAX TNAuthorizationList IDENTIFIED BY id-pe-TNAuthList
    }

  id-pe-TNAuthList OBJECT IDENTIFIER ::= { id-pe 26 }

  TNAuthorizationList ::= SEQUENCE SIZE (1..MAX) OF TNEntry

  TNEntry ::= CHOICE {
    spc    [0] ServiceProviderCode,
    range  [1] TelephoneNumberRange,
    one    [2] TelephoneNumber
    }

  ServiceProviderCode ::= IA5String

  -- Service Provider Codes may be OCNs, various SPIDs, or other
  -- SP identifiers from the telephone network

  TelephoneNumberRange ::= SEQUENCE {
    start TelephoneNumber,
    count INTEGER (2..MAX)
    }

  TelephoneNumber ::= IA5String (SIZE (1..15)) (FROM ("0123456789"))

  -- TN Access Descriptor

  id-ad-stirTNList OBJECT IDENTIFIER ::= { id-ad 14 }

  END
~~~
