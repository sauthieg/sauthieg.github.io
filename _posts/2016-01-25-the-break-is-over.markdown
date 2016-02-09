---
layout: post
title: "The break is over"
date: "2016-01-25"
image: /assets/article_images/san-diego.jpeg
categories: misc
tags: [misc]
comments: true
---

This has been a year since my last post...

In the meantime, quite a few things happened: a baby boy in february, a new flat,
some major OpenIG refactorings, and 2 releases.

Quite a busy year after all :)

Let's have some kind of retrospective on the year...

<!-- more -->

# January under the West Coast Sun

2015 started (from a professional point of view) very smoothly: we had our yearly
company meeting on the first week of January!

Heading to San Diego, US west coast, 7 time zones to cross, we started the journey
at 4am (CET) and finally arrived at 7pm (PST).

We had 3 wonderful days, meeting with Forgerockers from all around the globe.
Nice place and weather, cool team building activities, interesting people: everything
was here for a great week!

That was a pleasure to finally met with people that you usually only interact
through HipChat/Skype.
We had good feedback from sales and sales engineering on OpenIG.
Engineering breakout sessions had been organized (well un-organized, unconference
style :)), stateless session, new (common) projects were on the plate.

That was a very pleasant (to say the least) experience, leaving us both eager to attend
the next one and energized for the year :)

# New forgerock.org Web Site

You probably noticed already since this is not really new at the time of writing,
but we launched a whole new [forgerock.org](http://forgerock.org) web site:
good-bye Maven generated sites (well they are still [around](http://openig.forgerock.org)
because some content did not find yet its new home), welcome to the future:

* Gamification support (you gain points when you participate)
* Ease access to online resources ([downloads][downloads], [docs][docs], [sources][sources], [blogs][blogs], ...)
* General and per-project [forums][forums]
* CSS harmony ;)

# Rebranded Documentation

[Mark](https://marginnotes2.wordpress.com/) and his team did an amazing job this
year to refresh the documentation's style.

I have to admit that the first time I saw the new documentation, it was like ... **Wow**!

That was so refreshing, [reading the doc][openig-doc] became again a pleasure.

Note that the documentation team continued on their way and they also provided a
documentation that fits perfectly in [ForgeRock's backstage site][openig-backstage-doc]!

Good job guys !

# Great Git Migration

Ahhh, a technical item, finally (I can hear you, you know ;)).

Since day one, OpenIG's source code (and most probably other ForgeRock projects)
have been hosted on a Subversion server.

It was time to move on.

Frankly, I don't recall having used subversion for OpenIG :) The first thing I've
done when I've been hired was to `git svn clone` the OpenIG source code!

Over time, I demoed Git features to co-workers and team members, and gradually,
they did their own clones and start enjoying working on local branches, reworking history, ...

So, at the end, I think we were the most ready team for Git migration: from developers
to QA and doc writers everybody felt quite comfortable with Git!

We have the chance of being a small (but still complex enough) project. That make
OpenIG the candidate of choice for trying imports, giving feedback, and most
important: being the first product migrated!

Kudos to the release engineering team for achieving this huge task, providing
support for the whole company!

# New Hires !

The OpenIG project has welcomed 2 new hires in 2015: Laurent Vaills (Senior Developer)
and Joanne Henry (Doc Writer).

Laurent started in difficult conditions: his first day was all preparing the San
Diego trip, and then moving to the US! That could have been worse :)

Technically Joanne started in the beginning of 2016, just like Laurent: jumping in
with the annual company meeting.

Welcome to both of you.

# Not One but Two OpenIG Releases

The team did a tremendous job for releasing 2 OpenIG versions in 2015:

* OpenIG 3.1.1, a sustaining release with important bug fixes for customers
* OpenIG [4.0.0][download-openig], a major release with loads of features: UMA, PEP, STS (more
  on theses weirds acronyms in a later dedicated post), ...

This year in [numbers][openig-stats] in OpenIG-land:

* **68384** lines were added
* **101208** lines were deleted
* **521** commits
* **13** contributors
* **172** pull requests
* **2512** comments

That was a busy year, I've told you so :)

[downloads]: https://forgerock.org/downloads/
[docs]: https://forgerock.org/documentation/
[sources]: https://forgerock.org/projects/getting-the-source-code/
[blogs]: https://forgerock.org/blog/
[forums]: https://forgerock.org/forums/

[openig-doc]: http://openig.forgerock.org/doc/bootstrap/gateway-guide/index.html
[openig-backstage-doc]: https://backstage.forgerock.com/#!/docs/openig/4/gateway-guide
[openig-stats]: https://stash.forgerock.org/plugins/servlet/graphs?graph=activity&projectKey=OPENIG&repoSlug=openig&refId=all-branches&period=monthly
[download-openig]: https://backstage.forgerock.com/#!/downloads/OpenIG/Open%20Identity%20Gateway/4.0.0/OpenIG%204/war#list
