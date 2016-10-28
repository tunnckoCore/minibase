<p align="center">
  <a href="https://github.com/node-base/base">
    <img height="250" width="250" src="https://avatars1.githubusercontent.com/u/23032863?v=3&s=250">
  </a>
</p>

# [minibase][author-www-url] [![npmjs.com][npmjs-img]][npmjs-url] [![The MIT License][license-img]][license-url] [![npm downloads][downloads-img]][downloads-url] 

> MiniBase is minimalist approach to Base - [@node-base](https://github.com/node-base), the awesome framework. Foundation for building complex APIs with small units called plugins. Works well with most of the already existing [base][] plugins.

[![code climate][codeclimate-img]][codeclimate-url] [![standard code style][standard-img]][standard-url] [![travis build status][travis-img]][travis-url] [![coverage status][coveralls-img]][coveralls-url] [![dependency status][david-img]][david-url]

## Table of Contents
- [Install](#install)
- [Usage](#usage)
- [API](#api)
  * [MiniBase](#minibase)
  * [.delegate](#delegate)
  * [.define](#define)
  * [.use](#use)
  * [#delegate](##delegate)
  * [#define](##define)
  * [#extend](##extend)
- [Related](#related)
- [Contributing](#contributing)

## Install
> Install with [npm](https://www.npmjs.com/)

```sh
$ npm i minibase --save
```

## Usage
> For more use-cases see the [tests](./test.js)

```js
const MiniBase = require('minibase').MiniBase

// or instance directly

const minibase = require('minibase')
```

## API

### [MiniBase](index.js#L47)
> Creates an instance of `MiniBase` with optional `options` object - if given, otherwise the `minibase.options` defaults to empty object. _**Never throws - emit events!™**_

**Params**

* `[options]` **{Object}**: optional, pass `silent: true` to not add default error listener    

**Example**

```js
const MiniBase = require('minibase').MiniBase
const app = require('minibase')

// when `silent: true` it will not add
// the default event listener to `error` event
const minibase = MiniBase({ silent: true })

// nothing is printed, until you add
// listener `.on('error', fn)`
minibase.use(() => {
  throw new Error('foo bar')
})

// if you work with defaults
// you will get this error printed
// because the default error handler
app.use(function () {
  console.log(this.options.silent) // => undefined
  throw new Error('default error handler works')
})
```

### [.delegate](index.js#L137)
> Copy properties from `provider` to `this` instance of `MiniBase`, using [delegate-properties][] lib.

**Params**

* `<provider>` **{Object}**: object providing properties    
* `returns` **{Object}**: Returns instance of `MiniBase` for chaining  

**Example**

```js
const minibase = require('minibase')

minibase.use((app) => {
  // `app` is `minibase`

  app.delegate({
    foo: 'bar',
    qux: (name) => {
      console.log(`hello ${name}`)
    }
  })
})

// or directly use `.delegate`,
// not through plugin
minibase.delegate({ cats: 'dogs' })

console.log(minibase.cats) // => 'dogs'
console.log(minibase.foo) // => 'bar'
console.log(minibase.qux('kitty!')) // => 'hello kitty!'
```

### [.define](index.js#L191)
> Used for adding non-enumerable property `key` with `value` on the instance, using [define-property][] lib.

**Params**

* `key` **{String}**: name of the property to be defined or modified    
* `value` **{any}**: descriptor for the property being defined or modified    
* `returns` **{Object}**: Returns instance of `MiniBase` for chaining  

**Example**

```js
const minibase = require('minibase')

minibase.use(function (app) {
  // `app` and `this` are instance of `MiniBase`,
  // and so `minibase`

  this.define('set', function set (key, value) {
    this.cache = this.cache || {}
    this.cache[key] = value
    return this
  })
  app.define('get', function get (key) {
    return this.cache[key]
  })
})

minibase
  .set('foo', 'bar')
  .set('qux', 123)
  .set('baz', { a: 'b' })

// or directly use `.define`,
// not through plugin
minibase.define('hi', 'kitty')
console.log(minibase.hi) // => 'kitty'

console.log(minibase.get('foo')) // => 'bar'
console.log(minibase.get('qux')) // => 123
console.log(minibase.get('baz')) // => { a: 'b' }

// or access the cache directly
console.log(minimist.cache.baz) // => { a: 'b' }
console.log(minimist.cache.qux) // => 123
```

### [.use](index.js#L239)
> Define a plugin `fn` function to be called immediately upon init. It is recommended `fn` to be synchronous and should not expect asynchronous plugins to work correctly - use plugins for this. Uses [try-catch-callback][] under the hood to handle errors and completion of that synchronous function. _**Never throws - emit events!™**_

See [try-catch-callback][] and/or [try-catch-core][] for more details.

**Params**

* `fn` **{Function}**: plugin to be called with `ctx, cb` arguments, where both `ctx` and `this` of `fn` are instance of `MiniBase` and `cb` is callback - use with caution and in rare cases    
* `returns` **{Object}**: Returns instance of `MiniBase` for chaining  

**Events**
* `emits`: `error` when plugin `fn` throws an error  
* `emits`: `use` on successful completion with `fn` and `result` arguments, where the `result` is returned value of the plugin  

**Example**

```js
const MiniBase = require('minibase').MiniBase
const app = MiniBase({ silent: true, foo: 'bar' })

app
  .once('error', (err) => console.error(err.stack || err))
  .on('use', function (fn, res) {
    // called on each `.use` call
    console.log(res) // => 555
  })
  .use((app) => {
    console.log(app.options) // => { silent: true, foo: 'bar' }
    return 555
  })
  .use(function () {
    // intentionally
    foo bar
  })
```

### [#delegate](index.js#L283)
> Static method to delegate properties from `provider` to `receiver` and make them non-enumerable.

See [delegate-properties][] for more details, it is exact mirror.

**Params**

* `receiver` **{Object}**: object receiving properties    
* `provider` **{Object}**: object providing properties    

**Example**

```js
const MiniBase = require('minibase').MiniBase

const obj = { foo: 'bar' }

MiniBase.delegate(obj, {
  qux: 123
})

console.log(obj.foo) // => 'bar'
console.log(obj.qux) // => 123
```

### [#define](index.js#L310)
> Static method to define a non-enumerable property on an object.

See [define-property][] for more details, it is exact mirror.

**Params**

* `obj` **{Object}**: The object on which to define the property    
* `prop` **{Object}**: The name of the property to be defined or modified    
* `descriptor` **{any}**: The descriptor for the property being defined or modified    

**Example**

```js
const MiniBase = require('minibase').MiniBase

const obj = {}
MiniBase.define(obj, 'foo', 123)
MiniBase.define(obj, 'bar', () => console.log('qux'))

console.log(obj.foo) // => 123
console.log(obj.bar()) // => 'qux'
```

### [#extend](index.js#L347)
> Static method for inheriting the prototype and static methods of the `MiniBase` class. This method greatly simplifies the process of creating inheritance-based applications.

See [static-extend][] for more details.

**Params**

* `Ctor` **{Function}**: constructor to extend    
* `methods` **{Object}**: optional prototype properties to mix in    

**Example**

```js
const MiniBase = require('minibase').MiniBase

function MyApp (options) {
  MiniBase.call(this, options)
}

MiniBase.extend(MyApp)

console.log(MyApp.extend) // => function
console.log(MyApp.define) // => function
console.log(MyApp.delegate) // => function

const app = new MyApp()

console.log(app.use) // => function
console.log(app.define) // => function
console.log(app.delegate) // => function
```

## Related
- [base-app](https://www.npmjs.com/package/base-app): Starting point for creating a base application, with a few light plugins… [more](https://github.com/node-base/base-app) | [homepage](https://github.com/node-base/base-app "Starting point for creating a base application, with a few light plugins for running tasks and writing to the file system, and a functional CLI.")
- [base-plugins-enhanced](https://www.npmjs.com/package/base-plugins-enhanced): Error handling and extras for `.use` and `.run` methods of your Base… [more](https://github.com/tunnckocore/base-plugins-enhanced#readme) | [homepage](https://github.com/tunnckocore/base-plugins-enhanced#readme "Error handling and extras for `.use` and `.run` methods of your Base apps. Modifies `.use` method to be able to 1) accept array of functions, 2) options object as second argument. Emits `error` event if some plugin fails.")
- [base-plugins](https://www.npmjs.com/package/base-plugins): Upgrade's plugin support in base applications to allow plugins to be called… [more](https://github.com/node-base/base-plugins) | [homepage](https://github.com/node-base/base-plugins "Upgrade's plugin support in base applications to allow plugins to be called any time after init.")
- [base-task](https://www.npmjs.com/package/base-task): base plugin that provides a very thin wrapper around <https://github.com/doowb/composer> for adding… [more](https://github.com/node-base/base-task) | [homepage](https://github.com/node-base/base-task "base plugin that provides a very thin wrapper around <https://github.com/doowb/composer> for adding task methods to your application.")
- [base](https://www.npmjs.com/package/base): base is the foundation for creating modular, unit testable and highly pluggable… [more](https://github.com/node-base/base) | [homepage](https://github.com/node-base/base "base is the foundation for creating modular, unit testable and highly pluggable node.js applications, starting with a handful of common methods, like `set`, `get`, `del` and `use`.")
- [generate](https://www.npmjs.com/package/generate): Command line tool and developer framework for scaffolding out new GitHub projects… [more](https://github.com/generate/generate) | [homepage](https://github.com/generate/generate "Command line tool and developer framework for scaffolding out new GitHub projects. Generate offers the robustness and configurability of Yeoman, the expressiveness and simplicity of Slush, and more powerful flow control and composability than either.")
- [lazy-cache](https://www.npmjs.com/package/lazy-cache): Cache requires to be lazy-loaded when needed. | [homepage](https://github.com/jonschlinkert/lazy-cache "Cache requires to be lazy-loaded when needed.")
- [use](https://www.npmjs.com/package/use): Easily add plugin support to your node.js application. | [homepage](https://github.com/jonschlinkert/use "Easily add plugin support to your node.js application.")
- [verb](https://www.npmjs.com/package/verb): Documentation generator for GitHub projects. Verb is extremely powerful, easy to use… [more](https://github.com/verbose/verb) | [homepage](https://github.com/verbose/verb "Documentation generator for GitHub projects. Verb is extremely powerful, easy to use, and is used on hundreds of projects of all sizes to generate everything from API docs to readmes.")

## Contributing
Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](https://github.com/node-minibase/minibase/issues/new).  
But before doing anything, please read the [CONTRIBUTING.md](./CONTRIBUTING.md) guidelines.

## [Charlike Make Reagent](http://j.mp/1stW47C) [![new message to charlike][new-message-img]][new-message-url] [![freenode #charlike][freenode-img]][freenode-url]

[![tunnckoCore.tk][author-www-img]][author-www-url] [![keybase tunnckoCore][keybase-img]][keybase-url] [![tunnckoCore npm][author-npm-img]][author-npm-url] [![tunnckoCore twitter][author-twitter-img]][author-twitter-url] [![tunnckoCore github][author-github-img]][author-github-url]

[base]: https://github.com/node-base/base
[define-property]: https://github.com/jonschlinkert/define-property
[delegate-properties]: https://github.com/jonschlinkert/delegate-properties
[static-extend]: https://github.com/jonschlinkert/static-extend
[try-catch-callback]: https://github.com/tunnckocore/try-catch-callback
[try-catch-core]: https://github.com/tunnckocore/try-catch-core

[npmjs-url]: https://www.npmjs.com/package/minibase
[npmjs-img]: https://img.shields.io/npm/v/minibase.svg?label=minibase

[license-url]: https://github.com/node-minibase/minibase/blob/master/LICENSE
[license-img]: https://img.shields.io/npm/l/minibase.svg

[downloads-url]: https://www.npmjs.com/package/minibase
[downloads-img]: https://img.shields.io/npm/dm/minibase.svg

[codeclimate-url]: https://codeclimate.com/github/node-minibase/minibase
[codeclimate-img]: https://img.shields.io/codeclimate/github/node-minibase/minibase.svg

[travis-url]: https://travis-ci.org/node-minibase/minibase
[travis-img]: https://img.shields.io/travis/node-minibase/minibase/master.svg

[coveralls-url]: https://coveralls.io/r/node-minibase/minibase
[coveralls-img]: https://img.shields.io/coveralls/node-minibase/minibase.svg

[david-url]: https://david-dm.org/node-minibase/minibase
[david-img]: https://img.shields.io/david/node-minibase/minibase.svg

[standard-url]: https://github.com/feross/standard
[standard-img]: https://img.shields.io/badge/code%20style-standard-brightgreen.svg

[author-www-url]: http://www.tunnckocore.tk
[author-www-img]: https://img.shields.io/badge/www-tunnckocore.tk-fe7d37.svg

[keybase-url]: https://keybase.io/tunnckocore
[keybase-img]: https://img.shields.io/badge/keybase-tunnckocore-8a7967.svg

[author-npm-url]: https://www.npmjs.com/~tunnckocore
[author-npm-img]: https://img.shields.io/badge/npm-~tunnckocore-cb3837.svg

[author-twitter-url]: https://twitter.com/tunnckoCore
[author-twitter-img]: https://img.shields.io/badge/twitter-@tunnckoCore-55acee.svg

[author-github-url]: https://github.com/tunnckoCore
[author-github-img]: https://img.shields.io/badge/github-@tunnckoCore-4183c4.svg

[freenode-url]: http://webchat.freenode.net/?channels=charlike
[freenode-img]: https://img.shields.io/badge/freenode-%23charlike-5654a4.svg

[new-message-url]: https://github.com/tunnckoCore/ama
[new-message-img]: https://img.shields.io/badge/ask%20me-anything-green.svg

