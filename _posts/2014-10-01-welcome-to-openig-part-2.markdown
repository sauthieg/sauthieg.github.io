---
layout: post
title:  "Introduction to OpenIG"
subtitle: "Part 2: First contact..."
date:   2014-10-01
categories: openig
tags: [guide, openig, tomcat, json]
image: /assets/article_images/introduction-to-openig-p2/cosmo_earth_star_trek_trek_first_contact_artwork_1920x1080_86137.jpg
comments: true
---

# What do I need to start ?

First, you need to install OpenIG in the root context of a web
container (Tomcat used for the example):

<!-- more -->

 1. [Download OpenIG](http://forgerock.com/products/open-identity-stack/openig/)
 2. Grab a fresh [Apache Tomcat][tomcat] instance (Tomcat 7.0 or 8.0) and unzip it
 3. Remove all war files from the `webapps/` directory
 4. Move `OpenIG-3.0.0.war` to that directory **and** rename it to `ROOT.war`
    (OpenIG needs to be installed in the root context: `/`)
 5. Start Tomcat (`bin/catalina.sh run` from within the tomcat directory).

    Don't worry if you see some warnings about *Offending classes* during startup:
    that's because OpenIG web-app provides its own EL implementation (useful when the
    web container doesn't provide one). This pollutes the startup logs but does
    not prevent OpenIG to run properly.

If you're curious, you can go to http://localhost:8080 (or whatever port
number you're using), you should see the OpenIG's welcome page (admire the ASCII art).

![OpenIG 3.0.0 - Welcome Page](/assets/article_images/introduction-to-openig-p2/welcome.png)

If you don't see anything, or an error, make sure that you don't have a previous
OpenIG configuration file in `$HOME/.openig/config/config.json`
(`$APP_DATA\ForgeRock\OpenIG\config\config.json` for our windows users).

You might wonder why OpenIG requires to be installed in the root context.
The reason is simple: most of the time, when you want to access a web site,
you just wanna type a domain name in your browser navbar (like `http://app.example.com`).

That gives you a very easy to use and to remember entry point URL as well as the
flexibility of mapping incoming requests to any internal URL (really routing
requests in fact).

## Your first configuration

What would you think of an OpenIG **Hello World** example?
After all that reading, I'm sure you'll be happy to get your hands dirty :)

So, finally some code (let's face it; it's JSON configuration really).

{% gist sauthieg/17ebb198470e90e249be hello-world.json %}

Place that configuration file inside the `$HOME/.openig/config/routes/` directory.
It will be auto-(re)loaded by OpenIG.

It's quite self-descriptive: if the incoming request's path matches `^/hello`
(starts with `/hello`), it is dispatched to this route
(a JSON file placed in the `routes/` directory is called a *route*). This route
is configured with a single `StaticResponseHandler` that simply return
a `Hello World` message (we'll see what a `Handler` is in an upcoming post) to
the user-agent.

![OpenIG Hello World](/assets/article_images/introduction-to-openig-p2/hello-world.png)

At first look, that doesn't seem very fancy, but you've been exposed to
multiple core concepts of OpenIG (they'll be covered in more details in
subsequent posts):

 * The OpenIG configuration file format (expressed as JSON)
 * The [expression language][doc-expression] (`${}` for expressing conditions
   based the value of some properties)
 * The `Exchange` HTTP model (`exchange.request.uri.path` in the expression)
 * The route concept (a simple way to isolate processing chains in different
   files and to ease setup with auto-(re)loading)

In the next post, we'll have a look at the concepts providing the backbone
of OpenIG (`Exchange`, `Handler` and `Filter`).

See you soon.

[doc-expression]: http://docs.forgerock.org/en/openig/3.0.0/reference/index.html#expressions-conf
[tomcat]: http://tomcat.apache.org
