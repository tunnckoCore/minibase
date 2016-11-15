<p align="center">
  <a href="https://github.com/node-minibase">
    <img height="250" width="250" src="https://avatars1.githubusercontent.com/u/23032863?v=3&s=250">
  </a>
</p>

# minibase [![NPM version](https://img.shields.io/npm/v/minibase.svg?style=flat)](https://www.npmjs.com/package/minibase) [![NPM downloads](https://img.shields.io/npm/dm/minibase.svg?style=flat)](https://npmjs.org/package/minibase) [![npm total downloads][downloads-img]][downloads-url]

> MiniBase is minimalist approach to Base - @node-base, the awesome framework. Foundation for building complex APIs with small units called plugins. Works well with most of the already existing [base][] plugins.

[![code climate][codeclimate-img]][codeclimate-url] [![standard code style][standard-img]][standard-url] [![travis build status][travis-img]][travis-url] [![Windows Build Status](https://img.shields.io/appveyor/ci/node-minibase/minibase.svg?style=flat&label=AppVeyor)](https://ci.appveyor.com/project/node-minibase/minibase) [![coverage status][coveralls-img]][coveralls-url] [![dependency status][david-img]][david-url]

You might also be interested in [base](https://github.com/node-base/base).

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
- [Building docs](#building-docs)
- [Running tests](#running-tests)
- [Author](#author)
- [License](#license)

_(TOC generated by [verb](https://github.com/verbose/verb) using [markdown-toc](https://github.com/jonschlinkert/markdown-toc))_

## Install
Install with [npm](https://www.npmjs.com/)

```
$ npm install minibase --save
```

or install using [yarn](https://yarnpkg.com)

```
$ yarn add minibase
```

## Usage
> For more use-cases see the [tests](test.js)

```js
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

### [.delegate](index.js#L129)
> Copy properties from `provider` to `this` instance of `MiniBase`, using [delegate-properties][] lib.

**Params**

* `<provider>` **{Object}**: object providing properties    
* `returns` **{Object}**: Returns instance for chaining  

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

### [.define](index.js#L183)
> Used for adding non-enumerable property `key` with `value` on the instance, using [define-property][] lib.

**Params**

* `key` **{String}**: name of the property to be defined or modified    
* `value` **{any}**: descriptor for the property being defined or modified    
* `returns` **{Object}**: Returns instance for chaining  

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

### [.use](index.js#L218)
> Define a synchronous plugin `fn` function to be called immediately upon init. _**Never throws - emit events!™**_

**Params**

* `fn` **{Function}**: plugin passed with `ctx` which is the instance    
* `returns` **{Object}**: Returns instance for chaining  

**Events**
* `emits`: `error` when plugin `fn` throws an error  

**Example**

```js
const MiniBase = require('minibase').MiniBase
const app = MiniBase({ silent: true, foo: 'bar' })

app
  .once('error', (err) => console.error(err.stack || err))
  .use((app) => {
    console.log(app.options) // => { silent: true, foo: 'bar' }
    return 555
  })
  .use(function () {
    console.log(this.options) // => { silent: true, foo: 'bar' }
    // intentionally
    foo bar
  })
```

### [#delegate](index.js#L263)
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

### [#define](index.js#L290)
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

### [#extend](index.js#L327)
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
- [base](https://www.npmjs.com/package/base): base is the foundation for creating modular, unit testable and highly pluggable node.js applications, starting with… [more](https://github.com/node-base/base) | [homepage](https://github.com/node-base/base "base is the foundation for creating modular, unit testable and highly pluggable node.js applications, starting with a handful of common methods, like `set`, `get`, `del` and `use`.")
- [generate](https://www.npmjs.com/package/generate): Command line tool and developer framework for scaffolding out new GitHub projects. Generate offers the robustness… [more](https://github.com/generate/generate) | [homepage](https://github.com/generate/generate "Command line tool and developer framework for scaffolding out new GitHub projects. Generate offers the robustness and configurability of Yeoman, the expressiveness and simplicity of Slush, and more powerful flow control and composability than either.")
- [lazy-cache](https://www.npmjs.com/package/lazy-cache): Cache requires to be lazy-loaded when needed. | [homepage](https://github.com/jonschlinkert/lazy-cache "Cache requires to be lazy-loaded when needed.")
- [minibase-better-define](https://www.npmjs.com/package/minibase-better-define): Plugin for [base][] and [minibase][] that overrides the core `.define` method to be more better. | [homepage](https://github.com/node-minibase/minibase-better-define#readme "Plugin for [base][] and [minibase][] that overrides the core `.define` method to be more better.")
- [minibase-create-plugin](https://www.npmjs.com/package/minibase-create-plugin): Utility for [minibase][] and [base][] that helps you create plugins | [homepage](https://github.com/node-minibase/minibase-create-plugin#readme "Utility for [minibase][] and [base][] that helps you create plugins")
- [minibase-is-registered](https://www.npmjs.com/package/minibase-is-registered): Plugin for [minibase][] and [base][], that adds `isRegistered` method to your application to detect if plugin… [more](https://github.com/node-minibase/minibase-is-registered#readme) | [homepage](https://github.com/node-minibase/minibase-is-registered#readme "Plugin for [minibase][] and [base][], that adds `isRegistered` method to your application to detect if plugin is already registered and returns true or false if named plugin is already registered on the instance.")
- [minibase-visit](https://www.npmjs.com/package/minibase-visit): Plugin for [minibase][] and [base][], that adds `.visit` method to your application to visit a method… [more](https://github.com/node-minibase/minibase-visit#readme) | [homepage](https://github.com/node-minibase/minibase-visit#readme "Plugin for [minibase][] and [base][], that adds `.visit` method to your application to visit a method over the items in an object, or map visit over the objects in an array. Using using [collection-visit][] package.")
- [try-catch-callback](https://www.npmjs.com/package/try-catch-callback): try/catch block with a callback, used in [try-catch-core][]. Use it when you don't care about asyncness… [more](https://github.com/hybridables/try-catch-callback#readme) | [homepage](https://github.com/hybridables/try-catch-callback#readme "try/catch block with a callback, used in [try-catch-core][]. Use it when you don't care about asyncness so much and don't want guarantees. If you care use [try-catch-core][].")
- [use](https://www.npmjs.com/package/use): Easily add plugin support to your node.js application. | [homepage](https://github.com/jonschlinkert/use "Easily add plugin support to your node.js application.")
- [verb](https://www.npmjs.com/package/verb): Documentation generator for GitHub projects. Verb is extremely powerful, easy to use, and is used on… [more](https://github.com/verbose/verb) | [homepage](https://github.com/verbose/verb "Documentation generator for GitHub projects. Verb is extremely powerful, easy to use, and is used on hundreds of projects of all sizes to generate everything from API docs to readmes.")

## Contributing
Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](https://github.com/node-minibase/minibase/issues/new).  
Please read the [contributing guidelines](CONTRIBUTING.md) for advice on opening issues, pull requests, and coding standards.

**In short:** If you want to contribute to that project, please follow these things

1. Please DO NOT edit [README.md](README.md), [CHANGELOG.md](CHANGELOG.md) and [.verb.md](.verb.md) files. See ["Building docs"](#building-docs) section.
2. Ensure anything is okey by installing the dependencies and run the tests. See ["Running tests"](#running-tests) section.
3. Always use `npm run commit` to commit changes instead of `git commit`, because it is interactive and user-friendly. It uses [commitizen][] behind the scenes, which follows Conventional Changelog idealogy.
4. Do NOT bump the version in package.json. For that we use `npm run release`, which is [standard-version][] and follows Conventional Changelog idealogy.

Thanks a lot! :)

## Building docs
Documentation and that readme is generated using [verb-generate-readme][], which is a [verb][] generator, so you need to install both of them and then run `verb` command like that

```
$ npm install verbose/verb#dev verb-generate-readme --global && verb
```

_Please don't edit the README directly. Any changes to the readme must be made in [.verb.md](.verb.md)._

## Running tests
Clone repository and run the following in that cloned directory

```
$ npm install && npm test
```

## Author
**Charlike Mike Reagent**

+ [github/tunnckoCore](https://github.com/tunnckoCore)
+ [twitter/tunnckoCore](http://twitter.com/tunnckoCore)

## License
Copyright © 2016, [Charlike Mike Reagent](http://www.tunnckocore.tk). Released under the [MIT license](LICENSE).

***

_This file was generated by [verb-generate-readme](https://github.com/verbose/verb-generate-readme), v0.2.0, on November 16, 2016._

[base]: https://github.com/node-base/base
[collection-visit]: https://github.com/jonschlinkert/collection-visit
[commitizen]: https://github.com/commitizen/cz-cli
[define-property]: https://github.com/jonschlinkert/define-property
[delegate-properties]: https://github.com/jonschlinkert/delegate-properties
[minibase]: https://github.com/node-minibase/minibase
[standard-version]: https://github.com/conventional-changelog/standard-version
[static-extend]: https://github.com/jonschlinkert/static-extend
[try-catch-callback]: https://github.com/hybridables/try-catch-callback
[try-catch-core]: https://github.com/hybridables/try-catch-core
[verb-generate-readme]: https://github.com/verbose/verb-generate-readme
[verb]: https://github.com/verbose/verb

[downloads-url]: https://www.npmjs.com/package/minibase
[downloads-img]: https://img.shields.io/npm/dt/minibase.svg

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

