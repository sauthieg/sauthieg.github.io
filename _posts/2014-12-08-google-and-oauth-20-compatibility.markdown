---
layout: post
title: "Google and OAuth 2.0 Compatibility"
subtitle: "Eliminate all other factors, and the one which remains must be the truth. (Sherlock Holmes)"
date: "2014-12-08"
categories: openig
tags: [google, oauth2, compat, spec, standard, fail]
image: /assets/article_images/google-oauth2-fail/square_peg_in_a_round_hole.jpg
comments: true
---


Over the weekend I was pinged by my QA engineers about new failures in our OAuth 2.0 test suite.
Not a good sign, just a few days before the OpenIG 3.1 release date ! Anyway, I've made my hands
dirty to track down the problem up to the source. Here is the story :)

<!-- more -->

<div style="font-weight: bolder;color: firebrick;text-align: center;font-size: larger;">
 This issue has been fixed the 12th of December 2014
 (<a href="http://stackoverflow.com/a/27434915/2831176">see stack overflow answer</a>).
</div>

# Everything starts with ...

... a failure, at least I had the screenshot of a nice Exception, pretty explicit by the way:

![The failure]({{ site.url }}/assets/article_images/google-oauth2-fail/test-failure.png)

# ... then you start digging

... on your side. Yeah, you must have done something wrong in your code, Google is
*just working* (Apple ©), right ?

The part of the code referred by that stack trace is dealing with the Access Token
Response JSON structure returned from the OAuth 2.0 Token endpoint. But this has not
been changed since it was first checked in (or almost, nothing recent at least), and the
QA tests that were failing on friday have been working previously.

I've tried to think about every code checkin' to see the potential impact on the
expected behaviour, found nothing.

On the QA side, they re-run an older OpenIG's version (that was working, remember ?), and now it's failing too !?

WTF ?

# ... looking more closely to what Google send

At that point, it's time to open your IDE and debug OpenIG, this way we'll see what see OpenIG see...

No wonder, it's really receiving a successful access token response of the following form:

```json
{
  "access_token": "ya29.1gD56tBWtHW3K7oZ0FINTnsqa4VYiE2YGZeQXgJ4ID79E-mZxNWoyYi7pKrs_Vyxj8FZbuxh_RGTJw",
  "token_type": "Bearer",
  "expires_in": "3600",
  "refresh_token": "1/dGjGYC7sDFaBwpdUVpkJP2mYFYTU8HAh7T6szsKGYTs"
}
```

`expires_in` is really a `String`, even if semantically it's a `Number` !

Ok Google is now sending a `String` instead of a `Number` for this attribute.

**My world collapse: Google can't be wrong for something that simple !**

That reminds me a famous Sherlock Holmes quote: `Eliminate all other factors, and the one which remains must be the truth.`

Sad...

# ... what about the spec ?

As per the [OAuth 2.0 Spec][oauth2], a successful response looks like the following:

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache
```
```json
{
  "access_token":"2YotnFZFEjr1zCsicMWpAA",
  "token_type":"example",
  "expires_in":3600,
  "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
  "example_parameter":"example_value"
}
```

They even give a [syntax](https://tools.ietf.org/html/rfc6749#appendix-A.14) for
`expires_in` attribute:

```
expires-in = 1*DIGIT
```

Pretty simple, though ?

# ... let's find a solution

Hopefully, it was not very hard to fix: I just made one of our class [a little bit smarter][fix]
(dealing especially with the `String` case).

Tested and committed, ready for the release :)

# ... wait a minute, what changed on Google's side ?

They seem to have **updated their OAuth 2.0 service implementation**.

Take a look at the [OpenID Connect Discovery](https://accounts.google.com/.well-known/openid-configuration) endpoint returned JSON:

```json
{
  "issuer": "accounts.google.com",
  "authorization_endpoint": "https://accounts.google.com/o/oauth2/auth",
  "token_endpoint": "https://www.googleapis.com/oauth2/v3/token",
  "userinfo_endpoint": "https://www.googleapis.com/plus/v1/people/me/openIdConnect",
  "revocation_endpoint": "https://accounts.google.com/o/oauth2/revoke",
  "jwks_uri": "https://www.googleapis.com/oauth2/v2/certs",
  "response_types_supported": [
    "code", "token", "id_token",
    "code token", "code id_token", "token id_token", "code token id_token",
    "none"
  ],
  "subject_types_supported": [ "public" ],
  "id_token_alg_values_supported": [ "RS256" ]
}
```

And, more closely to the `token_endpoint` value: it's using a `v3` API.
It's not even advertised on the [Google API Explorer][explorer] (*yet ?*).

Another hint that something changed was in the *last edited* timestamp at the bottom
of the [OpenIDConnect](https://developers.google.com/accounts/docs/OpenIDConnect) and
[OAuth2WebServer](https://developers.google.com/accounts/docs/OAuth2WebServer)
google developers pages:

![Edited on Friday the 5th](/assets/article_images/google-oauth2-fail/OpenID_Connect__OAuth_2_0_for_Login__-_Google_Accounts_Authentication_and_Authorization.png)

# Funny facts

I played with [Google OAuth Playground][playground] this morning (BTW, it's pretty neat and usable, good stuff).

Surprisingly, when you choose the `Google` OAuth endpoints set, you'll see they're not
using the `v3` one yet: they're using `https://accounts.google.com/o/oauth2/token`.

When manually re-configured to use the v3 API, you clearly see the `String`:

![Configured with v3 endpoints](/assets/article_images/google-oauth2-fail/OAuth_2_0_Playground.png)

# Links

* Post picture ['Square Peg In A Round Hole'](http://adam-purcell.com/ministry/ministry-in-a-box/)
* Stack Overflow question: http://stackoverflow.com/questions/27363481/cannot-authenticate-anymore-with-new-google-oauth-2-0-token-endpoint-v3

[oauth2]: https://tools.ietf.org/html/rfc6749
[playground]: https://developers.google.com/oauthplayground
[fix]: http://sources.forgerock.org/changelog/openig?cs=800
[explorer]: https://developers.google.com/apis-explorer/#search/oauth2/
