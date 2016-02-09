---
layout: post
title:  "Introduction to OpenIG"
subtitle: "Part 5: Ease your development with Routes"
date:   2014-12-03
categories: openig
tags: [guide, openig, route]
image: /assets/article_images/introduction-to-openig-p5/IMG_1547.jpg
comments: true
---

A [route][route] is an OpenIG configuration fragment that supports hot reloading. This is
the perfect tool for developing your configuration, trying it, fixing it with a
very fast feedback. This is also very handy for managing your configuration
complexity by splitting it into smaller, more cohesive, sections.

<!-- more -->

# Route

Here is a route configuration file ... just to give you a taste.

{% gist sauthieg/165f33f4a5f9cde96476 %}

Except for the `condition` property, it's really looking like a regular `config.json`
content, right ?

You'll recognise some attributes from the main configuration file:

 * `heap/objects`: where you describe all of your route components and how they're
   linked to each others
 * `handler`: specify the main entry point of your route
 * `baseURI`: where the request should be re-routed (optional)

## Inheritance

An interesting feature of routes is that they inherit objects from their parent route
(`config.json` content being the primary route, ancestor of all others). That means that
any named object defined in a parent config file (let's say `config.json`) can be re-used
by the child route defined components:

Assuming that `config.json` defines a `ClientHandler` instance named `Forwarder`, someone
could write in his own route that would includes this `Chain`:

~~~json
{
  "name": "OutgoingChain",
  "type": "Chain",
  "config": {
    "filters": [ ],
    "handler": "Forwarder"
  }
}
~~~

This feature makes it very easy to share pieces of configuration logic in a common/shared
place (the parent route or `config.json`).

## Isolation

Each route is having a dedicated namespace (some kind of private area) where objects
declared in its `heap/objects` array are kept.
That means that a route cannot access objects defined in another route (except if
the object is declared in the parent route, thanks to inheritance), effectively
providing content isolation.

That's utmost useful for multiple reasons:

 * When writing the configuration, errors may happen (we cannot always be right on
   the first shot), having isolation limits the number of issues that may arise due
   to unmanaged (or unseen) dependencies.
 * When hot-reloading is enabled (the default), this permits fragmented update of
   your system (just update what you need, not the whole configuration)

## Exchange processing

The `heap/objects` array contains all of your route components (declaration, configuration and bindings with other components). One of the declared `Handler` has to be referenced through the `handler` top level attribute:

~~~json
{
  "heap": {
    "objects": [
      {
        "name": "EntryPoint",
        "type": "StaticResponseHandler",
        "config": {
          "status": 200
        }
      }
    ]
  },
  "handler": "EntryPoint"
}
~~~

Notice that the `handler` attribute is **required**, an error will be thrown if not set.

## Conditional execution

Unlike `config.json`, a route is **conditionally invoked** given the result of a condition:

~~~json
{
  "condition": "${exchange.request.form['forward'] == true}"
}
~~~

The `condition` is expressed as an `Expression` and gives you access to
[all of the properties][exchange-model] of the `Exchange` being processed. It has
to return a `boolean` (any other return type will be considered as `false`).

If the `condition` attribute is absent (or `null`), the route will always accept
the `Exchange` (if proposed).

Here are some examples of useful conditions:

* Only requests whose paths are starting with `/wordpress/`:

  ~~~ json
  {
    "condition": "${matches(exchange.request.uri, '^/wordpress/')}"
  }
  ~~~

* Requests paths starting with a value or another:

  ~~~ json
  {
    "condition": "${matches(exchange.request.uri.path, '^(/carousel|/openid)')}"
  }
  ~~~

## Uri Rebasing

When an `Exchange` is accepted in a route, its `request.uri` may be rebased (if
there is a `baseURI` top level attribute). Notice this is the very first thing
happening to the `Exchange` inside of the route (even before being handled by
your main `handler` object).

# Activation / Deactivation

The [`RouterHandler`][router] (which is the heap object managing routes) is observing a given
directory (`${openig.base}/routes/` unless specified to something else) for route
files. Each file that ends with `.json` is considered as a route and is tried to
be activated. If everything goes well, the new route is available into the system,
otherwise, an error is displayed into the logs.

Activating a route is as simple as dropping a `.json` file in the right folder.

And deactivating as simple as removing that file.

Notice that you can simply rename the file with a different extension to have it
ignored by the system (and thus uninstalled if it was previously active):

~~~
routes/
  wordpress.json -> wordpress.json.disabled
~~~

## Scan interval

The route file scan is done at most 1 time per `scanInterval` (default
value to **10** seconds) and that this scan is not done by a background thread but
by the thread that handle the request (ie don't wait for a log message saying a
new route was discovered :) ).

## Ordering

The `RouteHandler` imposes a lexicographical order when trying to hand-off the current
`Exchange` to one of the available route. The file name is used as a default value
when there is no `name` top level attribute in the route configuration.

For example, given the following `routes/` folder content (no `name` attribute
defined in any of the routes), the routes would be tried in this order: `00-main.json`,
`next.json` and `zz-default.json`.

~~~
routes/
  00-main.json    (1)
  zz-default.json (3)
  next.json       (2)
~~~

You can override the default system provided name (based on the file name) by specifying
a `name` attribute in the route:

~~~json
{
  "name": "my-name"
}
~~~

# Conclusion

Routes is a very handy tool for OpenIG users, there is even a global
[`Router`][router] in the default configuration: just drop your json files in
`${openig.base}/routes/` and you're good to go !

# Next

In the next post we'll talk about OAuth 2.0.

[exchange-model]: http://docs.forgerock.org/en/openig/3.0.0/reference/index.html#exchange-object-model-conf
[route]: http://docs.forgerock.org/en/openig/3.0.0/reference/index.html#Route
[router]: http://docs.forgerock.org/en/openig/3.0.0/reference/index.html#Router
