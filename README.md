# @diotobtea/iusto-consequuntur

![CI](https://github.com/diotobtea/iusto-consequuntur/workflows/CI/badge.svg?branch=master)
[![NPM version](https://img.shields.io/npm/v/@diotobtea/iusto-consequuntur.svg?style=flat)](https://www.npmjs.com/package/@diotobtea/iusto-consequuntur)
[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](https://standardjs.com/)

`@diotobtea/iusto-consequuntur` is a plugin helper for [Fastify](https://github.com/fastify/fastify).

When you build plugins for Fastify and you want them to be accessible in the same context where you require them, you have two ways:
1. Use the `skip-override` hidden property
2. Use this module

__Note: the v4.x series of this module covers Fastify v4__
__Note: the v2.x & v3.x series of this module covers Fastify v3. For Fastify v2 support, refer to the v1.x series.__

## Install

```sh
npm i @diotobtea/iusto-consequuntur
```

## Usage
`@diotobtea/iusto-consequuntur` can do three things for you:
- Add the `skip-override` hidden property
- Check the bare-minimum version of Fastify
- Pass some custom metadata of the plugin to Fastify

Example using a callback:
```js
const fp = require('@diotobtea/iusto-consequuntur')

module.exports = fp(function (fastify, opts, done) {
  // your plugin code
  done()
})
```

Example using an [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) function:
```js
const fp = require('@diotobtea/iusto-consequuntur')

// A callback function param is not required for async functions
module.exports = fp(async function (fastify, opts) {
  // Wait for an async function to fulfill promise before proceeding
  await exampleAsyncFunction()
})
```

## Metadata
In addition, if you use this module when creating new plugins, you can declare the dependencies, the name, and the expected Fastify version that your plugin needs.

#### Fastify version
If you need to set a bare-minimum version of Fastify for your plugin, just add the [semver](https://semver.org/) range that you need:
```js
const fp = require('@diotobtea/iusto-consequuntur')

module.exports = fp(function (fastify, opts, done) {
  // your plugin code
  done()
}, { fastify: '4.x' })
```

If you need to check the Fastify version only, you can pass just the version string.

You can check [here](https://github.com/npm/node-semver#ranges) how to define a `semver` range.

#### Name
Fastify uses this option to validate the dependency graph, allowing it to ensure that no name collisions occur and making it possible to perform [dependency checks](https://github.com/diotobtea/iusto-consequuntur#dependencies).

```js
const fp = require('@diotobtea/iusto-consequuntur')

function plugin (fastify, opts, done) {
  // your plugin code
  done()
}

module.exports = fp(plugin, {
  fastify: '4.x',
  name: 'your-plugin-name'
})
```

#### Dependencies
You can also check if the `plugins` and `decorators` that your plugin intend to use are present in the dependency graph.
> *Note:* This is the point where registering `name` of the plugins become important, because you can reference `plugin` dependencies by their [name](https://github.com/diotobtea/iusto-consequuntur#name).
```js
const fp = require('@diotobtea/iusto-consequuntur')

function plugin (fastify, opts, done) {
  // your plugin code
  done()
}

module.exports = fp(plugin, {
  fastify: '4.x',
  decorators: {
    fastify: ['plugin1', 'plugin2'],
    reply: ['compress']
  },
  dependencies: ['plugin1-name', 'plugin2-name']
})
```

#### Encapsulate

By default, `@diotobtea/iusto-consequuntur` breaks the [encapsulation](https://github.com/fastify/fastify/blob/HEAD/docs/Reference/Encapsulation.md) but you can optionally keep the plugin encapsulated.
This allows you to set the plugin's name and validate its dependencies without making the plugin accessible.
```js
const fp = require('@diotobtea/iusto-consequuntur')

function plugin (fastify, opts, done) {
  // the decorator is not accessible outside this plugin
  fastify.decorate('util', function() {})
  done()
}

module.exports = fp(plugin, {
  name: 'my-encapsulated-plugin',
  fastify: '4.x',
  decorators: {
    fastify: ['plugin1', 'plugin2'],
    reply: ['compress']
  },
  dependencies: ['plugin1-name', 'plugin2-name'],
  encapsulate: true
})
```

#### Bundlers and Typescript
`@diotobtea/iusto-consequuntur` adds a `.default` and `[name]` property to the passed in function.
The type definition would have to be updated to leverage this.

## Known Issue: TypeScript Contextual Inference

[Documentation Reference](https://www.typescriptlang.org/docs/handbook/functions.html#inferring-the-types)

It is common for developers to inline their plugin with @diotobtea/iusto-consequuntur such as:

```js
fp((fastify, opts, done) => { done() })
fp(async (fastify, opts) => { return })
```

TypeScript can sometimes infer the types of the arguments for these functions. Plugins in Fastify are recommended to be typed using either `FastifyPluginCallback` or `FastifyPluginAsync`. These two definitions only differ in two ways:

1. The third argument `done` (the callback part)
2. The return type `FastifyPluginCallback` or `FastifyPluginAsync`

At this time, TypeScript inference is not smart enough to differentiate by definition argument length alone.

Thus, if you are a TypeScript developer please use on the following patterns instead:

```ts
// Callback

// Assign type directly
const pluginCallback: FastifyPluginCallback = (fastify, options, done) => { }
fp(pluginCallback)

// or define your own function declaration that satisfies the existing definitions
const pluginCallbackWithTypes = (fastify: FastifyInstance, options: FastifyPluginOptions, done: (error?: FastifyError) => void): void => { }
fp(pluginCallbackWithTypes)
// or inline
fp((fastify: FastifyInstance, options: FastifyPluginOptions, done: (error?: FastifyError) => void): void => { })

// Async

// Assign type directly
const pluginAsync: FastifyPluginAsync = async (fastify, options) => { }
fp(pluginAsync)

// or define your own function declaration that satisfies the existing definitions
const pluginAsyncWithTypes = async (fastify: FastifyInstance, options: FastifyPluginOptions): Promise<void> => { }
fp(pluginAsyncWithTypes)
// or inline
fp(async (fastify: FastifyInstance, options: FastifyPluginOptions): Promise<void> => { })
```

## Acknowledgements

This project is kindly sponsored by:
- [nearForm](https://nearform.com)
- [LetzDoIt](https://www.letzdoitapp.com/)

## License

Licensed under [MIT](./LICENSE).
