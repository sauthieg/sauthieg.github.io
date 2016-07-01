---
layout: post
title: "Blocking on Promises"
subtitle: "Hard-learned lessons on asynchronous programming"
date: "2016-06-27"
categories: openig
tags: [openig, async, promise]
image: /assets/article_images/rockhands01.jpg
comments: true
---

OpenIG is now 100% asynchronous! In other words, we're using a lot of [Promises][promise-apidoc].
Recently, we faced a strange issue where a thread remained in the `WAITING`
state, waiting for an HTTP response to come.

<!-- more -->

Here is the thread dump we got:

~~~
"I/O dispatcher 1" #13 prio=5 os_prio=31 tid=0x00007f8f930c3000 nid=0x5b03 in Object.wait() [0x000070000185d000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076b155b80> (a org.forgerock.util.promise.PromiseImpl)
	at java.lang.Object.wait(Object.java:502)
	at org.forgerock.util.promise.PromiseImpl.await(PromiseImpl.java:618)
	- locked <0x000000076b155b80> (a org.forgerock.util.promise.PromiseImpl)
	at org.forgerock.util.promise.PromiseImpl.getOrThrow(PromiseImpl.java:144)
~~~

Ok, to tell the truth, the code was performing a blocking call on a
`Promise<Response>`, so we got what we deserved, right? Well, that code has
been around (in more or less the same form) for a long time, and, AFAIK, nobody
had experienced a thread blockage issue.

Here is the code where the blocking call happened:

~~~java
try {
  Promise<JsonValue, OAuth2ErrorException> promise = registration.getUserInfo(context, session);
  return promise.getOrThrow(); // < - - - - - - block here
} catch (OAuth2ErrorException e) {
  logger.error(...);
} catch (InterruptedException e) {
  logger.error(...);
}
~~~

Dead simple, isn't it?

The strangest thing happened when we engaged a timeout on the promise
(using [`getOrThrow(10, SECONDS)`][get-or-throw]). After the timeout expired,
the Promise unblocked and we saw a real `Response` inside (with an associated `SocketTimeoutException`), just as if it was already there, but without the
promise triggering callbacks.

How could this be possible? Having a thread waiting for a result of another HTTP
request, when the HTTP client library in use (Apache HttpAsyncClient in our case)
is supposed to handle threads by itself (and correctly).

Well, we had to dig, but we found the key deep inside the HTTP library:

~~~java
// Distribute new channels among the workers
final int i = Math.abs(this.currentWorker++ % this.workerCount);
this.dispatchers[i].addChannel(entry);
~~~

### What is this code doing?

This code is called when an NIO event comes back into the HTTP library (such as
the content of a response). The code basically selects one of the worker threads
to be responsible for processing the response.

### Is this wrong?

Depends on your point of view ;) Initially, I thought that it was plain
wrong: this code doesn't know if the thread is busy doing something else or blocked.

After a bit more thought, it’s not that obvious - because responses are processed
asynchronously, the request and response flows are clearly decoupled, so there is
no easy way to know if the requestor thread is the same thread as the response thread.

### So what happened?

The scenario is quite simple:

* Create a CHF `HttpClientHandler`
* Send the first HTTP request
* When the response is there, trigger another HTTP call
* See the blocked thread

In practice, you probably have to configure the number of workers, until you can
find a setting where the distribution function re-assigns the response to the
requestor's thread. The easiest configuration is to use a single-thread :)

Here is a code sample to reproduce the "issue":

~~~java
// Create an HTTP Client with a single thread
Options options = Options.defaultOptions()
                         .set(AsyncHttpClientProvider.OPTION_WORKER_THREADS, 1);
HttpClientHandler client = new HttpClientHandler(options);

// Perform a first request
Promise<Response, NeverThrowsException> main;
Request first = new Request().setMethod("GET").setUri("http://forgerock.org");
main = client.handle(new RootContext(), first)
             .then(value -> {
                 // Perform a second request on the thread used to receive the response
                 try {
                     Request second = new Request().setMethod("GET")
                                                   .setUri(URI.create("http://www.apache.org"));
                     return client.handle(new RootContext(), second)
                                  // and block here
                                  .getOrThrow(5, TimeUnit.SECONDS);
                 } catch (InterruptedException e) {
                     return newInternalServerError(e);
                 } catch (TimeoutException e) {
                     return newInternalServerError(e);
                 }
             });

// Get the response on the "main" thread
Response response = main.getOrThrow();
long length = response.getHeaders().get(ContentLengthHeader.class).getLength();
System.out.printf("response size: %d bytes%n", length);
~~~

Note that you can clone the [`sauthieg/blocking-on-promise`][sample] GitHub
repository if you want to play with that code by yourself.

### The solution

Avoid the blocking call and use `Promise` with appropriately typed callbacks in
every step of the processing.

To prevent any form of thread blockage, register callbacks (`ResultHandler`,
`Function` or `AsyncFunction`) instead of actively waiting for a result/failure.

So now, the caller thread is not blocked. It will be available for its next
task after all callbacks are registered on the promise.

#### Bad code example

~~~java
try {
  Promise<JsonValue, OAuth2ErrorException> promise = registration.getUserInfo(context, session);
  JsonValue info = promise.getOrThrow(); // < - - - - - - block here
  return new Response(Status.OK).setEntity(info);
} catch (OAuth2ErrorException e) {
  return newInternalServerError(e);
} catch (InterruptedException e) {
  logger.error(...);
}
~~~

#### Good code example

~~~java
return registration.getUserInfo(context, session)
                   .then((info) -> {
                     // process the result when it will be available
                     return new Response(Status.OK).setEntity(info);
                   },
                   (e) -> {
                     // Convert exception
                     return newInternalServerError(e);
                   })
~~~

### The conclusion

**Never block any threads when you're doing asynchronous processing**.

The async programming model is designed to maximize use of machine's resources, and implicitly
requires that there are no blocking call on the stack. As there should be no threads
blocked at anytime, any thread can be selected to process a response. That explains
why our HTTP library is not even trying to see if the elected thread is busy or not.

More pragmatically, when using our `Promise` API, you'll know that you're in
trouble (and a potential victim of that threading issue) if you see code
that uses one of the `get()` method variations on the `Promise` interface.

In OpenIG, this can be in any `Filter` / `Handler` that you write by yourself, or
in any Groovy script. So take a look at the code you execute in OpenIG, we make a
point to write 100% asynchronous / non-blocking code, what about you?

#### Exhaustive list of blocking methods in `Promise`

* `Promise.get()` / `Promise.get(long, TimeUnit)`
* `Promise.getOrThrow()` / `Promise.getOrThrow(long, TimeUnit)`
* `Promise.getOrThrowUninterruptibly()` / `Promise.getOrThrowUninterruptibly(long, TimeUnit)`

### Some links

* The `Promise` [API Documentation][promise-apidoc]
* Github [Code Sample][sample]

[promise-apidoc]: https://backstage.forgerock.com/static/docs/openig/4/apidocs/org/forgerock/util/promise/package-summary.html
[get-or-throw]: https://backstage.forgerock.com/static/docs/openig/4/apidocs/org/forgerock/util/promise/Promise.html#getOrThrow(long,%20java.util.concurrent.TimeUnit)
[sample]: https://github.com/sauthieg/blocking-on-promise
