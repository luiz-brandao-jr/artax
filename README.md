
Woah, slow down, cowboy! **THIS IS THE DEV BRANCH** and this README has not yet been updated. It
reflects code from the master branch. If you want to know how to use this branch you need to check
out the [example files](https://github.com/rdlowrey/Artax/tree/dev/examples).


----------------------------------------------------------------------------------------------------


# Artax HTTP Client

Artax is a full-featured HTTP/1.1 client as specified in RFC 2616. Its API is designed to simplify
standards-compliant HTTP resource traversal and RESTful web service consumption without obscuring the
underlying HTTP protocol. The code manually implements the HTTP over TCP sockets; as such it has no
dependency on PHP's `curl_*` API and requires no non-standard PHP extensions.

> **HEY!** Checkout out the [EXAMPLES SECTION](https://github.com/rdlowrey/Artax/tree/dev/examples)
> to see more of what the Artax client can do.

#### FEATURES

 - Exposes non-blocking, parallel and synchronous APIs
 - Retains persistent "keep-alive" connections
 - Transparently follows redirects
 - Requests and decodes gzipped entity bodies
 - Provides access to all raw request/response headers and message data
 - Streams entity bodies for managing memory usage with large transfers
 - Supports all standard and custom request methods
 - Trivializes HTTP form submissions via the `FormBody` API
 - Provides fully customizable and secure-by-default TLS (https://) support
 - Offers advanced connection limiting options on a per-host basis
 - Transparently supports cookies and sessions
 - Transparently functions behind HTTP proxy servers
 - Eschews any cURL/libcurl dependency


#### PROJECT GOALS

* Model all code as closely as possible to the relevant HTTP protocols;
* Implement an HTTP/1.1 client built on raw sockets with no libcurl dependency;
* Build all components using [SOLID][solid], readable and tested code;

#### INSTALLATION

###### Git:

```bash
$ git clone --recursive https://github.com/rdlowrey/Artax.git
$ cd Artax
$ composer.phar install
```

> **NOTE:** The recursive option is necessary to pull in the SSL certificate bundle for encrypted *https://* communication.

###### Composer:

```bash
$ php composer.phar require rdlowrey/Artax:~0.8.0
```


#### REQUIREMENTS

* PHP 5.4+
* [Alert][alert-github]
* [After][after-github]
* `ext/openssl` PHP extension for TLS (https://) encryption support
* `ext/zlib` PHP extension for requesting/decompressing gzipped responses


#### ASYNC vs. SYNC

Artax offers two APIs for your HTTP needs:

- **Async:** `Artax\Client` is fully asynchronous and runs inside a non-blocking event loop.
The asynchronous client allows for full IO and computational parallelization. But with great power
comes great responsibility; an understanding of non-blocking IO is needed to effectively write code
using this paradigm.

- **Sync:** `Artax\BlcokingClient` exposes an easy-to-use synchronous wrapper around the non-blocking
client code. You can use it to execute requests invidually or in parallel, but its method calls are
always synchronous and will block to completion.




@TODO
----------------------------------------------------------------------------------------------------
Haven't updated anything below here yet




#### BASIC USAGE

###### Simple GET Request

Often we only care about simple GET retrieval. For such cases Artax accepts a basic HTTP URI as the
request parameter:

```php
<?php

$client = new Artax\Client;
$response = $client->request('http://www.google.com');

echo "Response status code: ", $response->getStatus(), "\n";
echo "Response reason:      ", $response->getReason(), "\n";
echo "Response protocol:    ", $response->getProtocol(), "\n";

print_r($response->getAllHeaders());

echo $response->getBody();
```

###### Customized Request Message

For non-trivial requests Artax allows you to construct messages piece-by-piece. This example
sets the request method to POST and assigns an entity body. HTTP veterans will notice that
we don't bother to set a `Content-Length` or `Host` header. Aerys will automatically add/normalize
missing headers for us so we don't need to worry about it. The only property that _MUST_ be assigned
when sending an `Artax\Request` is the absolute *http://* or *https://* request URI:

```php
<?php

$client = new Artax\Client;
$request = new Artax\Request;
$request->setUri('http://httpbin.org/post');
$request->setProtocol('1.1');
$request->setMethod('POST');
$request->setAllHeaders([
    'Content-Type' => 'text/plain; charset=utf-8',
    'Cookie' => ['Cookie1=val1', 'Cookie2=val2']
]);
$request->setBody('woot!');

$response = $client->request($request);
```

In the above example the raw request message sent to the server will look like this:

```
POST /post HTTP/1.1
Host: httpbin.org
Content-Length: 5
Content-Type: text/plain; charset=utf-8
Cookie: Cookie1=val1
Cookie: Cookie2=val2

woot!
```

###### Form Submission

Assume that `httpbin.org/post` contains the following HTML form:

```html
<form action="http://httpbin.org/post" enctype="multipart/form-data" method="post">
   <P>
   What is your name? <input type="text" name="name"><br>
   What files are you sending?<br>
   <input type="file" name="file1"><br>
   <input type="file" name="file2"><br>
   <input type="submit" value="Send">
 </form>
```

We can easily submit this form using the `Artax\FormBody` API:

```php
<?php

$body = new Artax\FormBody;

$body->addField('name', 'Zoroaster');
$body->addFileField('file1', '/hard/path/to/some/file1');
$body->addFileField('file2', '/hard/path/to/some/file2');

$client = new Artax\Client;
$request = new Artax\Request;
$request->setBody($body);
$request->setUri('http://httpbin.org/post');
$request->setMethod('POST');

$response = $client->request($request);
```

It's *important* to note that `Artax\FormBody` will **stream** any file fields you specify as
part of your form submission. This means you can easily upload huge files without issue.

###### Synchronous Parallel Requests

The synchronous `Artax\Client::requestMulti` retrieves multiple requests in parallel and alerts
the relevant callbacks each time a request in the batch completes:

```php
<?php

$client = new Artax\Client;

$onResponse = function($requestKey, Artax\Response $response) {
    echo 'Response: (', $requestKey, ') ', $response->getStatus(), "\n";
};
$onError = function($requestKey, Exception $error) {
    echo 'Error: (', $requestKey, ') ', $error->getMessage(), "\n";
};
$requests = [
    'google' => 'http://www.google.com',
    'google news' => 'http://news.google.com',
    'bing' => 'http://www.bing.com',
    'yahoo' => 'http://www.yahoo.com',
    'php' => 'http://www.php.net'
];

$client->requestMulti($requests, $onResponse, $onError);
```

Note that though the individual requests in the `$requests` batch are retrieved in parallel the
`Client::requestMulti` call itself will block until all of the requests complete (or error out).

###### Still Want More?

Check out [more examples](https://github.com/rdlowrey/Artax/tree/master/examples) demonstrating
features such as cookies, progress bars, TLS support, asynchronous requests, etc ...


#### WHAT'S WITH THE NAME?

Children of the 1980s are likely familiar with [The NeverEnding Story][neverending] and may remember
the scene where Atreyu's faithful steed, Artax, died in the Swamp of Sadness. The name is an homage.


[rfc2616]: http://www.w3.org/Protocols/rfc2616/rfc2616.html
[alert-github]: https://github.com/rdlowrey/Alert
[after-github]: https://github.com/rdlowrey/After
[solid]: http://en.wikipedia.org/wiki/SOLID_(object-oriented_design) "S.O.L.I.D."
[neverending]: http://www.imdb.com/title/tt0088323/ "The NeverEnding Story"

