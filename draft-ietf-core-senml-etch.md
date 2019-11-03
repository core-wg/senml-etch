---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-senml-etch-latest
keyword: Internet-Draft
cat: std
pi:
  toc: 'yes'
  tocompact: 'yes'
  tocdepth: '3'
  tocindent: 'yes'
  symrefs: 'yes'
  sortrefs: 'yes'
  comments: 'yes'
  inline: 'yes'
  strict: 'no'
  compact: 'no'
  subcompact: 'no'
title: FETCH & PATCH with Sensor Measurement Lists (SenML)
abbrev: FETCH & PATCH with SenML
date: 2019
author:
- ins: A. Keranen
  name: Ari Keranen
  org: Ericsson
  email: ari.keranen@ericsson.com
- ins: M. Mohajer
  name: Mojan Mohajer
  org: u-blox UK
  email: Mojan.Mohajer@u-blox.com
normative:
  RFC8428:
informative:
  IPSO:
    target: http://www.openmobilealliance.org/tech/profiles/lwm2m/3311.xml
    title: 'IPSO Light Control Smart Object'
    author:
    - org: IPSO
    date: 2018


--- abstract

The Sensor Measurement Lists (SenML) media type and data model can be
used to send collections of resources, such as batches of sensor data or
configuration parameters. The CoAP FETCH, PATCH, and iPATCH methods
enable accessing and updating parts of a resource or multiple resources
with one request. This document defines new media types for the CoAP
FETCH, PATCH, and iPATCH methods for resources represented with the SenML
data model.

--- middle


# Introduction {#intro}

The Sensor Measurement Lists (SenML) media type {{RFC8428}} and data
model can be used to transmit collections of resources, such as
batches of sensor data or configuration parameters.

An example of a SenML collection is shown below:

~~~
[
 {"bn":"2001:db8::2/3311/0/", "n":"5850", "vb":true},
 {"n":"5851", "v":42},
 {"n":"5750", "vs":"Ceiling light"}
]
~~~

Here three resources "3311/0/5850", "3311/0/5851", and "3311/0/5750",
of an IPSO dimmable light smart object {{IPSO}} are represented using
a single SenML Pack with three SenML Records. All resources share the
same base name "2001:db8::2/3311/0/", hence full names for resources
are "2001:db8::2/3311/0/5850", etc.

The CoAP {{!RFC7252}} FETCH, PATCH, and iPATCH methods {{!RFC8132}}
enable accessing and updating parts of a resource or multiple resources
with one request.

This document defines two new media types, one using the JavaScript
Object Notation (JSON) {{!RFC8259}} and one using the Concise Binary
Object Representation (CBOR) {{!RFC7049}}, which can be used with the CoAP
FETCH, PATCH, and iPATCH methods for resources represented with the SenML
data model. The rest of the document uses term "(i)PATCH" when referring
to both methods as the semantics of the new media types are the same for
the CoAP PATCH and iPATCH methods.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in
all capitals, as shown here.

Readers should also be familiar with the terms and concepts discussed
in {{RFC8132}} and {{RFC8428}}. The following additional terms are used in
this document:

Fetch Record:
: One set of parameters that is used to match SenML Record(s).

Fetch Pack:
: One or more Fetch Records in an array structure.

Patch Record:
: One set of parameters similar to Fetch Record but also containing
instructions on how to change existing SenML Pack(s).

Patch Pack:
: One or more Patch Records in an array structure.

Target Record:
: A Record in a SenML Pack that matches the selection criteria of
a Fetch or Patch Record and hence is a target for a Fetch or Patch
operation.

Target Pack:
: A SenML Pack that is a target for a Fetch or Patch operation.

(i)PATCH:
: A term that refers to both CoAP "PATCH" and "iPATCH" methods when
there is no difference in this specification in which one is used.

# Using FETCH and (i)PATCH with SenML

The FETCH/(i)PATCH media types for SenML are modeled as extensions to the
SenML media type to enable re-use of existing SenML parsers and
generators, in particular on constrained devices. Unless mentioned
otherwise, FETCH and PATCH Packs are constructed with the same rules and
constraints as SenML Packs.

The key differences to the SenML media type are allowing the use of a
"null" value for removing records with the (i)PATCH method and lack of
value fields in Fetch Records. Also the Fetch and Patch Records do not
have default time or base version when the fields are omitted.

## SenML FETCH

The FETCH method can be used to select and return a subset of records, in
sequence, of one or more SenML Packs. The SenML Records are selected by
giving a set of names that, when resolved, match resolved names in a
Target SenML Pack. The names for a Fetch Pack are given using the SenML
"name" and/or "base name" fields. The names are resolved by concatenating
the base name with the name field as defined in {{RFC8428}}.

For example, to select the IPSO resources "5850" and "5851" from the
example in {{intro}}, the following Fetch Pack can be used:

~~~
[
 {"bn":"2001:db8::2/3311/0/", "n":"5850"},
 {"n":"5851"}
]
~~~

The result to a FETCH request with the example above would be:

~~~
[
 {"bn":"2001:db8::2/3311/0/", "n":"5850", "vb":true},
 {"n":"5851", "v":42},
]
~~~

When SenML Records contain also time values, a name may no longer
uniquely identify a single Record. When no time is given in a Fetch
Record, all SenML Records with the given name are matched (i.e., unlike
with SenML Records, lack of time field in a Fetch Record does not imply
time value zero). When time is given in the Fetch Record, only a SenML
Record (if any) with equal resolved time value and name is matched.

For example, if the IPSO resource "5850" would have multiple sensor
readings (SenML Records) with different time values, the following Fetch
Pack can be used to retrieve the Record with time "1.276020091e+09":

~~~
[
 {"bn":"2001:db8::2/3311/0/", "n":"5850", "t":1.276020091e+09}
]
~~~

The resolved form of records (Section 4.6 of {{RFC8428}}) is used when
comparing the names and times of the Target and Fetch Records to
accommodate for differences in use of the base values. In resolved form
the SenML name in the example above becomes "2001:db8::2/3311/0/5850".
Since there is no base time in the Pack, the time in resolved form is
equal to the time in the example.

All other Fetch Record fields than name, base name, time, and base time
MUST be ignored.

## SenML (i)PATCH

The (i)PATCH method can be used to change the fields of SenML Records,
to add new Records, and to remove existing Records. The names and
times of the Patch Records are given and matched in same way as for
the Fetch Records, except each Patch Record can match at most one
Target Record. Patch Packs can also include new values and other SenML
fields for the Records. Application of Patch Packs is idempotent; hence
PATCH and iPATCH methods for SenML Packs are equivalent.

When the name in a Patch Record matches with the name in an existing
Record, the resolved time values are compared. If the time values either
do not exist in both Records or are equal, the Target Record is replaced
with the contents of the Patch Record. All Patch Records MUST contain at
least a SenML Value or Sum field. A Patch Pack with invalid Records MUST
be rejected.

If a Patch Record contains a name, or combination of a time value and
a name, that do not exist in any existing Record in the Pack, the
given Record, with all the fields it contains, is added to the Pack.

If a Patch Record has a value ("v") field with value null, it MUST NOT be
added but the matched Record (if any) is removed from the Target Pack.

For example, the following document could be given as an (i)PATCH payload
to change/set values of two SenML Records for the example in
{{intro}}:

~~~
[
 {"bn":"2001:db8::2/3311/0/", "n":"5850", "vb":false},
 {"n":"5851", "v":10}
]
~~~

If the request is successful, the resulting representation of the
example SenML Pack would be as follows:

~~~
[
 {"bn":"2001:db8::2/3311/0/", "n":"5850", "vb":false},
 {"n":"5851", "v":10},
 {"n":"5750", "vs":"Ceiling light"}
]
~~~

As another example, the following document could be given as an (i)PATCH
payload to remove the two SenML Records:

~~~
[
 {"bn":"2001:db8::2/3311/0/", "n":"5850", "v":null},
 {"n":"5851", "v":null}
]
~~~

# Fragment Identification

Fragment identification for Records of Fetch and Patch Packs uses the
same mechanism as SenML JSON/CBOR fragment identification (see Section 9
of {{RFC8428}}), i.e., "rec" scheme followed by a comma-separated list of
Record positions or range(s) of Records. For example, to select the 3rd
and 5th Record of a Fetch or Patch Pack, a fragment identifier "rec=3,5"
can be used in the URI of the Fetch or Patch Pack resource.

# Security Considerations {#seccons}

The security and privacy considerations of SenML apply also with the
FETCH and (i)PATCH methods.

In FETCH and (i)PATCH requests, the client can pass arbitrary names to
the target resource for manipulation. The resource implementer must
take care to only allow access to names that are actually part of (or
accessible through) the target resource.

If the client is not allowed to do a GET or PUT on the full target
resource (and thus all the names accessible through it), access
control rules must be evaluated for each record in the pack.

# IANA Considerations {#iana}

This document registers two new media types and CoAP Content-Format IDs
for both media types.

Note to RFC Editor: Please replace all occurrences of "RFC-AAAA" with
the RFC number of this document.

## CoAP Content-Format Registration

IANA is requested to assign CoAP Content-Format IDs for the SenML PATCH
and FETCH media types in the "CoAP Content-Formats" sub-registry, within
the "CoRE Parameters" registry {{RFC7252}}. The assigned IDs are shown in
{{tbl-coap-content-formats}}.

| Media type                   | Encoding | ID      |
| application/senml-etch+json  | -        | TBD-320 |
| application/senml-etch+cbor  | -        | TBD-322 |
{: #tbl-coap-content-formats cols="l l" title="CoAP Content-Format IDs"}


## senml-etch+json Media Type

Type name: application

Subtype name: senml-etch+json

Required parameters: N/A

Optional parameters: N/A

Encoding considerations: binary

Security considerations: See {{seccons}} of RFC-AAAA.

Interoperability considerations: Applications MUST ignore any key
value pairs that they do not understand unless the key ends with the
'_' character in which case an error MUST be generated. This allows
backwards compatible extensions to this specification.

Published specification: RFC-AAAA

Applications that use this media type: Applications that use the
SenML media type for resource representation.

Fragment identifier considerations: Fragment identification for
application/senml-etch+json is supported by using fragment identifiers as
specified by RFC AAAA.

Additional information:

Deprecated alias names for this type: N/A

Magic number(s): N/A

File extension(s): senml-etchj

Windows Clipboard Name: "SenML FETCH/PATCH format"

Macintosh file type code(s): N/A

Macintosh Universal Type Identifier code: org.ietf.senml-etch-json
conforms to public.text

Person & email address to contact for further information:
Ari Keranen ari.keranen@ericsson.com

Intended usage: COMMON

Restrictions on usage: N/A

Author: Ari Keranen ari.keranen@ericsson.com

Change controller: IESG

## senml-etch+cbor Media Type

Type name: application

Subtype name: senml-etch+cbor

Required parameters: N/A

Optional parameters: N/A

Encoding considerations: binary

Security considerations: See {{seccons}} of RFC-AAAA.

Interoperability considerations: Applications MUST ignore any key
value pairs that they do not understand unless the key ends with the
'_' character in which case an error MUST be generated. This allows
backwards compatible extensions to this specification.

Published specification: RFC-AAAA

Applications that use this media type: Applications that use the
SenML media type for resource representation.

Fragment identifier considerations: Fragment identification for
application/senml-etch+cbor is supported by using fragment identifiers as
specified by RFC AAAA.

Additional information:

Deprecated alias names for this type: N/A

Magic number(s): N/A

File extension(s): senml-etchc

Macintosh file type code(s): N/A

Macintosh Universal Type Identifier code: org.ietf.senml-etch-cbor
conforms to public.data

Person & email address to contact for further information:
Ari Keranen ari.keranen@ericsson.com

Intended usage: COMMON

Restrictions on usage: N/A

Author: Ari Keranen ari.keranen@ericsson.com

Change controller: IESG

# Acknowledgements {#acks}

The use of FETCH and (i)PATCH methods with SenML was first introduced by
the OMA SpecWorks LwM2M v1.1 specification. This document generalizes the
use to any SenML representation. The authors would like to thank Carsten
Bormann, Christian Amsuess, Jaime Jimenez, Klaus Hartke, Michael
Richardson, and other participants from the IETF CoRE and OMA SpecWorks
DMSE working groups who have contributed ideas and reviews.


--- back
