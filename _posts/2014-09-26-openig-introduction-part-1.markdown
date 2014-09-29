---
layout: post
title:  "Introduction to OpenIG"
subtitle: "Part 1: Use cases"
date:   2014-09-26
categories: openig
featured: true
tags: [guide, openig, usecases, openid-connect, oauth2, saml, rest]
image: /assets/article_images/introduction-to-openig-p1/IMG_0919.jpg
comments: true
#published: false
---

# Welcome

You've probably landed here because you want to know something about [OpenIG][openig].
This is the right place to be :)

This post is the first one of a serie of OpenIG-related articles that would
gives you hand-on samples with explanations.

<!-- more -->

## Identity Gateway

[OpenIG][openig] stands for **Open** **I**dentity **G**ateway, it is an identity/security specialized
reverse proxy. That means that it *control* the access to a set of internal services.

By *controling the access*, I mean that it intercepts all the requests coming to the
protected service (be it a [RESTful][restful] API or a web application) and process them
before (and after) forwarding them to the server.

Different kind of processing can be handled:

 * Request authorization
 * Capture and password replay
 * Message logging
 * Transformation

## Commons use cases

Ok, that was a bit of a generalist description (transformation is intentionally
vague :) ). Having some real-life use cases will help to have a better
feeling/understanding of what OpenIG is capable of.

### SAML Application Federation

In this use case, OpenIG acts as a SAML-enabled facade to a somehow legacy
application that cannot be adapted to support SAML federation. The Identity
Provider (could be [OpenAM][openam]) will consider OpenIG as a standard SAML
Service Provider.

![SAML CoT with OpenIG used as a facade to a legacy application](/assets/article_images/introduction-to-openig-p1/circle-of-trust.jpg)

### Application Authentication

Here, OpenIG acts as an [OpenID Connect][oidc] Relying Party ([OIDC terminology][oidc-term]
 for client) and requires the user to authenticate to an OpenID Connect Provider
 (the identity provider) before giving him/her access to the protected application.

Authenticated user's profile information (such as name, email, address, picture, ...)
are available to enrich the user experience, or make further verifications.

![OpenID Connect - OpenIG Relying Party](/assets/article_images/introduction-to-openig-p1/openid-connect.jpg)

### RESTful Services Protection

This simple case shows OpenIG verifying request to a proxified RESTful API: each
request must contains a valid [OAuth 2.0][oauth2] Bearer Token to be allowed to
reach the service API. In this case, OpenIG acts as an OAuth 2.0 Resource Server.

Very useful if you have an old-fashioned REST API that you cannot easily update
to deal with OAuth 2.0 tokens.

![OAuth 2.0 - OpenIG ResourceServer](/assets/article_images/introduction-to-openig-p1/oauth2.jpg)

# Not enough ?

Well, we're done with the *'marketing'* stuff.
In the next post, we will start to play with OpenIG.

See you soon!

[openig]:       http://openig.forgerock.org
[openam]:       http://openam.forgerock.org
[oidc]:         http://openid.net/connect/
[oidc-term]:    http://openid.net/specs/openid-connect-core-1_0.html#Terminology
[oauth2]:       http://tools.ietf.org/html/rfc6749
[restful]:      http://en.wikipedia.org/wiki/Representational_state_transfer
