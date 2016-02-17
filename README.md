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

Guzzle 6+ OAuth 1.0a Middleware
===============================

Signs HTTP requests using OAuth 1.0. Requests are signed using a consumer key,
consumer secret, OAuth token, and OAuth secret.

This version only works with Guzzle 6.0 and up!

(Forked from https://github.com/guzzle/oauth-subscriber and separated from original repo to be able to use continuous
integration server and other analysis tools).

Installing
==========

This project can be installed using Composer. Add the following to your
composer.json:

.. code-block:: javascript

    {
        "require": {
            "serendipity_hq/oauth-guzzle-middleware": "~0.1"
        }
    }



Using the Middleware
====================

Here's an example showing how to send an authenticated request to the Twitter
REST API:

.. code-block:: php

    use GuzzleHttp\Client;
    use GuzzleHttp\HandlerStack;
    use GuzzleHttp\Middleware\OpenAuthentication\Oauth10a;

    $stack = HandlerStack::create();

    $middleware = new Oauth10a([
        'consumer_key'    => 'my_key',
        'consumer_secret' => 'my_secret',
        'token'           => 'my_token',
        'token_secret'    => 'my_token_secret'
    ]);
    $stack->push($middleware);

    $client = new Client([
        'base_uri' => 'https://api.twitter.com/1.1/',
        'handler' => $stack
    ]);

    // Set the "auth" request option to "oauth" to sign using oauth
    $res = $client->get('statuses/home_timeline.json', ['auth' => 'oauth']);

You can set the ``auth`` request option to ``oauth`` for all requests sent by
the client by extending the array you feed to ``new Client`` with auth => oauth.

.. code-block:: php

    use GuzzleHttp\Client;
    use GuzzleHttp\HandlerStack;
    use GuzzleHttp\Middleware\OpenAuthentication\Oauth10a;

    $stack = HandlerStack::create();

    $middleware = new Oauth10a([
        'consumer_key'    => 'my_key',
        'consumer_secret' => 'my_secret',
        'token'           => 'my_token',
        'token_secret'    => 'my_token_secret'
    ]);
    $stack->push($middleware);

    $client = new Client([
        'base_uri' => 'https://api.twitter.com/1.1/',
        'handler' => $stack,
        'auth' => 'oauth'
    ]);

    // Now you don't need to add the auth parameter
    $res = $client->get('statuses/home_timeline.json');

.. note::

    You can omit the token and token_secret options to use two-legged OAuth.

Using the RSA-SH1 signature method
==================================

.. code-block:: php

    use GuzzleHttp\Middleware\OpenAuthentication\Oauth10a;

    $stack = HandlerStack::create();

    $middleware = new Oauth10a([
        'consumer_key'    => 'my_key',
        'consumer_secret' => 'my_secret',
        'private_key_file' => 'my_path_to_private_key_file',
        'private_key_passphrase' => 'my_passphrase',
        'signature_method' => Oauth10a::SIGNATURE_METHOD_RSA,
    ]);
    $stack->push($middleware);

    $client = new Client([
        'handler' => $stack
    ]);

    $response = $client->get('http://httpbin.org', ['auth' => 'oauth']);
