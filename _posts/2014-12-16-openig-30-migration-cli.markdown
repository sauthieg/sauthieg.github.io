---
layout: post
title: "OpenIG 3.0 Migration CLI"
date: "2014-12-16"
categories: openig
tags: [openig migrate json config]
image: /assets/article_images/command-line.jpeg
comments: true
fork-me: sauthieg/openig-migration
---

ForgeRock OpenIG 3.1 has been recently [released][31] (12th of December, on time ;) ) and
it's time for you to give it a try! Already done that ? Great! But wait a minute,
what are all theses deprecation warning messages you see on the logs ?

<!-- more -->

```
WED DEC 17 16:46:03 CET 2014 (WARNING) file:/.../config/config.json
The configuration field heap/objects has been deprecated. Heap objects should now be listed directly in the top level "heap" field, e.g. { "heap" : [ objects... ] }.
------------------------------
WED DEC 17 16:46:03 CET 2014 (WARNING) file:/.../config/config.json
[/] The 'handlerObject' attribute is deprecated, please use 'handler' instead
------------------------------
```

Don't worry, this messages are only warnings, your old configurations are still parseable
by OpenIG 3.1: everything is backward compatible, we just warn you that, in the next
major release, theses elements are likely to be unsupported (in other words, they
would probably be ignored).

Hopefully, in order to reduce the burden of migrating your config files manually,
which is always an error-prone operation, I wrote a small [toolkit][github] that will massage
your existing JSON configuration and produce a 3.1 compatible JSON.

At time of writing, the [openig-migration][github] toolkit supports the following migration actions:

 * `heap/objects` array has been simplified to just heap
 * Heap object declaration inline (when possible)
 * Remove `"config": {}` (empty element)
 * Rename `RedirectFilter` to `LocationHeaderFilter`
 * Rename `handlerObject` to `handler`
 * Rename `OAuth2ResourceServerFilter` deprecated attributes to new names

# Usage

As there is still no binary available, you have to build the binary yourself (make
sure you use a JDK 8 at both compile and runtime):

```sh
git clone https://github.com/sauthieg/openig-migration.git
cd openig-migration
mvn clean install
```

Then, you have to execute it: it expects a path to the JSON document to migrate:

```sh
java -jar target/openig-migration-1.0-SNAPSHOT-jar-with-dependencies.jar .../config.json
```

This command line outputs the transformed JSON on `System.out`.

# Example

This example is extracted from OpenIG 3.0 documentation:

```json
{
  "heap": {
    "objects": [
      {
        "name": "DispatchHandler",
        "type": "DispatchHandler",
        "config": {
          "bindings": [
            {
              "condition": "${exchange.request.uri.path == '/login'}",
              "handler": "LoginChain",
              "baseURI": "http://TARGETIP"
            },
            {
              "handler": "OutgoingChain",
              "baseURI": "http://TARGETIP"
            }
          ]
        }
      },
      {
        "name": "LoginChain",
        "type": "Chain",
        "config": {
          "filters": [
            "LoginRequest"
          ],
          "handler": "OutgoingChain"
        }
      },
      {
        "name": "LoginRequest",
        "type": "StaticRequestFilter",
        "config": {
          "method": "POST",
          "uri": "https://TARGETIP/login",
          "form": {
            "USER": [
              "myusername"
            ],
            "PASSWORD": [
              "mypassword"
            ]
          }
        }
      },
      {
        "name": "OutgoingChain",
        "type": "Chain",
        "config": {
          "filters": [
            "CaptureFilter"
          ],
          "handler": "ClientHandler"
        }
      },
      {
        "name": "CaptureFilter",
        "type": "CaptureFilter",
        "config": {
          "captureEntity": false,
          "file": "/tmp/gateway.log"
        }
      },
      {
        "name": "ClientHandler",
        "comment": "Responsible for sending all requests to remote servers.",
        "type": "ClientHandler",
        "config": {}
      }
    ]
  },
  "handlerObject": "DispatchHandler"
}
```

... and is migrated to:

```json
{
  "heap": [
    {
      "name": "DispatchHandler",
      "type": "DispatchHandler",
      "config": {
        "bindings": [
          {
            "condition": "${exchange.request.uri.path == '/login'}",
            "handler": {
              "name": "LoginChain",
              "type": "Chain",
              "config": {
                "filters": [
                  {
                    "name": "LoginRequest",
                    "type": "StaticRequestFilter",
                    "config": {
                      "method": "POST",
                      "uri": "https://TARGETIP/login",
                      "form": {
                        "USER": [
                          "myusername"
                        ],
                        "PASSWORD": [
                          "mypassword"
                        ]
                      }
                    }
                  }
                ],
                "handler": "OutgoingChain"
              }
            },
            "baseURI": "http://TARGETIP"
          },
          {
            "handler": "OutgoingChain",
            "baseURI": "http://TARGETIP"
          }
        ]
      }
    },
    {
      "name": "OutgoingChain",
      "type": "Chain",
      "config": {
        "filters": [
          {
            "name": "CaptureFilter",
            "type": "CaptureFilter",
            "config": {
              "captureEntity": false,
              "file": "/tmp/gateway.log"
            }
          }
        ],
        "handler": {
          "name": "ClientHandler",
          "comment": "Responsible for sending all requests to remote servers.",
          "type": "ClientHandler"
        }
      }
    }
  ],
  "handler": "DispatchHandler"
}
```

Notice the now inlined object declaration that make it easier to follow the execution
flow and understand what happen to your request. The empty un-necessary elements
have been removed too.

# Limitations

At time of writing, some old JSON files may be incorrectly handled by the migration CLI:

 * Incorrectly escaped regular expressions (in `EntitytExtractFilter`)
 * Multi-line Strings (without a line ending `\`)

# Contributions

Feel free to [open issues][issues], and/or [fork the repository][fork] for any
improvements you can think of.

I'll be happy to consider all [pull requests][pr].

Happy hacking !

[31]: {% post_url 2014-12-12-openig-31-minor-release-but-loads-of-improvements %}
[github]: https://github.com/sauthieg/openig-migration
[issues]: https://github.com/sauthieg/openig-migration/issues
[fork]: https://github.com/sauthieg/openig-migration/fork
[pr]: https://github.com/sauthieg/openig-migration/pulls
