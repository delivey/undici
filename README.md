# undici

[![Node CI](https://github.com/nodejs/undici/actions/workflows/nodejs.yml/badge.svg)](https://github.com/nodejs/undici/actions/workflows/nodejs.yml) [![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](http://standardjs.com/) [![npm version](https://badge.fury.io/js/undici.svg)](https://badge.fury.io/js/undici) [![codecov](https://codecov.io/gh/nodejs/undici/branch/main/graph/badge.svg?token=yZL6LtXkOA)](https://codecov.io/gh/nodejs/undici)

A HTTP/1.1 client, written from scratch for Node.js.

> Undici means eleven in Italian. 1.1 -> 11 -> Eleven -> Undici.
It is also a Stranger Things reference.

Have a question about using Undici? Open a [Q&A Discussion](https://github.com/nodejs/undici/discussions/new) or join our official OpenJS [Slack](https://openjs-foundation.slack.com/archives/C01QF9Q31QD) channel.

## Install

```
npm i undici
```

## Benchmarks

AMD EPYC 7502P 32 Core, Node 15

The benchmark is a simple `hello world` [example](benchmarks/index.js) using a
number of unix sockets (connections) with a pipelining depth of 10.

### Connections 1

| Slower tests        | Samples |        Result  | Tolerance | Difference with slowest |
|---------------------|---------|----------------|-----------|-------------------------|
| http - no keepalive |      99 |   812 reqs/sec |  ± 0.22 % |                         |
| http - keepalive    |      99 |   819 reqs/sec |  ± 0.20 % | + 0.82 %                |
| undici - pipeline   |      99 |  6632 reqs/sec |  ± 0.63 % | + 716.73 %              |
| undici - request    |      99 |  6645 reqs/sec |  ± 1.34 % | + 718.34 %              |
| undici - stream     |      99 |  7366 reqs/sec |  ± 0.59 % | + 807.11 %              |
| undici - dispatch   |      99 |  7404 reqs/sec |  ± 0.37 % | + 811.76 %              |

### Connections 50

| Slower tests        | Samples |        Result  | Tolerance | Difference with slowest |
|---------------------|---------|----------------|-----------|-------------------------|
| http - no keepalive |      99 | 12968 reqs/sec |  ± 1.86 % |                         |
| http - keepalive    |      99 | 14745 reqs/sec |  ± 1.59 % | + 13.70 %               |
| undici - pipeline   |      99 | 20051 reqs/sec |  ± 2.34 % | + 54.62 %               |
| undici - stream     |     100 | 26456 reqs/sec |  ± 3.50 % | + 104.00 %              |
| undici - request    |      99 | 29342 reqs/sec |  ± 1.26 % | + 126.26 %              |
| undici - dispatch   |      99 | 35323 reqs/sec |  ± 0.77 % | + 172.38 %              |

## Quick Start

```js
import { request } from 'undici'

const {
  statusCode,
  headers,
  trailers,
  body
} = await request('http://localhost:3000/foo')

console.log('response received', statusCode)
console.log('headers', headers)

for await (const data of body) {
  console.log('data', data)
}

console.log('trailers', trailers)
```

## Common API Methods

This section documents our most commonly used API methods. Additional APIs are documented in their own files within the [docs](./docs/) folder and are accessible via the navigation list on the left side of the docs site.

### `undici.request([url, options]): Promise`

Arguments:

* **url** `string | URL | object`
* **options** [`RequestOptions`](./docs/api/Dispatcher.md#parameter-requestoptions)
  * **dispatcher** `Dispatcher` - Default: [getGlobalDispatcher](#undicigetglobaldispatcherdispatcher)
  * **method** `String` - Default: `PUT` if `options.body`, otherwise `GET`
  * **maxRedirections** `Integer` - Default: `0`

Returns a promise with the result of the `Dispatcher.request` method.

Calls `options.dispatcher.request(options)`.

See [Dispatcher.request](./docs/api/Dispatcher.md#dispatcherrequestoptions-callback) for more details.

### `undici.stream([url, options, ]factory): Promise`

Arguments:

* **url** `string | URL | object`
* **options** [`StreamOptions`](./docs/api/Dispatcher.md#parameter-streamoptions)
  * **dispatcher** `Dispatcher` - Default: [getGlobalDispatcher](#undicigetglobaldispatcherdispatcher)
  * **method** `String` - Default: `PUT` if `options.body`, otherwise `GET`
  * **maxRedirections** `Integer` - Default: `0`
* **factory** `Dispatcher.stream.factory`

Returns a promise with the result of the `Dispatcher.stream` method.

Calls `options.dispatcher.stream(options, factory)`.

See [Dispatcher.stream](docs/api/Dispatcher.md#dispatcherstream) for more details.

### `undici.pipeline([url, options, ]handler): Duplex`

Arguments:

* **url** `string | URL | object`
* **options** [`PipelineOptions`](docs/api/Dispatcher.md#parameter-pipelineoptions)
  * **dispatcher** `Dispatcher` - Default: [getGlobalDispatcher](#undicigetglobaldispatcherdispatcher)
  * **method** `String` - Default: `PUT` if `options.body`, otherwise `GET`
  * **maxRedirections** `Integer` - Default: `0`
* **handler** `Dispatcher.pipeline.handler`

Returns: `stream.Duplex`

Calls `options.dispatch.pipeline(options, handler)`.

See [Dispatcher.pipeline](docs/api/Dispatcher.md#dispatcherpipeline) for more details.

### `undici.connect([url, options]): Promise`

Starts two-way communications with the requested resource using [HTTP CONNECT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT).

Arguments:

* **url** `string | URL | object`
* **options** [`ConnectOptions`](docs/api/Dispatcher.md#parameter-connectoptions)
  * **dispatcher** `Dispatcher` - Default: [getGlobalDispatcher](#undicigetglobaldispatcherdispatcher)
  * **maxRedirections** `Integer` - Default: `0`
* **callback** `(err: Error | null, data: ConnectData | null) => void` (optional)

Returns a promise with the result of the `Dispatcher.connect` method.

Calls `options.dispatch.connect(options)`.

See [Dispatcher.connect](docs/api/Dispatcher.md#dispatcherconnect) for more details.

### `undici.upgrade([url, options]): Promise`

Upgrade to a different protocol. See [MDN - HTTP - Protocol upgrade mechanism](https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism) for more details.

Arguments:

* **url** `string | URL | object`
* **options** [`UpgradeOptions`](docs/api/Dispatcher.md#parameter-upgradeoptions)
  * **dispatcher** `Dispatcher` - Default: [getGlobalDispatcher](#undicigetglobaldispatcherdispatcher)
  * **maxRedirections** `Integer` - Default: `0`
* **callback** `(error: Error | null, data: UpgradeData) => void` (optional)

Returns a promise with the result of the `Dispatcher.upgrade` method.

Calls `options.dispatcher.upgrade(options)`.

See [Dispatcher.upgrade](docs/api/Dispatcher.md#dispatcherupgradeoptions-callback) for more details.

### `undici.setGlobalDispatcher(dispatcher)`

* dispatcher `Dispatcher`

Sets the global dispatcher used by Common API Methods.

### `undici.getGlobalDispatcher()`

Gets the global dispatcher used by Common API Methods.

Returns: `Dispatcher`

## Specification Compliance

This section documents parts of the HTTP/1.1 specification which Undici does
not support or does not fully implement.

### Expect

Undici does not support the `Expect` request header field. The request
body is  always immediately sent and the `100 Continue` response will be
ignored.

Refs: https://tools.ietf.org/html/rfc7231#section-5.1.1

### Pipelining

Uncidi will only use pipelining if configured with a `pipelining` factor
greater than `1`.

Undici always assumes that connections are persistent and will immediately
pipeline requests, without checking whether the connection is persistent.
Hence, automatic fallback to HTTP/1.0 or HTTP/1.1 without pipelining is
not supported.

Undici will immediately pipeline when retrying requests afters a failed
connection. However, Undici will not retry the first remaining requests in
the prior pipeline and instead error the corresponding callback/promise/stream.

Undici will abort all running requests in the pipeline when any of them are
aborted.

* Refs: https://tools.ietf.org/html/rfc2616#section-8.1.2.2
* Refs: https://tools.ietf.org/html/rfc7230#section-6.3.2

## Collaborators

* [__Ethan Arrowood__](https://github.com/ethan-arrowood), <https://www.npmjs.com/~ethan_arrowood>
* [__Daniele Belardi__](https://github.com/dnlup), <https://www.npmjs.com/~dnlup>
* [__Tomas Della Vedova__](https://github.com/delvedor), <https://www.npmjs.com/~delvedor>
* [__Matteo Collina__](https://github.com/mcollina), <https://www.npmjs.com/~matteo.collina>
* [__Robert Nagy__](https://github.com/ronag), <https://www.npmjs.com/~ronag>

### Releasers

* [__Matteo Collina__](https://github.com/mcollina), <https://www.npmjs.com/~matteo.collina>
* [__Robert Nagy__](https://github.com/ronag), <https://www.npmjs.com/~ronag>

## License

MIT
