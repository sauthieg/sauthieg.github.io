---
layout: post
title:  "Introduction to OpenIG"
subtitle: "Part 4: Troubleshooting"
date:   2014-10-12
categories: openig
tags: [guide, openig, debug, capture, troubleshooting]
image: /assets/article_images/introduction-to-openig-p4/100_6522.jpg
comments: true
---

As transformations are dictated by the set of filters/handlers in your
configuration, they not always trivial, it's becoming very important quickly
to capture the messages at different phases of the processing.

<!-- more -->

# See the flow

First thing to understand when trying to debug a configuration is "where the
hell are all my messages going ?" :)

This achievable simply by activating `DEBUG` traces in your `LogSink` heap object:

{% gist sauthieg/9622ec69b2db77c99b51 %}

When the `DEBUG` traces are `on`, you should see something like:

{% gist sauthieg/03f4f76c1ba7e5d3598b %}

You'll see a new line each time an Exchanges comes into a Handler/Filter and
each time it's flowing out of the element (you also get a performance measurement).

# Capture the messages (requests/responses)

OpenIG provides a simple, way to see the HTTP message (being a
request or a response), including both headers and (optionaly) the entity (if
that's a textual content): the [`CaptureFilter`][capture-filter].

Here is an output example you can obtain when you install a `CaptureFilter`:

{% gist sauthieg/502b5827774fe1ca877c %}

## Install a CaptureFilter

Being a filter, it has to be installed as part of a Chain:

{% gist sauthieg/b4658a6b111620f99705 %}

It is usually best placed either as the OpenIG entry point (the first element
to be invoked), that helps to see what the User-Agent send and receive (as
it's perceived by OpenIG) or just before a `ClientHandler` (that represents
a sort of endpoint, usually your protected application).

# Capture what you want

`CaptureFilter` is sufficient for simple capturing needs. When what you want
to observe is not contained in the HTTP message, we have to use the OpenIG
swiss-knife: [`ScriptableFilter`][scriptable-filter].

This is a special filter that allows you to execute a [Groovy][groovy] script when
traversed by an Exchange.

Here is a sample script that prints the content of the Exchange's session:

{% gist sauthieg/511c1818b1d84553f31a %}

Copy this script into `~/.openig/scripts/groovy/PrintSessionFilter.groovy`
and configure your heap object:

{% gist sauthieg/4ff822e430ce4c4e1741 %}

# Seeing the messages on wire

Sometimes, all of the previous solutions are not applicable, because you want
to see the on-wire message content (as opposed to modelled by OpenIG).

For this case, the only solution is to start your OpenIG with a couple of
system properties that will activate deep traces of the http client library
we're using: [Apache HTTP Client][http-client].

``` sh
>$ bin/catalina.sh -Dorg.apache.commons.logging.Log=org.apache.commons.logging.impl.SimpleLog \
                   -Dorg.apache.commons.logging.simplelog.log.httpclient.wire=debug \
                   -Dorg.apache.commons.logging.simplelog.log.org.apache.commons.httpclient=debug \
                   run
```

See the [HTTP Client Logging page][hc-logging] for more informations.

# Next

In the next post we'll explain how routes can speed-up your configuration preparation.

[capture-filter]: http://docs.forgerock.org/en/openig/3.0.0/reference/index.html#CaptureFilter
[scriptable-filter]: http://docs.forgerock.org/en/openig/3.0.0/reference/index.html#ScriptableFilter
[groovy]: http://groovy.codehaus.org
[http-client]: http://hc.apache.org/httpclient-3.x/
[hc-logging]: http://hc.apache.org/httpclient-3.x/logging.html
