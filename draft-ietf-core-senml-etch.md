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
date: 2018
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
    target: https://github.com/IPSO-Alliance/pub
    title: 'IP for Smart Objects - IPSO Objects'
    author:
    - org: IPSO
    date: 2018


--- abstract

The Sensor Measurement Lists (SenML) media type and data model can be
used to send collections of resources, such as batches of sensor data
or configuration parameters. The CoAP iPATCH, PATCH, and FETCH methods
enable accessing and updating parts of a resource or multiple
resources with one request. This document defines semantics for the
CoAP iPATCH, and FETCH methods for resources represented with the
SenML data model.

--- middle


# Introduction {#intro}

The Sensor Measurement Lists (SenML) media type {{RFC8428} and data
model can be used to transmit collections of resources, such as
batches of sensor data or configuration parameters.

Example of a SenML collection is shown below:

~~~
[
 {"bn":"2001:db8::2/3306/0/", "n":"5850", "vb":true},
 {"n":"5851", "v":42},
 {"n":"5750", "vs":"Ceiling light"}
]
~~~

Here three resources "3306/0/5850", "3306/0/5851", and "3306/0/5750",
of an IPSO dimmable light smart object {{IPSO}} are represented using
a single SenML Pack with three SenML Records. All resources share the
same base name "2001:db8::2/3306/0/", hence full names for resources
are "2001:db8::2/3306/0/5850", etc.

The CoAP {{!RFC7252}} iPATCH and FETCH methods {{!RFC8132}} enable
accessing and updating parts of a resource or multiple resources with
one request.

This document defines semantics for the CoAP iPATCH and FETCH
methods for resources represented with the SenML data model. Same
semantics apply also for the CoAP PATCH method.


# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in
all capitals, as shown here.

Readers should also be familiar with the terms and concepts discussed
in {{RFC8132}} and {{RFC8428}}. Also the following terms are used in
this document:

Fetch Record:
: One set of parameters that is used to match SenML Record(s).

Fetch Pack:
: One or more Fetch Records in an array structure. Presented using the
SenML media type.

Patch Record:
: One set of parameters similar to Fetch Record but also containing
instructions on how to change existing SenML Pack(s).

Patch Pack:
: One or more Patch Records in an array structure.

Target Record:
: A Record in a SenML Pack that is matching the selection criteria of
a Fetch or Patch Record and hence is a target for a Fetch or Patch
operation.

# Using FETCH and iPATCH with SenML

The FETCH and iPATCH methods use the same SenML media type to enable
re-use of existing SenML parsers and generators, in particular on
constrained devices.

NOTE: it is currently under consideration whether new media types
should be registered for FETCH/iPATCH instead of re-using the SenML
media types.

## SenML FETCH

The FETCH method can be used to select and return parts of one or more
SenML Packs. The SenML Records are selected by giving the name(s) of
the resources using the SenML "name" and/or "base name" Fields.

For example, to select resources "5850" and "5851" from the example in
{{intro}}, the following Fetch Pack can be used:

~~~
[
 {"bn":"2001:db8::2/3306/0/", "n":"5850"},
 {"n":"5851"}
]
~~~

The result to a FETCH request with the example above would be:

~~~
[
 {"bn":"2001:db8::2/3306/0/", "n":"5850", "vb":true},
 {"n":"5851", "v":42},
]
~~~

When SenML Records contain also time values, a name may no longer
uniquely identify a single Record. When no time is given in a Fetch
Record, all SenML Records with the given name are matched. When time
is given in the Fetch Record, only a SenML Record (if any) with equal
time value and name is matched.

The resolved form of records (Section 4.6 of {{RFC8428}}) is used when
comparing the names and times of the Target and Fetch Records to
accommodate for differences in use of the base values.

## SenML iPATCH

The iPATCH method can be used to change the values of SenML Records,
to add new Records, and to remove existing Records. The names and
times of the Patch Records are given and matched in same way as for
the Fetch Records, except each Patch Record can match at most one
Target Record. Patch Packs can also include new values and other SenML
Fields for the Records.

When the name in a Patch Record matches with the name in an existing
Record, the time values are compared. If the time values do not exist
or are equal in both Records, the Target Record is replaced with the
contents of the Patch Record.

If a Patch Record contains a name, or combination of a time value and
a name, that do not exist in any existing Record in the Pack, the
given Record, with all the fields it contains, is added to the Pack.

If a Patch Record has a value field with value null, the matched
Record (if any) is removed from the Pack.

For example, the following document could be given as iPATCH payload
to change/set values of two SenML Records for the example in
{{intro}}:

~~~
[
 {"bn":"2001:db8::2/3306/0/", "n":"5850", "vb":false},
 {"n":"5851", "v":10}
]
~~~

If the request is successful, the resulting representation of the
example SenML Pack would be as follows:

~~~
[
 {"bn":"2001:db8::2/3306/0/", "n":"5850", "vb":false},
 {"n":"5851", "v":10},
 {"n":"5750", "vs":"Ceiling light"}
]
~~~


# Security Considerations {#seccons}

The security and privacy considerations of SenML apply also with the
FETCH and iPATCH methods.

In FETCH and iPATCH requests, the client can pass arbitrary names to
the target resource for manipulation. The resource implementer must
take care to only allow access to names that are actually part of (or
accessible through) the target resource.

If the client is not allowed to do a GET or PUT on the full target
resource (and thus all the names accessible through it), access
control rules must be evaluated for each record in the pack.


# Acknowledgements {#acks}

The use of FETCH and iPATCH methods with SenML was first introduced by
the OMA SpecWorks LwM2M v1.1 specification. This document generalizes
the use to any SenML representation. The authors would like to thank
Carsten Bormann, Christian Amsuess, Jaime Jimenez, Klaus Hartke, and
also everyone in the IETF CoRE and OMA SpecWorks DMSE working groups
for their contributions and reviews.


--- back