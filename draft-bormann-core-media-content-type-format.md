---
docname: draft-bormann-core-media-content-type-format-latest
stand_alone: true
ipr: trust200902
cat: std
#consensus: 'yes'
#submissiontype: IETF
pi:
  compact: 'yes'
  text-list-symbols: o*+-
  subcompact: 'no'
  sortrefs: 'yes'
  symrefs: 'yes'
  strict: 'yes'
  toc: 'yes'
title: On Media-Types, Content-Types, and related terminology
abbrev: Content-Types
date: 2021-02-22
author:
  -
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org
  -
    name: Henk Birkholz
    org: Fraunhofer SIT
    abbrev: Fraunhofer SIT
    email: henk.birkholz@sit.fraunhofer.de
    street: Rheinstrasse 75
    code: '64295'
    city: Darmstadt
    country: Germany

normative:
  IANA.core-parameters: core-parameters
  IANA.media-types: media-types
  IANA.http-parameters: http-parameters
informative:
#  RFC0020: ascii
#  RFC3629: utf8
  RFC4288: mediatype-reg-old
  RFC5234: abnf
  RFC6838: mediatype-reg
  RFC7231: http
  RFC7252: coap
  I-D.keranen-core-senml-data-ct: senml-ct
  RFC8152: cose
  I-D.ietf-cose-rfc8152bis-struct-15: cosebis

--- abstract

There is a lot of confusion about media-types, content-types, and
related terminology.

This memo is an attempt at clearing it up, so we can use consistent
terminology in CoRE and related specifications.
It also defines some ABNF that can be used in these specifications.

--- middle

# Introduction

{::boilerplate bcp14}

{{?RFC1590}} introduced media types and their registration.
That document took MIME types from {{?RFC1521}} and gave them a new name.
At that time, the term "media type" was often used just for the major
type ("text", "audio"), and what we call a media-type now was the
combination of a type and a subtype.  This lives on in {{-mediatype-reg}},
which does not even have an ABNF {{-abnf}} production for media type.
{{-mediatype-reg}}'s predecessor, {{-mediatype-reg-old}}, supplied the ABNF shown in ({{abnf-type-subtype1}}).

<blockquote markdown="1">
~~~ abnf;old
type-name = reg-name
subtype-name = reg-name

reg-name = 1*127reg-name-chars
reg-name-chars = ALPHA / DIGIT / "!" /
                 "#" / "$" / "&" / "." /
                 "+" / "-" / "^" / "_"
~~~
{: #abnf-type-subtype1 artwork-align="center" title="ABNF for type and
subtype, cited from RFC 4288"}
</blockquote>

{{-mediatype-reg}}, obsoleting {{-mediatype-reg-old}}, restricts the
first character of a reg-name to alphanumeric.
It contains the otherwise semantically equivalent ABNF shown in
{{ABNF-type-subtype2}}, however adding prose comments that further
limit the use of "."  and "+".

~~~~ abnf
type-name = restricted-name
subtype-name = restricted-name

restricted-name = restricted-name-first *126restricted-name-chars
restricted-name-first  = ALPHA / DIGIT
restricted-name-chars  = ALPHA / DIGIT / "!" / "#" /
                         "$" / "&" / "-" / "^" / "_"
restricted-name-chars =/ "." ; Characters before first dot always
                             ; specify a facet name
restricted-name-chars =/ "+" ; Characters after last plus always
                             ; specify a structured syntax suffix
~~~~
{: #ABNF-type-subtype2 title="ABNF for type and subtype, as defined from RFC 6838"}


# Media-Type

Today, the term "media type" is now generally used for a registered
combination of a type-name and a subtype-name, as well as for the
specification that defines the semantics of this combination.
We further disambiguate by calling the former a *media type name*.
An ABNF definition of `Media-Type-Name`:

~~~ abnf
Media-Type-Name = type-name "/" subtype-name
~~~
{: #ABNF-Media-Type-Name title="Definition of Media-Type-Name"}

For the purposes of this memo, we define:

Media-Type-Name:
: A combination of a type-name and a subtype-name
  registered in {{-media-types}}, conventionally identified by the two
  names separated by a slash.

(This leaves the term "Media Type" for the actual specification that
is registered under the Media-Type-Name.)

# Content-Type

Media types can have parameters {{-mediatype-reg}}, some of which are
defined by the media type specification to be mandatory.
In HTTP and many other protocols, media-type-names and parameters are
then used together in a "Content-Type" header field.
HTTP {{-http}} uses the ABNF in {{http-ct}}:

<blockquote markdown="1">
~~~ abnf;old
Content-Type = media-type
media-type = type "/" subtype *( OWS ";" OWS parameter )
type       = token
subtype    = token
token          = 1*tchar
tchar          = "!" / "#" / "$" / "%" / "&" / "'" / "*"
               / "+" / "-" / "." / "^" / "_" / "`" / "|" / "~"
               / DIGIT / ALPHA
OWS        = *( SP / HTAB )
~~~
{: #http-ct artwork-align="center" title="Content-Type ABNF from RFC 7231"}
</blockquote>

In the ABNF as established by
{{?RFC2616}}, parts of which became {{-http}}, the rule name
media-type is used for a Media-Type-Name with parameters attached.
We don't follow this inclusive use of media-type; note that
{{RFC2616}} was quite confused about this term by claiming ({{Section
3.7 of RFC2616}}):

>   Media-type values are registered with the Internet Assigned Number
>   Authority (IANA \[19]).

This clearly reverts to the understanding of Media-Type-Name we use.

In order to resolve some of this confusion, we define as a separate term:

Content-Type:
: A Media-Type-Name, optionally associated with parameters (separated from
  the media type name and from each other by a semicolon).

Removing the legacy HTAB characters now shunned in polite conversation,
as well as some other cobwebs, we define the conventional textual
representation of a Content-Type with the ABNF in {{ABNF-Content-Type}}:

~~~ abnf
Content-Type   = Media-Type-Name *( *SP ";" *SP parameter )
parameter      = token "=" ( token / quoted-string )

token          = 1*tchar
tchar          = "!" / "#" / "$" / "%" / "&" / "'" / "*"
               / "+" / "-" / "." / "^" / "_" / "`" / "|" / "~"
               / DIGIT / ALPHA
quoted-string  = %x22 *qdtext %x22
qdtext         = SP / %x21 / %x23-5B / %x5D-7E

~~~
{: #ABNF-Content-Type title="Definition of Content-Type"}

Note that there is a slight inconsistency between the "token" used
here and the "reg-name"/"restricted-name" used above; since media type
parameters probably will be defined within the guard rails set by
{{-http}}, we need to use HTTP's more comprehensive definition here.

# Content-Coding

{{Section 3.5 of RFC2616}} also introduced the term Content-Coding, a
registered name for an encoding transformation that has been or can be
applied to a representation:

~~~ abnf
content-coding   = token
~~~
{: #ABNF-content-coding title="Definition of content-coding as in RFC 2616"}

Confusingly, in HTTP the Content-Coding is then given in a header
field called "Content-Encoding"; we **never** use this term (except when
we are in error).  Instead we define:

Content-Coding:
: a registered name for an encoding transformation that has been or
  can be applied to a representation.

Content-Codings are registered in the HTTP Content Coding Registry, a
subregistry of {{-http-parameters}}.  We often use the "identity"
Content-Coding, which is the identity transformation, and often fail
to identify that Content-Coding by name, instead calling it "no
Content-Coding".

# Content-Format

CoAP, in {{Section 1 of RFC7252}}, defines a Content-Format as the
combination of a Content-Type and a Content-Coding, identified by a
numeric identifier defined in the "CoAP Content-Formats" registry (a
subregistry of {{-core-parameters}}), but in more confusing words (it
did not have the benefit of the present specifications).

Content-Format:
: the combination of a Content-Type and a Content-Coding, identified
  by a numeric identifier defined by the "CoAP Content-Formats"
  subregistry of {{-core-parameters}}.

Note that there has not been a conventional string representation of
just the combination of a Content-Type and a Content-Coding;
Content-Formats so far always are identified by their registered
Content-Format numbers.  However, there are applications where that is
useful {{-senml-ct}}, so we define:

~~~ abnf
Content-Format = "0" / (POS-DIGIT *DIGIT)
Content-Format-String   = Content-Type ["@" content-coding]
~~~
{: #ABNF-Content-Format-String title="Definition of Content-Format/-String"}

This allows the use of Content-Format-Strings such as
"application/json@deflate" in place of the less self-describing
content-format "11050", or other combinations that do not have a
content-format number defined yet.

Content-Format-Strings MUST NOT explicitly use the content-coding value of
"identity" (i.e., if an identity content-coding is desired, the entire
optional part including the "@" sign is left out).

Note that a quoted string inside a content-type parameter might
contain an "@" sign, so the parsing of Content-Format-Strings cannot
be done in a too simplistic way.

# Remaining ABNF

This specification uses the ABNF given in {{abnf-boilerplate}}, as
originally defined in {{-abnf}} and {{?RFC8866}}:

~~~ abnf
DIGIT     =  %x30-39           ; 0 – 9
POS-DIGIT =  %x31-39           ; 1 – 9
ALPHA     =  %x41-5A / %x61-7A ; A – Z / a – z
SP        =  %x20
~~~
{: #abnf-boilerplate title="Commonly Used ABNF Definitions"}

# Abbreviations

Media type names are sometimes abbreviated as "mt", and Content-Types
as "ct".  We propose not to use those abbreviations: Where the long
form of the values can be used, the long form "Content-Type" can also
be used to name them.

For historical reasons, both {{?RFC6690}} and {{-coap}} use the
abbreviation "ct" for Content-Format (think first and last character).

For Content-Coding, the abbreviation "cc" can be used.

# Discussion

The ABNF given here is provisional and may need some more cleanup,
such as unifying the various forms of reg-name, token, etc.

(ABNF just shown for illustration is centered, in a blockquote, and tagged with
`<artwork type="abnf;old"...>` in the XML, while the normative ABNF of this memo is
left-aligned and tagged with `<sourcecode type="abnf"...>`.)

The XPath expression `//sourcecode[@type='abnf']/text()` can be used
on the XML form of this specification to extract the ABNF defined here.

We need to discuss case-insensitivity at some point, which is usually
rather insensitive.


# Suggested usage

## COSE

{{Section 3.1 of RFC8152}} defines a common COSE header parameter
(number 3) called "content type" in the description, to indicate the
type of the data in the payload or ciphertext fields.

This header parameter can either be an unsigned integer, indicating a CoRE
Content-Format number, or a text string.
The latter alternative is only defined in general terms.
It points to {{Section 4.2 of RFC6838}} for 'text values following the
syntax of "\<type-name>/\<subtype-name>"...', but also discusses the
use of parameters and subparameters; no ABNF or similar detail
specification is provided.
The text does not discuss the use of Content-Coding in the text string
form, probably because nothing like the present document existed at
the time, creating a weird gap compared with numeric
Content-Format values.
(The text only has trivial changes in its updated version in {{Section
3.1 of I-D.ietf-cose-rfc8152bis-struct-15}}.)

The present specification suggests using the production
`Content-Format-String` as a more formal definition of the text string
that can go into the "content type" (number 3) common header parameter
in COSE.

## SenML

As discussed above, {{Section 3 of I-D.keranen-core-senml-data-ct}}
makes use of the present specification.

## ...

(to be filled in along further use cases)

# IANA Considerations

While this memo talks a lot about IANA registries, it does not
require any action from IANA.

# Security Considerations

Confusion about terminology may, in the worst case, cause security
problems, as can loosely defined syntax elements of a specification.
No other security considerations are known to be raised by the present
specification.

--- back

# Acknowledgements {#acknowledgements}
{: numbered="no"}

{{{Matthias Kovatsch}}} forced the authors to make up their minds about this.
{{{Ari Keränen}}} forced them to write it up, then, and created a convincing
use case of Content-Format-Strings.
{{{John Mattsson}}} alerted us to a mistake.
{{{Alexey Melnikov}}} suggested reviving this draft after a year of dormancy.


<!--  LocalWords:  subtype HTAB subregistry ciphertext subparameters
 -->
