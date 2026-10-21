---
v: 3

title: "The 2.06 (No Content) Response Code for the Constrained Application Protocol (CoAP)"
abbrev: CoAP No Content
docname: draft-jimenez-core-coap-no-content-latest

category: std
stream: IETF
area: Applications
workgroup: CoRE Working Group
updates: 7641

venue:
  mail: core@ietf.org
  github: jaimejim/draft-jimenez-core-coap-no-content

author:
- name: Jaime Jiménez
  org: Ericsson
  email: jaime@iki.fi
- name: Carsten Bormann
  org: Universität Bremen TZI
  street: Postfach 330440
  city: D-28359 Bremen
  country: Germany
  phone: "+49-421-218-63921"
  email: cabo@tzi.org

normative:
  RFC7252: coap
  RFC7641: observe
  RFC8132: etch
  RFC8075: http-mapping
  RFC9110: http-semantics

informative:
  I-D.ietf-core-coap-pubsub:
    title: "A publish-subscribe architecture for the Constrained Application Protocol (CoAP)"
    author:
    - name: Jaime Jiménez
      org: Ericsson
    - name: Michael Koster
      org: Dogtiger Labs
    - name: Ari Keränen
      org: Ericsson
    date: 2026-07-02
    seriesinfo:
      Internet-Draft: draft-ietf-core-coap-pubsub-21
  I-D.jimenez-core-coap-diag:
    title: "CoAP Diagnostic Message Notation"
    author:
    - name: Jaime Jiménez
      org: Ericsson
    - name: Carsten Bormann
      org: Universität Bremen TZI
    date: 2026-07-04
    seriesinfo:
      Internet-Draft: draft-jimenez-core-coap-diag-00

entity:
  SELF: "[RFC-XXXX]"

--- abstract

This document defines the 2.06 (No Content) response code for the Constrained Application Protocol (CoAP). The code indicates that the target resource exists but has no current representation to transfer. It allows a client to retrieve or observe a resource before that resource has any content, without the server signaling an error. This document updates RFC 7641 to allow 2.06 (No Content) notifications, and specifies how cross-proxies map the code to and from the HTTP 204 (No Content) status code.

--- middle

# Introduction {#introduction}

CoAP {{-coap}} defines a set of success response codes in {{Section 5.9.1 of -coap}}. A GET request that succeeds returns 2.05 (Content) with a representation of the target resource. A GET request on a resource that does not exist returns 4.04 (Not Found). CoAP has no code for the case in between: the resource exists, but no representation of it is available yet, or the last representation has been removed or has expired.

This case occurs in several CoRE applications. In the publish-subscribe architecture of {{I-D.ietf-core-coap-pubsub}}, a topic-data resource has no content between topic creation and the first publication. Because a GET or Observe registration on such a resource cannot succeed, that specification introduces a "half created" topic state, answers requests in that state with 4.04 (Not Found), and provides an "initialize" property so that a creator can install placeholder data. A client then has to infer the topic state from which operations fail. A sensor whose last reading has expired, or a resource whose value has been cleared, have the same gap.

Servers fill the gap today in one of three ways, each with a drawback. A 4.04 (Not Found) response terminates an Observe registration ({{Section 4.2 of -observe}}), so a subscriber cannot register interest until data exists and has to poll instead. A 2.05 (Content) response with an empty payload is ambiguous: a zero-length payload is a valid representation in some Content-Formats (for instance, an empty text/plain string), so the client cannot tell "the value is empty" from "there is no value". A 5.03 (Service Unavailable) response signals a server-side failure when no failure has occurred.

HTTP addresses this case with 204 (No Content) ({{Section 15.3.5 of -http-semantics}}): the server has fulfilled the request and there is no content to send. The HTTP-to-CoAP mapping in {{-http-mapping}} maps HTTP 204 only to 2.02 (Deleted) and 2.04 (Changed), which are not used in response to GET.

This document defines 2.06 (No Content) for CoAP, specifies its use with GET, FETCH, and Observe, and gives the mapping to and from HTTP 204 (No Content). It does not change the semantics of any existing response code, and it does not define how a resource comes to have no representation; that is up to the application.

# Terminology {#terminology}

{::boilerplate bcp14-tagged-bcp14}

This document uses the terms "resource", "representation", "origin server", "proxy", and "cross-proxy" as defined in {{-coap}}, and "observer", "registration", and "notification" as defined in {{-observe}}.

# The 2.06 (No Content) Response Code {#code}

## Semantics {#semantics}

The 2.06 (No Content) response code indicates that the server has fulfilled the request, that the target resource exists, and that the resource has no current representation to transfer in the response.

A server MUST NOT include a payload in a 2.06 (No Content) response. A server MUST NOT include a Content-Format Option or an ETag Option in a 2.06 (No Content) response, since there is no representation to describe or tag.

A client that receives 2.06 (No Content) learns that the resource exists. If the client holds a representation of the resource from an earlier response, it MUST NOT treat that representation as the current state of the resource.

## Applicable Methods {#methods}

A server uses 2.06 (No Content) in response to GET and FETCH {{-etch}} requests. For FETCH, the code indicates that the target resource exists and the selected part of it has no representation.

A server MUST NOT use 2.06 (No Content) in response to PUT, POST, PATCH, iPATCH, or DELETE requests. Those methods already have success codes that carry no payload: 2.04 (Changed) and 2.02 (Deleted).

## Distinction from 2.05 (Content) and 4.04 (Not Found) {#distinction}

A 2.05 (Content) response with a zero-length payload carries a representation of length zero in the indicated Content-Format. A 2.06 (No Content) response carries no representation at all. A server that has an empty representation of a resource, such as the empty CBOR array or an empty string, returns it with 2.05 (Content). A server that has no representation returns 2.06 (No Content).

A 4.04 (Not Found) response indicates that the server has no resource at the target URI. A server MUST NOT return 2.06 (No Content) for a URI at which no resource exists.

## Caching {#caching}

A 2.06 (No Content) response is cacheable. Caches use the Max-Age Option to determine freshness ({{Section 5.6.1 of -coap}}). A server SHOULD set Max-Age to the time after which it expects the resource may have a representation, if known. A cached 2.06 (No Content) response satisfies subsequent GET requests with the same cache key while it is fresh.

A client MAY include ETag Options in a GET request on a resource that currently has no representation. Since none of the entity tags can match, the server responds with 2.06 (No Content).

## Clients That Do Not Recognize the Code {#unknown}

{{Section 5.9 of -coap}} defines no generic success code. A client that does not recognize 2.06 (No Content) can only determine that the request was successful. Such a client might process the empty payload as a representation, which is the ambiguity this code is meant to remove.

For this reason, a server SHOULD NOT send 2.06 (No Content) unless it knows the client supports it, either through a specification that requires support (such as an application protocol built on this document) or through configuration. Otherwise, the server continues to use the response it would have used without this document.

# Use with Observe {#observe}

This section updates {{Section 4.2 of -observe}}.

## Registration {#observe-registration}

A client registers interest in a resource that has no current representation by sending a GET request with an Observe Option set to 0. A server that supports this document and is willing to add the client to the list of observers responds with 2.06 (No Content) and includes an Observe Option. As with any 2.xx response that carries an Observe Option ({{Section 3.1 of -observe}}), this confirms the registration.

## Notifications {#observe-notifications}

{{Section 4.2 of -observe}} states that a notification SHOULD have a 2.05 (Content) or 2.03 (Valid) response code. This document adds 2.06 (No Content) to that list.

When the resource of an observed registration gains a representation, the server sends a 2.05 (Content) notification as usual. When the resource loses its representation, for instance because the data was removed or has expired, the server sends a 2.06 (No Content) notification that includes an Observe Option. The client remains on the list of observers. On receipt, the client discards the previously notified representation, per {{semantics}}.

A 2.06 (No Content) notification follows the ordering rules of {{Section 3.4 of -observe}} and the freshness rules of {{Section 3.2 of -observe}}, using its Observe sequence number and Max-Age Option.

## Content-Format Consistency {#observe-ct}

{{Section 4.2 of -observe}} requires the Content-Format of a 2.xx notification to be the same as the one used in the initial response. This requirement does not apply to 2.06 (No Content) notifications, which carry no Content-Format Option.

If the initial response to a registration is 2.06 (No Content), the Content-Format of the first 2.05 (Content) notification determines the Content-Format for all subsequent 2.05 (Content) notifications of that registration. A client that needs a specific Content-Format SHOULD include an Accept Option in the registration request, so that the server can reject the registration with 4.06 (Not Acceptable) if it cannot produce that format.

# Proxying and HTTP Mapping {#proxy}

2.06 (No Content) is an end-to-end response code. A CoAP-to-CoAP proxy forwards it unchanged and caches it per {{caching}}.

An HTTP-to-CoAP cross-proxy following {{-http-mapping}} maps a CoAP 2.06 (No Content) response to an HTTP 204 (No Content) response. This adds a row to Table 2 in {{Section 7 of -http-mapping}}. The proxy translates Max-Age into HTTP freshness information as specified in {{-http-mapping}}.

A CoAP-to-HTTP cross-proxy ({{Section 10.1 of -coap}}) maps an HTTP 204 (No Content) response to a GET request into a CoAP 2.06 (No Content) response. For other methods, the existing mappings to 2.04 (Changed) and 2.02 (Deleted) apply.

# Example: Publish-Subscribe Topics {#example}

This section is informative. Messages are written in the CoAP Diagnostic Message Notation {{I-D.jimenez-core-coap-diag}}. The section shows how a publish-subscribe broker following {{I-D.ietf-core-coap-pubsub}} could use 2.06 (No Content), assuming the broker creates the topic-data resource together with the topic.

A subscriber reads the topic-data resource before any data has been published, then registers for notifications. Both requests succeed:

~~~~ coap
GET /ps/data/1bd0d6d

2.06 No Content
Max-Age: 600
~~~~
{: #fig-read title="Reading a Topic with No Data"}

~~~~ coap
GET /ps/data/1bd0d6d
Observe: 0
Accept: 110 (application/senml+json)

2.06 No Content
Observe: 1
Max-Age: 600
~~~~
{: #fig-subscribe title="Subscribing to a Topic with No Data"}

A publisher then publishes the first value. Since the resource already exists, the broker returns 2.04 (Changed), and notifies the subscriber:

~~~~ coap
PUT /ps/data/1bd0d6d
Content-Format: 110 (application/senml+json)

[{"bn":"urn:dev:os:32473-123456","n":"temperature","u":"Cel","v":23}]

2.04 Changed
~~~~
{: #fig-publish title="First Publication"}

~~~~ coap
2.05 Content
Observe: 2
Content-Format: 110 (application/senml+json)
Max-Age: 15

[{"bn":"urn:dev:os:32473-123456","n":"temperature","u":"Cel","v":23}]
~~~~
{: #fig-notify title="Notification of the First Value"}

If the data later expires, the broker sends a 2.06 (No Content) notification. The subscriber stays registered and stops using the value of 23 Cel:

~~~~ coap
2.06 No Content
Observe: 3
Max-Age: 600
~~~~
{: #fig-expire title="Notification That the Data Has Expired"}

With this code, the broker needs neither a separate "half created" state nor 4.04 (Not Found) responses on an existing topic, and subscribers can register at any time after the topic is created.

# Security Considerations {#security}

The security considerations of {{-coap}} and {{-observe}} apply.

A 2.06 (No Content) response tells the client that a resource exists at the target URI, which a 4.04 (Not Found) response would not. A server that does not want to disclose the existence of a resource to an unauthorized client MUST respond with the same code it uses for nonexistent resources, as it would for any other request from that client.

A 2.06 (No Content) response carries no payload and is smaller than the request that triggered it, so it does not add an amplification vector. Observe registrations accepted on resources with no representation keep observer state on the server for longer than registrations that would otherwise fail with 4.04 (Not Found). Servers apply the same limits on the number of observers that they apply to any other resource ({{Section 7 of -observe}}).

# IANA Considerations {#iana}

## CoAP Response Codes {#iana-codes}

IANA is asked to add the following entry to the "CoAP Response Codes" registry within the "Constrained RESTful Environments (CoRE) Parameters" registry group, defined in {{Section 12.1.2 of -coap}}:

| Code | Description | Reference |
|------|-------------|-----------|
| 2.06 | No Content  | {{&SELF}} |
{: #tab-iana title="New CoAP Response Code"}

The code 2.06 is a suggested value; it is unassigned in the registry, and the registration policy is IETF Review or IESG Approval.

Note to RFC Editor: replace {{&SELF}} with this document's number.

--- back

# Design Alternatives {#alternatives}

This appendix records the alternatives considered. It is to be removed before publication.

2.05 (Content) with an empty payload:
: Requires no new code, but cannot distinguish an empty representation from an absent one, and constrains the Content-Format of an Observe registration before any data exists.

4.04 (Not Found):
: Conflates "no resource" with "no content", and terminates Observe registrations, so subscribers have to poll until data appears.

5.03 (Service Unavailable) with Max-Age:
: Signals a temporary server failure; caches and clients treat it as an error, and it also terminates Observe registrations.

Placeholder representation:
: An application-defined "empty" value, as with the "initialize" property of {{I-D.ietf-core-coap-pubsub}}, works for formats that have an empty value, but each application defines its own and generic clients cannot recognize it.

2.04 (Changed) in response to GET:
: Used for this purpose in early revisions of {{I-D.ietf-core-coap-pubsub}}. 2.04 (Changed) is defined for the result of PUT and POST, and reusing it for GET overloads its meaning.

# Acknowledgments {#acknowledgments}
{:numbered="false"}

Mike Bishop suggested a No Content success code during the AD review of {{I-D.ietf-core-coap-pubsub}}. Michael Koster proposed a dedicated 2.06 code in issue 4 of the {{I-D.ietf-core-coap-pubsub}} repository, including the requirement that a client stop using a stale value.
