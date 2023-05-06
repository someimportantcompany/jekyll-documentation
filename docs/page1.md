---
title: http-assert-plus

header:
  title: Jekyll Documentation
sidebar:
  Documentation:
    - label: jekyll-documentation
      url: /
    - label: http-assert-plus
      url: /docs/page1/
    - label: jsonapi-resolvers
      url: /docs/page2/
footer:
  content: >-
    Copyright © 2023 <a href="https://someimportantcompany.github.io" target="_blank">Some Important Company</a>.
    <a href="https://github.com/someimportantcompany/jekyll-documentation/blob/master/LICENSE" target="_blank">MIT License</a>.
  gh-edit-link:
    repository: someimportantcompany/http-assert-plus
    branch: master
    view: tree
    path: README.md
---

# http-assert-plus

[![NPM](https://badge.fury.io/js/http-assert-plus.svg)](https://npm.im/http-assert-plus)
[![CI](https://github.com/someimportantcompany/http-assert-plus/actions/workflows/ci.yml/badge.svg?branch=master&event=push)](https://github.com/someimportantcompany/http-assert-plus/actions/workflows/ci.yml)
[![Typescript](https://img.shields.io/badge/TS-TypeScript-%230074c1.svg)](https://www.typescriptlang.org)
<!-- [![Coverage](https://coveralls.io/repos/github/someimportantcompany/http-assert-plus/badge.svg)](https://coveralls.io/github/someimportantcompany/http-assert-plus) -->

More assertions with status codes.

```js
const assert = require('http-assert-plus');
// or
import assert from 'http-assert-plus';

const username = 'jdrydn';
assert(username === 'not-jdrydn', 403, 'Authorization failed', {
  code: 'NOT_AUTHORIZED',
  username,
});

// Error: Authorization failed
//     at http-assert-plus/README.md:15:7 {
//   code: 'NOT_AUTHORIZED',
//   statusCode: 403,
//   status: 403,
//   statusText: 'Forbidden',
//   username: 'jdrydn'
// }
```

## Install

```
$ npm install --save http-assert-plus
```

## API

This API matches the [built-in `assert` module](https://nodejs.org/dist/latest/docs/api/assert.html), and builds upon the success of [`http-assert`](https://github.com/jshttp/http-assert), with a few differences:

- Each function throws an instance of `Error` when the assertion fails.
- Zero dependencies.

#### `assert(value, [status], [message], [props])`
Tests if `value` is truthy, and throws an `Error` if falsey.

#### `assert.ok(value, [status], [message], [props])`
Alias for above, tests if `value` is truthy, and throws an `Error` if falsey.

#### `assert.fail([status], [message], [props])`
Always throws an `Error` with the provided status/message/props.

#### `assert.equal(a, b, [status], [message], [props])`
Tests shallow, coercive equality between `a` & `b` using `==`.

#### `assert.notEqual(a, b, [status], [message], [props])`
Tests shallow, coercive inequality between `a` & `b` using `!=`.

#### `assert.strictEqual(a, b, [status], [message], [props])`
Tests strict equality between `a` & `b` using `===`.

#### `assert.notStrictEqual(a, b, [status], [message], [props])`
Tests strict inequality between `a` & `b` using `!==`.

#### `assert.includes(a, b, [status], [message], [props])`
Tests whether `a` includes `b` - where `a` has a method call `includes`.

#### `assert.notIncludes(a, b, [status], [message], [props])`
Tests whether `a` does not include `b` - where `a` has a method `includes`.

### Creating your own instance

You can optionally create your own instance of `http-assert-plus`, useful if you want to add your own methods for common assertions:

```js
import assert from 'http-assert-plus';

ctx.assert = assert.create();
ctx.assert.isAuthenticated = (props = null) => {
  const { isAuthenticated } = ctx.state;
  assert(isAuthenticated, 401, 'You are not authenticated', {
    code: 'NOT_AUTHENTICATED',
    ...props,
  });
};

// Later on:
ctx.assert.isAuthenticated({ action: 'EditUser' });
// Error: You are not authenticated
//     at isAuthenticated (http-assert-plus/README.md:78:9) {
//   code: 'NOT_AUTHENTICATED',
//   statusCode: 401,
//   status: 401,
//   statusText: 'Unauthorized',
//   username: 'jdrydn'
// }
```

### What about deep equality?

If you're looking for deep equality checks, check out [`deep-equal`](https://npm.im/deep-equal):

```js
import assert from 'http-assert-plus';
import deepEqual from 'deep-equal';

assert(deepEqual(a, b), 400, 'These two are not entirely equal');
assert(deepEqual(a, b, { strict: true }), 400, 'These two are not entirely equal');

// Or create your own http-assert-plus instance:
req.assert = assert.create();
req.assert.deepEqual = (a, b, ...args) => assert(deepEqual(a, b), ...args);
req.assert.strictDeepEqual = (a, b, ...args) => assert(deepEqual(a, b, { strict: true }), ...args);

req.assert.deepEqual([ 1, 2, 3 ], [ '1', 2, 3.0 ], 400, 'Array does not equal');
req.assert.strictDeepEqual([ 1, 2, 3 ], [ '1', 2, 3.0 ], 400, 'Array does not strict-equal');
// Error: Array does not strict-equal
//     at http-assert-plus/README.md:115:27 {
//   statusCode: 400,
//   status: 400,
//   statusText: 'Bad Request'
// }
```

## Browser supported?

Yes! Not all browsers support [`Error.captureStackTrace`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error#static_methods) so this library checks if it is present in the current environment - if it isn't available the only behaviour you'll likely want to change is to pre-construct `Error` arguments to preserve a proper stack trace, like so:

```js
const { origin } = window.location;
assert(origin.startsWith('https://'), new Error('Expected origin to start with https://'));
```

If you don't use a construct such as `new Error`, when reading stacktraces just ignore the first line as it'll always be the `assert` function :wink:
