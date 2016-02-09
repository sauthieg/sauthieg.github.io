---
layout: post
title: "OpenIG 4.0 is now available"
date: "2016-02-09 23:03:56"
image: /assets/article_images/forgerock-wall.jpg
categories: openig
tags: [openig, release]
comments: true
---

January’s release of the ForgeRock Identity Platform includes OpenIG 4.
This release brings new API gateway features, better integration with OpenAM,
extended support for standards, and increased performance.

<!-- more -->

OpenIG 4’s new [audit framework][audit] now handles audit events in
a common way across the whole ForgeRock platform. For example, OpenIG 4 can
track interactions across OpenAM, OpenDJ, and OpenIDM. Audit logs can be
centralized and transactions can be traced across the platform. Additionally,
the audit framework supports logging to files, databases, and the UNIX system log.

Improved [monitoring data][monitoring] for the servers, applications, and APIs
provides a better view of how OpenIG 4 and its routes are used. Delivered
through REST endpoints, data includes request and response statistics, such
as the number of requests, time to respond, and throughput.

The new [throttling][throttling] feature limits access to applications and
APIs, increasing security and fairness. Throttling can enforce flexible rate
limits for a variety of use cases, such as to limit the number of requests
per minute from clients at the same network address.

Several new features improve integration with OpenAM:

* A new [policy enforcement filter][pep] allows only authorized access to
  protected resources.  You can now use OpenIG instead of an OpenAM agent for
  authorization, and centralize all your access control policies in OpenAM.
* SSO and federation for applications has been extended by a
  [token transformation filter][sts] to use with the OpenAM REST Security
  Token Service. By using the filter, a mobile app with an OpenID Connect
  token can now access resources held by a federated service provider.
* A new [password replay filter][replay] simplifies the configuration for
  replaying credentials in common use cases.

Support for standards has been extended:

* [OpenID Connect Discovery][discovery] makes it possible for users themselves,
  instead of system administrators, to select identity providers.
* Initial support is available for a [User Managed Access][uma] resource server,
  where users can control who accesses their resources, when, and under what conditions.

Behind the scenes, OpenIG 4 internals have been refactored to improve
scalability - because we are no longer blocking threads, a single deployment
can handle more requests at the same time.

These are just some of the changes in OpenIG 4. Check the [Release Notes][release-notes]
for a full list of what's new in this release, and download the software from
[ForgeRock’s BackStage][download].

We love your feedback. Please feel free to ask questions, make suggestions,
and tell us what you think of OpenIG by joining the [community][community]
and getting on the [forum][forum] and [mailing list][mailing-list].

- - -

**Note**: If you happened to notice that my english has noticeably improved in
this post, there is a reason :) I therefore would like to give full credits to
Joanne, our new tech writer, who wrote all of this.

[audit]: https://backstage.forgerock.com/#!/docs/openig/4/gateway-guide#audit-event-handlers
[monitoring]: https://backstage.forgerock.com/#!/docs/openig/4/gateway-guide#monitoring
[throttling]: https://backstage.forgerock.com/#!/docs/openig/4/gateway-guide#throttling
[pep]: https://backstage.forgerock.com/#!/docs/openig/4/gateway-guide#chap-pep
[sts]: https://backstage.forgerock.com/#!/docs/openig/4/reference#TokenTransformationFilter
[replay]: https://backstage.forgerock.com/#!/docs/openig/4/reference#PasswordReplayFilter
[discovery]: https://backstage.forgerock.com/#!/docs/openig/4/gateway-guide#oidc-discovery
[uma]: https://backstage.forgerock.com/#!/docs/openig/4/gateway-guide#chap-uma
[release-notes]: https://backstage.forgerock.com/#!/docs/openig/4/release-notes
[download]: https://backstage.forgerock.com/#!/docs/openig/4/release-notes
[community]: https://forgerock.org/openig/
[forum]: https://forgerock.org/forum/fr-projects/openig/
[mailing-list]: https://lists.forgerock.org/mailman/listinfo/openig
