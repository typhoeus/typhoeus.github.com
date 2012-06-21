---
title: "Getting Started"
layout: article
---

Like a modern code version of the mythical beast with 100 serpent heads,
Typhoeus runs HTTP requests in parallel while cleanly encapsulating handling
logic. To be a little more specific, it’s a library for accessing web services
in Ruby. It’s specifically designed for building RESTful service oriented
architectures in Ruby that need to be fast enough to process calls to multiple
services within the client’s HTTP request/response life cycle.

Some of the awesome features are parallel request execution, memoization of
request responses (so you don’t make the same request multiple times in a
single group), built in support for caching responses to memcached (or
whatever), and mocking capability baked in. It uses libcurl and libcurl-multi
to work this speedy magic. I wrote the bindings myself so it’s yet another
Ruby libcurl library, but with some extra awesomeness added in. FFI is used to
interface with the library so it works with any Ruby implementation.

##  Installation

Typhoeus requires you to have a current version of libcurl installed. The
easiest solution is to use your system’s package manager to install it. If
that doesn’t work, you can grab a package off of [the curl
website](http://curl.haxx.se/download.html) and manually install it following
the instructions given there. Typhoeus will work with version 7.19.4 or higher
(earlier versions might work but no guarantees are provided).

To install Typhoeus, simply run:

    gem install typhoeus

If you’re on Debian or Ubuntu and getting errors while trying to install, it
could be because you don’t have the latest version of libcurl installed. Do
this to fix:

    sudo apt-get install libcurl4-gnutls-dev

If you’re still having issues, please let me know on [the mailing
list](http://groups.google.com/group/typhoeus).

There’s one other thing you should know. The Easy object (which is just a
libcurl thing) allows you to set timeout values in milliseconds. However, for
this to work you need to build libcurl with c-ares support built in.

##  Windows Support

Typhoeus runs perfectly on Windows. The tricky part is knowing how to install
libcurl in the absence of a package manager.

To install libcurl, simply grab [the latest libcurl
package](http://curl.haxx.se/download.html#Win32) off of the curl website,
extract the bin directory, and then add the path to the bin directory into the
PATH environment variable. Ruby with then be able to find libcurl properly and
everything will just work.

##  Usage

The primary interface for Typhoeus is comprised of three classes: Request,
Response, and Hydra. Request represents an HTTP request object, response
represents an HTTP response, and Hydra manages making parallel HTTP
connections.

<script src="https://gist.github.com/2965397.js?file=typhoeus_getting_started_1.rb"></script>

### Making Quick Requests

The request object has some convenience methods for performing single HTTP
requests. The arguments are the same as those you pass into the request
constructor.

<script src="https://gist.github.com/2965397.js?file=quick_requests.rb"></script>

### Handling HTTP errors

You can query the response object to figure out if you had a successful
request or not. Here’s some example code that you might use to handle errors.

<script src="https://gist.github.com/2965397.js?file=handling_http_errors.rb"></script>

This also works with serial (blocking) requests in the same fashion. Both
serial and parallel requests return a Response object.

### Handling file uploads

A File object can be passed as a param for a POST request to handle uploading
files to the server. Typhoeus will upload the file as the original file name
and use Mime::Types to set the content type.

<script src="https://gist.github.com/2965397.js?file=handling_file_uploads.rb"></script>

### Making Parallel Requests

<script src="https://gist.github.com/2965397.js?file=making_parallel_requests.rb"></script>

The execution of that code goes something like this. The first and second
requests are built and queued. When hydra is run the first and second requests
run in parallel. When the first request completes, the third request is then
built and queued up. The moment it is queued Hydra starts executing it.
Meanwhile the second request would continue to run (or it could have completed
before the first). Once the third request is done, hydra.run returns.

### Specifying Max Concurrency

Hydra will also handle how many requests you can make in parallel. Things will
get flakey if you try to make too many requests at the same time. The built in
limit is 200. When more requests than that are queued up, hydra will save them
for later and start the requests as others are finished. You can raise or
lower the concurrency limit through the Hydra constructor.

<script src="https://gist.github.com/2965397.js?file=specifying_max_concurrency.rb"></script>

<!--
### Memoization

Hydra memoizes requests within a single run call. You can also disable
memoization.

<script src="https://gist.github.com/2965397.js?file=memoization.rb"></script>

### Caching

Hydra includes built in support for creating cache getters and setters. In the
following example, if there is a cache hit, the cached object is passed to the
on\_complete handler of the request object.

<script src="https://gist.github.com/2965397.js?file=caching.rb"></script>

The example shows that you can provide a block for cache\_getter and cache\_setter
which is responsible for extracting the information from a caching solution you
prefer.

### Direct Stubbing

Hydra allows you to stub out specific urls and patterns to avoid hitting
remote servers while testing.

<script src="https://gist.github.com/2965397.js?file=direct_stubbing.rb"></script>

The queued request will hit the stub. The on\_complete handler will be called
and will be passed the response object. You can also specify a regex to match
urls.

<script src="https://gist.github.com/2965397.js?file=direct_stubbing_2.rb"></script>
-->

### The Singleton

All of the quick requests are done using the singleton hydra object. If you
want to enable caching or stubbing on the quick requests, set those options on
the singleton.

<script src="https://gist.github.com/2897980.js?file=singleton.rb"></script>

### Timeouts

No exceptions are raised on HTTP timeouts. You can check whether a request
timed out with the following methods:

<script src="https://gist.github.com/2897980.js?file=timeouts.rb"></script>

### Following Redirections

Use `:follow_location => true`, eg:

<script src="https://gist.github.com/2897980.js?file=follow_redirect.rb"></script>

### Basic Authentication

<script src="https://gist.github.com/2897980.js?file=basic_auth.rb"></script>

### SSL

SSL comes built in to libcurl so it’s in Typhoeus as well. If you pass in a
url with “https” it should just work assuming that you have your [cert
bundle](http://curl.haxx.se/docs/caextract.html) in order and the server is
verifiable. You must also have libcurl built with SSL support enabled.

Now, even if you have libcurl built with OpenSSL you may still have a messed
up cert bundle or if you’re hitting a non-verifiable SSL server then you’ll
have to disable peer verification to make SSL work. Like this:

    Typhoeus::Request.get("https://mail.google.com/mail", :ssl_verifypeer => true)

If you are getting “SSL: certificate subject name does not match target host
name” from curl (ex:- you are trying to access to b.c.host.com when the
certificate subject is \*.host.com). You can disable host verification. Like
this:

    Typhoeus::Request.get("https://mail.google.com/mail", :ssl_verifyhost => false)

<!--
### LibCurl

Typhoeus also has a more raw libcurl interface. These are the Easy and Multi
objects. If you’re into accessing just the raw libcurl style, those are your
best bet.

However, by using this raw interface, you do not get access to Hydra-specific
features, such as stubbing/mocking.

SSL Certs can be provided to the Easy interface:

<script src="https://gist.github.com/2897980.js?file=libcurl_1.rb"></script>

or directly to a Typhoeus::Request :

<script src="https://gist.github.com/2897980.js?file=libcurl_2.rb"></script>

##  Advanced authentication

Thanks for the authentication piece and this description go to Oleg Ivanov
(morhekil). The major reason to start this fork was the need to perform NTLM
authentication in Ruby, and other libcurl’s authentications method were made
possible as a result. Now you can do it via Typhoeus::Easy interface using the
following API.

<script src="https://gist.github.com/2897980.js?file=advanced_auth.rb"></script>

### Other authentication types

The following authentication types are available:

  * CURLAUTH\_BASIC
  * CURLAUTH\_DIGEST
  * CURLAUTH\_GSSNEGOTIATE
  * CURLAUTH\_NTLM
  * CURLAUTH\_DIGEST\_IE
  * CURLAUTH\_AUTO

The last one (CURLAUTH\_AUTO) is really a combination of all previous methods
and is provided by Typhoeus for convenience. When you set authentication to
auto, Typhoeus will retrieve the given URL first and examine it’s headers to
confirm what auth types are supported by the server. The it will select the
strongest of available auth methods and will send the second request using the
selected authentication method.

### Authentication via the quick request interface

There’s also an easy way to perform any kind of authentication via the quick
request interface:

<script src="https://gist.github.com/2897980.js?file=auth_via_request_interface.rb"></script>

All methods listed above is available in a shorter form – :basic, :digest,
:gssnegotiate, :ntlm, :digest\_ie, :auto.

### Query of available auth types

After the initial request you can get the authentication types available on
the server via Typhoues::Easy#auth\_methods call. It will return a number

that you’ll need to decode yourself, please refer to easy.rb source code to
see the numeric values of different auth types.
-->

##  Verbose debug output

Sometime it’s useful to see verbose output from curl. You may now enable it:

    Typhoeus::Request.get("http://example.com", :verbose => true)

Just remember that libcurl prints it’s debug output to the console (to
STDERR), so you’ll need to run your scripts from the console to see it.

<!---
##  Benchmarks

I set up a benchmark to test how the parallel performance works vs Ruby’s
built in NET::HTTP. The setup was a local evented HTTP server that would take
a request, sleep for 500 milliseconds and then issued a blank response. I set
up the client to call this 20 times. Here are the results:

      net::http  0.030000   0.010000   0.040000 ( 10.054327)
      typhoeus   0.020000   0.070000   0.090000 (  0.508817)

We can see from this that NET::HTTP performs as expected, taking 10 seconds to
run 20 500ms requests. Typhoeus only takes 500ms (the time of the response
that took the longest.) One other thing to note is that Typhoeus keeps a pool
of libcurl Easy handles to use. For this benchmark I warmed the pool first. So
if you test this out it may be a bit slower until the Easy handle pool has
enough in it to run all the simultaneous requests. For some reason the easy
handles can take quite some time to allocate.
-->
