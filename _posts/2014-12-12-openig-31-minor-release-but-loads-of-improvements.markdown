---
layout: post
title: "OpenIG 3.1, minor release but loads of improvements"
date: "2014-12-12"
categories: openig
tags: [openig release 3.1]
image: /assets/article_images/Red_Christmas_present_on_white_background.jpg
comments: true
---

Ho ho ho, Christmas is happening sooner this year! I'm very pleased to announce
[OpenIG 3.1][download]. This minor release focused on usability improvements,
monitoring enablement, session management and, obviously ... bug fixing.

<!-- more -->

For this Christmas release, the [OpenIG team][team] has been very hard at work
during the last weeks to deliver (on time) a delightful version.

# Improve configuration file readability/usability

We learned the hard way that trying to understand the `config.json` and sibling
route's configuration files can gives headaches! Mentally representing a graph of
objects when its representation is completely flat (a list with named pointers)
is too hard.

So we decided to make it easier: life will be easier for you when writing your
configurations and for us when you'll send your not-working configurations for debug ;)

Here is the list of improvements in regards to the ease of configuration:

 * [Object declarations can be inlined][inline]: when an attribute is a reference to another
   defined heap object, you can now directly include the whole object configuration in
   place of the object name. Notice that inlining only makes sense when the referenced object
   is not used elsewhere in your configuration.
   * Inline declarations don't require a `name` attribute (but this is still useful for
     identifying source objects when looking at log messages)
   * If your configuration just has one main handler (as it's usually the case), you can
     even omit the `heap` object array and directly define your object inside of
     the `handler` element
 * Empty `"config": { }` elements can be removed
 * Aligned configuration attributes to use the same name when possible:
   * OAuth 2.0 [Client][client] and [ResourceServer][rs] filter have been aligned
     on `providerHandler`, `requireHttps`, `cacheExpiration` and `scopes` (that is
     now, in both cases, a list of `Expression` instead of just `String`)
   * `config.json`: `handlerObject` is now `handler` (like in route config files)

Better with an example, here is your config before:

```json
{
  "heap": {
    "objects": [
      {
        "name": "Chain",
        "type": "Chain",
        "config": {
          "filters": [
            "ReplaceHostFilter"
          ],
          "handler": "Router"
        }
      },
      {
        "name": "ReplaceHostFilter",
        "type": "HeaderFilter",
        "config": {
          "messageType": "REQUEST",
          "remove": [
            "host"
          ],
          "add": {
            "host": [
              "example.com"
            ]
          }
        }
      },
      {
        "name": "Router",
        "type": "Router",
        "config": {}
      }
    ]
  },
  "handler": "Chain"
}
```

And now:

```json
{
  "handler": {
    "type": "Chain",
    "config": {
      "filters": [
        {
          "type": "HeaderFilter",
          "config": {
            "messageType": "REQUEST",
            "remove": [
              "host"
            ],
            "add": {
              "host": [
                "example.com"
              ]
            }
          }
        }
      ],
      "handler": {
        "type": "Router"
      }
    }
  }
}
```

Notice that most of this tedious work can be done with the [OpenIG Migration Tool][migration].

**I did not lied when I said it was Christmas ;)**

# Console Logs

I can't resists to show you an excerpt of your new logs (at least when you choose a `ConsoleLogSink`):

```
TUE DEC 02 17:36:10 CET 2014 (INFO) _Router
Added route 'oauth2-resources.json' defined in file '/Users/guillaume/tmp/demo/config/routes/oauth2-resources.json'
------------------------------
TUE DEC 02 17:36:11 CET 2014 (INFO) _Router
Added route 'monitor.json' defined in file '/Users/guillaume/tmp/demo/config/routes/monitor.json'
------------------------------
TUE DEC 02 17:36:12 CET 2014 (INFO) {SamlFederationHandler}/heap/1/config/bindings/0/handler
FederationServlet init directory: /Users/guillaume/tmp/demo/SAML
------------------------------
TUE DEC 02 17:36:12 CET 2014 (WARNING) SamlSession
JWT session support has been enabled but no encryption keys have been configured. A temporary key pair will be used but this means that OpenIG will not be able to decrypt any JWT session cookies after a configuration change, a server restart, nor will it be able to decrypt JWT session cookies encrypted by another OpenIG server.
------------------------------
TUE DEC 02 17:36:12 CET 2014 (INFO) _Router
Added route 'wordpress-federation.json' defined in file '/Users/guillaume/tmp/demo/config/routes/wordpress-federation.json'
------------------------------
TUE DEC 02 17:36:19 CET 2014 (ERROR) _Router
The route defined in file '/Users/guillaume/tmp/demo/config/routes/openid-connect-carousel.json' cannot be added
------------------------------
TUE DEC 02 17:36:19 CET 2014 (ERROR) _Router
/heap/0/type: java.lang.ClassNotFoundException: ConsoleLogSinkA
[       JsonValueException] > /heap/0/type: java.lang.ClassNotFoundException: ConsoleLogSinkA
[   ClassNotFoundException] > ConsoleLogSinkA
------------------------------
TUE DEC 02 17:36:46 CET 2014 (WARNING) {HttpClient}/heap/2/config/httpClient
[/heap/2/config/httpClient/config] The 'truststore' attribute is deprecated, please use 'trustManager' instead
------------------------------
```

The logs are now much more readable and concise: the first line is a header line
and gives you the log timestamp (date formatted accordingly to your Locale), the
message's log level and its source name (name of the heap object that produced
the message), then you have the message itself until the blank line separator is
reached.

The attentive reader noticed that Exception messages are handled in a different way:
each line correspond to a summary of an exception in the chain, up to the root cause.

If the `ConsoleLogSink` is configured with the `DEBUG` level, the full exception
stack-trace will be printed, instead of just the condensed rendering.

# Performances

On the performance side, there were also a number of enhancements:

 * A brand new [clustering section][cluster] has been added in the documentation.
   After reading this, the technics to load-balance OpenIG and configure it for fail-over
   will have no secrets for you. This doc tells you how to achieve this with Tomcat and
   Jetty as examples.
 * OAuth 2.0 caches enablement: the `OAuth2ResourceServerFilter` was already capable
   of caching token info data from the IDP, minimizing the network latency, now the
   `OAuth2ClientFilter` is able to cache (and load on-demand) the content of the
   user-info endpoint.
 * [JWT Session][jwt] support, in other word: how to deport session storage from server-side
   to client-side without sacrificing confidentiality. This technic for storing session
   data outside of the server is perfect when you want scalability: no content is stored
   on server, so it is stateless and can be replicated easily. [JSON Web Tokens][jwt-spec] are used to
   have both an easily serializable and deserializable session format and to reach message
   confidentiality (the content is encrypted, only OpenIG can read it again).

# Features

OK, we couldn't resist to add at least a few new features in OpenIG 3.1!

The most core one is [decorator support][decorator]: this feature enable an easy
way to add behaviours to existing heap objects without having to change the code
of theses objects.

Let's have the following example: in OpenIG 3.0, when you wanted to capture the
messages flowing in and out of a given Handler, you had to:

 * Create a `CaptureFilter`
 * Create a `Chain`, configure it with the `CaptureFilter` and using your observed
   handler as terminal handler
 * Update the source reference to your observed handler to use the Chain name

Definitely not user-friendly, and in 3.0 there was no inline heap object declaration,
making this process even more tedious :'(

```json
{
  "heap": {
    "objects": [
      {
        "name": "Chain",
        "type": "Chain",
        "config": {
          "filters": [
            "CaptureFilter"
          ],
          "handler": "Observed"
        }
      },
      {
        "name": "CaptureFilter",
        "type": "CaptureFilter",
        "config": {
          "file": "..."
        }
      },
      {
        "name": "ConsumerOfObserved",
        "type": "....",
        "config": {
          "handler": "Chain"
        }
      },
      {
        "name": "Observed",
        "type": "ClientHandler",
        "config": { }
      }
    ]
  }
}
```

Now, in 3.1, you just add a `capture` decorator in your observed object declaration and you're done!

```json
{
  "heap": [
    {
      "name": "Observed",
      "type": "ClientHandler",
      "capture": [ "request", "response" ]
    }
  ]
}
```

You just added a `capture` decorator to the `Observed` object, now for each object
invokation, the decorator will be called and will do its job (here capturing the
Exchange's content). Easier !

A number of decorators are available out-of-the-box:

 * [`capture`][capture]: as seen earlier this permit easy route debugging and error finding
 * [`timer`][timer]: compute the time spent inside of objects for performance tracking
 * [`audit`][audit]: track messages inside of the system (monitoring)

# Monitoring

Monitoring how your service is behaving is now as easy as it should be :)

Thanks to the decorator framework, your can [audit your routes][monitor] and receive
notifications when observed components (or routes) are traversed.

This is just as simple as adding an audit 'tag' to your route configuration, (this is
used to qualify the notifications):

```json
{
  "handler": {
    "...": "..."
  },
  "audit": [ "wordpress" ]
}
```

Then, you just add a route that activates the `MonitoringEndpointHandler` (a simple
audit agent), just like what @markcraig [explained in his blog](http://marginnotes2.wordpress.com/2014/12/01/openig-a-new-audit-framework/).

At the end, you should have a nice monitoring console:

![Monitoring Console](https://ludopoitou.files.wordpress.com/2014/12/screen-shot-2014-12-09-at-13-15-18.png)
See @ludomp  blog for details on the [monitoring console](https://ludopoitou.wordpress.com/2014/12/10/new-features-in-openig-3-1-statistics/).

# Thanks

Let finish with a huge thanks to all involved in producing this release: developers,
QA engineers, technical writers, architects, product managers, ...!

Now it's up to you to confirm this success: [download and try it by yourself][download] :)

And have nice Christmas vacations!

# Resources

 * [OpenIG 3.1 Releases Notes](http://docs.forgerock.org/en/openig/3.1.0/release-notes/index.html) ([JIRA](https://bugster.forgerock.org/jira/browse/OPENIG-311?filter=11945))
 * [Download OpenIG 3.1][download]
 * [OpenIG Community](https://forgerock.org/openig/) (Check it out, it has been completely renewed!)
 * [ForgeRock Announcement](http://forgerock.com/news-articles/product-announcement-december-2014/)
 * [OpenIG 3.0 Migration Tool][migration]

[download]:  https://backstage.forgerock.com/#/downloads/enterprise/OpenIG
[team]:      http://sources.forgerock.org/users/openig/?sort=commits&d=asc&page=1
[inline]:    http://docs.forgerock.org/en/openig/3.1.0/gateway-guide/index.html#the-configuration
[client]:    http://docs.forgerock.org/en/openig/3.1.0/reference/index.html#OAuth2ClientFilter
[rs]:        http://docs.forgerock.org/en/openig/3.1.0/reference/index.html#OAuth2ResourceServerFilter
[migration]: {% post_url 2014-12-16-openig-30-migration-cli %}
[cluster]:   http://docs.forgerock.org/en/openig/3.1.0/gateway-guide/index.html#load-balancing
[jwt]:       http://docs.forgerock.org/en/openig/3.1.0/reference/index.html#JwtSession
[jwt-spec]:  https://tools.ietf.org/html/draft-ietf-oauth-json-web-token
[decorator]: http://docs.forgerock.org/en/openig/3.1.0/reference/index.html#decorators-conf
[capture]:   http://docs.forgerock.org/en/openig/3.1.0/reference/index.html#CaptureDecorator
[timer]:     http://docs.forgerock.org/en/openig/3.1.0/reference/index.html#TimerDecorator
[audit]:     http://docs.forgerock.org/en/openig/3.1.0/reference/index.html#AuditDecorator
[monitor]: http://docs.forgerock.org/en/openig/3.1.0/gateway-guide/index.html#chap-auditing
