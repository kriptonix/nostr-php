# nostr-php

![CI](https://github.com/swentel/nostr-php/actions/workflows/ci.yml/badge.svg)
![Packagist PHP Version](https://img.shields.io/packagist/dependency-v/swentel/nostr-php/php)
![GitHub contributors](https://img.shields.io/github/contributors/swentel/nostr-php)
![GitHub issues](https://img.shields.io/github/issues/swentel/nostr-php)
![GitHub last commit (branch)](https://img.shields.io/github/last-commit/swentel/nostr-php/main)

This is a PHP Helper library for Nostr.
More info about Nostr: https://github.com/nostr-protocol/nostr.

## Installation

To use in your project with Composer:

```console
$ composer require swentel/nostr-php
```

Install dependencies if you would like to test / code some things out for yourself with the code example snippets below. 

```console
$ composer install
```

## Create an event

This will create an event object with a short text message (kind 1).

```php
use swentel\nostr\Event\Event;

$note = new Event();
$note->setKind(1);
$note->setContent('Hello world!');
$note->setTags([
  ['e', $relayUrl],
  ['p', $public_key, $relayUrl],
  ['r', $relayUrl],
]);
// or use addTag()
$note->addTag(['p', $public_key, $relayUrl]);
```

## Signing an event

Generates the id and signature for an event. The 'pubkey', 'id' and 'sig' 
properties are added to the event object.

```php
use swentel\nostr\Event\Event;
use swentel\nostr\Sign\Sign;

$note = new Event();
$note->setContent('Hello world!');
$note->setKind(1);

$signer = new Sign();
$signer->signEvent($note, $private_key);
```

## Generating a message

Generate an event message : `["EVENT", <event JSON as created above with id and sig>]`

```php
use swentel\nostr\Sign\Sign;
use swentel\nostr\Message\EventMessage;

$signer = new Sign();
$signer->signEvent($note, $private_key);

$eventMessage = new EventMessage($note);
$message_string = $eventMessage->generate();
```

## Publish an event to a relay

Publish an event with a note that has been prepared for sending to a relay.

```php
use swentel\nostr\Event\Event;
use swentel\nostr\Message\EventMessage;
use swentel\nostr\Relay\Relay;

$note = new Event();
$note->setContent('Hello world');
$note->setKind(1);

$signer = new Sign();
$signer->signEvent($note, $private_key);

$eventMessage = new EventMessage($note);

$relayUrl = 'wss://nostr-websocket.tld';
$relay = new Relay($relayUrl, $eventMessage);
$result = $relay->send();
```

## Read events from a relay

Fetch events from a relay. 

```php
$subscription = new Subscription();
$subscriptionId = $subscription->setId();

$filter1 = new Filter();
$filter1->setKinds([1, 3]); // You can add multiple kind numbers
$filter1->setLimit(25); // Limit to fetch only a maximum of 25 events
$filters = [$filter1]; // You can add multiple filters.

$requestMessage = new RequestMessage($subscriptionId, $filters);

$relayUrl = 'wss://nostr-websocket.tld';
$relay = new Relay($relayUrl);
$relay->setMessage($requestMessage);

$request = new Request($relayUrl, $requestMessage);
$response = $request->send();
```

`$response` is an multidimensional array with elements containing each a response message (JSON string) decoded to an array from the relay and sorted by the relay.
Output example:
```php
[
  'wss://nostr-websocket.tld' => [
    0 => [
      "EVENT",
      "A8kWzjCVUHSD1rmuwGqyK2PxsolZMO9YXditbg05fch6p3Q4eT7vRFLEJINBna",
      [
        'id' => '1e8534623845629d40f7761c0577edf10f778c490e7b95a524845d9280c7c25a',
        'kind' => 1,
        'pubkey' => '06639a386c9c1014217622ccbcf40908c4f1a0c33e23f8d6d68f4abf655f8f71',
        'created_at' => 1718723787,
        'content' => 'Losing your social graph can feel the same for some I think 😮 ',
        'tags' => [
          ['e', 'f754a238947b7f32168f872650a8dd0b9376493e58005d7e0b8be52f6f229364', 'wss://nos.lol/', 'root'],
          ['e', 'fe7dd6ba22fa0aa39370aa160226b8bc2413460621c8d67ce862205ad5a02c24', 'wss://nos.lol/', 'reply'],
          ['p', 'fb1366abd5e4c92a8a950791bc72d51bde291a83555cb2c629a92fedd78068ac', '', 'mention']
        ],
        'sig' => '888c9b5d9e0b69eba3510dd2b5d03eddcf0a680ab0e7673820fb36a56448ad80701042a669c7ef9918593c5a41c8b3ccc1d82ade50f32b62dd843144f32df403'
    ],
    1 => [
      "EVENT",
      "A8kWzjCVUHSD1rmuwGqyK2PxsolZMO9YXditbg05fch6p3Q4eT7vRFLEJINBna",
      [
        ...Nostr event
      ]
    ],
    2 => [
      ...
    ],
    3 => [
      ...
    ],
    4 => [
      ...
    ]
  ]
]

```

## Read events from a set of relays

Read events from a set of relays with the `RelaySet` class.
It's basically the same snippet as above with the difference you create a `RelaySet` class and pass it through the `Request` object.

```php
$subscription = new Subscription();
$subscriptionId = $subscription->setId();

$filter1 = new Filter();
$filter1->setKinds([1]);
$filter1->setLimit(5);
$filters = [$filter1];
$requestMessage = new RequestMessage($subscriptionId, $filters);
$relays = [
    new Relay('wss://nostr-websocket-1.tld'),
    new Relay('wss://nostr-websocket-2.tld'),
    new Relay('wss://nostr-websocket-3.tld'),
];
$relaySet = new RelaySet();
$relaySet->setRelays($relays);

$request = new Request($relaySet, $requestMessage);
$response = $request->send();
```

## Generating a private key and a public key

```php
use swentel\nostr\Key\Key;

$key = new Key();

$private_key = $key->generatePrivateKey();
$public_key  = $key->getPublicKey($private_key);

```

## Converting keys

Convert bech32 encoded keys (npub, nsec) to hex.

```php
use swentel\nostr\Key\Key;

$public_key = 'npub10elfcs4fr0l0r8af98jlmgdh9c8tcxjvz9qkw038js35mp4dma8qzvjptg';
$key = new Key();
$hex = $key->convertToHex($public_key);
```

Convert hex keys to bech32 (npub, nsec).

```php
use swentel\nostr\Key\Key;

$public_key = '7e7e9c42a91bfef19fa929e5fda1b72e0ebc1a4c1141673e2794234d86addf4e';
$private_key = '67dea2ed018072d675f5415ecfaed7d2597555e202d85b3d65ea4e58d2d92ffa';
$key = new Key();
$bech32_public = $key->convertPublicKeyToBech32($public_key);
$bech32_private = $key->convertPrivateKeyToBech32($private_key);
```

## Run tests

All tests can be found in `tests`.

```console
$ php vendor/bin/phpunit
```

## nostr-php script (cli client)

The library ships with a simple CLI client (`bin/nostr-php`) to post a short text note to a Nostr relay.

```console
Usage:
$ bin/nostr-php --content "Hello world!" --key /home/path/to/nostr-private.key --relay wss://nostr.pleb.network
```

Note: the key arguments expects a file with your private key! Do not paste your
private key on command line.

## Roadmap

- [x] Keypair generation and validation
  - [x] Convert from hex to bech32-encoded keys
- [x] Event signing with Schnorr signatures (`secp256k1`)
- [x] Event validation (issue [#17](https://github.com/swentel/nostr-php/issues/17))
- [ ] Support NIP-01 basic protocol flow description
  - [x] Publish events
  - [x] Request events (issue [#55](https://github.com/nostrver-se/nostr-php/pull/55) credits to [kriptonix](https://github.com/kriptonix))
  - [ ] Implement all types of relay responses
    - [ ] EVENT - sends events requested by the client
    - [ ] OK - indicate an acceptance or denial of an EVENT message
    - [ ] EOSE - end of stored events
    - [ ] CLOSED - subscription is ended on the server side
    - [ ] NOTICE - used to send human-readable messages (like errors) to clients
- [ ] Improve handling relay responses
- [ ] Support NIP-19 bech32-encoded identifiers
- [ ] Support NIP-42 authentication of clients to relays
- [ ] Support NIP-45 event counts
- [ ] Support NIP-50 search capability
- [ ] Support multi-threading (async concurrency) for handling requests simultaneously
- [ ] Support realtime (runtime) subscriptions with the `bin/nostr-php` CLI client to listen to new events from relays

## Community

If you need any help, please join this Telegram group: https://t.me/nostr_php

## Funding

In May 2024 OpenSats granted Sebastian Hagens for further development of this library for one year. If you would like to support this project with a donation, you could send some lightning sats to `sebastian@lnd.sebastix.com` or on-chain to `bc1p3p6jq2sxsf650lgllv57st9h97xj37fflg5t8d265saz6yqzcdyqd7pzun`. 

## Maintainers
 
* [@sebastix](https://github.com/Sebastix)  `npub1qe3e5wrvnsgpggtkytxteaqfprz0rgxr8c3l34kk3a9t7e2l3acslezefe`
* [@swentel](https://github.com/swentel) (original author, inactive)  `npub1z8n2zt0vzkefhrhpf60face4wwq2nx87sz7wlgcvuk4adddkkycqknzjk5`
