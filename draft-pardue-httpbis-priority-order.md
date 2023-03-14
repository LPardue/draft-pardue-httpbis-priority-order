---
title: "HTTP priority order extension"
abbrev: "HTTP priority order"
category: std

docname: draft-pardue-httpbis-priority-order-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "HTTP"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "HTTP"
  type: "Working Group"
  mail: "ietf-http-wg@w3.org"
  arch: "https://lists.w3.org/Archives/Public/ietf-http-wg/"
  github: "LPardue/draft-pardue-httpbis-priority-order"
  latest: "https://LPardue.github.io/draft-pardue-httpbis-priority-order/draft-pardue-httpbis-priority-order.html"

author:
 -
    fullname: Lucas Pardue
    organization: Cloudflare
    email: lucaspardue.24.7@gmail.com

normative:
  RFC9113:
    display: HTTP/2

informative:


--- abstract

The send-order parameter for the HTTP extensible prioritization scheme allows
explicit ordering indication independent of request order. This can be used to
as an additional input signal to scheduling decisions, to support alternative
sending behaviors.


--- middle

# Introduction

The extensible prioritization scheme for HTTP provides guidance for servers
using priority signals to schedule the sending of stream data; {{Section 10 of
!RFC9218}}. It recommends that when there are multiple items with the same
urgency servers choose how to allocate bandwidth by considering the stream
ID and incremental parameter, and possibly other signals.

In web use cases, an HTML document can include other resources and user agents
tend to issue requests in an ordered sequence that matches the actions of HTML
or HTTP receiver processing. Requests are made using ascending stream IDs where
lower-numbered streams typically relate to objects that were discovered earlier
than higher-numbered streams. Because stream IDs represent an ordering that
closely matches the receiver processing needs, the scheduling guidance in
{{RFC9218}} can be effectively used to send data that is optimal for this
consumption pattern. As a simple example, a client that issues non-incremental
requests of the same urgency on streams 0, 4, 8 and 12 would expect a server ,
that is following {{Section 10 of RFC9218}}, to serve the response in the order
0, 4, 8 and 12.

A strictly singular serving order is unlikely to meet the needs of HTTP-based
applications. The urgency and incremental parameters augment stream ID, urgency
allows more important resources (from a processing perspective) to be sent ahead
of less important resources, while incremental streams share bandwidth among
resources that can be effectively loaded in parallel.

There are other use cases where the user agent may not be able to influence the
order of request emission in the most optimal sequence, or may discover an
important resource late on and would like the response to "jump the queue" of
other streams within a single urgency level. The send-order parameter defined by
this document can be used in these situations. While it might be possible to
achieve a similar outcome by careful use of the urgency parameter, such as
reserving an urgency level, the limited number of levels can make this approach
difficult in practice. Alternatively, rebalancing or reprioritization of
concurrent stream urgencies might also achieve these goals but encounters issues
related to churn and responsiveness due to the latency of reprioritization
signals.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the following terminology from {{Section 3 of
!STRUCTURED-FIELDS=RFC8941}} to specify syntax and parsing: Boolean, Dictionary,
and Integer.

Example HTTP requests and responses use the HTTP/2-style formatting from
{{RFC9113}}.

This document uses the variable-length integer encoding from
{{!QUIC=RFC9000}}.

# The send-order parameter

The send-order (`bikeshed-order-name`) parameter value is Integer (see {{Section
3.3.1 of STRUCTURED-FIELDS}}), between 0 and 2<sup>32</sup>. The range is
ascending, higher values have precedence over lower values.

Endpoints use send-order to communicate an ordering precedence that is
independent of the request order. There is no default value, omission of the
parameter means that there is no additional ordering preference.

The send-order parameter satisfies the urgency and incremental parameter
compatibility requirements in {{Section 4.3 of RFC9218}}.

The following example shows an HTTP/3 client issuing multiple requests on the
same connection, without the send-order parameter, and responses sent by a
server following the guidance in {{Section 10 of !RFC9218}}.

~~~
Client                              Server

HEADERS (stream 0)
  :method = GET
  :scheme = https
  :authority = example.net
  :path = /image1.jpg
  priority = u=1

HEADERS (stream 4)
  :method = GET
  :scheme = https
  :authority = example.net
  :path = /image2.jpg
  priority = u=1

HEADERS (stream 8)
  :method = GET
  :scheme = https
  :authority = example.net
  :path = /image3.jpg
  priority = u=1

                                  HEADERS (stream 0)
                                    :status = 200
                                    content-type = image/jpeg

                                  DATA (stream 0)
                                    {binary data}

                                  HEADERS (stream 4)
                                    :status = 200
                                    content-type = image/jpeg

                                  DATA (stream 4)
                                    {binary data}

                                  HEADERS (stream 8)
                                    :status = 200
                                    content-type = image/jpeg

                                  DATA (stream 8)
                                    {binary data}
~~~

The following example shows an HTTP/3 client issuing multiple requests on the
same connection, without the send-order parameter, and responses sent by a
server following the guidance in {{server-scheduling}}.

~~~
Client                              Server

HEADERS (stream 0)
  :method = GET
  :scheme = https
  :authority = example.net
  :path = /image1.jpg
  priority = u=1

HEADERS (stream 4)
  :method = GET
  :scheme = https
  :authority = example.net
  :path = /image2.jpg
  priority = u=1,bikeshed-order-name=25

HEADERS (stream 8)
  :method = GET
  :scheme = https
  :authority = example.net
  :path = /image3.jpg
  priority = u=1,bikeshed-order-name=15

                                  HEADERS (stream 4)
                                    :status = 200
                                    content-type = image/jpeg

                                  DATA (stream 4)
                                    {binary data}

                                  HEADERS (stream 8)
                                    :status = 200
                                    content-type = image/jpeg

                                  DATA (stream 8)
                                    {binary data}

                                  HEADERS (stream 0)
                                    :status = 200
                                    content-type = image/jpeg

                                  DATA (stream 0)
                                    {binary data}
~~~

# Server Scheduling

This document updates the scheduling guidance in {{Section 10 of RFC9218}}.

Non-incremental responses of the same urgency SHOULD be served by prioritizing
bandwidth allocation in ascending order of send-order. If there are no responses
with a send-order, allocation SHOULD use ascending order of stream ID.

Incremental responses of the same urgency are not affected by send-order.

# Security Considerations

The considerations in {{RFC9218}} apply. There are not believed to be any
additional considerations.


# IANA Considerations

TODO


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
