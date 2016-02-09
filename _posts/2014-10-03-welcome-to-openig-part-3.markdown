---
layout: post
title:  "Introduction to OpenIG"
subtitle: "Part 3: Concepts"
date:   2014-10-03
categories: openig
tags: [guide, openig, exchange, request, response, handler, filter]
image: /assets/article_images/introduction-to-openig-p3/IMG_5052.jpg
comments: true
---

The previous posts exposed you to OpenIG: use cases and an initial configuration
example. Before going further, I would like to introduce the underlying concepts
that you should understand.

<!-- more -->

# HTTP Exchanges Requests and Responses

OpenIG is a specialized HTTP reverse-proxy: it only deals with HTTP messages.
The purpose of OpenIG is to give a complete control over the messages
flowing through itself (incoming requests and outgoing responses).

In order to ease handling of the whole message processing (request and response),
OpenIG uses the concept of [`Exchange`][doc-exchange]: it's a simple way to link a `Request` and
a `Response` together. It also contains a `Session` instance that can be used to
store session-scoped properties. The `Exchange` itself is a `Map` so it's a natural
container for request-scoped properties.

When a request comes through OpenIG, an `Exchange` object is created and populated
with an initial `Request` and a `Session` instance.

A [`Request`][doc-request] is the OpenIG model for an incoming HTTP message, it captures both
the message's entity and all of its headers. It also has dedicated accessors for
message's target URI, cookies and form/query parameters.

A [`Response`][doc-response] is the complementary model object for outgoing HTTP messages. In
addition to the entity and headers accessors, `Response` provides setter for
the HTTP status code (`20x` -> ok, `30x` -> redirect, `50x` -> server error, ...).

# Exchange processing

Now we have a model of the message's content, what can we do to apply
transformations to it?

OpenIG offers a simple, but powerful, API to process exchanges:

 * [`Handler`][doc-handler] that are responsible to produce a `Response` object into the `Exchange`
 * [`Filter`][doc-filter] that can intercept the flowing `Exchange` (incoming and outgoing)

## Handler

What do the doc says about `Handler.handle()` ?

> Called to request the handler respond to the request.
>
> A handler that doesn't hand-off an exchange to another handler downstream
> is responsible for creating the response in the exchange object.

Hmmm, an example would help, right ?

OpenIG offers a rich set of Handlers, the most significative one would be the `ClientHandler`.
This is a highly used component usually ending `Exchange`'s processing that
simply forward the `Request` to the required URI and wraps the returned HTTP
message into the `Exchange`'s `Response`.

In other words, it acts as a **client** to the protected resource (hence the name).

## Filter

Again, what do the doc says about `Filter.filter()` ?

> Filters the request and/or response of an exchange.
>
> Initially, exchange.request contains the request to be filtered. To pass the
> request to the next filter or handler in the chain, the filter calls
> next.handle(exchange). After this call, exchange.response contains the
> response that can be filtered.
>
> This method may elect not to pass the request to the next filter or handler,
> and instead handle the request itself. It can achieve this by merely avoiding
> a call to next.handle(exchange) and creating its own response object the exchange.
> The filter is also at liberty to replace a response with another of its own
> after the call to next.handle(exchange).

This is easier to understand I think, everyone is used to **interceptors** nowadays...

The traditional example of a `Filter` is the `CaptureFilter`: this filter simply
prints the content of the incoming request, then call the *next handler in chain*
and finally prints the outgoing response's content.

`Filters` are contained inside a special `Handler` called a `Chain`. The chain
is responsible to sequentially invoke each of the filter declared in its
configuration, before handing the flow to its terminal `Handler`.

# All together: a Chain example

If you have your OpenIG up and running, please shut it down and replace
its configuration file (`config.json`) with the following content:

{% gist sauthieg/daec0561679c0ad9c24f %}

Compared to previous, this configuration will enhance the response message
with an additional HTTP header named `X-Hello`.

The message flow is depicted in the following diagram:

![Exchange Flow](http://www.plantuml.com:80/plantuml/png/ZP7DJiCm48JFx5EimWKayW9H5GaXQ7hgeGUGog4ctiQMZ1qSz-_jSVsJfi21IpA7-NOziq9omgqnxiCS0Vm7YsLFUZ4ly5R9JhZEqWbkUcQTR6NFjCi6d3D71tOga0su8hjNv70s6sLTNsCDAMLUZLNyIJ2fprGKdedY9_78UO0QOfpi6NYHoddbYQJ-ND8mpLK4ilH4bXuXpJ7aNPTrVc-52zsQJwaxjFIrey41iDR9lK-PFG19bFL8hSZjUcIewV2kdu-jOBgZx4C_FsIhK8JrTvGSzvVXmG0GE_10Z5RXDln7uY6Di1CqMW5I6nvEAVz59oz0hcRAzrQURnqABQuiQIcdtuCkxyWX6EUuUIl3c0sASCc9BMQVZCg9nZPozHDFAcmzAo_7M-vSP-BnXjxeWQP0Sdq3)

A `Filter` can intercept both the `Request` and the `Reponse` flows. In this
case, our `HeaderFilter` is configured to only act on the response flow (because
the outgoing message is a handy way to observe a filter in action from an
outside perspective).

# Wrap up

OpenIG provides a low-level HTTP model API that let you alter HTTP messages
in many ways. The message processing is handled through some kind of pipeline
composed of handlers and filters.

All the processing logic you want to apply to your messages finally depends
on the way you compose your handlers and filters together.

# Next

In the next post we'll see how to debug your configurations.

[doc-exchange]: http://docs.forgerock.org/en/openig/3.0.0/apidocs/org/forgerock/openig/http/Exchange.html
[doc-request]:  http://docs.forgerock.org/en/openig/3.0.0/apidocs/org/forgerock/openig/http/Request.html
[doc-response]: http://docs.forgerock.org/en/openig/3.0.0/apidocs/org/forgerock/openig/http/Response.html
[doc-handler]:  http://docs.forgerock.org/en/openig/3.0.0/apidocs/org/forgerock/openig/handler/Handler.html
[doc-filter]:   http://docs.forgerock.org/en/openig/3.0.0/apidocs/org/forgerock/openig/filter/Filter.html
