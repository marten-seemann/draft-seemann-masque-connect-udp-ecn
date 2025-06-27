---
title: "Using ECN when Proxying UDP in HTTP"
abbrev: "CONNECT-UDP ECN"
category: std

docname: draft-seemann-masque-connect-udp-ecn-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Multiplexed Application Substrate over QUIC Encryption"
keyword:
 - UDP
 - ECN
 - Proxying
venue:
  group: "Multiplexed Application Substrate over QUIC Encryption"
  type: "Working Group"
  mail: "masque@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/masque/"
  github: "marten-seemann/draft-seemann-masque-connect-udp-ecn"
  latest: "https://marten-seemann.github.io/draft-seemann-masque-connect-udp-ecn/draft-seemann-masque-connect-udp-ecn.html"

author:
 -
    fullname: Marten Seemann
    organization: Smallstep
    email: martenseemann@gmail.com

normative:

informative:


--- abstract

This document describes how to proxy the ECN bits when proxying UDP in HTTP.

--- middle

# Introduction

Explicit Congestion Notification marking {{!RFC3168}} uses two bits in the IP
header to signal congestion from a network to endpoints.

{{!RFC9298}} describes how UDP datagrams can be proxied in HTTP. This allows the
proxying of the payload of UDP datagrams, however, it is not possible to proxy
the ECN bits. This document defines an extension to {{!RFC9298}} that allows the
proxying of the ECN bits without imposing any encoding overhead.

When establishing a tunnel, the client registers four context identifiers, one
for each ECN marking. These context identifiers are then used to:

1. For UDP datagrams sent from the client to the proxy: To request the proxy to
   set the ECN marking on the UDP datagram sent to the target.
2. For UDP datagrams sent from the proxy to the client: To inform the client
   about the ECN marking of the UDP datagram received from the target.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Proxying ECN

The proxy fulfills a dual role: First, it sends UDP datagrams received from the
client over HTTP to the target, and sends UDP datagrams received from the target
over HTTP to the client. Second, it also acts as a router that can experience
congestion in both directions.

## Sending UDP datagrams from the client to the proxy

When sending UDP datagrams over the tunnel, the client uses the context
identifier as negotiated during establishment of the tunnel (see
{{negotiation}}). Under normal circumstances, the proxy MUST set the ECN marking
on the UDP datagram sent to the target based on the context identifier. However,
if the proxy is experiencing congestion on the link to the target, it SHOULD
apply ECN markings according to {{!RFC3168}} and {{!RFC8331}}.

## Sending UDP datagrams from the proxy to the client

When receiving UDP datagrams from the target, the proxy uses the context
identifier negotiated during establishment of the tunnel to indicate the ECN
marking the UDP datagram was received with. Similarly, if the HTTP connection to
the client is experiencing congestion, the proxy SHOULD apply ECN markings.

# Negotiating Extension and Registration of Context Identifiers {#negotiation}

To support ECN mode, both clients and proxies need to include the "Proxy-ECN"
header field. This indicates support for ECN mode and registers the context
IDs.

    proxy-ecn = ?1; not-ect = 2; ect1 = 100; ect0 = 1234; ce = 42

"Proxy-ECN" is an Item Structured Header {{!RFC8941}}. Its value MUST be a
boolean.

If the client wants to enable proxying of ECN markings, it sets the value to
"?1". The client MUST add the following four parameters: "not-ect", "ect1",
"ect0", and "ce", each of which is of type sf-integer. The values are used to
register the context IDs for the different ECN markings. The numbers MUST be even
according to the rules for context identifiers in Section 4 of {{!RFC9298}}.

It is RECOMMENDED to use context identifier values that can be encoded using the
same QUIC Variable-Length Integer encoding (see Section 16 of {{!RFC9000}}).

If the proxy wants to enable proxying of ECN markings, it sets the value to
"?1". It MUST NOT add any of the four parameters defined above.

If the proxy wants to disable proxying of ECN markings, it either omits the
"Proxy-ECN" header field or sets the value to "?0". This also refuses the
registration of the context IDs.


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
