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
date: 2021-02-18
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

<aside markdown="1">
~~~ abnf;old
type-name = reg-name
subtype-name = reg-name

reg-name = 1*127reg-name-chars
reg-name-chars = ALPHA / DIGIT / "!" /
                 "#" / "$" / "&" / "." /
                 "+" / "-" / "^" / "_"
~~~
{: #abnf-type-subtype1 gi="artwork" artwork-align="center" title="ABNF for type and
subtype, cited from RFC 4288"}
</aside>

{{-mediatype-reg}} contains the semantically equivalent ABNF in
{{ABNF-type-subtype2}}, which also provides comments that limit the
use of "." and "+".

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

However, the term "media type" is now generally used for a registered
combination of a type-name and a subtype-name.  We further
disambiguate by calling this a *media type name*.
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

Media types can have parameters {{-mediatype-reg}}, some of which are mandatory.
In HTTP and many other protocols, these are then used in a
"Content-Type" header field.
HTTP {{-http}} uses:

<aside markdown="1">
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
{: #http-ct gi="artwork" artwork-align="center" title="Content-Type ABNF from RFC 7231"}
</aside>

We don't follow this inclusive use established by {{?RFC2616}}, parts
of which became {{-http}}, namely to use the term media-type for a
Media-Type-Name with parameters; note that {{RFC2616}} was quite confused
about this by claiming (Section 3.7):

>   Media-type values are registered with the Internet Assigned Number
>   Authority (IANA \[19]).

This clearly reverts to the understanding of Media-Type-Name we use.
We instead define as a separate term:

Content-Type:
: A Media-Type-Name, optionally associated with parameters (separated from
  the media type name and from each other by a semicolon).

Removing the legacy HTAB characters now shunned in polite conversation,
as well as some other cobwebs, we define the conventional textual
representation of a Content-Type as:

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

{{RFC2616}} also introduced the term Content-Coding, a registered name
for an encoding transformation that has been or can be applied to a
representation:

~~~ abnf
content-coding   = token
~~~
{: #ABNF-content-coding title="Definition of content-coding"}

Confusingly, in HTTP the Content-Coding is then given in a header
field called "Content-Encoding"; we NEVER use this term (except when
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

CoAP {{-coap}} defines a Content-Format as the combination of a
Content-Type and a Content-Coding, identified by a numeric identifier
defined by the "CoAP Content-Formats" registry (a subregistry of
{{-core-parameters}}), but in more confusing
words (it did not have the benefit of the present memo).

Content-Format:
: the combination of a Content-Type and a Content-Coding, identified
  by a numeric identifier defined by the "CoAP Content-Formats"
  registry.

Note that there has not been a conventional string representation of
just the combination of a Content-Type and a Content-Coding;
Content-Formats so far always are identified by their registered
Content-Format numbers.  However, there are applications where that is
useful {{-senml-ct}}, so we define:

~~~ abnf
Content-Format = 1*DIGIT
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

# Abbreviations

Media type names are sometimes abbreviated as "mt", and Content-Types
as "ct".  We propose not to use those abbreviations: Where the long
form of the values can be used, the long form "Content-Type" can also
be used to name them.

For historical reasons, both {{?RFC6690}} and {{-coap}} use the
abbreviation "ct" for Content-Format (think first and last character).

For Content-Coding, the abbreviation "cc" can be used.

# Discussion

The ABNF given here is provisional and may need some more cleanup:
We need to unify the various forms of reg-name, token, etc.

(ABNF just shown for illustration is centered, in an aside, and tagged with
type "abnf;old" in the XML, while the normative ABNF of this memo is
left-aligned and tagged with type "abnf".)

We need to discuss case-insensitivity, which is usually rather
insensitive.

# Suggested usage

## COSE

The production `Content-Format-String` is suggested as a more formal
definition of the text string choice for the "content type" generic
header (number 3) in {{Section 3.1 of RFC8152}}.
While the text in this RFC only discusses numeric Content-Format
labels and calls the text version "content type" without defining
exactly what can go in there, defining this more exactly as
content-format-strings fills a weird gap.

## SenML

As discussed above, {{-senml-ct}} makes use of the present specification.

## ...

(to be filled in further)

# IANA Considerations

While this memo talks a lot about IANA registries, it does not
require any action from IANA.

# Security Considerations

Confusion about terminology may, in the worst case, cause security
problems.  No other security considerations are known to be raised by
the present memo.

--- back

# Acknowledgements {#acknowledgements}
{: numbered="no"}

Matthias Kovatsch forced the author to make up his mind about this.
Ari Keranen forced him to write it up, then, and created a convincing
use case of Content-Format-Strings.
John Mattsson alerted us to a mistake.
Alexey Melnikov suggested to revive this draft after a year of dormancy.


<!--  LocalWords:  subtype HTAB subregistry
 -->
