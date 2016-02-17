[![Latest Stable Version](https://poser.pugx.org/serendipity_hq/oauth-guzzle-middleware/v/stable)](https://packagist.org/packages/serendipity_hq/oauth-guzzle-middleware)
[![Build Status](https://travis-ci.org/SerendipityHQ/oauth-guzzle-middleware.svg?branch=master)](https://travis-ci.org/SerendipityHQ/oauth-guzzle-middleware)
[![Total Downloads](https://poser.pugx.org/serendipity_hq/oauth-guzzle-middleware/downloads)](https://packagist.org/packages/serendipity_hq/oauth-guzzle-middleware)
[![License](https://poser.pugx.org/serendipity_hq/oauth-guzzle-middleware/license)](https://packagist.org/packages/serendipity_hq/oauth-guzzle-middleware)
[![Code Climate](https://codeclimate.com/github/SerendipityHQ/oauth-guzzle-middleware/badges/gpa.svg)](https://codeclimate.com/github/SerendipityHQ/oauth-guzzle-middleware)
[![Test Coverage](https://codeclimate.com/github/SerendipityHQ/oauth-guzzle-middleware/badges/coverage.svg)](https://codeclimate.com/github/SerendipityHQ/oauth-guzzle-middleware/coverage)
[![Issue Count](https://codeclimate.com/github/SerendipityHQ/oauth-guzzle-middleware/badges/issue_count.svg)](https://codeclimate.com/github/SerendipityHQ/oauth-guzzle-middleware)
[![StyleCI](https://styleci.io/repos/51864555/shield)](https://styleci.io/repos/51864555)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/60961656-1bd1-40dd-9b37-2f9418e3bc1f/mini.png)](https://insight.sensiolabs.com/projects/60961656-1bd1-40dd-9b37-2f9418e3bc1f)
[![Dependency Status](https://www.versioneye.com/user/projects/56c38a3b18b2710036c8dee1/badge.svg?style=flat)](https://www.versioneye.com/user/projects/56c38a3b18b2710036c8dee1)

Guzzle 6+ OAuth Middleware
===============================

Signs HTTP requests using OAuth.

This version only works with Guzzle 6.0 and up!

(Forked from https://github.com/guzzle/oauth-subscriber and separated from original repo to be able to use continuous
integration server and other analysis tools).

Installing
==========

**Requires [PHP OAuth extension](http://php.net/manual/en/book.oauth.php).**

This project can be installed using Composer. Add the following to your
`composer.json`:

    {
        "require": {
            "serendipity_hq/oauth-guzzle-middleware": "~0.1"
        }
    }

# About OAuth
From [OAuth Core 1.0a](http://oauth.net/core/1.0a/):

>The OAuth protocol enables websites or applications (Consumers) to access Protected Resources from a web service
 (Service Provider) via an API, without requiring Users to disclose their Service Provider credentials to the Consumers.
 More generally, OAuth creates a freely-implementable and generic methodology for API authentication.
>
>An example use case is allowing printing service printer.example.com (the Consumer), to access private photos stored on
 photos.example.net (the Service Provider) without requiring Users to provide their photos.example.net credentials to
 printer.example.com.
>
>OAuth does not require a specific user interface or interaction pattern, nor does it specify how Service Providers
 authenticate Users, making the protocol ideally suited for cases where authentication credentials are unavailable to
 the Consumer, such as with OpenID.
>
>OAuth aims to unify the experience and implementation of delegated web service authentication into a single,
 community-driven protocol. OAuth builds on existing protocols and best practices that have been independently
 implemented by various websites. An open standard, supported by large and small providers alike, promotes a consistent
 and trusted experience for both application developers and the users of those applications.

## OAuth Versions

- [2007] OAuth Core 1.0 ([DEPRECATED] Community - [OAuth.net](http://oauth.net/core/1.0/))
- [2009] OAuth Core 1.0a (Community - [OAuth.net](http://oauth.net/core/1.0a/))
- [2010] OAuth Protocol 1.0 (Informational - [RFC5849](http://tools.ietf.org/html/rfc5849))
- [2012] OAuth Protocol 2.0 (Standards Track - [RFC6749](https://tools.ietf.org/html/rfc6749))

## This library supports

- OAuth Core 1.0a
- OAuth Protocol 1.0 (IS THE SAME IDENTICAL CLASS USED FOR OAuth Core 1.0a - Sorry :( ... for the moment :))
- OAuth Protocol 2.0 (to implement)

# How to use the OAuth Core 1.0a Middleware

You can send both 2-legged and 3-legged `Requests` by passing or not the `token` and `token_secret` values to the
`OAuth*` middleware.

See the [Terminology paragraph on hueniverse.com](http://hueniverse.com/oauth/guide/terminology/) for more information
about n-legged requests.

## Send a 3-legged `Request`

```php
/**
 * Sends ALL THE REQUESTS authenticated with OAuth.
 * @see [docs/examples/Twitter/](docs/examples/Twitter/) for more details.
 */

use GuzzleHttp\Client;
use GuzzleHttp\Handler\CurlHandler;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware\OpenAuthentication\OAuth10a;
use GuzzleHttp\RequestOptions;

// Consumer Credentials: created at Precondition 2
$consumerKey    = 'your-consumer-key';
$consumerSecret = 'your-consumer-secret';

// Set here the access token generated at Precondition 3
$accessTokenKey    = 'your-token-key';
$accessTokenSecret = 'your-token-secret';

// The home_timeline endpoint
$resourceUrl = 'statuses/home_timeline.json';

try {
    // Instantiate the Guzzle Client and the OAuth10a middleware
    $handler = new CurlHandler();
    $stack = HandlerStack::create($handler);
    $middleware = new OAuth10a([
        'consumer_key'     => $consumerKey,
        'consumer_secret'  => $consumerSecret,
        'token'            => $accessTokenKey,
        'token_secret'     => $accessTokenSecret,
        'request_method'   => OAuth10a::REQUEST_METHOD_HEADER,
        'signature_method' => OAuth10a::SIGNATURE_METHOD_HMAC,
    ]);
    $stack->push($middleware);

    // Set the client params
    $clientParams = [
        'base_uri'                      => 'https://api.twitter.com/1.1/',
        'handler'                       => $stack,
        // Set the oauth authentication for ALL REQUESTS
        RequestOptions::AUTH            => 'oauth',
        RequestOptions::HTTP_ERRORS     => false,
        RequestOptions::DEBUG           => true,
        RequestOptions::ALLOW_REDIRECTS => ['track_redirects' => true]
    ];

    $guzzleClient = new Client($clientParams);

    $tweetsList = $guzzleClient->get($resourceUrl);
    dump(json_decode($tweetsList->getBody()->__toString(), true));
} catch (\Exception $e) {
    dump($e->getMessage());
    echo "&lt;br/&gt;";
    dump($e);
}
```

To authenticate only a single `Request` with OAuth middleware, remove the `RequestOptions::AUTH => 'oauth'` option from
the `$clientParams` array and put it into a `$requestParams` array:

```php
/**
 * @see [docs/examples/Twitter/](docs/examples/Twitter/) for more details.
 */

...

try {
    ...
    $stack->push($middleware);

    // Set the client params
    $clientParams = [
        'base_uri'                      => 'https://api.twitter.com/1.1/',
        'handler'                       => $stack,
        // Remove the options from the CLient parameters
        // RequestOptions::AUTH            => 'oauth',
        RequestOptions::HTTP_ERRORS     => false,
        RequestOptions::DEBUG           => true,
        RequestOptions::ALLOW_REDIRECTS => ['track_redirects' => true]
    ];

    $guzzleClient = new Client($clientParams);

    $requestParams = [
        RequestOptions::AUTH => 'oauth'
    ];

    // Use OAuth on a pre Request basis
    $tweetsList = $guzzleClient->get($resourceUrl, $requestParams);

    dump(json_decode($tweetsList->getBody()->__toString(), true));
} catch (\Exception $e) {
    dump($e->getMessage());
    echo "&lt;br/&gt;";
    dump($e);
}
```

## Send a 2-legged `Request`

To send 2-legged `Requests` simply omit `token` and `token_secret` parameters.

```php
/**
 * Sends ALL THE REQUESTS authenticated with OAuth.
 * @see [docs/examples/Twitter/](docs/examples/Twitter/) for more details.
 */

use GuzzleHttp\Client;
use GuzzleHttp\Handler\CurlHandler;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware\OpenAuthentication\OAuth10a;
use GuzzleHttp\RequestOptions;

// Consumer Credentials: created at Precondition 2
$consumerKey    = 'your-consumer-key';
$consumerSecret = 'your-consumer-secret';

// DO NOT SET THOSE PARAMETERS
// $accessTokenKey    = 'your-token-key';
// $accessTokenSecret = 'your-token-secret';

// The home_timeline endpoint
$resourceUrl = 'statuses/home_timeline.json';

try {
    // Instantiate the Guzzle Client and the OAuth10a middleware
    $handler = new CurlHandler();
    $stack = HandlerStack::create($handler);
    $middleware = new OAuth10a([
        'consumer_key'     => $consumerKey,
        'consumer_secret'  => $consumerSecret,
        // DO NOT SET THOSE PARAMETERS
        // 'token'            => $accessTokenKey,
        // 'token_secret'     => $accessTokenSecret,
        'request_method'   => OAuth10a::REQUEST_METHOD_HEADER,
        'signature_method' => OAuth10a::SIGNATURE_METHOD_HMAC,
    ]);
    $stack->push($middleware);

    // Set the client params
    $clientParams = [
        'base_uri'                      => 'https://api.twitter.com/1.1/',
        'handler'                       => $stack,
        // Set the oauth authentication for ALL REQUESTS
        RequestOptions::AUTH            => 'oauth',
        RequestOptions::HTTP_ERRORS     => false,
        RequestOptions::DEBUG           => true,
        RequestOptions::ALLOW_REDIRECTS => ['track_redirects' => true]
    ];

    $guzzleClient = new Client($clientParams);

    $tweetsList = $guzzleClient->get($resourceUrl);
    dump(json_decode($tweetsList->getBody()->__toString(), true));
} catch (\Exception $e) {
    dump($e->getMessage());
    echo "&lt;br/&gt;";
    dump($e);
}
```

Using the RSA-SH1 signature method
==================================

```php

    use GuzzleHttp\Middleware\OpenAuthentication\Oauth10a;

    $stack = HandlerStack::create();

    $middleware = new Oauth10a([
        'consumer_key'           => 'my_key',
        'consumer_secret'        => 'my_secret',
        'private_key_file'       => 'my_path_to_private_key_file',
        'private_key_passphrase' => 'my_passphrase',
        'signature_method'       => Oauth10a::SIGNATURE_METHOD_RSA,
    ]);
    $stack->push($middleware);

    $client = new Client([
        'handler' => $stack
    ]);

    $response = $client->get('http://httpbin.org', ['auth' => 'oauth']);
```
